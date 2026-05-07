# ADR-0015: 제품 자유도 단계화 + 디자인 시스템 구조 분리

- Status: Accepted (2026-05-07)
- Constraints affected: A1 (No Canvas), A2 (시맨틱 HTML), A3 (CSS 변수), B4 (viewport-relative), B5 (mobileTransform)
- Related ADRs: ADR-0003 (No Canvas), ADR-0004 (Section-based, Not Free-form), ADR-0009 (DESIGN.md), ADR-0014 (Cross-review)
- Related features: F-013 (DOM 편집 보드), F-015 (인스펙터), F-035 (Design System Presets, 신설), F-100 (결제), F-102 (모바일 편집 부분 활성)

## Context

Codex 두 번째 informal sanity check (제품 방향 점검, 2026-05-07)에서 사용자 본질 니즈 5개 도출:

1. 자유로운 배치
2. 컴포넌트 자율 커스터마이징 (외관 스타일링 _완전통제_)
3. 자기만의 디자인 시스템 _여러 개_ 저장/불러오기
4. 사용자별 포폴 그리는 화면 + 별도 배포 사이트
5. 무료 1개 / 유료 여러 버전 관리 (ADR-0016 짝)

(2)(3)이 _Phase 1부터 풀리면_ Framer/Webflow 수준 일정 폭발 + 모바일 반응형/ATS/SEO/접근성 손상. ADR-0004(섹션 기반)는 _레이아웃 단위_ 결정만 박았지 _자유도 단계화_ 와 _컴포넌트 스타일 vs 디자인 시스템 분리_ 는 미박음. 본 ADR이 그 빈자리.

ADR-0009 (DESIGN.md) 짝 — DESIGN.md는 _C3 에디터 자체_ 시각 SSOT. 사용자 _페이지_ 디자인 시스템은 _다른 territory_. 둘 분리 명시.

## Decision

### 1. 자유도 Phase 1/2/3 단계화

| Phase   | 사용자 페이지 자유도                                                                               |
| ------- | -------------------------------------------------------------------------------------------------- |
| Phase 1 | 섹션 순서 자유 + 섹션 추가/삭제 + 섹션 _내부_ 제한적 자유 배치 (CSS Grid + viewport-relative 좌표) |
| Phase 2 | 섹션 내부 _고급_ grid/layout control (column span, row span, advanced gap 등)                      |
| Phase 3 | advanced custom layout / custom CSS sandbox / custom component builder                             |

**Phase 1 허용**:

| 항목                       | 허용 여부            |
| -------------------------- | -------------------- |
| 페이지 전체 자유 배치      | _제한_               |
| 섹션 순서 변경             | 허용                 |
| 섹션 추가/삭제             | 허용                 |
| 섹션 내부 요소 배치        | _제한적_ 자유 허용   |
| 텍스트/이미지/CTA 수정     | 허용                 |
| 섹션별 _외관_ 커스터마이징 | 허용 (제한적, 토큰)  |
| 완전 custom component 제작 | Phase 2/3 이후       |
| 모바일 별도 편집           | Phase 2 이후 (F-102) |

이유: 모바일 반응형, ATS 통과, SEO, 접근성 — _완전 자유_ 가 이 4 축을 동시에 손상 (ADR-0004 일관). _통제된 자유_ 는 사용자 만족 + 4 축 동시 보호.

### 2. 컴포넌트 스타일 ↔ 디자인 시스템 _분리_

두 _다른 layer_ 로 명시 분리:

```
Component-level style (페이지 안 _개별_ 컴포넌트 override):
  - 이 Hero만 배경색 다르게
  - 이 Project card만 radius 다르게
  → page.theme_override_json (page meta)
  → F-015 (인스펙터 + 테마 패널)이 다룸

Design-system-level (전역 _저장 가능_):
  - 색상 scale / 타이포 scale / spacing scale / radius / shadow
  - button style / card style / section background
  → design_systems table (tokens_json + component_variants_json + is_default)
  → F-035 (Design System Presets, 신설)이 다룸
```

**왜 분리하나**:

- 사용자가 _개별 컴포넌트_ 변경한 것을 _디자인 시스템 전체_ 로 격상 가능 (F-035 _"이 변경을 디자인 시스템에 저장"_ UX)
- 디자인 시스템 _저장/불러오기_ (F-035) 가 _개별 페이지 override_ 와 충돌 없이 공존 — page.design_system_id (preset 적용) + page.theme_override_json (페이지별 override)
- _과금이 자연스러움_ — 무료는 1 design system + 1 페이지, 유료는 N (ADR-0016)

### 3. 데이터 모델 (F-035 작업 시점 박힘, _개념_ 만 명시)

```
user (F-004)
  → pages[]                                F-019: page schema (status, slug, tree_json, theme_override_json)
                                           F-035: page.design_system_id 외래키 추가
  → design_systems[]                       F-035 신설: tokens_json, component_variants_json, is_default, plan_limit
```

마이그레이션은 D12 (append-only) 정합 — F-019 page 테이블은 design*system_id 컬럼 *빈 채로\_ 추가, F-035가 backfill + 외래키.

### 4. ADR-0009 (DESIGN.md) 짝 — Territory 분리

| Territory                      | SSOT                                | 적용 대상                                                  |
| ------------------------------ | ----------------------------------- | ---------------------------------------------------------- |
| C3 에디터 자체 (관리자/편집기) | `DESIGN.md` (ADR-0009)              | `src/app/(auth)/`, `src/app/editor/`, `src/app/dashboard/` |
| 사용자 페이지                  | `design_systems` 테이블 + page meta | `src/app/p/[username]/[slug]`                              |

_DESIGN.md에 사용자 페이지 디자인 박지 마_ — 사용자 페이지는 사용자 결정. ADR-0009 territory 침범 X. F-008 (Hero 앵커) 패턴 참고하되 _복제 X_ (사용자 페이지 디자인은 _런타임 derive_, DESIGN.md는 빌드 타임 derive).

### 5. F-015 진화 — page-level theme override _만_

F-015 (인스펙터 + 테마 편집 패널) 현재 description: _"테마 변경이 page meta에만 저장 (전역 DESIGN.md 미변경)"_ — _맞음, 유지_. 본 ADR이 _design system preset 저장/불러오기는 별도 feature(F-035)_ 명시 박음.

F-015 _작업 시 안 할 것_ (notes 추가):

- design system 저장/불러오기 UI 만들지 마 (F-035 영역)
- design_systems 테이블 만들지 마 (F-035 영역)

### 6. F-013 (DOM 편집 보드) 정합

F-013 description은 이미 *"섹션 단위 *구조 조작* 중심"* + _"자유 배치 정밀 조정은 F-015 인스펙터로 흡수"_ 박힘 (plan 0007). 본 ADR이 _그 결정의 사유_ 영구화 — Phase 1 자유도가 _구조 조작 중심_ 이라 F-013이 _레이아웃 보드_ 가 아닌 _트리 조작_ 인 게 자연.

## Consequences

- _완전 자유_ 라는 사용자 기대 다스림 — 베타 출시 일정 보호 + 모바일/ATS/SEO/접근성 동시 보호
- 디자인 시스템 _저장/불러오기_ 가 _과금 자연스러움_ (ADR-0016 짝) — _무료 1개 / 유료 N개_ 가 인위적 제한 X, 데이터 모델 분리의 자연 결과
- _개별 페이지 override_ 와 _전역 디자인 시스템_ 동시 지원 — 사용자가 둘 중 하나 선택 X, 둘 다 사용 가능 (preset 적용 후 페이지별 미세 조정)
- ADR-0009 territory 명시 분리 — _C3 에디터_ 디자인 (DESIGN.md, 우리 제어) vs _사용자 페이지_ 디자인 (런타임, 사용자 제어) 두 머릿속 모델 분리
- F-013 (DOM 편집 보드) + F-015 (인스펙터) + F-035 (Design System Presets) 3 feature 명확한 boundary — 작업 시 _침범 안 함_ 가시성

## Alternatives Considered

- **Phase 1부터 완전 자유 배치 (Framer/Webflow 수준)**: 일정 폭발 + 모바일/ATS/SEO/접근성 손상 — 거부 (베타 마감 2026-11-30 위협)
- **컴포넌트 스타일과 디자인 시스템 _묶음_**: page meta에 모든 스타일 저장 — _저장/불러오기_ feature 만들 때 데이터 분리 비용 폭발 — 거부 (지금 분리)
- **DESIGN.md를 사용자 페이지에도 적용**: 사용자별 DESIGN.md? 파일 시스템 vs 데이터베이스 충돌 + 다중 사용자 + 다중 페이지 + 다중 디자인 시스템에 부적절 — 거부 (테이블 기반)
- **모든 자유도 Phase 1에 풀고 _제약 X_**: AGENTS.md 17 제약 (특히 A1/A2/B4) 대거 위반 가능 — 거부 (제약 _보호_ 가 ADR의 본질)
- **design*systems를 \_Phase 2* 부터 도입**: 사용자 _첫 인상_ 에 "내 디자인 시스템 저장" 부재 — 베타 후 _Phase 1.5_ 로 풀어 유료화 _전_ 가치 입증

## 의도적으로 안 할 것

- Phase 1에 _완전 자유 배치_ 풀기
- Phase 1에 _완전 통제 컴포넌트 커스터마이징_ (color/font/자간/행간/여백/radius/shadow/background/border/grid/image crop/hover/dark mode/animation/custom CSS/custom breakpoint/token set/component variant 17개 항목)
- DESIGN.md에 사용자 페이지 디자인 박기 (territory 위반)
- F-015 작업 시 design system 저장/불러오기 UI 만들기 (F-035 영역)
- design*systems 테이블에 *개별 페이지 override\_ 박기 (page.theme_override_json 영역)
- Phase 1.5 schema phase enum 추가 _지금_ — Phase 1.5 _활성화_ 시점(베타 도달 후) 결정. F-035/F-036/F-037은 phase: phase*1로 박되 description에 *"Phase 1.5 — 베타 후 폴리시"\_ 표지

## 짝 ADR

ADR-0016 (Free/Pro/Team 비즈니스 모델) — 본 ADR의 _데이터 모델 분리_ 가 _과금 모델 자연스러움_ 의 토대. 둘은 _제품 자유도/구조_ vs _비즈니스 모델_ 다른 카테고리지만 같은 sanity check 결과로 짝 도입.
