# 프로토콜 ‘설계/개선’에 머물러 ‘실행(압축 산출)’ 타이밍을 놓친 시행착오

> 파일명: `EP3_ChatGPT_프로토콜_개선에_매몰돼_압축_실행_누락.md`

> **작성자:** 박예준
> **날짜:** 2026-03-05
> **에피소드:** EP3 (3주차)

---

## Situation (상황)

- **문제 상황:** 세션 재개를 위해 “대화 내용 압축(=Restart Package 산출)”이 필요했는데, 대화가 길어지면서 모델이 내부 compaction을 어떻게 하든 사용자 입장에선 확실한 재개용 컨텍스트가 필요해졌다.
- **사용 도구:** ChatGPT Web (브라우저), 대화 기반 프롬프트 설계(AGENTS.MD/CLAUDE.md 스타일 참고)
- **배경:** 임시 채팅/긴 대화를 나중에 이어가기 위해, “세션 마지막에 재개 패키지를 생성해 다음 세션에서 첨부 없이 재개”하는 운영을 만들고 있었다.

## Task (과제)

- **목표:** 세션 종료 시점에 “프로토콜을 **실행**”해서, 다음 세션에 그대로 붙여넣을 수 있는 압축 산출물(Anchor/Changelog/Verification 포함)을 생성한다.

## Action (행동)

### 시행착오
- 프로토콜(END-OF-SESSION) 문구를 만들고 다듬는 과정에서, 사용자가 원하는 것은 “프로토콜 텍스트 개선”이 아니라 “지금 당장 그 프로토콜을 실행해서 **압축 결과물**을 내는 것”이었는데도,
  - 모델이 계속 **프로토콜 설계/최적화 모드**로 해석하고
  - 실제 압축 산출(Anchor 등)을 즉시 출력하지 못했다.
- 왜 잘 안 됐는지
  - 요청의 핵심이 “행동(실행)”인지 “문서(개선)”인지 구분이 흐려졌고,
  - 대화 흐름이 “가이드/베스트 프랙티스/포맷 개선”으로 오래 이어지면서 마지막 사용자의 의도가 “실행 전환”임을 놓쳤다.
  - 사용자가 프로토콜 본문을 붙여넣었을 때, 이를 “실행 트리거”로 못 박지 않아 모드 혼동이 발생했다.

### 해결 방법
- 프로토콜을 **Execution-locked**로 수정하여, 프로토콜이 등장하면 “설계/평가”가 아니라 “즉시 실행”하도록 강제했다.
- 또한 출력물이 다음 세션에서 바로 쓰일 수 있도록, 결과 전체가 곧 RESTART PROMPT가 되게 **Single-block Output(v2.3)** 형식으로 변경했다.
- 사용한 프롬프트/설정/도구
  - Execution-locked 규칙: “이 메시지가 등장하면 실행 명령이며, Output Format 1~6만 출력하고 멈춘다.”
  - Single-block 규칙: “출력 전체가 단 하나의 RESTART PROMPT 블록이어야 하며, 블록 밖 텍스트 금지.”

## Result (결과)

- 최종 결과는 어땠는지
  - 프로토콜 등장 = 실행 트리거로 고정되어, “개선 모드로 도망가는” 문제가 줄었다.
  - 단일 블록 산출물로 정리되어, 다음 세션에 그대로 복붙해 재개할 수 있는 형태가 됐다.
- 성공/부분 성공/실패 여부
  - **부분 성공 → 성공으로 수습:** 초기에는 실행 누락(실패)이 있었으나, Execution-locked + Single-block으로 운영 안정성을 개선했다.

## Learning (배움)

- **결정적 활용팁:** “프로토콜 문서”와 “프로토콜 실행”을 절대 섞지 말고, **트리거 문장(Execution-locked)**과 **단일 산출물 포맷(Single-block)**으로 모드 혼동을 차단하자.
- **배운 것:**
  - 대화가 ‘설계/최적화’ 흐름으로 길어질수록, 사용자가 원한 것은 종종 “지금 당장 실행”이다.
  - 실행이 목표인 프로토콜은 “평가/개선 금지, 지정 포맷만 출력”처럼 **행동을 강제하는 규칙**이 있어야 안정적으로 반복 사용된다.

## 결과물
```markdown
# END-OF-SESSION PROTOCOL (Restart Package v2.3 — SINGLE-BLOCK OUTPUT)

## Mission
이 세션의 내용을 다음 세션에서 “첨부 없이” 바로 이어가기 위해,
다음 세션 첫 메시지로 그대로 붙여넣을 수 있는 **단일 RESTART PROMPT 블록**을 생성한다.

## Mode & Trigger (MOST IMPORTANT)
- 이 메시지가 등장하면 이는 “프로토콜 실행 명령”이다.
- 출력은 오직 **하나의 블록**이어야 하며, 그 블록이 곧 “다음 세션 첫 메시지(=RESTART PROMPT)”다.
- 따라서 별도의 해설/평가/대안/추가 텍스트를 블록 밖에 쓰지 않는다.

## Hard Rules
- [CRITICAL] 추측 금지: 대화에 없는 사실을 추가하지 않는다.
- [CRITICAL] 불확실하면 `UNKNOWN`으로 남긴다.
- [HIGH] 모순/충돌은 임의로 선택하지 말고 `CONFLICT`로 표시한다.
- [HIGH] 숫자/기한/버전/고유명사/규칙/결정사항은 가능한 한 원문 표현을 보존한다.
- [HIGH] 우선순위: Constraints/Don’ts > Decisions (Locked) > Next Actions > 나머지.

## Length Budget (inside the single block)
- ANCHOR: 20–30줄 이내
- Decisions/Constraints: 각 5–12개 이내
- EXCERPTS: 3–7개 이내 (hard details만)

## Output Format (SINGLE BLOCK ONLY)
아래 형식 그대로, **단 하나의 블록**만 출력하라:

<<<RESTART_PROMPT>>>
[INSTRUCTIONS TO ASSISTANT]
- 이 메시지는 이전 세션의 재개용 패키지다. 아래 ANCHOR를 기준으로 진행하라.
- 추측 금지 / UNKNOWN / CONFLICT 유지.
- 기본 출력 언어: 한국어(내가 요청하지 않으면 변경하지 말 것).
- 먼저 VERIFICATION 질문 3개에 답해 “컨텍스트가 맞는지” 확인한 뒤 본 작업을 시작하라.

[SESSION ANCHOR]
- Mission:
- Current State:
- Decisions (Locked):
- Constraints / Don’ts:
- Definitions (if any):
- Assumptions / Unknowns (must include UNKNOWN):
- Open Questions:
- Next Actions (1–3):

[CHANGELOG]
- Added:
- Updated:
- Removed:
(각 항목에 근거 한 줄. 불확실하면 UNKNOWN)

[VERIFICATION]
1)
2)
3)

[CONFLICT RESOLUTION] (only if any)
- CONFLICT:
  - Question needed to resolve:

[OPTIONAL EXCERPTS] (only if needed)
- (3–7 hard-detail excerpts)
<<<END_RESTART_PROMPT>>>

## Stop Condition
- 위 SINGLE BLOCK 외에는 아무것도 출력하지 말고 즉시 멈춘다.
```
