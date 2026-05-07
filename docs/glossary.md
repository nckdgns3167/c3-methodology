# 용어 & 개념 사전

jch가 누적 정리하는 개인 학습 리소스. 비자명한 기술 용어를 작업 중 등장 시점에 추가. **도메인별 분류** (카테고리 내부 알파벳 순), **영문 용어는 번역하지 않음** (한글 alias 추가 금지, 한국어 설명 prose는 유지). 각 항목 끝 "**이 프로젝트에서**:" 줄로 도메인 적용 적시.

---

## 아키텍처 & 데이터

### ACID

데이터베이스 트랜잭션의 4가지 속성 — Atomicity, Consistency, Isolation, Durability. 한 트랜잭션이 모두 성공하거나 모두 실패해야 한다는 약속.

**이 프로젝트에서**: git 커밋도 ACID처럼 다룸 = 한 커밋이 하나의 완결된 논리 변경. 부분 커밋 금지. AGENTS.md 운영 원칙 3.3.

### Append-only

한번 쓰인 레코드를 수정/삭제하지 않고, 변경은 새 레코드 추가로만 표현하는 방식. 이력 보존 + 동시성 충돌 회피.

**이 프로젝트에서**: `supabase/migrations/`는 append-only 기본값. DROP/RENAME은 `MIGRATION_DESTRUCTIVE` 플래그 필수 (제약 #12).

### JSON Resume

이력서 데이터를 표현하는 오픈 스펙 (https://jsonresume.org). basics/work/education/skills/projects/awards/certificates/languages/interests/references/volunteer/publications 등 표준 12 필드.

**이 프로젝트에서**: 사용자 데이터의 _데이터 계약_. 모든 페이지 트리는 JSON Resume과 호환되도록 export 가능해야 함 (제약 #14, F-022, ADR-0002).

### SSOT (Single Source of Truth)

어떤 정보의 진실을 정의하는 _단 하나의_ 위치. 같은 정보가 두 곳에 있으면 동기화 비용 + 불일치 위험.

**이 프로젝트에서**: AGENTS.md = 운영 규칙의 SSOT. feature_list.json = 백로그의 SSOT. DESIGN.md = 시각/UX의 SSOT. git 커밋 = done 이벤트의 SSOT. CLAUDE.md/.cursorrules는 stub. AGENTS.md Section 0.

---

## 보안

### RLS (Row Level Security)

PostgreSQL 기능. 같은 테이블이라도 행(row) 단위로 누가 읽고 쓸 수 있는지 SQL 정책으로 정의. 애플리케이션 코드 우회로 데이터 접근해도 정책이 차단.

**이 프로젝트에서**: 모든 사용자 데이터 테이블에 RLS 활성화 + `auth.uid() = owner_id` 정책 강제 (제약 #11). 마이그레이션 시 `_rls.sql` 별도 파일로 append.

---

## 소프트웨어 설계 원칙

### DRY (Don't Repeat Yourself)

같은 정보·로직을 두 곳 이상에 복제하지 않는다는 원칙. 중복은 동기화 부담 + 불일치 버그를 낳음.

**이 프로젝트에서**: AGENTS.md 규칙을 CLAUDE.md에 다시 적지 않는 결정의 근거 (ADR-0006). 디자인 토큰을 컴포넌트마다 하드코딩 안 하는 근거 (ADR-0009).

### Idempotent

같은 작업을 여러 번 실행해도 결과가 1회 실행과 동일한 성질. 네트워크 재시도·자동 복구의 전제.

**이 프로젝트에서**: `init.sh`는 멱등(idempotent)이어야 함 — 두 번 돌려도 부작용 없이 같은 상태 도달. Stripe 웹훅 처리도 idempotent (P2-2, Phase 2).

---

## 운영 방법론

### ADR (Architecture Decision Record)

아키텍처 결정과 사유를 기록한 짧은 문서. 표준 4섹션: Context / Decision / Consequences / Alternatives Considered.

**이 프로젝트에서**: `docs/adr/`에 누적 (현재 9개, F-001 산출물). 비타협 결정 발생 시 코드보다 ADR 먼저. 17개 제약은 *결과*이고 ADR이 _사유_. AGENTS.md Section 0.1.

### Harness Engineering

1인+AI 위임 환경에서 *코드 작성 자체*보다 *어떤 작업을 어떤 게이트로 통과시킬지*의 인프라가 더 비싼 보험이라는 접근. 4대 프리미티브: AGENTS.md / feature_list.json / init.sh / claude-progress.md.

**이 프로젝트에서**: 첫 git 커밋 전에 이 4개 + DESIGN.md + glossary 박는 결정의 근거. 모든 후속 작업이 이 하네스를 통과해서 들어옴.

### Pass-state Gating

어떤 통과 조건(테스트, lint, schema 검증 등)을 만족할 때만 다음 단계로 넘어가게 하는 게이트. ACID의 atomic처럼 통과 or 비통과만 존재, 부분 통과 없음.

**이 프로젝트에서**: husky pre-commit + GitHub Actions가 이 게이트를 구현. init.sh 한 단계라도 실패 시 비제로 종료. AGENTS.md 3.3.

### Plan-as-Code

외부 도구(Notion, Linear) 대신 plan을 *Markdown + git*으로 영구화하는 접근. SSOT 분산 회피 + 도구 락인 0 + git이 형상관리. ADR-0010이 우리 구현.

**이 프로젝트에서**: `docs/plans/NNNN-<topic>.md` 표준. frontmatter(id/status/supersedes 등) + Pre-mortem + Post-mortem 섹션 강제 (ADR-0013). Claude Code plan 모드의 `.claude/plans/<random>.md` draft → 사용자 승인 후 백포팅.

### Project Inception

프로젝트의 _시작 단계_. 비전·요구사항·아키텍처·팀 구성 등 기본 frame을 박는 phase. RUP/Lean 용어. 산업에선 보통 "Phase 0" 또는 "0단계"로 부르기도.

**이 프로젝트에서**: 첫 git 커밋 *전*에 박은 모든 작업(AGENTS.md / 17 제약 / ADR / feature_list.json / DESIGN.md / glossary)이 inception. *Harness Engineering*과 결합해 솔로+AI 위임 게이트까지 함께 박음.

### RFC (Request for Comments)

큰 변경·신규 기능 *제안*을 markdown 문서로 작성·토론·승인 추적하는 패턴. Rust, IETF, 대형 OSS 표준. plan과 비슷하지만 _공개 토론_ 색이 더 강함.

**이 프로젝트에서**: 솔로 환경이라 직접 RFC 운영 X. `docs/plans/`이 비슷한 역할 (plan = inner-RFC). Phase 2 협업 합류 시 RFC 패턴 도입 검토.

### Spike (Architecture Spike)

XP/Agile 용어. _미지의 기술·구현 가능성_ 탐색을 위한 짧은 작업. 결과는 코드 또는 아키텍처 결정. 일반 작업 일정과 분리.

**이 프로젝트에서**: F-013(드래그+@dnd-kit) 같은 위험 항목이 spike 성격 — 미경험 기술 검증. *작은 spike*는 plan 안 박고 바로 시도, *큰 spike*는 plan에 위험 명시 + 시간 박스로 제한.

### Sprint Zero (Day 0)

Agile/DevOps 용어. *첫 sprint*를 실제 작업이 아닌 셋업(인프라·CI·도구·골격)에 쓰는 것. 운영 시작 _전_ 작업이라 "Day 0".

**이 프로젝트에서**: 부트스트랩(F-001+F-005+F-006) 묶음이 정확히 Sprint Zero. 일반 셋업과 다른 점 — *AI 에이전트 위임 게이트*까지 박는 *Harness Engineering*을 함께.

### WIP=1 (Work In Progress = 1)

동시에 진행 중인 작업이 정확히 1개로 제한된 상태. 컨텍스트 스위칭 비용 0, 한 작업의 완결성 보장.

**이 프로젝트에서**: feature_list.json `max_in_progress: 1`로 강제. paused 상태는 허용 (이유 기록 강제). `wip_lock=false` 메타 작업은 카운트 제외. 솔로+AI에서 큰 효과 (ADR-0007).

---

## 회고 & 학습

### 5 Whys

문제의 표면 원인이 아닌 *시스템 원인*까지 5번 "왜?"를 반복해 내려가는 분석 기법. Toyota 발. Postmortem의 핵심 도구.

**이 프로젝트에서**: `docs/postmortems/_template.md`의 `## Why (5 Whys)` 섹션에서 강제. 5번째 답이 *시스템 변경*으로 나와야 함 (사람 비난 X, ADR-0013 Blameless).

### Action item

회고/postmortem에서 도출된 _실행 가능한_ 후속 작업. 단순 결심·다짐이 아닌 _시스템 변경_. 형식: `- [ ] <변경 내용>` 체크박스.

**이 프로젝트에서**: postmortem 또는 plan Post-mortem의 모든 `- [ ]` 항목은 `feature_list.json`에 새 F-XXX로 매핑 강제 (ADR-0013). "회고 문서만 남고 실행 안 됨"의 가장 흔한 실패 차단.

### Blameless

postmortem 작성 시 *사람*이 아닌 *시스템*을 분석하는 원칙. "X가 실수했다"가 아니라 "X 실수를 막을 가드레일이 없었다". 비난이 들어가면 다음 사건은 숨겨짐 — postmortem의 가장 큰 적.

**이 프로젝트에서**: 1인이라도 자기에게 적용 — "내가 멍청해서" X / "스키마 검증이 없어서" O. ADR-0013 + `docs/postmortems/README.md`에 명시.

### Devlog

매 작업 세션의 작업 일지 — 무엇을 시도했고, 무엇이 막혔고, 다음에 뭘 할지. 회고가 아닌 _메모리 외부화_.

**이 프로젝트에서**: `claude-progress.md`가 정확히 devlog. AGENTS.md Section 7 1번 규칙 (새 세션 시작 시 첫 번째 파일로 읽음). 사소한 디버깅도 여기 한두 줄.

### Feature Retro

feature 하나(F-XXX) 마무리 직후 5~10분 짧게 적는 회고. 의식이 아니라 *반사*에 가까움. PDF는 `docs/retros/feature/F-XXX.md` 별도 파일 권장이지만 우리는 plan 통합.

**이 프로젝트에서**: ADR-0013으로 plan 파일 안 `## Post-mortem` 섹션 강제 (별도 파일 X). Pre-mortem(plan 작성 시) ↔ Post-mortem(feature done 시) 페어링이 핵심. validate-feature-list.ts가 done 처리 시 비어있지 않음 검증.

### KPT (Keep / Problem / Try)

Sprint Retro의 가장 흔한 포맷. Keep(계속할 것) / Problem(문제였던 것) / Try(다음에 시도할 것).

**이 프로젝트에서**: Sprint Retro 자체를 안 함 (ADR-0013) — 1인+AI 환경엔 부담 vs 가치 비효율. Phase 2 합류 시 재검토. KPT 패턴은 알아두되 직접 사용 X.

### Postmortem

사건/장애 발생 _후_ 분석 문서. 무슨 일이 일어났고, 왜 일어났고, 어떻게 고쳤고, 어떻게 재발 방지할 건지. 트러블슈팅 회고가 거의 다 여기 속함. Blameless가 표준.

**이 프로젝트에서**: `docs/postmortems/YYYY-MM-DD-<title>.md` (ADR-0013). 1페이지 이내 lightweight template. 사소한 디버깅은 `claude-progress.md`에 메모 — 경계는 "다른 사람이 같은 문제 겪을 가능성 있는가?" yes면 정식 postmortem.

### Pre-mortem

작업 _시작 전_ "이 plan이 6개월 후 실패한다면 _왜_ 실패했을까?" 미리 적기. 보이지 않는 위험을 강제로 가시화하는 도구. Post-mortem(사후 회고)의 거울.

**이 프로젝트에서**: `docs/plans/` 모든 plan에 Pre-mortem 섹션 강제 (ADR-0010). 실행 후 Post-mortem에서 _예측 vs 실제_ 비교가 학습의 코어 (ADR-0013 Pre-mortem ↔ Post-mortem 페어링).

### Sprint Retro

일정 주기(1주/2주) 끝에 그 기간의 *프로세스*를 평가하는 회고. KPT 포맷이 흔함. 결과물 평가가 아니라 *일하는 방식*에 대한 평가.

**이 프로젝트에서**: 안 함 (ADR-0013). 1인+AI 환경엔 부담 vs 가치 비효율. Phase 2 합류 또는 외부 기여자 발생 시 재검토.

---

## 코드 품질

### ESLint

JavaScript/TypeScript 코드의 _논리_ 문제를 검사하는 정적 분석 도구. 미사용 변수, await 빠진 비동기, console.log 잔존 등 잡음.

**이 프로젝트에서**: `.eslintrc.cjs`에 `eslint-plugin-boundaries` 박아서 `src/server`↔`src/components` import 차단 + `no-restricted-imports`로 Canvas 라이브러리 차단(A1). ESLint 8 사용 (boundaries v5 + ESLint 9 호환 미검증으로 fallback).

### Prettier

코드의 _모양_ 통일 — 들여쓰기, 줄 바꿈, 따옴표 종류, 트레일링 콤마. 의견 분쟁이 없게 strict하게 정형화.

**이 프로젝트에서**: `.prettierrc`로 singleQuote / trailingComma=all / printWidth=100. lint-staged가 staged 파일에 자동 적용.

### Stylelint

CSS의 무효 속성, 비표준 사용, 패턴 위반 등을 잡는 린터.

**이 프로젝트에서**: `.stylelintrc.json`에 `custom-property-pattern: ^(color|space|font|radius|shadow|z|breakpoint)-` 강제 → A3(CSS 변수만 사용) 자동 검증.

### TypeScript

JavaScript의 정적 타입 시스템. 컴파일 시점에 타입 오류(숫자에 문자열 더하기, 없는 함수 호출 등) 차단.

**이 프로젝트에서**: strict 모드 + path aliases(`@/*` → `src/*`). `tsconfig.json`. `pnpm typecheck` (`tsc --noEmit`)가 init.sh 첫 단계.

---

## Git 게이트

### commitlint

커밋 메시지 형식을 강제하는 도구. conventional commits (`feat:`, `fix:` 등) + 우리 커스텀 룰.

**이 프로젝트에서**: `commitlint.config.cjs`에 feature ID(`F-\d{3}`) 포함 강제. 누락 시 커밋 거부. `feat: F-002 lib/env.ts에 zod 검증` 형태 표준. git log를 done 이벤트의 SSOT로 만드는 핵심.

### husky

git의 _훅_(pre-commit, commit-msg 등 특정 시점에 자동 실행되는 스크립트)을 관리하는 도구.

**이 프로젝트에서**: `.husky/pre-commit`이 lint-staged → init.sh 호출. `.husky/commit-msg`가 commitlint 호출. 1차 게이트 (AGENTS.md 3.3).

### lint-staged

git에 staged된 _변경된 파일만_ 골라서 검사 도구(prettier/ESLint 등)에 넘기는 도구. 매 커밋마다 전체 코드베이스 검사하면 느려서.

**이 프로젝트에서**: `.lintstagedrc.json` 또는 package.json에 정의. staged TS/CSS 파일에 prettier + ESLint/Stylelint 적용. husky pre-commit이 호출.

---

## CI

### GitHub Actions

GitHub의 CI 시스템. 저장소 이벤트(push, PR 등)에 반응해 가상 머신에서 워크플로 자동 실행.

**이 프로젝트에서**: `.github/workflows/ci.yml`이 PR/push 시 init.sh 호출 → 모든 게이트 재실행. 2차 게이트 — 로컬 우회한 경우도 못 빠져나감 (AGENTS.md 3.3).

### Lighthouse

Google 제공의 웹 페이지 품질 측정 도구. SEO/성능/접근성/PWA 점수를 0-100으로 산출.

**이 프로젝트에서**: F-021에서 `.github/workflows/lighthouse.yml`로 PR마다 자동 측정. SEO 점수 ≥ 90 게이트 (E17 자동 검증의 핵심).

---

## 디자인 & 타이포그래피

### AI Slop

AI가 생성한 결과물이 _평균적·매력 없는_ 상태. 누구나 코드를 뽑아낼 수 있는 시대에 차별점 부재로 모든 결과물이 비슷해지는 현상. 영상 *"AI로 만든 제품이 안 팔리는 이유"*의 핵심 키워드.

**이 프로젝트에서**: DESIGN.md(ADR-0009) + 한국어 타이포 강제(ADR-0011) + 외부 reference + 우리만의 색깔 결정으로 회피. AI 슬롭 회피의 진짜 도구는 _정교한 디자인 시스템_. AGENTS.md Section 6.1이 작업 시작 전 필수 단계로 강제.

### Coolicons

(https://coolicons.cool) 일관 규칙으로 제작된 무료 SVG 아이콘 라이브러리. 무료 아이콘 섞어 쓸 때 발생하는 스타일 불일치를 방지.

**이 프로젝트에서**: ADR-0012로 채택. F-008 작업 시 30개 큐레이션 → `src/lib/icons/` 또는 manifest로 export. 라이선스 정독 + F-026 법적 페이지에 명시.

### Letter-spacing (자간)

문자 사이 간격. CSS `letter-spacing` 속성. 한국어는 영어보다 자간이 _좁아야_ 자연스러움 — KR best practice는 `-0.02em`.

**이 프로젝트에서**: ADR-0011로 한국어 텍스트 기본값 `-0.02em` 강제. DESIGN.md typography에 박음 (`letter-spacing-default`). 토스/네이버/다음/카카오 KR 서비스 표준.

### Line-height (행간)

줄 사이 간격. CSS `line-height` 속성. 한국어 가독성에 _결정적_ — 너무 좁으면 답답, 너무 넓으면 산만.

**이 프로젝트에서**: ADR-0011로 한국어 body 기본값 `1.55` (range 1.5~1.6) 강제. heading은 `1.3~1.4`, caption은 `1.4`. KR 서비스 표준.

### Pretendard

한국어 디자인 표준 폰트 (https://github.com/orioncactus/pretendard). 토스/네이버/다음/카카오 등 주요 KR 서비스 사용. 영문 글자도 자연스럽게 처리. Variable 폰트 지원.

**이 프로젝트에서**: ADR-0011로 한국어 1차 폰트 강제. fallback chain: Pretendard Variable → Pretendard → -apple-system → system-ui → Apple SD Gothic Neo → Noto Sans KR → Malgun Gothic → sans-serif. CDN 또는 npm `pretendard` 자체 호스팅 (F-008 결정).

### Wanted Montage

(https://montage.wanted.kr) 원티드(Wanted)가 공개한 한국어 디자인 시스템. 한국어 폰트 행간·버튼 깨짐 등 KR 디자인 디테일 반영. 상업 이용 + 수정 자유.

**이 프로젝트에서**: ADR-0012의 외부 reference 후보. F-008 벤치마킹 단계에서 컴포넌트 _구조_ reference로 활용 가능. 라이선스 정독 필수 (작업 시점 재검증).

---

## 프레임워크 & 툴링

### ajv

JSON Schema 검증 엔진 (Another JSON Validator). draft-2020-12 등 최신 사양 지원.

**이 프로젝트에서**: `scripts/validate-feature-list.ts`가 ajv로 `feature_list.schema.json`을 사용해 `feature_list.json` 검증. F-005 핵심 라이브러리.

### Boundary plugin (eslint-plugin-boundaries)

ESLint 플러그인. 폴더/파일 그룹 간 import 방향을 정의해서 위반 시 빌드 실패 처리.

**이 프로젝트에서**: `src/server` → `src/components` import 차단, `src/components` → `src/server` 차단, 'use client' 파일 → 'server-only' 모듈 차단 (제약 #10).

### Docker Compose

여러 컨테이너를 정의·기동·관리하는 도구 (단일 머신용). `docker-compose.yml`로 서비스 정의.

**이 프로젝트에서**: 부트스트랩에선 stub만(`services: {}`). F-003에서 supabase CLI가 자체 docker-compose 관리 (`supabase start/stop`).

### Drizzle (drizzle-orm)

TypeScript ORM. raw SQL에 가까운 API + 타입 안전. drizzle-zod로 zod 스키마 자동 생성.

**이 프로젝트에서**: DB 접근의 유일 경로 (제약 #8). Prisma보다 가벼움 + 마이그레이션 SQL 명시적 (Prisma의 schema.prisma DSL과 다름). drizzle.config.ts 출력 경로 = `supabase/migrations/`.

### Next.js

React 기반 풀스택 프레임워크. App Router (Server Components + Server Actions), 파일 기반 라우팅, SSR/SSG/ISR 통합.

**이 프로젝트에서**: 15.x App Router 사용. React 19 동반. `src/app/p/*`(사용자 페이지)는 SSR + 가벼운 번들, `src/app/editor/*`(편집기)는 클라이언트 heavy.

### pnpm

빠르고 디스크 효율적인 Node.js 패키지 매니저. npm/yarn 대체. 의존성을 단일 글로벌 store에 보관 + 프로젝트는 symlink.

**이 프로젝트에서**: 10.x 사용. `package.json`의 `packageManager: pnpm@10.33.4`로 버전 잠금. lock 파일 `pnpm-lock.yaml`.

### Server-only

Next.js 패키지. 모듈 상단에 `import 'server-only'` 명시하면 클라이언트 번들에 포함 시 빌드 에러.

**이 프로젝트에서**: `src/db/`, `src/server/` 모듈 모두 server-only로 격리. DB 접근·시크릿 사용 코드가 클라이언트에 새는 것을 빌드 시점에 차단 (제약 #10).

### Supabase

Postgres + Auth + Storage + Realtime 통합 BaaS (Backend as a Service). 자체 호스팅 또는 클라우드.

**이 프로젝트에서**: 데이터·인증·파일 저장의 단일 백엔드. RLS 정책으로 권한 강제 (제약 #11). Phase 2에 Supabase Sync Engine으로 Stripe 통합 가능.

### Tailwind CSS

유틸리티-퍼스트 CSS 프레임워크. 클래스로 직접 스타일링.

**이 프로젝트에서**: 3.4.x 사용 (Phase 1, 4.x는 Phase 2 검토). 디자인 토큰은 `tailwind.config.ts`에 박지만 *F-007 build-tokens.ts*가 DESIGN.md에서 derive해서 채움. 컴포넌트 작업 시 `--space-md` 등 토큰 경유.

---

## 비즈니스 모델 & 수익화

### Design system preset

사용자가 _저장_ 한 디자인 토큰 묶음 (color/typography/spacing/radius 등). 페이지에 _적용_ 가능. 일반 _개별 페이지 theme override_ 와 다름 — preset은 _전역 저장_, override는 _페이지별 미세 조정_.

**이 프로젝트에서**: F-035 (Design System Presets) 영역. design*systems 테이블 + tokens_json 컬럼 + page.design_system_id 외래키로 적용. F-015 (인스펙터)의 page meta theme override와 *분리\_ (ADR-0015). 무료 1개 / 유료 N개 (ADR-0016).

### Feature flag

기능을 코드에 박은 채 _런타임_ 활성/비활성 toggle. **soft flag**: UI 표시 + 차단 X (사용자 가시성 + upgrade 권유). **hard flag**: 강제 차단. 보통 soft → hard 단계화로 사용자 학습 + 결제 트리거.

**이 프로젝트에서**: F-036 (Portfolio Version Limits) plan*limit이 soft flag (F-100 결제 활성 *전\_). F-100 활성 시 hard 차단으로 한 줄 변경 (단일 enforcement 함수 패턴, ADR-0016).

### Free / Pro / Team

SaaS 표준 3 tier 비즈니스 모델. **Free**: 진입 장벽 0, 핵심 기능 일부 + 표시 (badge 등). **Pro**: 개인 유료 — 양적 제한 해제 + 부가 기능 (custom domain 등). **Team**: 조직 유료 — multi-user + 권한 + 협업.

**이 프로젝트에서**: ADR-0016 명세. Free = 1 발행 포폴 + 1 design system + C3 subdomain + C3 badge. Pro = N 포폴 + N design systems + custom domain + JSON Resume 고급 + no badge. Team = multi-user + 기업 채용 페이지 + shared design system + 지원자 유입 분석. F-100 (Stripe) 활성 시 plan_limit hard.

### Page-level theme override

design system preset _위에_ _이 페이지만_ 미세 조정. 예: preset은 보라색이지만 _이 포트폴리오만_ 파란색. 전역 preset은 변하지 않음.

**이 프로젝트에서**: F-015 (인스펙터) 영역 — page.theme*override_json 컬럼에 저장. design system preset(F-035)과 분리 (ADR-0015 territory). preset 적용 후 *페이지별 미세 조정\_ 자유.

### Plan limit

플랜별 사용량 제한 (페이지 N개 / 디자인 시스템 M개 / 기능 활성 여부). DB 필드로 박힘 — 결제 활성 여부와 _독립_ 으로 enforcement 단일 경로.

**이 프로젝트에서**: F-036 — user.plan ('free' default) + user.plan_limit (jsonb {pages: 1, design_systems: 1}) 컬럼. F-035/F-037 작업 시 plan_limit 체크 통과해야 새 리소스 생성. F-100 활성 시 soft → hard 전환.

### Upgrade CTA

무료 사용자에게 _유료 전환 권유_ UI. _기능 차단 시점_ 보다 _자연 사용 패턴 도달 시점_ 에 띄우는 게 효과적 (예: 무료 1개 한도 도달 _직전_, 페이지 복제 시도 직후).

**이 프로젝트에서**: F-036 + F-037 영역. _기능 제한 시 거부 메시지_ 가 아닌 _다중 포폴이 가치 있는 자연 시점_ 에 띄움 (ADR-0016 _자연 사용 패턴 기반 과금_ 정합).
