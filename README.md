# 출장비 부정 청구 탐지 시스템

> 영수증 이미지와 출장 경로 데이터를 Upstage Solar LLM, Kakao Map, n8n으로 연결한 지능형 비용 감사 자동화 시스템

## 프로젝트 요약

기업 출장비 심사는 영수증 텍스트만 확인하면 허위 출장, 경로 이탈, 업무 무관 품목, 과다한 유류비를 놓치기 쉽습니다. 이 프로젝트는 영수증에서 지출 정보를 추출하고, 출장 경로·기간·지출 품목·금액을 함께 검증해 감사 결과를 자동으로 생성합니다.

핵심 방향은 `LLM이 읽는 정보`와 `API·수식으로 확인하는 사실`을 분리한 뒤 n8n에서 하나의 판정 흐름으로 결합하는 것입니다.

## Demo

- [발표자료](./docs/presentation.pdf)
- [n8n 워크플로 이미지](./assets/workflow.png)
- 데모 영상: 업로드 후 링크 추가 예정

GitHub README에는 동영상 파일을 직접 커밋하기보다 YouTube, Loom 또는 GitHub Issue 댓글에 영상을 업로드한 뒤 링크를 연결하는 방식을 권장합니다.

```md
[![Demo video](https://img.youtube.com/vi/<VIDEO_ID>/0.jpg)](https://youtu.be/<VIDEO_ID>)
```

## 문제 정의

- 육안 검수와 신뢰에 의존하는 출장비 검증
- 영수증 OCR만으로는 지리적 정합성·유류비 적정성 판단 불가
- 다수 영수증 처리와 감사 결과 리포팅에 반복 작업 발생

## 해결 방식

1. 사용자가 영수증 이미지/PDF, 출장 시작일·복귀일, 출발지·목적지를 제출합니다.
2. Upstage Solar Pro 2가 영수증에서 지출일자, 지출처, 지출내용, 금액을 구조화합니다.
3. Solar Pro 2가 지출 항목을 회계 분류 체계로 분류합니다.
4. Kakao Local API와 Kakao Mobility Directions API로 출발지·목적지 좌표와 이동 거리를 계산합니다.
5. AI Judgement Agent가 출장 기간, 품목, 유류비 한도, 출장 경로 정보를 바탕으로 `합격/불합격`과 사유를 JSON으로 반환합니다.
6. 결과를 Google Sheets에 기록하고, 요약 리포트를 웹 화면으로 출력합니다. 이메일 전송은 선택 기능입니다.

## 시스템 흐름

```text
영수증/PDF + 출장 정보
        |
        v
n8n Webhook
        |
        +--> 다중 파일 분리
        |       +--> Solar Pro 2 정보 추출
        |       +--> Solar Pro 2 지출 분류
        |
        +--> Kakao 주소 검색 + 경로 거리 계산
                    |
                    v
        AI Judgement + Structured Output Parser
                    |
                    +--> Google Sheets 기록
                    +--> 결과 요약 화면
                    +--> 이메일 전송(선택)
```

## 핵심 기술

### Context-aware 영수증 분석

단순 텍스트 OCR 결과가 아니라 다음 필드를 JSON으로 추출합니다.

- `expense_date`
- `vendor`
- `description`
- `amount`

지출 분류 노드는 `유류대`, `여비교통비`, `복리후생비` 등 회계 항목을 분류합니다.

### Fact와 Claim 결합

- Fact: Kakao API가 계산한 출발지·목적지 좌표와 이동 거리
- Claim: 영수증에서 추출한 지출일자·지출처·지출내용·금액
- Logic: 출장 기간, 지출 품목, 유류비 한도, 경로 관련 규칙
- Output: `judgment`, `reason`만 포함하는 구조화된 판정 결과

### LLM 출력 안정화

`Structured Output Parser`와 JSON schema를 사용해 AI Judgement 결과를 두 필드로 제한합니다.

```json
{
  "judgment": "합격",
  "reason": "출장 기간과 지출 기준을 충족한 이유로 합격"
}
```

## 현재 판정 규칙

- 시간: 지출일시가 출장 시작일 00:00부터 복귀일 23:59 사이인지 확인
- 품목: 주류, 담배, 상품권, 유흥 관련 키워드 포함 여부 확인
- 유류비: `지출분류`가 `유류대`일 때 `(총거리 ÷ 10) × 1700 × 1.3` 한도와 청구 금액 비교
- 위치: 출발지·목적지·지출처 주소의 행정구역 또는 경로 일치 여부를 판단하도록 설계

## 현재 한계와 개선 방향

포트폴리오에서는 구현 범위와 보완점을 함께 공개하는 것이 신뢰도를 높입니다.

- 현재 정보 추출 schema가 `지출처 주소`를 별도 필드로 추출하지 않아 결제 지점의 실제 경로상 여부는 완전 검증이 어렵습니다.
- Kakao Directions의 거리 단위와 유류비 수식 단위를 명시적으로 통일해야 합니다. 운영 전 미터/킬로미터 변환을 테스트해야 합니다.
- 실제 데이터셋 기반 precision, recall, false positive 비율을 추가 측정할 수 있습니다.
- API key, webhook URL, Google Sheets ID는 환경 변수와 n8n Credentials로 분리해야 합니다.
- 공개 운영 시 webhook 인증, 요청 크기 제한, rate limit, 개인정보 보관 정책이 필요합니다.

## 실행 방법

### 1. n8n workflow import

1. n8n에서 `workflow.json`을 import합니다.
2. Upstage, Kakao, Google Sheets, Gmail Credentials를 연결합니다.
3. `YOUR_SHEET_ID`와 `your-n8n-host.example` placeholder를 실제 환경 값으로 교체합니다.
4. workflow를 활성화합니다.

`workflow.json`은 공개용으로 credential 객체와 실제 webhook·Google Sheets 식별자를 제거한 파일입니다. 원본 export는 `private/`에 보관하며 Git에 올리지 않습니다.

### 2. 입력값

백엔드 webhook `POST /webhook/apply`에 multipart form-data로 다음 값을 전달합니다.

- `file`: 영수증 이미지/PDF. 여러 파일 가능
- `start_date`: 출장 시작일
- `end_date`: 출장 복귀일
- `origin`: 출발지
- `destination`: 목적지

프론트엔드 `GET /webhook/index`는 제출 화면을 반환합니다.

## 저장소 구조

```text
.
├── README.md
├── workflow.json              # 공개용 n8n workflow
├── assets/
│   └── workflow.png           # workflow 시각화
├── docs/
│   └── presentation.pdf       # 발표자료
└── private/                   # 원본 export·개인 메모, Git 제외
```

## 팀

- 천창용: 팀장, AI Agent 처리, 발표
- 조희종: n8n workflow 총괄, 기능 구현
- 최현호: 거리 계산 시스템 구현, 기능 구현

## 사용 기술

`Upstage Solar Pro 2` · `n8n` · `Kakao Local API` · `Kakao Mobility Directions API` · `Google Sheets` · `Gmail`
