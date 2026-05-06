---
name: C3 Editor (TBD until brand lock)
colors:
  primary: TBD
  secondary: TBD
  background: TBD
  surface: TBD
  text-primary: TBD
  text-secondary: TBD
  border: TBD
typography:
  # ADR-0011 한국어 타이포 강제 — Pretendard 1차 + KR best practice
  font-ko: "'Pretendard Variable', 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, Roboto, 'Helvetica Neue', 'Segoe UI', 'Apple SD Gothic Neo', 'Noto Sans KR', 'Malgun Gothic', sans-serif"
  font-en: 'Inter, -apple-system, BlinkMacSystemFont, system-ui, sans-serif'
  font-ja: "'Noto Sans JP', -apple-system, system-ui, sans-serif"
  letter-spacing-default: '-0.02em' # KR best practice (토스/네이버/다음 표준)
  letter-spacing-en: '0' # 영문 전용 텍스트
  line-height-heading: '1.35' # range 1.3~1.4
  line-height-body: '1.55' # range 1.5~1.6
  line-height-caption: '1.4'
  heading-xl:
    fontSize: TBD # F-008 작업 시 결정
    fontWeight: TBD
  heading-lg:
    fontSize: TBD
    fontWeight: TBD
  heading-md:
    fontSize: TBD
    fontWeight: TBD
  body-md:
    fontSize: TBD
    fontWeight: TBD
  body-sm:
    fontSize: TBD
    fontWeight: TBD
  caption:
    fontSize: TBD
    fontWeight: TBD
spacing:
  xs: TBD
  sm: TBD
  md: TBD
  lg: TBD
  xl: TBD
rounded:
  sm: TBD
  md: TBD
  lg: TBD
---

# C3 Editor — DESIGN.md

> **Phase 1 부트스트랩 단계 — placeholder만**. 실제 brand 값은 F-008 Hero 작업 시 첫 결정 박힘. 이후 컴포넌트 작업 따라 점진 채움. 베타 직전 F-030에서 Components/Do's-Don'ts 섹션 마무리 (3단 적용 단계 3 완료).
>
> **사용 범위**: C3 *에디터 제품 자체*의 시각/UX SSOT. 사용자가 만드는 페이지(`src/app/p/*`)의 디자인은 사용자가 결정 — DESIGN.md 적용 X.
>
> **변경 흐름**: DESIGN.md 갱신 → `pnpm scripts/build-tokens.ts` (F-007) → `src/lib/tokens/{index.css, index.ts}` derive → 컴포넌트가 토큰 경유. _DESIGN.md가 SSOT, tokens/는 derived_ (ADR-0009).

## Overview

(F-008 Hero 작업 시 brand 톤·관점 추가)

## Colors

(F-008 작업 시 hex 값 + 사용 맥락 추가 — primary/secondary/text-\* 등)

## Typography

(F-008 작업 시 heading scale + body 스케일 추가)

## Layout

(F-008 작업 시 spacing 스케일 + container 폭 + breakpoints 추가)

## Elevation & Depth

(F-015 인스펙터 작업 시 shadow 스케일 추가)

## Shapes

(F-008 작업 시 border-radius 스케일 추가)

## Icons (ADR-0012)

**라이브러리**: [Coolicons](https://coolicons.cool) — 일관 규칙 + 무료 + 상업 이용 OK.

**큐레이션 정책**:

- F-008 작업 시 자주 쓸 30개를 미리 골라 `src/lib/icons/` 또는 manifest로 export
- 컴포넌트 작업 중 큐레이션 외 아이콘 추가는 동일 라이브러리 안에서만 (다른 라이브러리 섞기 금지 — 일관성 깨짐, AI 슬롭 위험)
- 라이선스 파일/조건은 F-026(법적 페이지) 작업 시 `docs/legal/` 또는 `LICENSES.md`에 명시

(F-008 작업 시 30개 큐레이션 박힘)

## Components

(베타 직전 F-030 마무리 시점에 채움 — 5 컴포넌트 + 편집기 작업 누적 후. 추측 기반 가짜 컴포넌트 가이드 박지 마)

## Do's and Don'ts

(베타 직전 F-030 마무리 시점에 채움 — 실제 사용 경험 누적 후 박음)
