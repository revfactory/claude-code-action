# 마이그레이션 가이드: v0.x에서 v1.0으로

이 가이드는 Claude Code Action v0.x에서 v1.0으로 마이그레이션하는 방법을 안내합니다. 새 버전에서는 지능형 모드 감지와 간소화된 설정이 도입되었으며, 대부분의 사용 사례에서 하위 호환성이 유지됩니다.

## 변경 사항 개요

### 🎯 v1.0의 주요 개선 사항

1. **자동 모드 감지** - 더 이상 수동으로 `mode`를 설정할 필요가 없습니다
2. **간소화된 설정** - 통합된 `prompt` 및 `claude_args` 입력
3. **향상된 SDK 정합성** - Claude Code CLI와의 긴밀한 통합

### ⚠️ 호환성을 깨뜨리는 변경 사항

다음 입력값들은 더 이상 사용되지 않으며 대체되었습니다:

| 지원 중단된 입력값      | 대체 방법                             | 참고                                          |
| --------------------- | ------------------------------------ | --------------------------------------------- |
| `mode`                | 자동 감지                             | 컨텍스트에 따라 액션이 자동으로 선택합니다       |
| `direct_prompt`       | `prompt`                             | 직접 대체하여 사용할 수 있습니다                |
| `override_prompt`     | `prompt`                             | GitHub 컨텍스트 변수를 대신 사용하세요          |
| `custom_instructions` | `claude_args: --system-prompt`       | CLI 인수로 이동합니다                          |
| `max_turns`           | `claude_args: --max-turns`           | CLI 형식을 사용합니다                          |
| `model`               | `claude_args: --model`               | CLI를 통해 지정합니다                          |
| `allowed_tools`       | `claude_args: --allowedTools`        | CLI 형식을 사용합니다                          |
| `disallowed_tools`    | `claude_args: --disallowedTools`     | CLI 형식을 사용합니다                          |
| `claude_env`          | `settings`의 env 객체                 | settings JSON을 사용합니다                     |
| `mcp_config`          | `claude_args: --mcp-config`          | CLI 인수를 통해 MCP 설정을 전달합니다            |
| `timeout_minutes`     | GitHub Actions의 `timeout-minutes`   | 입력값이 아닌 작업(job) 수준에서 설정합니다       |

## 마이그레이션 예시

### 기본 대화형 워크플로 (@claude 멘션)

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mode: "tag"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    custom_instructions: "Follow our coding standards"
    max_turns: "10"
    allowed_tools: "Edit,Read,Write"
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --max-turns 10
      --system-prompt "Follow our coding standards"
      --allowedTools Edit,Read,Write
```

### 자동화 워크플로

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mode: "agent"
    direct_prompt: "Review this PR for security issues"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    model: "claude-3-5-sonnet-20241022"
    allowed_tools: "Edit,Read,Write"
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: |
      REPO: ${{ github.repository }}
      PR NUMBER: ${{ github.event.pull_request.number }}

      Review this PR for security issues
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --model claude-4-0-sonnet-20250805
      --allowedTools Edit,Read,Write
```

> **⚠️ 중요**: PR 리뷰 시에는 반드시 프롬프트에 리포지토리와 PR 컨텍스트를 포함하세요. 이렇게 해야 Claude가 어떤 PR을 리뷰해야 하는지 알 수 있습니다.

### 진행 상황 추적이 포함된 자동화 (v1.0 신규 기능)

**v0.x agent 모드의 추적 코멘트가 그리우신가요?** 새로운 `track_progress` 입력값이 이를 다시 제공합니다!

v1.0에서 자동화 모드(`prompt` 입력 사용)는 노이즈를 줄이기 위해 기본적으로 추적 코멘트를 생성하지 않습니다. 그러나 진행 상황의 가시성이 필요한 경우 `track_progress` 기능을 사용할 수 있습니다:

**이전 (추적 기능이 있는 v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mode: "agent"
    direct_prompt: "Review this PR for security issues"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**이후 (추적 기능이 있는 v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    track_progress: true # 추적 코멘트가 포함된 tag 모드를 강제 실행합니다
    prompt: |
      REPO: ${{ github.repository }}
      PR NUMBER: ${{ github.event.pull_request.number }}

      Review this PR for security issues
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

#### `track_progress`의 장점

1. **GitHub 컨텍스트 보존**: PR/이슈의 세부 사항, 코멘트, 첨부 파일을 자동으로 포함합니다
2. **추적 코멘트 복원**: v0.x agent 모드와 동일한 진행 상황 표시를 생성합니다
3. **커스텀 프롬프트와 호환**: 컨텍스트를 유지하면서 `prompt`가 커스텀 지시사항으로 주입됩니다

#### `track_progress`가 지원하는 이벤트

`track_progress` 입력값은 다음 GitHub 이벤트에서만 동작합니다:

**Pull Request 이벤트:**

- `opened` - 새 PR 생성
- `synchronize` - 새 커밋으로 PR 업데이트
- `ready_for_review` - 드래프트 PR이 리뷰 준비 완료로 변경
- `reopened` - 이전에 닫힌 PR이 다시 열림

**Issue 이벤트:**

- `opened` - 새 이슈 생성
- `edited` - 이슈 제목 또는 본문 수정
- `labeled` - 이슈에 레이블 추가
- `assigned` - 이슈에 담당자 지정

> **참고**: 지원되지 않는 이벤트에서 `track_progress: true`를 사용하면 오류가 발생합니다.

### 변수를 포함한 커스텀 템플릿

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    override_prompt: |
      Analyze PR #$PR_NUMBER in $REPOSITORY
      Changed files: $CHANGED_FILES
      Focus on security vulnerabilities
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: |
      REPO: ${{ github.repository }}
      PR NUMBER: ${{ github.event.pull_request.number }}

      Analyze this pull request focusing on security vulnerabilities in the changed files.

      Note: The PR branch is already checked out in the current working directory.
```

> **💡 팁**: 프롬프트에서 GitHub 컨텍스트 변수에 접근할 수 있지만, 일관성을 위해 표준 `REPO:` 및 `PR NUMBER:` 형식을 사용하는 것을 권장합니다.

### 환경 변수

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    claude_env: |
      NODE_ENV: test
      CI: true
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    settings: |
      {
        "env": {
          "NODE_ENV": "test",
          "CI": "true"
        }
      }
```

### 타임아웃 설정

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    timeout_minutes: 30
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**이후 (v1.0):**

```yaml
jobs:
  claude-task:
    runs-on: ubuntu-latest
    timeout-minutes: 30 # 작업(job) 수준으로 이동
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

## 모드 감지 동작 방식

액션은 이제 적절한 모드를 자동으로 감지합니다:

1. **`prompt`가 제공된 경우** → **자동화 모드**로 실행

   - @claude 멘션을 기다리지 않고 즉시 실행됩니다
   - 예약된 작업, PR 자동화 등에 적합합니다

2. **`prompt`가 없지만 @claude가 멘션된 경우** → **대화형 모드**로 실행

   - @claude 멘션을 대기하고 응답합니다
   - 진행 상황이 포함된 추적 코멘트를 생성합니다

3. **둘 다 해당하지 않는 경우** → 아무 동작도 수행하지 않습니다

## claude_args를 활용한 고급 설정

`claude_args` 입력값은 Claude Code CLI 인수에 직접 접근할 수 있게 해줍니다:

```yaml
claude_args: |
  --max-turns 15
  --model claude-4-0-sonnet-20250805
  --allowedTools Edit,Read,Write,Bash
  --disallowedTools WebSearch
  --system-prompt "You are a senior engineer focused on code quality"
  --mcp-config '{"mcpServers": {"custom": {"command": "npx", "args": ["-y", "@example/server"]}}}'
```

### 주요 claude_args 옵션

| 옵션                 | 설명                    | 예시                                   |
| ------------------- | ---------------------- | -------------------------------------- |
| `--max-turns`       | 대화 턴 수 제한          | `--max-turns 10`                       |
| `--model`           | Claude 모델 지정        | `--model claude-4-0-sonnet-20250805`   |
| `--allowedTools`    | 특정 도구 활성화         | `--allowedTools Edit,Read,Write`       |
| `--disallowedTools` | 특정 도구 비활성화       | `--disallowedTools WebSearch`          |
| `--system-prompt`   | 시스템 지시사항 추가     | `--system-prompt "Focus on security"`  |
| `--mcp-config`      | MCP 서버 설정 추가       | `--mcp-config '{"mcpServers": {...}}'` |

## 클라우드 공급자별 업데이트

### AWS Bedrock

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    use_bedrock: "true"
    claude_args: |
      --model anthropic.claude-4-0-sonnet-20250805-v1:0
```

### Google Vertex AI

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    use_vertex: "true"
    claude_args: |
      --model claude-4-0-sonnet@20250805
```

## MCP 설정 마이그레이션

### 커스텀 MCP 서버 추가

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mcp_config: |
      {
        "mcpServers": {
          "custom-server": {
            "command": "npx",
            "args": ["-y", "@example/server"]
          }
        }
      }
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    claude_args: |
      --mcp-config '{"mcpServers": {"custom-server": {"command": "npx", "args": ["-y", "@example/server"]}}}'
```

파일을 통해 MCP 설정을 전달할 수도 있습니다:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    claude_args: |
      --mcp-config /path/to/mcp-config.json
```

## 단계별 마이그레이션 체크리스트

- [ ] 액션 버전을 `@beta`에서 `@v1`으로 업데이트
- [ ] `mode` 입력값 제거 (이제 자동 감지됩니다)
- [ ] `direct_prompt`를 `prompt`로 교체
- [ ] `override_prompt`를 GitHub 컨텍스트를 사용하는 `prompt`로 교체
- [ ] `custom_instructions`를 `claude_args`의 `--system-prompt`로 이동
- [ ] `max_turns`를 `claude_args`의 `--max-turns`로 변환
- [ ] `model`을 `claude_args`의 `--model`로 변환
- [ ] `allowed_tools`를 `claude_args`의 `--allowedTools`로 변환
- [ ] `disallowed_tools`를 `claude_args`의 `--disallowedTools`로 변환
- [ ] `claude_env`를 `settings` JSON 형식으로 이동
- [ ] `mcp_config`를 `claude_args`의 `--mcp-config`로 이동
- [ ] `timeout_minutes`를 작업(job) 수준의 GitHub Actions `timeout-minutes`로 교체
- [ ] **선택 사항**: 자동화 모드에서 추적 코멘트가 필요한 경우 `track_progress: true` 추가
- [ ] 운영 환경이 아닌 환경에서 워크플로 테스트

## 도움 받기

마이그레이션 중 문제가 발생하면 다음을 참고하세요:

1. 자주 묻는 질문은 [FAQ](./faq.md)를 확인하세요
2. 참고할 수 있는 [예시 워크플로](../examples/)를 검토하세요
3. 지원이 필요하면 [이슈](https://github.com/anthropics/claude-code-action/issues)를 등록하세요

## 버전 호환성

- **v0.x 워크플로**는 계속 동작하지만 지원 중단 경고가 표시됩니다
- **v1.0**은 모든 새로운 워크플로에 권장되는 버전입니다
- 향후 버전에서는 지원 중단된 입력값이 완전히 제거될 수 있습니다
