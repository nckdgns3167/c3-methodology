# AGENTS.md — C3 (Create your Career on Canvas)

> 이 문서는 C3 프로젝트의 헌법이다. AI 에이전트와 사람 모두 작업 시작 전에 이 문서를 읽는다.
> 변경은 [git log](.) 커밋 단위로만 이루어진다.

---

## 0. 문서 구조 (SSOT)

C3 하네스는 역할별로 SSOT(Single Source of Truth)를 분리한다. 한 정보의 진실은 _단 하나의_ 위치에서만 정의되며, 같은 정보를 두 군데에 두지 않는다.

| SSOT                    | 역할                                                                                                     |
| ----------------------- | -------------------------------------------------------------------------------------------------------- |
| **AGENTS.md** (이 파일) | 운영 규칙·17개 비타협 제약·작업 절차                                                                     |
| **`feature_list.json`** | 백로그·작업 상태·WIP 정책                                                                                |
| **`DESIGN.md`**         | 시각/UX 의도 — colors, typography, spacing, components, do's & don'ts (Google Labs 사양, Apache 2.0)     |
| **`docs/adr/`**         | 비타협 _결정 근거_ (Context / Decision / Consequences / Alternatives Considered)                         |
| **`docs/plans/`**       | _실행 계획_ 영구화 (Plan-as-Code, ADR-0010). frontmatter + Pre-mortem 강제, supersede 체인으로 진화 추적 |
| **`docs/glossary.md`**  | 용어 사전 (사용자 학습 누적)                                                                             |
| **`git log`**           | 작업 완료(`done`) 이벤트의 진실                                                                          |

`CLAUDE.md`, `.cursorrules`, `.github/copilot-instructions.md` 등 *에이전트별 메모리 파일*은 모두 위 SSOT를 가리키는 stub이며, 규칙을 중복 정의하지 않는다.

**GitHub Milestones**는 SSOT가 _아니라_ Phase 게이팅의 시각적 그룹핑이다 — 백로그는 항상 `feature_list.json`이며, 마일스톤은 PR/커밋의 데드라인 메타데이터 용도로만 사용 (`Phase 1 — Beta` due 2026-11-30 / `Phase 2`). **GitHub Issues 백로그·Projects 칸반보드·Wiki는 의도적으로 사용하지 않는다** — DRY 위반과 SSOT 분산을 방지. Phase 2에서 외부 가시성이 필요해지면 `feature_list.json` → Projects 단방향 sync 스크립트만 도입 검토.

### 0.1 ADR 인덱스

비타협 결정은 `docs/adr/`에 ADR 형식으로 영구화된다. 17개 제약은 *결과*이고 ADR이 *사유*를 보존한다. 새 비타협 결정을 도입하는 PR은 ADR 추가 필수.

- ADR-0001 — Stack: Supabase + Drizzle + Next.js
- ADR-0002 — JSON Resume as Data Contract
- ADR-0003 — No Canvas Rendering (HTML/CSS only)
- ADR-0004 — Section-based Layout, Not Free-form
- ADR-0005 — Mobile Edit Deferred to Phase 2
- ADR-0006 — AGENTS.md as SSOT (CLAUDE.md is stub)
- ADR-0007 — WIP = 1
- ADR-0008 — feature_list.json as Backlog SSOT
- ADR-0009 — DESIGN.md as Visual/UX SSOT
- ADR-0010 — Plan-as-Code (docs/plans/) for Plan Versioning

각 ADR이 적용되는 제약은 Section 4의 각 제약 헤딩 옆에 `[ADR-XXXX]` 태그로 표시.

---

## 1. 프로젝트 정의

**C3 (Create your Career on Canvas)** — 리크루팅 특화 노코드 캔버스 플랫폼.
사용자가 자유롭게 UI 컴포넌트를 배치해 포트폴리오/채용 페이지를 만들고 자기 도메인에 즉시 배포하는 서비스.

- **Phase 1 (Seed) 마감**: 2026-11-30 베타 출시
- **Phase 1 KPI**: 개인 사용자 1,000명, 기업 레퍼런스 10곳
- **운영 형태**: 1인 개발자 + AI 에이전트 위임 체제

## 2. 용어 정의

이 문서와 코드베이스 전반에서 일관되게 사용한다.

- **사용자 페이지 (User Page)**: 사용자가 C3로 만들어 발행한 결과물 페이지. 17개 제약의 주된 적용 대상. `src/app/p/*` 또는 `[username].c3.app`에서 렌더링.
- **에디터 UI (Editor UI)**: C3 자체의 빌더 인터페이스. `/editor/[pageId]` 라우트. 데스크톱 전용.
- **공개 페이지 (Public Page)**: User Page 중 `status='published'`로 발행되어 anon read 가능한 상태.
- **섹션 컴포넌트 (Section Component)**: 사용자 페이지를 구성하는 단위. Hero, About, Skills, Projects, Contact 등. `mobileTransform` 메타 필수.

## 3. 운영 원칙

### 3.1 WIP = 1

`feature_list.json`의 `wip_policy.max_in_progress: 1`은 절대값이다.

- 새 작업 시작 시 `status: "in_progress"`인 feature가 없어야 함
- 차단(blocker) 발생 시 `paused`로 전환하되 **반드시 `reason` 기록**
- 다른 작업으로 옮긴 뒤 영원히 안 돌아오는 패턴을 하네스가 막는다

`wip_lock: false`인 feature(의존성 업데이트, 보안 패치 등 메타 작업)는 WIP 카운트에서 제외된다.

### 3.2 init.sh 게이팅

`./init.sh` 통과 없이는 코드 수정 자체를 시작하지 않는다.

- 의존성 설치 → 타입체크 → 린트 → 단위 테스트 → 마이그레이션 dry-run → `feature_list.json` 스키마 검증
- 실패 시 비영(non-zero) 종료. CI도 동일 스크립트 호출.

### 3.3 ACID 커밋 + Pass-state Gating

1 논리 작업 = 1 원자적 커밋. 모든 게이트 통과 후에만 커밋 허용.

- **1차 게이트**: husky + lint-staged (커밋 자체를 차단)
- **2차 게이트**: GitHub Actions
- **커밋 컨벤션**: `feat: F-002 lib/env.ts에 zod 검증 추가` 형태로 feature ID 명시 → `active_history`의 `done` 이벤트 대용

### 3.4 작업 시작 전 체크리스트

1. `./init.sh` 통과 확인
2. `feature_list.json`에서 다음 작업 후보 선정 (`status: "planned"` + `depends_on` 모두 `done`)
3. 해당 feature의 `applies_constraints` 항목을 이 문서에서 다시 정독
4. 해당 feature `status`를 `in_progress`로 갱신 + `active_history`에 `started` 이벤트 append
5. `acceptance.verifiable.tests`에 명시된 테스트 파일 **먼저 작성** (TDD 강제)
6. 구현
7. `init.sh` 통과 확인 → `git commit` (메시지에 feature ID 포함) → `feature_list.json`의 status는 사람/CI가 별도 단계에서 `done` 처리

---

## 4. Constraints

> 이 섹션의 ID는 `feature_list.json`의 `applies_constraints`가 참조한다.
> 모든 feature가 17개 제약 전부를 따라야 하지만, `applies_constraints`는 _특히 위반 위험이 큰_ 항목을 가이드로 명시한다.

### A. 결과물 형식 & 테마

**A1. Canvas 렌더링 엔진 사용 금지.** [ADR-0003]
사용자 페이지 출력은 오직 HTML/CSS. `konva`, `fabric`, `paper.js`, `react-konva` 등 import 금지. ESLint `no-restricted-imports`로 강제.
_Why: 접근성·SEO·ATS 통과율을 시각 레이어에서 동시에 확보하기 위함. 그래픽 합성이 필요한 경우는 SVG로 해결._

**A2. 사용자 페이지의 모든 텍스트는 시맨틱 HTML 태그로 출력.** [ADR-0003]
`h1~h6`, `p`, `ul/ol`, `blockquote`, `time`, `address` 등을 의미에 맞게 사용. `<div>` 안 raw 텍스트 직접 삽입 금지. ATS·SEO·a11y 통과율 100%의 토대.
_검증: 커스텀 ESLint 룰 + 컴포넌트 단위 테스트._

**A3. 사용자 페이지의 모든 색상·간격·타이포는 CSS 변수로 정의.** [ADR-0009]
`--color-primary`, `--space-md`, `--font-heading` 등. 컴포넌트 내부에 `#3B82F6`, `16px`, `'Inter'` 같은 하드코딩 값 금지.
_검증: Stylelint `custom-property-pattern`. 원클릭 테마 교체의 전제._

### B. 레이아웃 & 반응형

**B4. 사용자 페이지의 길이 단위는 viewport-relative 또는 `clamp()` 우선.** [ADR-0003, ADR-0004]
`%`, `vw`, `vh`, `fr`, `clamp()`, `min()`, `max()` 사용. 픽셀 고정값은 `border`, `box-shadow`, `outline`, `1px hairline` 외에는 금지.
_Why: 자동 반응형의 기반. 픽셀 절대좌표 도입 시 모바일 별도 레이아웃 비용이 폭발._

**B5. 모든 섹션 컴포넌트는 `mobileTransform` 메타데이터 선언 필수.** [ADR-0004, ADR-0005]
값: `'stack' | 'scroll' | 'hide' | 'reorder'` 중 하나. 메타 누락 시 빌드 실패.
_검증: TypeScript discriminated union으로 타입 단계에서 강제._

**B6. 사용자 페이지의 모든 인터랙티브 요소는 최소 44×44px 터치 타겟 충족.**
시각 크기는 작아도 되지만 hit area는 44px 이상. Apple HIG 기준.
_검증: 컴포넌트 라이브러리 디자인 토큰에 박아두면 자동 충족. PR 리뷰 체크포인트._

**B7. 모바일 편집 UI 작성 금지 (Phase 1 스코프).** [ADR-0005]
모바일 뷰포트에서 사용자에게 허용되는 행위는 다음 4가지로 제한:
(a) 자기 페이지 미리보기 (실제 배포 화면)
(b) 인라인 텍스트 수정 (오타 정정 수준 — 단일 텍스트 노드 in-place 편집만, 길이 제한 권장)
(c) 공유 링크 복사
(d) 대시보드 조회 (방문자 수, 조회 섹션 등)
컴포넌트 추가/삭제, 드래그, 레이아웃 변경, 스타일·테마 편집은 모두 차단.
_Why: 6개월 베타 일정 보호. 위반은 곧바로 6개월→12개월 리스크._

### C. 데이터 계층 & 보안

**C8. DB 접근은 Drizzle 쿼리 빌더 통해서만.** [ADR-0001]
raw SQL, `postgres-js` 클라이언트 직접 사용, Supabase JS 클라이언트의 `.from().select()` 사용 모두 금지. 단, **인증 흐름에 한해 Supabase Auth 클라이언트는 예외**.
_Why: 타입 안전성과 마이그레이션 추적성 보존. 검증: ESLint `no-restricted-imports`._

**C9. 모든 사용자 데이터 테이블은 Supabase RLS 정책 필수.** [ADR-0001]
RLS 비활성화 또는 정책 없는 테이블 생성 금지. 마이그레이션에 RLS enable + 정책 SQL이 함께 들어가야 통과.
_Why: 1인 개발자가 권한 버그를 만들 가능성을 DB 레벨에서 구조적으로 차단. 검증: 마이그레이션 린터._

**C10. server-only 모듈은 `'use client'` 컴포넌트에 import 금지.** [ADR-0001]
DB 클라이언트, 시크릿 접근, 외부 API 호출 모듈은 모두 파일 상단에 `import 'server-only'` 명시.
_Why: 시크릿 노출 사고를 빌드 타임에 차단. Next.js 런타임이 자동 강제._

**C11. 사용자 업로드는 Supabase Storage 통해서만 + 클라이언트·서버 양쪽 검증.**
클라이언트는 UX를 위한 1차 검증, 서버는 신뢰 가능한 2차 검증(MIME, 크기, 차원). 클라이언트 검증만 의존 금지.
_Why: 악성 업로드 방어의 기본._

### D. 무결성 & 운영

**D12. DB 마이그레이션은 append-only가 기본.**
`DROP`, `RENAME`, 컬럼 타입 변경은 (a) 데이터 백업 마이그레이션 선행 + (b) 파일명에 `MIGRATION_DESTRUCTIVE` 플래그 명시 시에만 허용.
_Why: 에이전트가 마이그레이션을 만들 때 가장 자주 일으키는 사고가 무신경한 데이터 파괴._

**D13. 데이터 mutation은 zod 스키마 검증 통과 후에만 실행.** [ADR-0001]
API route, server action, form submit 모두 zod로 입력 검증. 검증 없이 DB에 도달하는 mutation 경로 금지. zod 스키마는 Drizzle 스키마와 자동 동기화.
_Why: 런타임 신뢰 경계._

**D14. JSON Resume 호환 export 엔드포인트는 항상 200 + 유효 스키마.** [ADR-0002]
`GET /api/resume/[userId].json`. 표준 필드(`basics`, `work`, `education`, `skills`, `projects`, `awards`, `certificates`, `languages`, `interests`, `references`, `volunteer`, `publications`)에 매핑 가능한 데이터는 표준 필드로 출력. 사용자 정의 필드(자체 인증, 비표준 카테고리)는 `meta.custom` 영역에 보존하되 표준 필드를 임의 변형/누락 금지.
_Why: 데이터 주권 + 사용자 자유. 검증: CI contract test._

**D15. 시크릿은 환경변수로만 접근.**
`.env*` 파일은 git 추적 제외. 코드 안에 키·토큰·비밀번호 하드코딩 금지. 환경변수 키 이름은 `lib/env.ts`에서 zod 스키마로 검증 후 export.
_Why: 시크릿 노출은 솔로 운영자에게 가장 비싼 사고. 검증: gitleaks 또는 secretlint를 pre-commit 훅으로._

### E. 발견성 & 다국어

**E16. 사용자 페이지의 텍스트 콘텐츠 필드는 다국어 확장 가능한 구조로 저장.**
단일 문자열이 아닌 `{ default: string, translations?: Record<LocaleCode, string> }` 형태. Phase 1은 `default`만 사용·표시하지만 스키마는 다국어 수용.
_Why: Phase 2에서 사용자 페이지 다국어 UI 추가 시 마이그레이션 0. 검증: zod·Drizzle 스키마로 강제._

**E17. 사용자 페이지는 빌드 시 SEO 메타데이터 자동 생성·삽입.**
필수 항목: `<title>`, `<meta description>`, OpenGraph (`og:title`, `og:description`, `og:image`, `og:type=profile`), Twitter Card, JSON-LD Schema.org Person, canonical URL. 이미지는 `alt` 텍스트 필수(편집기에서 빈 alt 차단). 사용자가 `noindex`를 선택하면 robots meta로 제어 가능.
_Why: C3에서 SEO는 곧 마케팅. 사용자 입력으로부터 자동 도출. 빈 메타데이터로 빌드 시 빌드 실패. 검증: CI에서 Lighthouse SEO 점수 90 이상 게이트._

### Phase 2 별첨 (활성화 예정)

> 이 두 제약은 Phase 1 진행 중에는 *비활성*이지만, 결제 기능 PR이 처음 들어오는 순간 자동 활성화되도록 `feature_list.json` 안에서 의존성으로 묶여 있다.

**P2-1. 결제 정보 자체 저장 금지.**
카드번호·CVC·만료일 등 PCI 범위 데이터는 절대 C3 DB에 저장하지 않음. Stripe Customer ID, Subscription ID, Payment Method ID(토큰)만 저장. 결제 UI는 Stripe Checkout 또는 Stripe Elements 사용.
_Why: PCI DSS 회피의 단순한 길._

**P2-2. Stripe 웹훅 엔드포인트는 idempotent + 서명 검증 필수.**
모든 웹훅은 `stripe-signature` 검증, `event.id` 기반 중복 처리 차단(이미 처리한 이벤트 ID는 DB에 저장하고 200 응답). 처리 실패 시 5xx 반환하여 Stripe가 재시도.
_Why: 웹훅 사고는 결제 시스템에서 가장 자주 발생하는 사일런트 버그._

### 자동 검증 매트릭스

| 검증 방식                 | 제약 ID                                    |
| ------------------------- | ------------------------------------------ |
| 자동 (린터/타입체커/빌드) | A1, A2, A3, B4, B5, C8, C10, D13, D14, D15 |
| 코드 리뷰·테스트          | B6, B7, C9, C11, D12, E16, E17             |

10/17 자동 검증 가능. 자동 검증 항목은 에이전트가 어기는 즉시 피드백 루프가 작동한다.

---

## 5. 파일 시스템 컨벤션

```
c3/
├── AGENTS.md                       # 이 파일 — 헌법
├── feature_list.json               # 기계 판독 가능한 백로그
├── feature_list.schema.json        # 위 파일 검증용 JSON Schema
├── claude-progress.md              # 세션 인수인계 메모
├── init.sh                         # 환경 건전성 검사 스크립트
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
├── next.config.ts
├── drizzle.config.ts
├── supabase/
│   ├── config.toml
│   └── migrations/                 # append-only (D12)
├── src/
│   ├── app/                        # Next.js App Router
│   │   ├── (auth)/                 # 회원가입·로그인 그룹
│   │   ├── editor/                 # 데스크톱 전용 (B7)
│   │   ├── dashboard/
│   │   ├── p/                      # User Page 렌더 (HTML/CSS only, A1)
│   │   └── api/
│   ├── components/
│   │   ├── sections/               # 사용자 페이지 섹션 컴포넌트 (mobileTransform 필수, B5)
│   │   └── editor/                 # 에디터 UI 컴포넌트
│   ├── server/                     # server-only (C10) — DB, 시크릿, 외부 API
│   ├── lib/                        # 서버/클라 양쪽 가능한 유틸
│   │   ├── env.ts                  # zod 환경변수 검증 (D15)
│   │   └── tokens/                 # 디자인 토큰 (A3)
│   ├── db/                         # Drizzle 스키마 + 클라이언트 (server-only)
│   └── middleware.ts               # 인증·서브도메인 라우팅
├── e2e/                            # Playwright 회귀 테스트
└── scripts/
    └── validate-feature-list.ts    # init.sh가 호출하는 스키마 검증
```

### 아키텍처 경계 (ESLint `boundaries`로 강제)

- `src/server/*` → `src/components/*` import **금지**
- `src/components/*` → `src/server/*` import **금지** (대신 server actions 또는 props 주입)
- `'use client'` 컴포넌트 → `import 'server-only'` 모듈 import **금지** (C10)
- `src/components/sections/*` → 외부 API 호출 **금지** (server-only 모듈에서만, C10)

---

## 6. 컴포넌트 작업 패턴

**F-008 Hero가 이후 모든 섹션 컴포넌트의 패턴 레퍼런스다.** 새 섹션 컴포넌트 작업 시 다음을 모두 갖춘다:

```
src/components/sections/<Name>/
├── <Name>.tsx                # React 컴포넌트
├── manifest.ts               # defineComponent({ id, displayName, propsSchema, defaultProps, mobileTransform, ... })
└── __tests__/
    └── <Name>.test.tsx
```

- `defineComponent()` 헬퍼로 manifest + 컴포넌트를 함께 export (헬퍼는 `src/components/sections/define-component.ts`)
- `propsSchema`는 zod (D13)
- `mobileTransform` 메타 필수 (B5)
- 시맨틱 HTML, 토큰만 사용, mobile에서 깨지지 않음 (A2, A3, B4)
- 이미지 alt 강제 (E17)
- 단위 테스트 1개 이상

> **Storybook 미도입 (Phase 2 검토)**: Phase 1은 F-016 미리보기 드로어(편집기 내부)로 시각 검토를 대체한다. 컴포넌트 5개 + 솔로 환경에서 Storybook 셋업·유지 비용 대비 효과가 작고, F-016이 동일 역할을 더 저렴히 수행. Phase 2 합류자 발생 시 도입 재검토.

---

## 7. AI 에이전트에 대한 메모

이 프로젝트는 1인 개발자가 AI 에이전트에게 작업을 위임하는 방식으로 진행된다. 본 문서(AGENTS.md)는 *도구 중립*이며, 이를 따르는 모든 에이전트(Claude Code, Codex, Cursor, Windsurf 등)에 동등히 적용된다. 에이전트가 다음을 지키지 않으면 위임이 실패한다:

1. **새 세션 시작 시 `claude-progress.md` 먼저 읽기** — 직전 세션에서 어디까지 갔는지, 어떤 차단이 있는지. 파일명은 historical reasons로 `claude-progress.md`이지만 어느 에이전트가 읽고 써도 동일 형식.
2. **거짓말 금지** — `verifiable.tests`에 명시된 파일이 실제 존재하지 않거나 통과하지 않으면 done 처리하지 않는다. `init.sh`가 이걸 검증하지 못하면 PR 머지 차단.
3. **WIP=1 위반 금지** — 한 번에 하나만. 차단되면 명시적으로 `paused`로 옮기고 이유 기록.
4. **17개 제약 위반 금지** — `applies_constraints`에 명시된 항목은 작업 전 다시 정독. 사유 궁금하면 각 제약 헤딩 옆 `[ADR-XXXX]` 태그를 따라 `docs/adr/`에서 근거 확인.
5. **모르면 멈춰서 물어보기** — 추측해서 진행하지 말 것. 특히 데이터 모델·인증·결제 영역.
6. **결정은 결정해야 할 시점에** — 거시 결정(스택, 17개 제약, 페이즈 게이트)은 미리 박는다. 미시 결정(필드 길이, 모달 닫는 키, 마이크로 인터랙션 임계값)은 해당 feature 작업 시점에 컨텍스트 보고 정한다. 솔로+AI에서 컨텍스트 부족 상태로 박은 결정은 더 큰 재작업을 부른다. 헷갈릴 때 기준: "이 결정이 다른 결정의 *전제*가 되는가?" Yes면 미리, No면 미루기.
7. **비자명 용어 도입 시 `docs/glossary.md` 확인/추가** — 사용자는 실무 강함, 이론 용어 학습 중. 새 기술 용어(SSOT, ACID, RLS, Idempotent 등)를 코드/문서에 처음 도입할 때 사전에 있는지 확인하고, 없으면 인라인 짧은 설명 + 추가 제안. 형식: 도메인별 분류 + 각 항목 끝 "**이 프로젝트에서**:" 줄 강제, 영문 용어 한글 번역 alias 안 붙임.
8. **시각/UX 결정은 `DESIGN.md` 경유** — colors, typography, spacing, component 패턴, do's & don'ts는 `DESIGN.md`(Google Labs 사양)가 SSOT. `src/lib/tokens/`는 이를 derive한 결과물이지 SSOT 아님. 새 토큰·컴포넌트 패턴 도입 시 DESIGN.md 먼저 갱신, 그 다음 build-tokens 스크립트 경유.

---

## 8. 변경 이력

이 문서는 git log가 변경 이력의 진실의 원천이다. 직접 편집하지 말고 PR로만 수정한다.
