# ADR-0009: DESIGN.md as Visual/UX SSOT

- Status: Accepted (2026-05-06)
- Constraints affected: A3
- Related features: F-007 (build-tokens), F-008 (Hero — 첫 시각 결정), F-030 (베타 직전 폴리시)

## Context

AGENTS.md(규칙) / ADR(결정 사유) / feature*list.json(백로그) / glossary(용어) 모두 박혔지만 *시각·UX 의도의 SSOT 빈 자리\_. 그대로 두면 컴포넌트가 feature별로 즉흥적으로 자라 베타 직전 일관성 없는 더미. 솔로+AI에선 디자인 의도 SSOT 부재가 직접 일관성 손실로 이어짐.

Google Labs가 2026-04 오픈한 DESIGN.md 사양(Apache 2.0)이 정확히 이 빈 자리를 메우는 표준.

## Decision

Google Labs DESIGN.md 사양을 C3 *에디터 제품 자체*의 시각/UX SSOT로 채택. `c:\Users\jch\c3\DESIGN.md` (프로젝트 루트, AGENTS.md와 동급).

- **YAML front matter**: machine-readable tokens (colors, typography, spacing, rounded)
- **Markdown body**: human-readable rationale (Overview / Colors / Typography / Layout / Elevation / Shapes / Components / Do's and Don'ts)
- **`src/lib/tokens/`는 derived artifact** — F-007 build-tokens.ts가 DESIGN.md → CSS 변수 + TS 객체 derive. .gitignore 명시. 매 빌드 재생성.
- **사용 범위**: C3 에디터 자체만. 사용자가 만드는 페이지(`src/app/p/*`)의 디자인은 사용자 결정 — DESIGN.md 적용 X.

## Consequences

- 크로스 AI 에이전트 동일 파일 (Cursor, Codex, Stitch, Windsurf 등 모두 DESIGN.md 인식) — ADR-0006(도구 중립) 정합
- 디자인 토큰 단일 출처 → DRY 보존
- Phase 1엔 _스캐폴드만_ (값 모두 "TBD") — F-008 작업 시 첫 결정, 점진 채움, F-030 마무리 (3단 적용)
- `src/lib/tokens/`에 직접 토큰 박지 _마_ — DESIGN.md 경유 강제 (F-007 + init.sh stale 검출)

## Alternatives Considered

- **Figma**: 외부 의존, AI 도구 통합 약함, 토큰 수동 옮김 비용
- **자체 schema**: 재발명, 표준 도구 호환 0
- **토큰만 코드로**: rationale(왜 이 색·이 타이포) 손실 — 6개월 후 회복 비용 큼
