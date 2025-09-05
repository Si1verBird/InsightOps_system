# VoC Insight Hub

## 📌 소개
VoC Insight Hub는 고객의 목소리를 자동으로 수집·분석·배포하는 MSA 기반 서비스입니다.  
VoC 데이터를 인입/정규화하여 NLP로 분류하고, 정책 기반 인사이트를 산출해  
부서별 Action Plan과 알림 메시지로 전달합니다.

## 🔑 주요 기능
- Intake → Normalize → Analyze → Route → Message → Delivery → Dashboard
- Azure Teams/Outlook API 연동
- 불변식 기반 설계 (데이터 무결성, 신뢰도, 정책 버전 관리)

## 🛠️ 아키텍처
- Backend: Spring Boot, JPA, PostgreSQL, MongoDB
- Frontend: React/Next.js, Tailwind, Recharts
- Infra: Docker, Azure, GitHub Actions(CI/CD)

## 📂 레포지토리
- [집계 리포지토리](https://github.com/Si1verBird/InsightOps_system.git)
- [서비스별 리포지토리](#서비스별-리포지토리)

## 📊 MSA 보드
- [이벤트스토밍 보드](https://miro.com/app/board/uXjVJQ9PPwo=/?share_link_id=513714152746)
- [서비스 아키텍처 다이어그램](링크)

## 📑 참고 자료
- [Notion 문서](https://silverbirds.notion.site/25cf8f01056180c38be1f1c1cf16e14e?source=copy_link)
- 위의 노션 링크에 회의록도 나와있음
