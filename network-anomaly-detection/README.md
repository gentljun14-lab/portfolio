# AI 기반 실시간 네트워크 이상 트래픽 감시 시스템 (IPS)

> **"NFStream Flow Intelligence + Machine Learning"** — 머신러닝을 이용한 지능형 네트워크 방어 및 실시간 자동 차단 솔루션
> 

## 1. 프로젝트 개요

기존의 룰 기반 보안 시스템은 정해진 조건만 차단하므로 변형된 공격 패턴에 대응하기 어렵습니다. 본 시스템은 네트워크 트래픽을 **Flow(플로우)** 단위로 분석하고, 학습된 AI 모델을 통해 **DDoS, Port Scan, Brute Force** 등의 공격을 실시간으로 탐지하고 즉시 차단합니다.

## 핵심 가치

- **지능형 탐지:** 플로우 특징(패킷 수, 바이트 수, 간격 등)을 학습하여 신규 및 변형 공격에 유연하게 대응합니다.
- **실시간 대응:** 공격 감지 시 `iptables`를 통해 공격 IP를 즉시 차단하고 디스코드로 알림을 발송합니다.
- **경량화 모델:** 자원이 제한된 **라즈베리파이**에서도 원활히 구동되도록 가볍고 해석 가능한 **Random Forest** 모델을 채택했습니다.

---

## 2. 시스템 아키텍처

본 시스템은 데이터 수집부터 차단까지 총 6단계의 파이프라인으로 구성됩니다.

1. **Traffic Capture:** Managed Switch 또는 인터페이스 미러링을 통해 패킷 수집
2. **Flow Extraction (NFStream):** 원시 패킷을 분석 가능한 플로우 특징 데이터로 변환
3. **ML Inference:** 학습된 모델(`Random Forest`)을 통해 트래픽의 이상 여부 판단
4. **Priority Logic:** AI 예측값과 사전에 정의된 휴리스틱 규칙을 결합해 최종 판정
5. **Active Response:** 공격 IP 자동 차단(`iptables`) 및 Discord 경보 발송

---

## 3. 탐지 대상 공격 및 대응 로직

시스템은 AI 판단과 하드코딩된 규칙을 결합한 **우선순위 로직**을 사용하여 오탐을 줄입니다.

| **공격 유형** | **탐지 기준 및 특징** | **우선순위** |
| --- | --- | --- |
| **Brute Force** | SSH(22), FTP(21) 포트에 대한 단시간 반복 접속 | 1순위 (최상) |
| **DDoS** | 80번 포트(HTTP) 집중 공격 및 패킷/플로우 빈도 급증 | 2순위 |
| **Port Scan** | 1초 내에 10개 이상의 서로 다른 포트 접속 시도 | 3순위 |

---

## 4. 기술 스택

- **언어:** Python 3.x
- **데이터 분석/ML:** Pandas, Scikit-learn (Random Forest), Joblib
- **네트워크 분석:** NFStream (Flow-based Traffic Intelligence)
- **보안/인프라:** iptables (Linux Firewall), Discord Webhook
- **데이터셋:** CIC-IDS-2017 (Reliable Network Traffic Dataset)

---

## 5. 실행 방법

### 1) 데이터 준비 및 학습

`Friday-WorkingHours.pcap` 파일을 기반으로 공격자 IP와 시간을 매칭하여 라벨링된 학습 데이터를 생성합니다.

```bash
python 1_create_multiclass_data.py
python 2_train_model.py
```

### 2) 실시간 탐지 실행

수집 인터페이스(`eth0` 등)를 지정하여 실시간 모니터링을 시작합니다.

```bash
sudo python 3_realtime_detect_7.py
```


---
## 🚀 How It Works
**Traffic Capture:** NFStream이 NIC를 모니터링하며 패킷을 통계 데이터로 요약합니다.

**Feature Extraction:** 모델이 학습한 것과 동일한 형태로 실시간 특성을 가공합니다.

**Prediction:** 모델이 해당 플로우를 Normal 또는 Attack으로 분류합니다.

**Action:** Attack으로 분류된 IP는 시스템에 의해 즉시 격리됩니다.

---

## 💡 프로젝트 구성을 돕기 위한 추가 문서 제안

현재 자료는 핵심 로직과 학습 과정에 집중되어 있습니다. 프로젝트의 완성도를 높이기 위해 다음과 같은 문서를 추가하는 것을 추천합니다.

1. **하드웨어 연결 구성도 (Physical Topology)**
    - Managed Switch와 라즈베리파이, 그리고 타겟 서버가 어떻게 물리적으로 연결되어 트래픽을 미러링하는지 설명하는 그림과 문서가 필요합니다.
2. **피처 중요도 분석 보고서 (Feature Importance)**
    - Random Forest 모델이 공격을 판단할 때 어떤 피처(`dst_port`, `bidirectional_packets` 등)가 가장 큰 영향을 미쳤는지 시각화하여 제공하면 모델의 신뢰성을 높일 수 있습니다.
3. **성능 평가 지표 (Model Evaluation)**
    - Confusion Matrix 및 F1-Score를 포함한 상세 성능 지표가 필요합니다. (코드에는 포함되어 있으나 결과 수치가 문서화되면 더 좋습니다.)
4. **IP 차단 관리 가이드**
    - `iptables`로 차단된 IP 리스트를 확인하거나, 오탐 시 차단을 해제(`sudo iptables -F`)하는 방법 등 운영 매뉴얼이 보완되어야 합니다.
