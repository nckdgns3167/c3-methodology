# Postmortems

C3 사건 회고 (사고/장애/데이터 손실/보안 사건). ADR-0013 정책 정합.

## 명명 규칙

`YYYY-MM-DD-<title>.md` (날짜 기반 — ADR/plan과 다름)

예시:

- `2026-08-15-rls-policy-leak.md`
- `2026-09-22-deploy-rollback.md`
- `2026-10-03-secret-rotation-failure.md`

## 사용 시점

| 상황                                             | 어디                                 | 형식                               |
| ------------------------------------------------ | ------------------------------------ | ---------------------------------- |
| 본격 사고 (데이터 손실 / 보안 / production 장애) | `docs/postmortems/<날짜>-<title>.md` | `_template.md` 채움 (1페이지 이내) |
| 사소한 디버깅 (오타 / 설정 실수 / 환경 문제)     | `claude-progress.md`                 | 한두 줄 메모                       |

**경계**: "다른 사람(또는 6개월 뒤의 나)이 같은 문제 겪을 가능성 있는가?" — yes면 정식 postmortem, no면 메모.

## 작성 흐름

1. 사고 해결 직후 `_template.md` 복사 → `<날짜>-<title>.md`
2. 채움 (1페이지 이내). Severity / Detected / Resolved / Duration / Affected / What happened / Timeline / 5 Whys / Action items / What worked / What didn't.
3. **Action items의 `- [ ]` 항목은 `feature_list.json`에 새 F-XXX로 등록** (ADR-0013 의무).
4. git commit (`docs: postmortem <간단 요약>` — chore/docs 메타 type이라 feature ID 면제).

## Blameless 원칙

*사람이 아니라 시스템*을 분석.

- ❌ "X가 실수했다"
- ✅ "X 실수를 막을 가드레일이 없었다"

1인이라도 자기에게 적용:

- ❌ "내가 멍청해서"
- ✅ "스키마 검증이 없어서"

Action item이 *사람의 더 잘하기*가 아닌 *시스템 변경*으로 나와야 한다. 비난이 들어가면 다음번 사건은 _숨겨짐_ — postmortem의 가장 큰 적.

## 5 Whys

표면 원인이 아닌 *시스템 원인*까지 5번 내려가기.

예시:

1. 왜 RLS 정책이 누락됐나? → 마이그레이션에서 빠뜨림
2. 왜 빠뜨렸나? → AGENTS.md C9를 자기 검증할 때 "RLS enable됨"만 봄
3. 왜 enable과 policy를 구분 안 했나? → 둘은 별도 SQL 명령
4. 왜 자동 검증이 없었나? → init.sh에 RLS policy 존재 검증 단계 부재
5. 왜 부재했나? → F-005 설계 시 enable만 검사 항목으로 정의

→ Action item: `init.sh`에 `policies_exist_for_rls_enabled_tables()` 검사 추가 → 새 F-XXX

## 인덱스

(아직 없음 — 첫 사건 발생 시 추가)
