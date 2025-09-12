# InsightOps - VoC 분석 및 관리 플랫폼

<!--
README formatter scaffold
- 위쪽: 포맷/테이블/이모지로 가독성 강화 (내용 추가이지만 요약/정리용)
- 아래쪽: "📎 원문(수정 없음)" 섹션에 제공된 텍스트를 한 글자도 바꾸지 않고 그대로 보존
-->

<div align="center">

# 🧠 InsightOps — VoC 분석 & 집계 리포지토리

AI 기반 실시간 VoC 인입 → 정규화/분류 → 집계/시각화 → 메일 생성·발송까지 이어지는 **MSA 아키텍처** 프로젝트

</div>

---

## 🚀 빠른 시작 (로컬 실행 요약)

| # | 서비스 | 포트 | 실행 명령 |
|---|---|---|---|
| 1 | Voicebot Service | 3000 | `cd InsightOps-realtime-voicebot-main && npm install && npm run dev` |
| 2 | Classification Service | 8080 | `cd InsightOps-classfication-main && ./mvnw spring-boot:run` |
| 3 | Dashboard Backend | 3001 | `cd InsightOps-dashboard-backend-main && ./gradlew bootRun` |
| 4 | Dashboard Frontend | 3002 | `cd InsightOps-dashboard-frontend-main && npm install && npm run dev` |
| 5 | Mail Send Service | 8081 | `cd InsightOps-mail-send-main && ./gradlew bootRun` |
| 6 | MailContents Service | 8082 | `cd InsightOps_MailContents-main && ./gradlew bootRun` |
| 7 | Admin Service | 8083 | `cd InsightOps_Admin-main && ./gradlew bootRun` |

> 🔑 **환경변수**  
> `OPENAI_API_KEY`, `DATABASE_URL`, `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`,  
> `CLASSIFICATION_SERVICE_URL`, `MAIL_SERVICE_URL`, `DASHBOARD_API_URL`

---

## 🧩 마이크로서비스 & 역할

| 영역 | 서비스명 | 설명 |
|---|---|---|
| Frontend | InsightOps-dashboard-frontend | VoC 데이터 시각화 및 관리 대시보드 |
| Backend | InsightOps-dashboard-backend | 대시보드 API 서버 및 데이터 집계 |
| Backend | InsightOps-classification | AI 기반 VoC 분류/분석 |
| Backend | InsightOps-realtime-voicebot | 실시간 AI 음성 상담 |
| Backend | InsightOps-mail-send | 이메일 발송 |
| Backend | InsightOps-MailContents | 메일 내용 자동 생성 |
| Backend | InsightOps-Admin | 데이터베이스 관리 API |

---

## 🗺️ 서비스 호출 흐름(요약)

1) 고객 실시간 상담(Voicebot) → **voc_raw 저장**  
2) Classification이 자동 분류/분석 → **consulting_classification 저장**  
3) Dashboard Backend가 집계 생성(스케줄러) → **집계/캐시 테이블 저장**  
4) Dashboard에서 MailContents 호출 → **Admin/Classification 조회 후 템플릿 구성**  
5) Mail Send가 **Microsoft Graph API**로 메일 발송

---

## ☁️ Azure 배포 URL

| 서비스 | URL |
|---|---|
| Admin | (http://insightops-admin-bnbchyhyc3hzb8ge.koreacentral-01.azurewebsites.net/) |
| Voicebot | [https://insightops-voicebot.azurewebsites.net](https://insightops-voicebot-aud7gfhwc3fsb3h7.koreacentral-01.azurewebsites.net/) |
| Classification | [https://insightops-classification.azurewebsites.net](https://insightops-classification-d2acc8afftgmhubt.koreacentral-01.azurewebsites.net) |
| Dashboard Backend | [https://insightops-dashboard-back.azurewebsites.net](insightops-dashboard-back-a7hwh6hxcqcxcqgh.koreacentral-01.azurewebsites.net) |
| Dashboard Frontend | [https://insightops-dashboard-front.azurewebsites.net](insightops-dashboard-front-f9ambdbjbje5gbe0.koreacentral-01.azurewebsites.net) |
| Mail Contents | [https://insightops-mailcontents.azurewebsites.net](https://insightops-mailcontents-effpd4dmepa8czgm.koreacentral-01.azurewebsites.net/) |
| Mail Send | [https://insightops-mailsend.azurewebsites.net](insightops-mailsend-e4drbwhqhge4bzam.koreacentral-01.azurewebsites.net) |

---

## 🗄️ 데이터베이스 구조(스키마 분리)

| 스키마 | 주요 테이블/특징 |
|---|---|
| `admin` | `assignee`, `consulting_category` |
| `voicebot` | `voc_raw` |
| `normalization` | `consulting_classification` |
| `insightops_dashboard` | `message_preview_cache`, `agg_by_category_age_gender`, 일/주/월 집계 |

> 🔒 **원칙**: 서비스 간 통신은 **API Only**, 데이터 FK로 직접 참조하지 않음

---

## 🧪 헬스체크 & 모니터링

- **Actuator**: `/actuator/health`, `/actuator/info`, `/actuator/metrics`
- **Custom Health**: 각 서비스별 `/api/health`  
- **스케줄러**: 매분 실행 `@Scheduled(cron = "0 * * * * ?")`  
- **지표**: 분류 성공률(목표 95%+), 처리 시간, OpenAI 사용량, 재시도 횟수

---

## 📬 메일 생성/발송 플로우(요약)

`Dashboard → MailContents`  
→ `Admin(담당자/카테고리)` + `Classification(분석결과 최대 3개)` 조회  
→ **정적 템플릿으로 메일 내용 생성(LLM 없음)**  
→ `Mail Send`를 통해 **비동기 발송(Microsoft Graph API)**

---

## 🧰 기술 스택

**Frontend**: React 18 + TypeScript + Vite + Tailwind + Recharts  
**Backend**: Spring Boot 3.x, Java 17, JPA/Hibernate, WebFlux(메일), Next.js(Voicebot)  
**DB/ORM**: MySQL 8.0, JPA/Hibernate, Prisma  
**Infra**: Azure, Docker, Azure AD, Actuator

---

## 🧱 Docker 배포

```bash
docker-compose up -d

# buildx 예시
docker buildx build --platform linux/amd64 -t insightops-dashboard-backend ./InsightOps-dashboard-backend-main-2 --load
docker buildx build --platform linux/amd64 -t insightops-classification ./InsightOps-classfication-main --load
docker buildx build —platform linux/amd64 -t insightops-voicebot ./InsightOps-realtime-voicebot-main —load
docker buildx build —platform linux/amd64 -t insightops-mail ./InsightOps-mail-send-main —load
docker buildx build —platform linux/amd64 -t insightops-frontend ./InsightOps-dashboard-frontend-main-2 —load

### 서비스 소개

InsightOps는 VoC를 실시간으로 수집, 분석하여 비즈니스 인사이트를 제공하는 종합 플랫폼입니다.

VoC 데이터를 인입/정규화하여 NLP로 분류하고, 정책 기반 인사이트를 산출해 아웃룩을 통해 메일로 전달합니다.

AI 기반 자동 분류, 실시간 음성 상담, 대시보드 시각화, 자동 메일 발송까지 통합된 마이크로서비스 아키텍처로 구성되어 있습니다.

### 팀 정보

- **팀 이름**: 허준효
- **프로젝트명**: InsightOps - AI-Powered VoC Analytics Platform
- **한줄 소개**: AI와 실시간 음성 기술을 활용한 차세대 VoC 분석 플랫폼

### 팀 노션 링크

[Team notion link](https://www.notion.so/25cf8f01056180c38be1f1c1cf16e14e?pvs=21)

### 담당 서비스

1. admin (은세) : [admin](https://github.com/Si1verBird/InsightOps_Admin)
2. 인입 (준선) : [voicebot](https://github.com/s4nta1999/InsightOps-realtime-voicebot)
3. 정규화 (준선) : [classification](https://github.com/s4nta1999/InsightOps-classfication)
4. 대시보드 (인효) : [backend](https://github.com/s4nta1999/InsightOps-dashboard-backend) / [frontend](https://github.com/s4nta1999/InsightOps-dashboard-frontend)
5. 메일내용생성 (은세) : [mailcontents](https://github.com/Si1verBird/InsightOps_MailContents)
6. 메일보내기 (준선) : [mailsender](https://github.com/s4nta1999/InsightOps-mail-send) + [(최종본_은세)](https://github.com/Si1verBird/InsightOps_MailSender)

---

## 서비스 목록

### 1. Frontend Services

- **`InsightOps-dashboard-frontend`**: VoC 데이터 시각화 및 관리 대시보드

### 2. Backend Services

- **`InsightOps-dashboard-backend`**: 대시보드 API 서버 및 데이터 집계
- **`InsightOps-classification`**: AI 기반 VoC 분류 및 분석 서비스
- **`InsightOps-realtime-voicebot`**: 실시간 AI 음성 상담 시스템
- **`InsightOps-mail-send`**: 이메일 발송 마이크로서비스
- **`InsightOps-MailContents`**: 메일 내용 자동 생성 서비스
- **`InsightOps-Admin`**: 데이터베이스 관리 API 서비스

---

## 서비스 호출 흐름

### 1. 실시간 상담 및 데이터 수집

1. 고객이 실시간 음성 상담 시작 (`InsightOps-realtime-voicebot`)
2. OpenAI Realtime API를 통한 음성-텍스트 변환 및 AI 상담
3. 상담 내용을 `voc_raw` 테이블에 저장 (원본 데이터)

### 2. AI 분류 및 분석

1. 수집된 상담 데이터를 분류 서비스로 전송 (`InsightOps-classification`)
2. OpenAI GPT-4o-mini 모델을 통한 자동 카테고리 분류 (25개 주요 카테고리)
3. 감정 분석, 키워드 추출, 긴급도 판정, 권장 조치사항 생성
4. 분류 결과를 `consulting_classification` 테이블에 JSON 형태로 저장

### 3. 대시보드 데이터 집계

1. 대시보드 백엔드가 분류된 데이터 수집 (`InsightOps-dashboard-backend`)
2. 스케줄러를 통한 일/주/월 단위 통계 데이터 생성 (매분 실행)
3. 카테고리별 비중, 트렌드 분석, 집계 데이터 생성
4. `message_preview_cache`, 집계 테이블들에 결과 저장

### 4. 메일 내용 생성 및 발송

1. **메일 생성 요청**: 대시보드에서 `InsightOps-MailContents` 서비스 호출
2. **데이터 수집**: Admin API에서 카테고리명 + 담당자 정보 조회
3. **VOC 분석**: Classification API에서 VOC 분석 결과 조회 (최대 3개)
4. **템플릿 생성**: 정적 템플릿 기반 메일 내용 자동 생성 (LLM 없음)
5. **자동 발송**: `InsightOps-mail-send`를 통한 비동기 메일 발송

### 5. 시각화 및 액션

1. 프론트엔드에서 집계된 데이터 시각화 (`InsightOps-dashboard-frontend`)
2. React + Recharts를 통한 파이차트, 라인차트, 막대그래프 렌더링
3. 사용자가 인사이트 확인 및 메일 발송 액션 수행
4. 메일 서비스를 통한 담당자별 자동 알림 발송 (`InsightOps-mail-send`)

---

## 실행 방식

### 로컬 개발 환경

각 서비스를 개별적으로 실행:

```bash
# 1. Voicebot Service (포트: 3000)
cd InsightOps-realtime-voicebot-main
npm install && npm run dev

# 2. Classification Service (포트: 8080)
cd InsightOps-classfication-main
./mvnw spring-boot:run

# 3. Dashboard Backend (포트: 3001)
cd InsightOps-dashboard-backend-main
./gradlew bootRun

# 4. Dashboard Frontend (포트: 3002)
cd InsightOps-dashboard-frontend-main
npm install && npm run dev

# 5. Mail Service (포트: 8081)
cd InsightOps-mail-send-main
./gradlew bootRun

# 6. MailContents Service (포트: 8082)
cd InsightOps_MailContents-main
./gradlew bootRun

# 7. Admin Service (포트: 8083)
cd InsightOps_Admin-main
./gradlew bootRun

```

### 환경변수 설정 (로컬)

```bash
# OpenAI API 설정
export OPENAI_API_KEY="your-openai-api-key"

# 데이터베이스 설정
export DATABASE_URL="mysql://username:password@host:3306/database_name"

# Azure AD 설정 (메일 서비스)
export AZURE_TENANT_ID="your-tenant-id"
export AZURE_CLIENT_ID="your-client-id"
export AZURE_CLIENT_SECRET="your-client-secret"

# 서비스 간 연동 URL
export CLASSIFICATION_SERVICE_URL="<http://localhost:8080>"
export MAIL_SERVICE_URL="<http://localhost:8081>"
export DASHBOARD_API_URL="<http://localhost:3002>"

```

### 클라우드 배포 (Azure)

1. **Admin Service**: [https://insightops-admin.azurewebsites.net](http://insightops-admin-bnbchyhyc3hzb8ge.koreacentral-01.azurewebsites.net/)
2. **Voicebot Service**: [https://insightops-voicebot.azurewebsites.net](https://insightops-voicebot-aud7gfhwc3fsb3h7.koreacentral-01.azurewebsites.net/)
3. **Classification Service**: [https://insightops-classification.azurewebsites.net](https://insightops-classification-d2acc8afftgmhubt.koreacentral-01.azurewebsites.net)
4. **Dashboard Backend**: [https://insightops-dashboard-back.azurewebsites.net](insightops-dashboard-back-a7hwh6hxcqcxcqgh.koreacentral-01.azurewebsites.net)
5. **Dashboard Frontend**: [https://insightops-dashboard-front.azurewebsites.net](insightops-dashboard-front-f9ambdbjbje5gbe0.koreacentral-01.azurewebsites.net)
6. **Mail Contents Service:** [https://insightops-mailcontents.azurewebsites.net](https://insightops-mailcontents-effpd4dmepa8czgm.koreacentral-01.azurewebsites.net/)
7. **Mail Send Service**: [https://insightops-mailsend.azurewebsites.net](insightops-mailsend-e4drbwhqhge4bzam.koreacentral-01.azurewebsites.net)

---

## 데이터베이스 구성

### 서비스별 데이터베이스 분리

각 마이크로서비스는 독립적인 데이터베이스 스키마를 사용하여 데이터 격리와 서비스 자율성을 보장합니다. (데이터베이스 서버는 동일)

### 0. Admin Service Database

- **Database**: MySQL 8.0 (Azure Database for MySQL)
- **Schema**: `admin`
- **Connection**: JDBC (Spring Boot)
- **주요 테이블**:
    - `assignee` - 담당자 정보
    - `consulting_category` - 상담 카테고리 마스터, assignee 맵핑 정보 포함

```sql
-- assignee 테이블
CREATE TABLE assignee (
    assignee_id    CHAR(8) NOT NULL PRIMARY KEY,   -- 8자리 uuid
    assignee_email VARCHAR(255) NOT NULL,
    assignee_name  VARCHAR(100) NOT NULL,
    assignee_team  VARCHAR(100),
    assignee_phone VARCHAR(20),
    created_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;

-- consulting_category 테이블
CREATE TABLE consulting_category (
    category_id          CHAR(8) NOT NULL,
    assignee_id          CHAR(8) NOT NULL,
    consulting_category  VARCHAR(100) NOT NULL,
    created_at           TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at           TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (category_id),
    CONSTRAINT fk_consulting_category_assignee
        FOREIGN KEY (assignee_id) REFERENCES assignee(assignee_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

### 1. Voicebot Service Database

- **Database**: MySQL 8.0 (Prisma ORM)
- **Schema**: `voicebot`
- **주요 테이블**:
    - `voc_raw` - 원본 상담 데이터 (source_id, consulting_content, client_age, client_gender 등)

```sql
CREATE TABLE voc_raw (
    source_id VARCHAR(255) PRIMARY KEY,
    consulting_date DATETIME NOT NULL,
    client_gender VARCHAR(10) NOT NULL,
    client_age INT NOT NULL,
    consulting_turns INT NOT NULL,
    consulting_length INT NOT NULL,
    consulting_content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

```

### 2. Classification Service Database

- **Database**: MySQL 8.0
- **Schema**: `normalization`
- **주요 테이블**:
    - `voc_nomalized` - 분류된 상담 데이터 + AI 분석 결과 (JSON)

```sql
CREATE TABLE consulting_classification (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    source_id VARCHAR(255) NOT NULL UNIQUE,
    consulting_content TEXT NOT NULL,
    consulting_date DATETIME NOT NULL,
    client_age INT NOT NULL,
    client_gender VARCHAR(10) NOT NULL,
    consulting_category VARCHAR(100),
    category_id VARCHAR(50),
    confidence DOUBLE,
    analysis_result JSON,  -- AI 분석 결과 (summary, urgency, sentiment, keywords, recommendedAction)
    processing_time DOUBLE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

```

### 3. Dashboard Backend Database

- **Database**: MySQL 8.0 (Azure Database for MySQL)
- **Schema**: `insightops_dashboard`
- **ORM**: JPA/Hibernate (Spring Boot)
- **주요 테이블**:
    - `message_preview_cache` - 메일 발송 캐시
    - `agg_by_category_age_gender` - 카테고리/연령/성별 집계
    - 집계 테이블들 (daily_aggregations, weekly_aggregations, monthly_aggregations)

```sql
-- 메시지 프리뷰 캐시
CREATE TABLE message_preview_cache (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    voc_id VARCHAR(255),
    subject VARCHAR(500),
    content TEXT,
    consulting_category VARCHAR(100),
    assignee_email VARCHAR(255),
    status VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 카테고리별 집계 데이터
CREATE TABLE agg_by_category_age_gender (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    aggregation_date DATE NOT NULL,
    consulting_category VARCHAR(100) NOT NULL,
    client_age INT NOT NULL,
    client_gender VARCHAR(10) NOT NULL,
    count INT DEFAULT 0,
    avg_processing_time DOUBLE,
    avg_confidence DOUBLE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

### 5. Stateless Services

### MailContents Service

- **Database**: 없음 (API 연동)
- **연동 API**: Admin API, Classification API, Mail Send API

### MailSend Service

- **Database**: 없음 (Stateless)
- **외부 연동**:
    - Microsoft Graph API (메일 발송)
    - Admin Service API (담당자 이메일 조회)

### 마이크로서비스별 데이터 흐름

```
[고객 음성 상담]
    ↓ voc_raw 저장
[Voicebot Service - MySQL: voicebot]
    ↓ API 호출 (원본 데이터 전송)
[Classification Service - MySQL: normalization]
    ↓ API 호출 (분류된 데이터 조회)
[Dashboard Backend - MySQL: insightops_dashboard]
    ↓ API 호출 (메일 생성 요청)
[MailContents Service - Stateless]
    ↓ Admin API 호출 (담당자 조회)
[Admin Service - MySQL: admin_db]
    ↓ 메일 발송 요청
[Mail Service - Microsoft Graph API]

```

### 서비스 간 통신 원칙 (API Only - No Database FK)

- **Voicebot → Classification**: HTTP API (상담 데이터 전송)
- **Classification → Dashboard**: HTTP API (분류 결과 조회)
- **Dashboard → MailContents**: HTTP API (메일 생성 요청)
- **MailContents → Admin**: HTTP API (담당자/카테고리 조회)
- **MailContents → Classification**: HTTP API (VOC 분석 결과 조회)
- **MailContents → MailSender**: HTTP API (메일 발송 요청)

### 서비스 간 데이터 연동 방식

1. **Voicebot → Classification**: REST API 호출 (`POST /api/enhanced-classify`)
2. **Classification → Dashboard**: REST API 호출 (`POST /api/normalized/voc-list`)
3. **Dashboard → Mail**: REST API 호출 (`POST /api/sendmail`)
4. **Mail → Admin**: REST API 호출 (담당자 이메일 조회)

---

## 기술 스택 요약

### Frontend

- **Framework**: React 18 + TypeScript + Vite
- **UI**: Tailwind CSS + Recharts (파이차트, 라인차트, 막대그래프)
- **State Management**: React Hooks + Custom Hooks
- **Routing**: React Router DOM
- **HTTP Client**: Axios

### Backend Services

- **Dashboard Backend**: Spring Boot 3.x + Java 17 + JPA/Hibernate + MySQL
- **Classification Service**: Spring Boot 3.x + Java 17 + OpenAI API + MySQL
- **Voicebot Service**: Next.js 14 + TypeScript + OpenAI Realtime API + Prisma + MySQL
- **MailContents & MailSender Service**: Spring Boot 3.x + Java 17 + Microsoft Graph API + WebFlux

### Database & ORM

- **Database**: MySQL 8.0 (Azure Database for MySQL)
- **ORM**: JPA/Hibernate (Spring), Prisma (Next.js)
- **Connection Pool**: HikariCP

### Infrastructure

- **Cloud Platform**: Microsoft Azure
- **Container**: Docker
- **Authentication**: Azure AD (Mail Service)
- **Monitoring**: Spring Boot Actuator + Custom Health Endpoints

---

## 핵심 기능 및 특징

### AI 기반 자동 분류 시스템

- **OpenAI GPT-4o-mini 모델**: 상담 내용 자동 분류 (25개 주요 카테고리)
- **분류 정확도**: 평균 95% 신뢰도
- **분류 응답 시간**: 평균 7.3초 (OpenAI API 호출 포함)
- **재시도 메커니즘**: API 장애 시 최대 3회 자동 재시도
- **향상된 분석**: 감정 분석, 키워드 추출, 긴급도 판정, 권장 조치사항

```json
{
  "category": "한도관리_한도증액신청",
  "confidence": 0.95,
  "analysis": {
    "summary": "카드 한도 증액 신청 관련 문의",
    "urgency": "MEDIUM",
    "sentiment": "NEUTRAL",
    "keywords": ["한도", "증액", "신청"],
    "recommendedAction": "한도 증액 절차 안내"
  }
}

```

### 실시간 음성 상담 시스템

- **OpenAI Realtime API**: 실시간 음성-텍스트 변환 및 AI 상담
- **WebRTC 기반**: 양방향 실시간 음성 스트리밍
- **세션 관리**: 상담 세션 생성, 추적, 종료 처리
- **데이터 수집**: 상담 턴 수, 상담 길이, 고객 정보 자동 수집

### 마이크로서비스 아키텍처

- **독립 배포**: 각 서비스별 독립적인 Docker 컨테이너 배포
- **RESTful API 연동**:
    - Dashboard Backend: 19개 API 엔드포인트
    - Classification Service: 9개 API 엔드포인트
    - Voicebot Service: 10개 API 엔드포인트
    - Mail Contents Service: 2개 API 엔드포인트
    - Mail send Service: 5개 API 엔드포인트
- **데이터베이스 분리**: 서비스별 MySQL 스키마 분리 운영
- **비동기 처리**: Spring WebFlux 기반 메일 발송 처리

### 실시간 대시보드 및 시각화

- **KPI 카드**: 총 VoC 수, 일/주/월별 변화율
- **파이차트**: Big Category 비중 분석
- **라인차트**: 시계열 VoC 변화량 추이
- **막대그래프**: Small Category 트렌드 분석
- **필터링**: 연령대, 성별, 카테고리별 다차원 필터링
- **집계 데이터**: 카테고리/연령/성별 다차원 분석

### 자동 메일 발송 시스템

- **Microsoft Graph API**: Enterprise 급 메일 발송
- **카테고리별 담당자**: Admin Service 연동으로 자동 담당자 할당
- **HTML 메일**: 리치 컨텐츠 메일 발송 지원
- **발송 내역 관리**: 메일 발송 로그 및 상태 추적

---

## 모니터링 및 운영

### 실제 구현된 모니터링 기능

### 1. Spring Boot Actuator (Backend Services)

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: always

```

- **Health Check**: `/actuator/health` - 서비스 상태 및 의존성 확인
- **Info Endpoint**: `/actuator/info` - 애플리케이션 정보
- **Metrics**: `/actuator/metrics` - 성능 메트릭

### 2. 서비스별 Custom Health API

- **Dashboard Backend**: `GET /api/health` - 데이터베이스 연결 상태
- **Classification Service**: `GET /api/health` - OpenAI API 연결 상태
- **Voicebot Service**: `GET /api/health` - Prisma DB 연결 상태
- **Mail Service**: `GET /api/health` - Microsoft Graph API 연결 상태
- **MailContents Service**: `GET /api/mail/health` - API 연동 서비스 상태
- **Admin Service**: `GET /api/admin/health` - Admin DB 연결 상태

### 3. 실시간 메트릭 수집

```tsx
// Voicebot Service 메트릭
interface ServiceMetrics {
  totalSessions: number;        // 총 상담 세션 수
  activeSessions: number;       // 활성 세션 수
  avgSessionLength: number;     // 평균 상담 길이
  dailyVocCount: number;        // 일일 VoC 수집량
  classificationSuccess: number; // 분류 성공률
}

```

### 4. 분류 성능 모니터링

- **분류 성공률**: 95% 이상 유지 목표
- **처리 시간**: 평균 1-3초 (OpenAI API 응답 시간 포함)
- **OpenAI API 사용량**: 일일 토큰 사용량 추적
- **재시도 횟수**: 실패 시 재시도 통계

### 5. 데이터 집계 스케줄러 모니터링

```java
// Dashboard Backend 스케줄러
@Scheduled(cron = "0 * * * * ?") // 매분 실행
public void aggregateData() {
    // 일/주/월별 데이터 집계
    // 실행 시간 및 처리량 로깅
}

```

### 로깅 설정

```yaml
# 통합 로깅 설정
logging:
  level:
    root: INFO
    com.insightops: DEBUG
    com.hanacard: DEBUG
    com.example.microservicemail: DEBUG
    org.springframework.web: WARN
    org.hibernate.SQL: WARN
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"

```

### 에러 처리 및 복구 전략

### 1. OpenAI API 장애 대응

```java
@Retryable(value = {Exception.class}, maxAttempts = 3, backoff = @Backoff(delay = 1000))
public ClassificationResult classifyWithRetry(String content) {
    // 최대 3회 재시도, 1초 간격
}

```

### 2. 데이터베이스 연결 관리

```yaml
# HikariCP 설정
spring:
  datasource:
    hikari:
      connection-timeout: 60000
      maximum-pool-size: 20
      minimum-idle: 5
      idle-timeout: 300000
      max-lifetime: 1800000

```

### 3. 메일 발송 실패 처리

```java
// WebFlux 기반 비동기 재시도
public Mono<String> sendMailWithRetry(SendMailRequest request) {
    return mailService.sendMail(request)
        .retry(3)
        .timeout(Duration.ofSeconds(30));
}

```

### 보안 구현 사항

### 1. 입력 검증 및 SQL 인젝션 방지

```java
// 모든 API 요청에 대한 검증
@PostMapping("/api/classify")
public ResponseEntity<ApiResponse> classify(@Valid @RequestBody ClassificationRequest request) {
    // @Valid 어노테이션으로 입력 검증
    // JPA 파라미터 바인딩으로 SQL 인젝션 방지
}

```

### 2. CORS 설정

```java
@CrossOrigin(origins = "<http://localhost:3000>")
public class DashboardController {
    // 프론트엔드에서의 API 호출 허용
}

```

### 3. Azure AD 기반 인증 (Mail Service)

```yaml
# Microsoft Graph API 인증
azure:
  tenant-id: ${AZURE_TENANT_ID}
  client-id: ${AZURE_CLIENT_ID}
  client-secret: ${AZURE_CLIENT_SECRET}

```

### 4. 개인정보 보호

- 로그에서 민감정보 마스킹
- API 응답에서 개인식별정보 제거
- 데이터베이스 암호화 (Azure MySQL SSL 연결)

### 성능 최적화

### 1. 데이터베이스 인덱스 최적화

```sql
-- 주요 쿼리 성능 향상을 위한 인덱스
CREATE INDEX idx_consulting_date ON voc_raw (consulting_date);
CREATE INDEX idx_category_id ON consulting_classification (category_id);
CREATE INDEX idx_created_at ON insight_cards (created_at);

```

### 2. API 응답 캐싱

- 대시보드 집계 데이터 캐싱 (1분 TTL)
- 인사이트 카드 캐싱 (5분 TTL)
- 카테고리 정보 캐싱 (1시간 TTL)

### 3. 배치 처리 최적화

- VoC 분류 배치 크기: 10개 단위
- 동시 처리 제한: 최대 5개 배치
- 메모리 사용량 모니터링

---

## API 명세 요약

### 1. Dashboard Backend (19개 API)

**데이터 조회 API**

- `GET /api/dashboard/overview` - 오버뷰 KPI 데이터
- `GET /api/dashboard/big-category-share` - 카테고리 비중 (파이차트)
- `GET /api/dashboard/total-series` - 시계열 데이터 (라인차트)
- `GET /api/dashboard/small-trends` - 트렌드 분석 (막대그래프)
- `GET /api/dashboard/insights` - 인사이트 카드 Top 10
- `GET /api/dashboard/cases` - VoC 사례 목록

**시계열 분석 API**

- `GET /api/dashboard/timeseries` - 기본 시계열 데이터
- `GET /api/dashboard/category-timeseries` - 카테고리별 시계열
- `GET /api/dashboard/age-timeseries` - 연령대별 시계열
- `GET /api/dashboard/gender-timeseries` - 성별 시계열

**메일 연동 API**

- `GET /api/dashboard/mail/recent` - 최근 메일 발송 내역
- `POST /api/dashboard/mail/preview` - 메일 초안 생성
- `POST /api/dashboard/mail/send` - 메일 발송
- `POST /api/dashboard/mail/generate` - 카테고리 기반 메일 생성

**관리 API**

- `POST /api/dashboard/filtered-cases` - 필터링된 VoC 목록
- `POST /api/dashboard/manual-aggregation` - 수동 데이터 집계
- `GET /api/dashboard/aggregation-status` - 집계 상태 확인
- `GET /api/dashboard/voc-detail/{vocEventId}` - VoC 상세보기
- `GET /api/health` - 서비스 상태 확인

### 2. Classification Service (9개 API)

**분류 API**

- `POST /api/classify` - 기본 상담 분류
- `POST /api/enhanced-classify` - 향상된 분류 + 분석

**데이터 조회 API**

- `GET /api/normalization/voc_normalized` - 카테고리별 분석 결과 (MailContents용)
- `POST /api/normalized/voc-list` - VoC 목록 조회 (Dashboard용)
- `GET /api/normalized/voc-detail/{vocEventId}` - VoC 상세보기

**관리 API**

- `GET /api/health` - 서비스 상태 확인
- `GET /api/test` - 배포 상태 테스트
- `GET /api/categories` - 사용 가능한 카테고리 정보
- `GET /api/` - 서비스 정보

### 3. Voicebot Service (10개 API)

**상담 관리 API**

- `GET /api/consultations` - 상담 기록 조회
- `POST /api/save-conversation` - 상담 내용 저장
- `GET /api/session` - 세션 정보 조회

**분류 연동 API**

- `POST /api/batch-classify` - 배치 분류 처리
- `GET /api/classification-status` - 분류 상태 확인

**분석 API**

- `GET /api/data-analysis` - VoC 데이터 분석
- `GET /api/voc/count-summary` - VoC 카운트 요약
- `GET /api/responses` - 응답 데이터 조회
- `POST /api/safe-update-dates` - 안전한 날짜 업데이트
- `GET /api/health` - 서비스 상태 확인

### 4. MailContents Service (2개 API)

**메일 생성 API**
- `POST /api/mail/generate` - 메일 내용 자동 생성 및 발송
  - **요청**: `{"category_id": "CAT001"}`
  - **기능**: 
    1. Admin API에서 카테고리명 + 담당자 정보 조회
    2. Classification API에서 VOC 분석 결과 조회 (최대 3개)
    3. 정적 템플릿 기반 메일 내용 생성 (LLM 없음)
    4. Mail Send API로 비동기 자동 발송
  - **응답**: `{"subject": "[개선 리포트] 카테고리명 관련 개선 리포트 전달", "content": "템플릿 기반 메일 내용"}`

**관리 API**
- `GET /api/mail/health` - 서비스 상태 및 API 연동 확인

**서비스 특징**
- **Stateless**: 데이터베이스 없이 API 연동만으로 동작
- **템플릿 기반**: 사전 정의된 정적 템플릿으로 메일 생성
- **통합 연동**: Admin, Classification, Mail Send 3개 서비스 API 통합 호출
- **비동기 처리**: 메일 발송 요청을 별도 스레드에서 처리
- **에러 처리**: API 호출 실패 시 기본 메일 생성 후 발송

5. Mail Service (5개 API)

**메일 발송 API**
- `POST /api/send` - 직접 메일 발송
- `POST /api/mail/send` - 카테고리 기반 담당자 메일 발송
- `POST /api/sendmail` - MailContents 서비스 연동 발송

**관리 API**
- `GET /api/health` - 서비스 상태 확인
- `GET /api/` - 서비스 정보 및 엔드포인트 안내

6. Admin Service (6개 API)

**데이터베이스 관리 API** 
- `GET /api/admin/{tableName}` - 테이블 전체 데이터 조회
  - **예시**: `/api/admin/assignee`, `/api/admin/consulting_category`
  - **응답**: 테이블의 모든 데이터를 JSON 배열로 반환
- `GET /api/admin/tables` - 모든 테이블 목록 조회
- `GET /api/admin/{tableName}/columns` - 테이블 컬럼 정보 조회
- `GET /api/admin/{tableName}/count` - 테이블 레코드 수 조회

**담당자 조회 API** 
- `GET /api/admin/assignee?category_id={categoryId}` - 카테고리별 담당자 조회
  - **기능**: assignee와 consulting_category 테이블 JOIN 쿼리
  - **응답**: 담당자 정보 (name, email, team, phone)

**관리 API**
- `GET /api/admin/health` - 데이터베이스 연결 상태 확인

**서비스 특징**
- **실제 DB 연결**: MySQL admin_db 스키마에 직접 JDBC 연결
- **동적 테이블 조회**: 테이블명을 파라미터로 받아 동적 쿼리 실행
- **JOIN 쿼리**: assignee ↔ consulting_category 관계 조회
- **SQL 인젝션 방지**: 파라미터 바인딩 및 테이블명 검증
- **메타데이터 제공**: 테이블 컬럼, 레코드 수 등 메타정보 API

---

## 배포 및 운영 가이드

### Docker 컨테이너 배포

```bash
# 전체 서비스 Docker Compose 실행
docker-compose up -d

# 개별 서비스 buildx 빌드 및 실행
docker buildx build --platform linux/amd64 -t insightops-dashboard-backend ./InsightOps-dashboard-backend-main-2 --load
docker buildx build --platform linux/amd64 -t insightops-classification ./InsightOps-classfication-main --load
docker buildx build --platform linux/amd64 -t insightops-voicebot ./InsightOps-realtime-voicebot-main --load
docker buildx build --platform linux/amd64 -t insightops-mail ./InsightOps-mail-send-main --load
docker buildx build --platform linux/amd64 -t insightops-frontend ./InsightOps-dashboard-frontend-main-2 --load
```

### 데이터베이스 초기화

```sql
-- 1. 데이터베이스 생성
CREATE DATABASE insightops_dashboard DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE normalization_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE voicebot_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE admin_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 2. 사용자 생성 및 권한 부여
CREATE USER 'insightops_user'@'%' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON insightops_dashboard.* TO 'insightops_user'@'%';
GRANT ALL PRIVILEGES ON normalization_db.* TO 'insightops_user'@'%';
GRANT ALL PRIVILEGES ON voicebot_db.* TO 'insightops_user'@'%';
GRANT ALL PRIVILEGES ON admin_db.* TO 'insightops_user'@'%';
FLUSH PRIVILEGES;

-- 3. Prisma 마이그레이션 실행 (Voicebot Service)
npx prisma migrate deploy
npx prisma generate

```

### 운영 체크리스트

### 서비스 시작 전 확인사항

- [ ]  MySQL 데이터베이스 연결 확인
- [ ]  OpenAI API 키 설정 및 할당량 확인
- [ ]  Azure AD 앱 등록 및 권한 설정 (Mail Service)
- [ ]  환경변수 설정 완료
- [ ]  서비스 간 네트워크 연결 확인

### 정기 모니터링 항목

- [ ]  일일 VoC 수집량 확인
- [ ]  AI 분류 성공률 모니터링 (95% 이상 유지)
- [ ]  OpenAI API 사용량 추적
- [ ]  데이터베이스 성능 모니터링
- [ ]  메일 발송 성공률 확인
- [ ]  서비스별 헬스체크 상태 확인

### 장애 대응 절차

1. **서비스 다운 감지**: 헬스체크 API 실패
2. **로그 분석**: 각 서비스별 에러 로그 확인
3. **의존성 확인**: 데이터베이스, 외부 API 연결 상태
4. **서비스 재시작**: Docker 컨테이너 재시작
5. **데이터 정합성 확인**: 분류 누락 데이터 배치 처리
