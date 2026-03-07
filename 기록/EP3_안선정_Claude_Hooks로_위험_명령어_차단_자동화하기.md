# Claude Hooks로 위험 명령어 차단 및 개발 작업 자동화하기

> 파일명: `EP3_안선정_Claude_Hooks로_위험_명령어_차단_자동화하기.md`

> **작성자:** [안선정]
> **날짜:** 2026-03-07
> **에피소드:** EP3 (3주차)

---

## Situation (상황)

- **문제 상황:** 스터디에서 AI Agent가 위험한 명령어를 자동으로 실행한다는 사례가 공유되었습니다. 작성자 본인도 동일한 경험이 있었습니다. Claude Code에게 `.claude 파일 내 내용을 제외하고 GitHub에 올려줘`라고 요청했더니, `.claude` 폴더 내 파일 전체를 `rm -rf`로 삭제해 버리는 참사가 발생했습니다. 되돌려 달라고 요청했지만 복구가 불가능했습니다.
- **사용 도구:** Claude Code, Claude Hooks
- **배경:** 무의식적으로 `Yes`를 선택하는 습관으로 인해 돌이킬 수 없는 작업이 실행되었습니다. 이를 계기로 Agent가 위험한 명령어를 실행하기 전에 자동으로 차단하는 환경을 구성하고자 했습니다.

## Task (과제)

- **목표:** 기능 개발 후 아래 작업이 자동으로 수행되도록 환경을 구성합니다.
  1. 위험한 명령어 실행 시 자동 차단
  2. 코드 컨벤션에 맞게 코드 리뷰 수행
  3. Prettier 코드 포매터 자동 적용
  4. `npm run build` 성공 여부 자동 확인

## Action (행동)

### Claude Hooks란?

Claude가 어떤 작업을 수행하기 전/후에 자동으로 실행되는 핸들러입니다. `command`, `http`, `prompt`, `agent` 유형을 지원하며, 아래와 같은 기능을 수행할 수 있습니다.

- **command:** Prettier, Lint 적용 (예: `npm run lint`)
- **http:** Slack API를 활용하여 작업 완료 알림 전송
- **prompt:** 위험 명령어 판단
- **agent:** 코드 리뷰, 테스트 케이스 작성 및 실행

Skill에 "이 작업 완료 후 반드시 OO해줘"라고 명시해도 100% 실행을 보장할 수 없지만, **Hooks는 원하는 시점에 실행을 보장**한다는 것이 핵심 특징입니다.

### 해결 방법

`.claude/settings.json`에 아래와 같이 Hooks를 구성했습니다.

```json
{
    "hooks": {
        "PreToolUse": [
            {
                "matcher": "Bash",
                "hooks": [
                    {
                        "type": "command",
                        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous.sh"
                    }
                ]
            }
        ],
        "PostToolUse": [
            {
                "matcher": "Edit|Write",
                "hooks": [
                    {
                        "type": "command",
                        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/prettier-format.sh",
                        "async": true
                    },
                    {
                        "type": "agent",
                        "prompt": "수정된 파일이 프로젝트 컨벤션을 준수하는지 검토하세요.\n\n1. Read 툴로 '$CLAUDE_PROJECT_DIR/CODE_CONVENTIONS.md'를 읽어 체크리스트를 파악하세요.\n2. Read 툴로 수정된 파일을 읽으세요: $ARGUMENTS\n3. 체크리스트 기준으로 위반 사항을 확인하세요.\n\n위반이 있으면 항목별로 간결하게 알려주세요. 위반이 없으면 아무것도 반환하지 마세요.",
                        "model": "claude-haiku-4-5-20251001"
                    }
                ]
            }
        ],
        "Stop": [
            {
                "hooks": [
                    {
                        "type": "command",
                        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/build-check.sh",
                        "async": true
                    }
                ]
            }
        ]
    }
}
```

**위험 명령어 차단 Hook 구성** (`.claude/hooks/block-dangerous.sh`)

차단 방식은 목적에 따라 선택합니다.
- "이건 꼭 막고 싶어" → `command`로 직접 패턴 비교
- "위험해 보이는 건 한 번 더 체크만 해줘" → `prompt` / `agent`

```bash
#!/usr/bin/env bash
# PreToolUse(Bash) Hook - 위험한 명령어 자동 차단
# exit 0: 통과 / exit 2: Claude에 피드백 + 차단

set -euo pipefail

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // ""' 2>/dev/null || echo "")

if [ -z "$COMMAND" ]; then
    exit 0
fi

DANGEROUS_PATTERNS=(
    'rm[[:space:]]+-rf'
    'rm[[:space:]]+-fr'
    'git[[:space:]]+reset[[:space:]]+--hard'
    'git[[:space:]]+push[[:space:]].*--force'
    'git[[:space:]]+push[[:space:]].*-f[[:space:]]'
    'git[[:space:]]+clean[[:space:]]+-f'
    'git[[:space:]]+branch[[:space:]]+-D'
    'DROP[[:space:]]+TABLE'
    'DELETE[[:space:]]+FROM.*WHERE.*1[[:space:]]*=[[:space:]]*1'
)

for PATTERN in "${DANGEROUS_PATTERNS[@]}"; do
    if echo "$COMMAND" | grep -qiE "$PATTERN"; then
        REASON="위험한 명령어가 감지되어 차단되었습니다: $(echo "$COMMAND" | head -c 100)"
        echo "{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"⛔ $REASON\"}}"
        exit 2
    fi
done

exit 0
```

## Result (결과)

- 위험 명령어 차단 Hook이 정상적으로 동작하여, `rm -rf`와 같은 패턴이 감지되면 Claude가 자동으로 실행을 중단하고 차단 사유를 피드백으로 제공했습니다.
- Prettier 자동 포매팅과 코드 컨벤션 리뷰, 빌드 확인이 각 시점에 맞게 자동으로 수행되었습니다.

## Learning (배움)

- **결정적 활용 팁:** Agent가 반드시 지켜야 하는 규칙은 Skill이 아닌 **Hooks로 구성해야 실행을 보장할 수 있습니다.** 반복적으로 수행하던 작업도 Hooks를 통해 보다 확실하게 자동화할 수 있습니다.

- **배운 것:** Hooks는 Skill과 달리 Agent의 행동 자체를 직접 제어하는 레이어입니다. 추후 HTTP Hooks를 활용하여 Slack 연동 등 외부 서비스와 연계하는 자동화로도 확장할 수 있을 것으로 기대됩니다.
