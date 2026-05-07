# Cross-review 가이드 (Claude × Codex)

ADR-0014 정책 실행 가이드. risk_class `critical` plan은 **반드시** Codex로 cross-review (`mid`는 권장, `easy`는 skip).

## 사전 셋업 (1회, 사용자 액션)

### 1. ChatGPT Plus 구독 ($20/mo)

- https://chatgpt.com/codex/pricing 또는 ChatGPT 설정 → Plan
- Pro $200/mo는 Phase 1 솔로엔 과 — Plus 충분 (월 ~10~30 review 추정)
- 구독 결제 후 Codex 사용 가능

### 2. GitHub repo 연동

- https://chatgpt.com/codex 접속 → Settings → Repositories
- `nckdgns3167/c3` (private) 추가 + 권한 승인
- Codex가 우리 repo의 `AGENTS.md` 자동 인식 (17 제약 + ADR 인덱스를 review 기준으로 사용)

### 3. (선택) Automatic reviews 토글

- Settings → Code review → Automatic reviews ON
- 모든 PR에 자동 review (수동 `@codex review` 불필요)
- 솔로엔 *모든 PR이 cross-review 필요*가 아니라 *critical/mid만*이라 OFF 권장. 매번 수동 트리거.

## 워크플로 (매 critical/mid plan)

### Step 1 — plan 작성 + risk_class 분류

`docs/plans/NNNN-<topic>.md` frontmatter에 `risk_class: critical | mid | easy` 박음. ADR-0014 분류 기준 따름.

### Step 2 — branch + commit + PR

```bash
git checkout -b plan/NNNN-<topic>
git add docs/plans/NNNN-<topic>.md
git commit -m "docs: plan NNNN <topic> 작성 (risk_class: critical, cross-review 대기)"
git push -u origin plan/NNNN-<topic>
gh pr create --title "Plan NNNN — <topic>" --body "$(cat docs/plans/NNNN-<topic>.md)" --label cross-review
```

PR body에 plan 내용 통째로 — Codex가 review 시 컨텍스트 충분.

### Step 3 — Codex review 트리거

PR 페이지에서 코멘트:

```
@codex review
```

Codex가 P0/P1 priority 코멘트 남김 (보통 수십 초~수 분).

### Step 4 — 결과 분석 + plan 안 박음

plan 파일 끝에 `## Cross-review (Codex)` 섹션 (이미 표준 — `docs/plans/README.md` 참조).

- **Convergence**: Claude + Codex 일치 항목 (그대로 진행)
- **Divergence**: 의견 차이 + trade-off 분석 + 사용자 결정 + 근거
- **Codex 단독 catch**: P0/P1 코멘트 핵심 (Claude 미발견 위험)
- **Status**: review 시점·PR번호

### Step 5 — plan 갱신 (필요 시) + merge

- Divergence가 plan 변경 필요 시 갱신 commit
- 재 review 권장 (`@codex review` 다시)
- 통과 시 PR merge → main

### Step 6 — feature 작업 진행

이제 정상 implementation. plan에 cross-review 결과 박혀 있어 작업 중 reference 가능.

### Step 7 — feature done 시 Post-mortem (ADR-0013)

`## Post-mortem` 섹션에 _cross-review 효과_ 회고 포함:

- Codex가 잡은 위험 / 못 잡은 위험
- Convergence vs Divergence 비율
- 다음 plan에 가져갈 review 패턴

`validate-feature-list.ts`가 done 처리 시 (a) Post-mortem 섹션 (ADR-0013) + (b) critical이면 Cross-review 섹션 (ADR-0014) 둘 다 검증.

## 결과 형식 (plan 안 표준)

```markdown
## Cross-review (Codex) — risk_class: critical

**Convergence (Claude + Codex 일치)**:

- (수렴 항목 1)
- (수렴 항목 2)

**Divergence (의견 차이)**:

- Claude: A 제안
- Codex: B 제안
- **결정**: B (이유: ... AGENTS.md C9 더 엄격하게 따라가는 게 안전)

**Codex 단독 catch (Claude 미발견)**:

- P0: <코멘트 핵심>
- P1: <코멘트 핵심>

**Status**:

- review 트리거: 2026-MM-DD HH:MM (PR #N)
- review 통과: 2026-MM-DD HH:MM
- 갱신 후 재 review (있다면): 2026-MM-DD HH:MM
```

## 실패·예외 처리

### Codex 응답 안 함 (10분 이상)

- ChatGPT 상태 확인 (https://status.openai.com)
- Plus 한도 초과? → 다음 청구 주기 또는 Pro 업그레이드 검토
- repo 권한 만료? → Settings → Repositories 재승인

### Codex가 AGENTS.md 무시한 일반론적 review

- AGENTS.md를 더 명시적으로 (구체적 anti-pattern 추가)
- Codex prompt에 "AGENTS.md 17 제약을 _반드시 명시적으로_ 검증하고 위반 사항을 P0로 표시" 추가 (PR body 또는 코멘트)

### 사용자가 Plus 한도 초과

- 트라이얼 1개월 후 후속 ADR로 _영구 도입 vs 폐기_ 결정
- 한도 압박이 자주 → Pro $200 ROI 검토

## 자동화 (Phase 2 검토)

- GitHub Actions로 `risk_class: critical` PR 자동 감지 → @codex review 자동 트리거
- Codex CLI로 로컬에서 cross-review (PR 만들지 않고)
- Webhook으로 Slack/Discord 알림

지금은 수동 트리거 (Plus 한도 모니터링 + 가치 검증 우선).

## 트라이얼 회고 (도입 1개월 후 의무)

ADR-0014 결정 6번에 명시. 1개월(2026-06-07) 후 후속 ADR로:

- Convergence rate (Claude + Codex 일치 비율)
- Codex 단독 catch 횟수 + 진짜 catch했는지 (false positive 비율)
- 비용 vs 가치 ($20/mo × 사고 회피 가치)
- _영구 도입_ vs _폐기_ vs _Pro 업그레이드_ 결정

회고 결과를 `c3_cross_review.md` 메모리에 박고, 영구 도입 시 ADR-0014 + 이 가이드 보강.
