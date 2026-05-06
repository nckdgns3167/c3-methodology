# ADR-0001: Stack — Supabase + Drizzle + Next.js

- Status: Accepted (2026-05-06)
- Constraints affected: C8, C9, C10, D13

## Context

솔로 개발 + Phase 1 빠른 베타 출시(2026-11-30) + 데이터 주권 + Phase 2 결제 통합 가능성. 셋업·운영 비용을 솔로가 감당 가능한 수준으로.

## Decision

- **Frontend**: Next.js 15 App Router (React 19)
- **DB + Auth + Storage + Realtime**: Supabase
- **ORM**: Drizzle (raw SQL에 가깝지만 타입 안전, 마이그레이션 SQL 명시적)
- **Hosting**: Vercel (Next.js 정합)
- **Package manager**: pnpm

## Consequences

- 셋업 빠름 (auth/storage 기본 제공)
- 표준 stack (사용자 양도/외주 시 인수 비용 낮음)
- Supabase 락인 위험 — RLS·Storage·Realtime 의존 (Phase 2에서 마이그레이션 비용 큼)
- Drizzle은 Prisma보다 작아 솔로 환경에 적합, 다만 ecosystem(시각 도구 등)은 Prisma보다 약함

## Alternatives Considered

- **Neon + Drizzle**: auth 셋업 비용 +1~2주
- **Prisma**: 런타임 무거움, schema.prisma DSL이 마이그레이션 추적 약화
- **Supabase + Prisma**: 이중 관리
- **자체 Postgres + 자체 auth**: +4~6주, 솔로엔 과
