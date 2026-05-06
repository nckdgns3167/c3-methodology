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
  heading-xl:
    fontFamily: TBD
    fontSize: TBD
    fontWeight: TBD
  heading-lg:
    fontFamily: TBD
    fontSize: TBD
    fontWeight: TBD
  body-md:
    fontFamily: TBD
    fontSize: TBD
    fontWeight: TBD
  body-sm:
    fontFamily: TBD
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

## Components

(베타 직전 F-030 마무리 시점에 채움 — 5 컴포넌트 + 편집기 작업 누적 후. 추측 기반 가짜 컴포넌트 가이드 박지 마)

## Do's and Don'ts

(베타 직전 F-030 마무리 시점에 채움 — 실제 사용 경험 누적 후 박음)
