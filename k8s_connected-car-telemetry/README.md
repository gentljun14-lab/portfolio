# 🚀 Connected Car Telemetry System

CQRS 패턴을 기반으로 설계한 차량 텔레메트리 실시간 관리 시스템입니다.
대량의 차량 데이터를 실시간으로 수신하는 경로(Command)와 효율적인 조회를 위한 경로(Query)를 분리하여, 데이터 처리 성능과 확장성을 함께 확보하고자 했습니다.

```mermaid
graph LR
    Vehicle(차량 단말/Generator) --> Ingest[Ingest API]
    Ingest --> Kafka((Kafka))

    subgraph "Write Path"
        Kafka --> MongoConsumer[Mongo Consumer]
        MongoConsumer --> MongoDB[(MongoDB)]
    end

    subgraph "Read Path"
        Kafka --> RedisConsumer[Redis Consumer]
        RedisConsumer --> Redis[(Redis)]
        Redis --> QueryAPI[Query API]
    end

    QueryAPI --> Frontend[React Frontend]
 ```

## 🏗️ 시스템 구조 및 데이터 흐름

시스템은 책임 분리를 위해 **Command(쓰기)** 영역과 **Query(읽기)** 영역으로 나뉘며, Kafka를 통해 비동기적으로 데이터를 동기화합니다.

### 📥 Command: 데이터 수신 및 저장

- **Ingest API (FastAPI)**: 차량 시뮬레이터로부터 `POST /api/query/telemetry` 요청을 통해 상태 데이터를 수신합니다.
- **Kafka Producer**: 수신한 데이터를 `car.telemetry.events` 토픽으로 즉시 발행합니다.
- **MongoDB Consumer**: 전체 이력(`telemetry_history`)을 영구 저장하여 데이터 영속성을 보장합니다.

### 📤 Query: 데이터 조회 및 캐싱

- **Redis Consumer**: Kafka 이벤트를 구독하여 차량별 최신 상태(`vehicle:*:latest`)를 Redis에 업데이트합니다. (TTL 60초)
- **Query API (FastAPI)**: `GET /api/vehicles` 요청 시 DB가 아닌 Redis 캐시에서 데이터를 조회하여 빠른 응답을 제공합니다.
- **Frontend (React)**: Leaflet 지도를 활용해 실시간 차량 위치를 시각화합니다.

---

## ☸️ Kubernetes 인프라 구성 (Helm)

`k8s-manifests/` 디렉토리에는 서비스 성격에 따라 구분된 4개의 Helm 차트가 포함되어 있습니다.

| 레이어 | 컴포넌트 | 설명 |
| --- | --- | --- |
| **Infra** | Kafka, MongoDB, Redis | StatefulSet 기반으로 운영하며 데이터 보존 정책을 적용 |
| **Command** | Ingest API, Consumer | HPA(Horizontal Pod Autoscaler)를 통한 수평 확장 지원 |
| **Query** | Query API, Consumer | 읽기 전용 서비스 분리 배포 |
| **Frontend** | React, Nginx Ingress | VWorld Proxy 설정 및 경로 기반 라우팅 적용 |

> **보안 참고:** VWorld 지도 API 키는 Nginx Proxy를 통해 은닉 처리하여 클라이언트에 직접 노출되지 않도록 구성했습니다.
> 

---

## 📊 모니터링 및 운영

- **Prometheus / Grafana**: 시스템 메트릭 수집 및 대시보드 시각화
- **Alertmanager**: 리소스 부하 및 파드 장애 발생 시 **Discord**로 실시간 알림 전송
- **데이터 보존**: NFS Storage(10.0.2.100) 마운트를 통해 파드 재시작 시에도 데이터 유지
- **CronJob**: MongoDB(15분 주기 이력 정리), Redis(10분 주기 고아 키 정리) 작업 자동화

---

## 🛠️ CI/CD 및 배포

1. **CI/CD 파이프라인**: Jenkinsfile을 통해 빌드 → Harbor 이미지 푸시 → Helm 배포를 자동화했습니다.
2. **수동 배포 지원**: `docs/deploy.sh`를 통해 로컬 및 스테이징 환경에서 수동 배포가 가능하도록 구성했습니다.
3. **Secret 관리**: `secret-values.yaml`을 통해 민감 정보를 분리 관리하며, Git 추적 대상에서는 제외했습니다.

---

## 담당 기여

- Harbor 사설 레지스트리를 구축하여 내부 이미지 저장 및 배포 환경을 구성했습니다.
- Redis를 연동해 최신 상태 조회에 적합한 Query 구조를 반영했습니다.
- React 기반 프론트엔드를 설계하고 지도 기반 시각화 화면을 구현했습니다.
- Kafka 일부 로직을 수정하고 Redis와 연계되는 조회 흐름을 반영했습니다.
- Grafana 일부 설정과 네트워크 설정에 참여했습니다.

---

## 트러블슈팅

### MongoDB 인증 실패

- **원인**: Secret의 namespace 불일치와 배포 설정 내 인증 옵션 누락으로 인해 인증이 정상적으로 적용되지 않았습니다.
- **해결**: namespace를 수정하고 StatefulSet에 인증 옵션을 반영한 뒤, 관련 PVC를 재배포하여 문제를 해결했습니다.

### Harbor와 Jenkins 통신 오류

- **원인**: 네트워크 불안정과 HTTP/HTTPS 설정 차이로 인해 이미지 푸시 및 연동 과정에서 통신 오류가 발생했습니다.
- **해결**: Offline Installer 기반으로 구성을 전환하고, 필요한 구간에는 insecure registry 설정을 적용하여 문제를 해결했습니다.

---

## 💡 핵심 특징

- **확장성**: 차량 대수가 증가하더라도 Kafka Partition 확장과 API 파드 수평 확장을 통해 유연하게 대응할 수 있도록 설계했습니다.
- **성능**: Redis 캐싱 레이어를 적용해 DB 부하를 줄이고, UI 응답 속도를 최적화했습니다.
- **자동화**: Helm과 Jenkins를 활용해 인프라 구성과 배포 과정을 자동화했습니다.
