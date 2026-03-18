# AI·IoT 기반 능동형 루틴 관리 및 피드백 시스템

> **"입력하지 않아도 일상을 읽어내다"** — 행동 데이터 기반의 능동형 루틴 관리 및 AI 피드백 솔루션
> 

## 1. 프로젝트 개요

기존의 루틴 관리 앱(Todoist, Google Calendar 등)은 사용자가 직접 완료 여부를 기록해야 하는 **수동적(Passive)** 방식에 의존합니다. 이로 인해 실제 행동과 계획 사이의 괴리가 발생하며, 기록을 잊거나 거짓으로 기록할 경우 데이터의 신뢰성이 떨어집니다.

본 프로젝트는 **AI와 IoT 기술을 융합**하여 사용자의 실제 행동을 자동으로 인식하고, 계획된 루틴을 수행할 수 있도록 실시간 피드백을 제공하는 **능동형(Active)** 시스템을 지향합니다.

### 핵심 가치 (Core Value)

- **Focus Window:** 24시간 감시가 아닌, 사용자가 설정한 특정 루틴 시간대에만 작동하여 프라이버시를 보호합니다.
- **Multi-Modal Sensing:** 스마트폰 사용 로그와 IoT 센서(조도, 모션, BLE) 데이터를 결합하여 상황을 정밀하게 분석합니다.
- **Objective Feedback:** 주관적인 기억이 아닌 객관적인 데이터를 바탕으로 성공률을 예측하고 피드백을 제공합니다.

---

## 2. 주요 기능

- **실시간 루틴 모니터링:** 설정된 루틴 시간에 스마트폰 사용률이 높으면 "이탈(OFF_TASK)"로 판단하여 푸시 알림을 발송합니다.
- **AI 성공률 예측:** Random Forest 모델을 통해 과거 행동 패턴을 학습하고, 오늘의 루틴 성공 확률을 예측합니다.
- **수면 패턴 분석:** 조도 센서와 모션 센서를 활용하여 수면 시간 및 환경의 적절성을 분석합니다.
- **양방향 피드백 루프:** 알림에 대해 사용자가 "수행 중이에요" 등의 피드백을 주면 이를 시스템 판단에 반영합니다.
- **외부 서비스 연동:** Google Calendar API를 통해 기존 일정을 자동으로 동기화합니다.

---

## 3. 시스템 아키텍처 및 기술 스택

### 구성도

1. **Sensing Layer:** Flutter 앱(화면 상태 감지) 및 ESP32(조도/모션/BLE 비콘)
2. **Core Server Layer:** Raspberry Pi 기반 FastAPI 서버. 데이터 라우팅 및 Rule Engine 작동
3. **Action Layer:** 앱 푸시 알림 및 대시보드 업데이트

### 기술 스택 (Tech Stack)

| **분류** | **기술** |
| --- | --- |
| **Mobile** | Flutter, Screen State API, BLE Scanner |
| **IoT / Hardware** | ESP32, Raspberry Pi, Light/Motion Sensors |
| **Backend** | Python, FastAPI, SQLAlchemy, SQLite |
| **AI / ML** | Scikit-learn (Random Forest Classifier) |

---

## 4. 핵심 판단 로직

### 슬라이딩 윈도우 (Sliding Window)

순간적인 스마트폰 확인을 "이탈"로 오판하지 않기 위해 **최근 10분간의 데이터 흐름**을 분석합니다.

- **OFF_TASK (이탈):** 스마트폰 사용률 70% 이상
- **BORDERLINE (주의):** 스마트폰 사용률 30% 이상
- **ON_TASK (집중):** 스마트폰 사용률 30% 미만

### AI 학습 피처 (Features)

AI 모델은 다음 지표를 바탕으로 성공 확률을 계산합니다.

- 스마트폰 사용률 (Usage Ratio)
- 조도 비율 (Dark Ratio)
- 책상 체류 여부 (Seated Ratio / BLE RSSI)

---

## 5. 시작하기 (Setup)

### 하드웨어 구성

1. ESP32에 조도(LDR) 및 모션(PIR) 센서를 연결합니다.
2. Raspberry Pi에 FastAPI 서버 환경을 구축합니다.

### 소프트웨어 설치

```bash
# Backend (Raspberry Pi)
cd backend
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0

# Mobile (Flutter)
cd mobile
flutter pub get
flutter run
```

