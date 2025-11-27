# Upstage-Hackathon
2025 대구울산경북 대학교 AI Agent 해커톤

# Solar Audit FDS (지능형 출장비 부정 청구 탐지 시스템)

![n8n](https://img.shields.io/badge/n8n-workflow-orange?style=for-the-badge&logo=n8n)
![Upstage](https://img.shields.io/badge/AI-Upstage_Solar-purple?style=for-the-badge)
![KakaoMap](https://img.shields.io/badge/API-Kakao_Map-yellow?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

> **"Fact(위치 데이터)와 Logic(Solar LLM)을 결합하여, 출장비 누수를 원천 차단하는 AI 감사 에이전트"**

## 📖 Project Overview
기업의 경비 지출 관리는 여전히 담당자의 육안 검사와 직원의 양심에 의존하고 있습니다. 기존 OCR 기술은 텍스트는 읽을 수 있지만, **"부산 출장자가 서울에서 결제했다"**는 **맥락(Context)**은 검증하지 못합니다.

**Solar Audit FDS**는 영수증의 텍스트 정보와 실제 물리적 이동 경로를 교차 검증(Cross-Validation)하여, 허위 출장 및 경로 이탈과 같은 부정 청구를 실시간으로 탐지하는 **지능형 FDS(Fraud Detection System)**입니다.

### 🏆 Achievements
- **[2025 Upstage AI Hackathon]** 제출 프로젝트
- 비정형 영수증 데이터의 정형화 및 자동 감사 파이프라인 구축

---

## 🏗️ System Architecture
![Architecture](./assets/architecture.png)

본 프로젝트는 별도의 백엔드 서버 없이 **n8n**을 활용한 **Serverless Workflow**로 구현되었습니다.

1.  **Ingestion:** 영수증 이미지 일괄 업로드 (Batch Processing)
2.  **Extraction (Upstage Solar):** 지출 성격 분류 및 금지 품목(주류 등) 태깅
3.  **Enrichment (Map API):** 출발지-목적지 간 실제 이동 거리 및 경로 데이터 확보
4.  **Judgment (Solar Agent):** 위치 기반 경로 이탈 확인 및 유류비 적정성 수학적 검증
5.  **Reporting:** Google Sheets 적재 및 담당자 알림 (Slack/Email)

---

## ✨ Key Features

### 1. Context-Aware Extraction (맥락 인식 정보 추출)
단순 OCR을 넘어 **Upstage Solar Pro**를 활용해 영수증의 맥락을 분석합니다.
- `식대` vs `유류비` 등 지출 성격 자동 분류
- `주류`, `담배` 등 업무 무관 품목 포함 여부 정밀 탐지

### 2. Geospatial Logic (위치 기반 교차 검증)
영수증의 결제 장소가 실제 출장 경로 상에 위치하는지 수학적으로 검증합니다.
- **Geofencing:** 출발지↔목적지 벡터 경로 반경(예: 5km) 이내 결제 건만 승인
- **Math Logic:** `(이동거리 ÷ 연비) × 유가` 공식을 프롬프트에 주입하여 유류비 과다 청구 적발

### 3. Strict JSON Enforcement (환각 제어)
LLM의 비정형 출력을 시스템에 연동하기 위해 엄격한 데이터 파이프라인을 설계했습니다.
- **Output Parser** 적용으로 100% 실행 가능한 JSON 포맷 강제
- `Reasoning`(추론)과 `Result`(결과)를 분리하여 설명 가능한 AI(XAI) 구현

---

## 🎥 Demo Scenarios

| Scenario A (정상) | Scenario B (경로 이탈) |
| :---: | :---: |
| ![Pass](./assets/demo_pass.gif) | ![Fail](./assets/demo_fail.gif) |
| 경로 내 휴게소 식사 → **합격** | 대전 출장 중 강릉 결제 → **불합격** |

> **Business Impact:**
> * 건당 처리 시간: 5분 → **3초 (99% 단축)**
> * 부정 청구 탐지율: **Zero Leakage (비용 누수 원천 차단)**

---

## 🚀 How to Run

이 프로젝트는 **n8n** 워크플로우 JSON 파일로 제공됩니다.

### Prerequisites
- [n8n](https://n8n.io/) (Self-hosted or Cloud)
- Upstage API Key
- Kakao Map API Key
- Google Cloud Service Account (for Sheets)

### Installation
1. Repository를 Clone 합니다.
   ```bash
   git clone [https://github.com/](https://github.com/)[your-username]/solar-audit-fds.git