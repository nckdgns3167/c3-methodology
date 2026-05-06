# <간단 사건 제목>

- **Severity**: low / medium / high / critical
- **Detected**: YYYY-MM-DD HH:MM (수동 발견 / 모니터링 알람 / 사용자 신고)
- **Resolved**: YYYY-MM-DD HH:MM
- **Duration**: Xh Ym
- **Affected**: production / staging / 로컬 / 사용자 N명

## What happened

(1~3줄 요약 — 무슨 일이 일어났는가)

## Timeline

- HH:MM <event>
- HH:MM <event>
- ...

## Why (5 Whys)

1. 왜 X가 일어났나 → A
2. 왜 A 였나 → B
3. 왜 B 였나 → C
4. 왜 C 였나 → D
5. 왜 D 였나 → E (시스템 원인)

(Blameless — _사람이 아니라 시스템_ 분석. "X가 실수했다" X / "X 실수를 막을 가드레일이 없었다" O.)

## Action items

- [ ] <시스템 변경 1> → 새 feature F-XXX 추가 또는 기존 보강
- [ ] <시스템 변경 2>
- ...

(_반드시_ `feature_list.json`에 매핑. 회고 → 실행 단절 차단의 핵심.)

## What worked

- (잘 한 것 — 사고 발견·대응에서 효과적이었던 것)

## What didn't

- (놓친 것 — 운에 의존했거나 더 빨랐어야 할 것)
