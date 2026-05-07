# ADR-0014: Multi-agent Plan Cross-Review (Claude × Codex)

- Status: Accepted (2026-05-07)
- Constraints affected: (해당 없음 — 메타 운영 결정)
- Related ADRs: ADR-0006 (도구 중립), ADR-0010 (Plan-as-Code), ADR-0013 (Pre-mortem ↔ Post-mortem)

## Context

ADR-0010 + 0013으로 plan에 Pre-mortem (예측) + Post-mortem (회고) 강제. 다만 *Pre-mortem은 작성자 자기 시점*이라 사각지대 잡기 한계. 솔로 + AI 위임 환경에서 *동료 리뷰 부재*가 본질적 갭.

산업 트렌드: _multi-agent verification_ — 한 에이전트가 작성, 다른 에이전트(다른 base model)가 검증. *진짜 ensemble*은 같은 모델 self-review와 다름. Codex (OpenAI GPT 기반)는 Claude (Anthropic)와 학습 데이터·관점이 달라 교차 검증 가치 큼. Codex의 GitHub PR `@codex review` 통합은 mature, 우리 AGENTS.md를 자동 인식해 17 제약·ADR 기준으로 review.

다만 매 plan에 cross-review 강제하면 비용·시간 부담. *근거 기반 분류*로 critical만 강제, easy는 skip.

## Decision

**1. Risk classification (plan frontmatter `risk_class` 필수)**:

| 분류       | 트리거                                                                                                                  | Cross-review       |
| ---------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------ |
| `critical` | 새 ADR / D 제약(DB·시크릿·결제·JSON Resume) / migration destructive / auth 변경 / dependency major bump / 5d+ 위험 추정 | **강제**           |
| `mid`      | T3 큰 feature / 외부 API 통합 / 새 server action / 복잡 알고리즘                                                        | 권장 (사용자 판단) |
| `easy`     | T2 단순 컴포넌트 / 메타 작업 (chore/docs) / 작은 fix                                                                    | skip               |

자동 분류 보조: Pre-mortem 위험 5+ 박혔으면 critical 자동 추천 (Claude self-flag).

**2. Cross-review 도구**:

- 1차 도구: **Codex via ChatGPT Plus** ($20/mo) + GitHub PR `@codex review` 트리거
- AGENTS.md 자동 인식 (Codex가 우리 17 제약·ADR을 자기 룰로 따름)
- P0/P1 priority 코멘트만 (노이즈 필터)
- 1개월 트라이얼 후 _영구 도입 vs 폐기_ 후속 ADR로 회고 (사용자 결정)

**3. 워크플로**:

```
1. Claude: plan 작성 + frontmatter risk_class 명시
2. critical/mid 시 — branch + plan commit + PR
3. PR 코멘트 @codex review 또는 Automatic reviews 토글
4. Codex P0/P1 코멘트 받음
5. plan 파일 안 ## Cross-review (Codex) 섹션에 결과 박음:
   - Convergence (수렴) / Divergence (발산) / Codex가 단독 catch한 것
   - Divergence는 trade-off 비교 + 사용자 결정 + 근거
6. 필요 시 plan 갱신 + 재 review
7. PR merge → 정상 implementation
8. Feature done 처리 시 Post-mortem (ADR-0013)에 *cross-review 효과* 회고
```

**4. Easy plan**: branch 없이 main 직접 push (현재 흐름 유지). 모든 plan을 PR화하면 솔로엔 과.

**5. validate.ts 보강 (cross-validation 7번째)**:

- feature done 시 매핑 plan에 `risk_class: critical` 이면 `## Cross-review` 섹션 비어있지 않음 검증
- ADR-0013 Post-mortem 검증과 동일 패턴
- mid는 검증 안 함 (권장이지 강제 X)
- easy는 무관

**6. 결과 보존 (plan 파일 안 통합, plan-as-code 정합)**:

```markdown
## Cross-review (Codex) — risk_class: critical

**Convergence (Claude + Codex 일치)**:

- ...

**Divergence (의견 차이)**:

- Claude: A
- Codex: B
- 결정: B (이유 ...)

**Codex 단독 catch**:

- (P0/P1 코멘트 핵심)

**Status**:

- review 트리거: 2026-MM-DD HH:MM (PR #N)
- review 통과: 2026-MM-DD HH:MM
```

**7. feature_list.json schema 보강**: `risk_class` optional enum 필드 추가 — feature 작성 시 권장 분류 명시 (critical/mid features만 박고 easy는 absent). plan 작성 시 일관 박힘.

## Consequences

- *진짜 ensemble*로 사각지대 catching (Claude + Codex 다른 base model)
- AGENTS.md 자동 인식 → Codex가 우리 17 제약·ADR 기준 review (자연 통합)
- ADR-0013 Pre-mortem ↔ Post-mortem 페어링과 결합 — _예측 + 교차검증 + 회고_ 3단 학습
- 비용 $20/mo (Plus) — 솔로 스타트업 합리, 한 critical 사고 회피로 회수
- 솔로 흐름에 PR 마찰 (현재 main 직접 push) → critical/mid만 PR화로 분기
- 도구 의존 약화 — AGENTS.md SSOT라 Cursor/Codex/Windsurf 어느 도구든 동일 review 가능
- 1개월 후 후속 ADR로 _영구 도입 vs 폐기_ 결정 — 가치 없으면 정직 회고

## Alternatives Considered

- **Pre-mortem만 의존** (ADR-0013): 작성자 사각지대 catching 한계 → 거부 (cross-review가 보완)
- **Self-review (Claude가 자기 plan 재검토)**: 같은 모델 same blind spot → 거부 (다른 base model 가치 X)
- **인간 동료 review**: 솔로 환경 부재 → cross-review가 보상
- **Pro $200/mo**: Phase 1 솔로엔 과 (Plus 충분)
- **수동 ChatGPT 무료**: 매번 plan 텍스트 복사·붙여넣기 부담 + GitHub 통합 X → 거부
- **GitHub Copilot Code Review**: Microsoft GPT 기반이라 Codex와 유사. 다만 통합 마찰 + 무료 플랜 제한적. 미래 검토.

## 의도적으로 안 할 것

- 모든 plan에 cross-review 강제 (비용·시간 폭발)
- Codex CLI 자동화 _지금_ 도입 (Phase 2 검토, 트라이얼 결과 후)
- 결과를 plan 외부 파일에 (SSOT 분산, plan-as-code 위반)
- Divergence를 무시하고 Claude 의견 기본 채택 (cross-review 의미 0)
- 트라이얼 1개월 후 회고 안 하고 영구 도입 (가치 검증 안 됨, AI 슬롭 회피 정신 위반)

## Bootstrapping (이 plan 자체)

이 ADR을 도입하는 plan 0005 자체는 _cross-review 정책 도입_ 메타 작업이라 self-bootstrapping. plan 0005에 `risk_class: critical` 명시하지만 cross-review skip (정책 active되기 _전_). 첫 실제 cross-review = F-003 (Supabase + RLS) 또는 F-004 (Drizzle + user RLS) — 둘 다 critical.

## 비용·결제 미정 사항 (사용자 액션 필요)

ADR 도입 시점(2026-05-07): 사용자 ChatGPT Plus 미구독. 정책만 박고 *결제 + GitHub Codex 연동*은 새 세션 첫 critical plan(F-003 추정) 시점에 사용자 액션. 가이드: `docs/cross-review-guide.md`.
