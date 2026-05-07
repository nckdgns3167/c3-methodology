# ADR-0016: Free/Pro/Team 비즈니스 모델 + 다중 포폴

- Status: Accepted (2026-05-07)
- Constraints affected: P2-1 (PCI 범위 데이터 자체 저장 금지), P2-2 (Stripe webhook idempotent + 서명 검증)
- Related ADRs: ADR-0001 (Stack), ADR-0008 (feature_list.json as backlog), ADR-0014 (Cross-review), ADR-0015 (자유도 + 디자인 시스템 분리)
- Related features: F-019 (발행 토글 + slug), F-025 (대시보드), F-035 (Design System Presets, 신설), F-036 (Portfolio Version Limits, 신설), F-037 (Portfolio Variants UX, 신설), F-100 (Stripe 결제), F-101 (팀 협업), F-103 (템플릿 마켓플레이스)

## Context

Codex 두 번째 informal sanity check (제품 방향, 2026-05-07) 도출 결정 5번째 — _무료 1개 / 유료 다양한 버전 여러 개 관리_. 사용자 본질 니즈 = _포폴 하나 더 만들고 싶다_ (직무·회사·언어·맥락별 다른 버전).

ADR-0008 (feature*list as backlog)은 *무엇을 빌드할지\_ SSOT, 본 ADR은 _누구에게 무엇을 팔지_ SSOT. 비즈니스 모델 결정은 백로그(feature_list)·데이터 모델(F-019/F-035)·결제 활성화(F-100)에 직접 영향이라 ADR감.

## Decision

### 1. Free / Pro / Team 3 플랜

| 항목               | Free               | Pro                         | Team                            |
| ------------------ | ------------------ | --------------------------- | ------------------------------- |
| 발행 포폴 수       | 1                  | N (예: 5)                   | N (Team 공유)                   |
| 디자인 시스템 수   | 1 (기본)           | N (예: 5)                   | shared design system            |
| 도메인             | C3 subdomain       | custom domain               | custom domain                   |
| C3 badge           | 표시               | 제거                        | 제거                            |
| JSON Resume export | 기본               | 고급 export                 | 고급 export                     |
| SEO 설정           | 기본               | 고급 (custom OG, 키워드 등) | 고급                            |
| 분석               | 기본 (방문자 수만) | 고급 (referrer, 체류, 클릭) | 지원자 유입 분석                |
| 멀티 유저          | X                  | X                           | 허용 (owner/editor/viewer 권한) |
| 기업 채용 페이지   | X                  | X                           | 활성                            |
| AI 추천 (F-104)    | X                  | 일부 (이력서 개선 제안)     | 전체 (채용공고 매칭 추천 포함)  |

### 2. 다중 포폴 = 수익화 _핵심 축_

_기능 제한_ 이 아닌 _자연스러운 사용 패턴_ 활용:

```
사용자가 실제로 필요로 하는 다중 버전:
  - 프론트엔드 개발자 버전
  - 풀스택 개발자 버전
  - PM 지원용 버전
  - 스타트업 지원 버전
  - 대기업 지원 버전
  - 영어 버전 / 한국어 버전
  - 프로젝트 중심 / 경력기술서 중심
```

_무료 1개 → 유료 N개_ 는 _기능 제한이 인위적이지 않음_. 사용자가 _스스로_ 추가 버전 필요할 때 자연스럽게 유료 전환.

### 3. 기존 features에 박힘 (F-035/F-036/F-037 신설 + F-019 보강)

- **F-019 (발행 토글)**: published page count + plan_limit 검증 (생성 시점 가드). Free는 1번째 published만 허용, 2번째는 plan_limit error.
- **F-035 신설 (Design System Presets)**: design*systems 테이블 + tokens_json + page.design_system_id 외래키. Free는 *기본 1개\_, Pro는 N개 저장 가능. plan_limit 필드.
- **F-036 신설 (Portfolio Version Limits)**: pages count by user + plan*limit check + dashboard 생성 가드 + upgrade CTA. \*\*Stripe 연동 *전\_ 은 soft flag\*\* (DB에 plan="free" default, plan_limit 강제는 F-100 활성 시).
- **F-037 신설 (Portfolio Variants UX)**: page duplicate (dashboard "복제" 버튼) + rename version + version label + compare metadata + last published timestamp. _다중 포폴 UX_ 의 사용자 가시성.

### 4. F-100 (Stripe 결제) description 보강

F-100 description에 Free/Pro/Team 명세 박음. 결제 활성화 시 ADR-0017+ (결제 통합 전략) 박힘 — 그때 P2-1/P2-2 활성, plan_limit 강제 hard, soft flag 제거.

### 5. F-101 (팀 협업) description 보강

Team 플랜 짝 — multi-user + shared design system + role permission + 기업 채용 페이지 + 지원자 유입 분석. ADR-0017+ (CRDT vs OT) 박힘 짝.

### 6. Phase 매핑

```
Phase 1 (베타, 2026-11-30):
  - 모든 사용자 = Free 모델 (DB plan="free" default, soft flag만)
  - 1 발행 포폴 + 1 design system 강제 (F-019 + F-035 미존재 → page 1개만)
  - C3 subdomain (F-023)
  - C3 badge (F-020 footer)

Phase 1.5 (베타 후, 유료화 _전_, 표지만 description):
  - F-035 Design System Presets (1개 기본 디폴트, N개 저장은 plan_limit soft check)
  - F-036 Portfolio Version Limits (soft flag, 2개 페이지 시 upgrade CTA 표시 but 차단 X)
  - F-037 Portfolio Variants UX (페이지 duplicate + version 관리)
  - 가치 입증 후 Phase 2 진입

Phase 2 (결제 활성화):
  - F-100 Stripe + ADR-0017+
  - plan_limit 강제 hard (Free 1개 / Pro N개)
  - F-101 Team 협업 + shared design system
  - F-104 AI 추천 (Pro 일부 / Team 전체)
  - P2-1/P2-2 활성

Phase 3 (안정 후):
  - F-103 템플릿 마켓플레이스 + 유료 템플릿 (수익 분배)
  - custom CSS sandbox
  - design token import/export
```

### 7. 모두의창업 사업계획서 정합 (F-031)

본 모델은 정부지원사업 사업계획서에 _자연스러운 수익 모델_ 로 박음. 핵심 메시지:

> "초기 무료 사용자는 1개의 포트폴리오 페이지를 생성할 수 있고, 유료 사용자는 지원 직무/기업별로 여러 버전의 포트폴리오와 디자인 시스템을 관리할 수 있다."

F-031 (모두의창업 응모 패키지) 작업 시 _이 ADR 인용_ — 사업계획서 _수익 모델_ 섹션의 SSOT.

## Consequences

- _기능 제한_ 이 아닌 _자연 사용 패턴_ 기반 과금 — 사용자 _기능 못 쓴다_ 거부감 적음
- F-100 (결제) 활성 시 _즉시_ Free 사용자 _이미 1개 사용 중_ → upgrade trigger 자연
- 데이터 모델 (ADR-0015 design*systems + page.design_system_id) + 비즈니스 모델 (본 ADR plan_limit) *분리\_ — 기술 결정과 비즈니스 결정 독립
- Phase 1.5에 Free 모델 활성 + plan*limit \_soft* → 사용자 _Pro 가치_ 미리 학습 (F-036 upgrade CTA), Phase 2 결제 활성 시 전환 자연
- 정부지원사업 응모 (F-031)에 _진정성 있는 수익 모델_ 박힘 — 단순 무료 서비스 X
- Phase 2 결제 도입 _전_ Phase 1.5에 _구조_ 박혀 있어 결제 활성화 시 마찰 0 (F-100이 plan_limit soft → hard 만 전환)

## Alternatives Considered

- **무료 무한 + 광고 모델**: 채용·경력 페이지에 광고는 _신뢰성 손상_ — 거부
- **유료 단일 가격 (구독료만)**: _기능 제한_ 없으면 무료 → 유료 전환 트리거 없음 — 거부
- **포폴 무한 + 다른 기능 제한 (custom domain만 유료 등)**: 사용자 본질 니즈가 _다중 버전_ 인데 다른 축 제한은 부자연 — 거부
- **Pro 단일 플랜 (Team 미도입)**: _기업 채용 페이지_ 이 Phase 1 KPI (10곳)인데 multi-user/shared 없으면 기업 가치 0 — Team 필수
- **무료 1개도 _발행_ 불가 (체험만)**: 베타 KPI 1,000명 도달 어려움 — 거부 (무료도 _발행_ 가능, badge로 차별화)

## 의도적으로 안 할 것

- Phase 1에 plan*limit \_hard* 강제 (F-100 _전_ 강제하면 _결제 시스템 없는데 차단_ 모순) — soft flag만
- _기능 단위_ 가격 차별화 (basic/standard/premium 식 — 인위적 차별)
- 무료 사용자에게 _C3 badge 강제_ 외 _다른 시각적 제약_ (컴포넌트 잠금, 색상 잠금 등 — 사용자 거부감)
- AI 추천 (F-104)을 _무료에 완전 차단_ — 일부(이력서 개선 제안)는 무료 허용 (체험 가치)
- _구독료 + 트랜잭션 fee_ 둘 다 (F-103 마켓플레이스 수익 분배는 별도, 기본 모델은 구독)
- "커리어 브랜딩 OS" 같은 _마케팅 표현_ 본 ADR에 박기 — F-031 사업계획서 작업 시 _사용자 자기 표현_ 검증 후 별도 박음 (자의적 채택 위험 회피)

## 결제·인프라 미정 사항 (사용자 액션 필요, Phase 2 진입 시)

- Stripe 계정 + Customer/Subscription/Payment Method 동기화 인프라 (Supabase Sync Engine 검토)
- 가격 결정 (Pro 월/년 / Team 사용자당) — F-100 작업 시점에 시장 조사 + 모두의창업 결과 반영
- 한국 결제 게이트웨이 검토 (Stripe vs PortOne — 한국 사용자가 신용카드 입력에 익숙한 정도, KRW 결제, 세금계산서)
- ADR-0017+ (결제 통합 전략 + 가격 + 환불 정책 + 한국 결제 게이트웨이)는 F-100 plan 작업 시점 박음
