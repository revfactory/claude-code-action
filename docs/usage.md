# 사용법

저장소에 워크플로우 파일을 추가하세요 (예: `.github/workflows/claude.yml`):

```yaml
name: Claude Assistant
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, assigned, labeled]
  pull_request_review:
    types: [submitted]

jobs:
  claude-response:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # 또는 OAuth 토큰을 대신 사용:
          # claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}

          # 선택 사항: 자동화 워크플로우를 위한 프롬프트 제공
          # prompt: "Review this PR for security issues"

          # 선택 사항: Claude CLI에 고급 인자 전달
          # claude_args: |
          #   --max-turns 10
          #   --model claude-4-0-sonnet-20250805

          # 선택 사항: 커스텀 플러그인 마켓플레이스 추가
          # plugin_marketplaces: "https://github.com/user/marketplace1.git\nhttps://github.com/user/marketplace2.git"
          # 선택 사항: Claude Code 플러그인 설치
          # plugins: "code-review@claude-code-plugins\nfeature-dev@claude-code-plugins"

          # 선택 사항: 커스텀 트리거 문구 추가 (기본값: @claude)
          # trigger_phrase: "/claude"
          # 선택 사항: 이슈에 대한 담당자 트리거 추가
          # assignee_trigger: "claude"
          # 선택 사항: 이슈에 대한 라벨 트리거 추가
          # label_trigger: "claude"
          # 선택 사항: 추가 권한 부여 (해당하는 GitHub 토큰 권한 필요)
          # additional_permissions: |
          #   actions: read
          # 선택 사항: 봇 사용자의 Action 트리거 허용
          # allowed_bots: "dependabot[bot],renovate[bot]"
```

## 입력값

| 입력값                           | 설명                                                                                                                                                                                   | 필수 여부 | 기본값        |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- | ------------- |
| `anthropic_api_key`              | Anthropic API 키 (직접 API 사용 시 필수, Bedrock/Vertex 사용 시 불필요)                                                                                                                | 아니요\*  | -             |
| `claude_code_oauth_token`        | Claude Code OAuth 토큰 (anthropic_api_key의 대안)                                                                                                                                     | 아니요\*  | -             |
| `prompt`                         | Claude에 대한 지시 사항. 직접 프롬프트 또는 자동화 워크플로우를 위한 커스텀 템플릿 가능                                                                                                 | 아니요    | -             |
| `track_progress`                 | 추적 댓글이 포함된 태그 모드를 강제합니다. 특정 PR/이슈 이벤트에서만 동작합니다. GitHub 컨텍스트를 유지합니다                                                                            | 아니요    | `false`       |
| `include_fix_links`              | PR 코드 리뷰 피드백에 'Fix this' 링크를 포함하여 식별된 문제를 수정하기 위한 컨텍스트와 함께 Claude Code를 엽니다                                                                       | 아니요    | `true`        |
| `claude_args`                    | [Claude CLI에 직접 전달할 추가 인자](https://docs.claude.com/en/docs/claude-code/cli-reference#cli-flags) (예: `--max-turns 10 --model claude-4-0-sonnet-20250805`)                     | 아니요    | ""            |
| `base_branch`                    | 새 브랜치를 생성할 때 사용할 기본 브랜치 (예: 'main', 'develop')                                                                                                                       | 아니요    | -             |
| `use_sticky_comment`             | PR 댓글 전달 시 하나의 댓글만 사용합니다 (pull_request 이벤트 워크플로우에만 적용)                                                                                                      | 아니요    | `false`       |
| `github_token`                   | Claude가 사용할 GitHub 토큰. **자체 커스텀 GitHub 앱을 연결하는 경우에만 포함하세요!**                                                                                                  | 아니요    | -             |
| `use_bedrock`                    | 직접 Anthropic API 대신 OIDC 인증을 사용하는 Amazon Bedrock 사용                                                                                                                       | 아니요    | `false`       |
| `use_vertex`                     | 직접 Anthropic API 대신 OIDC 인증을 사용하는 Google Vertex AI 사용                                                                                                                     | 아니요    | `false`       |
| `assignee_trigger`               | Action을 트리거하는 담당자 사용자명 (예: @claude). 이슈 할당에만 사용됩니다                                                                                                             | 아니요    | -             |
| `label_trigger`                  | 이슈에 적용될 때 Action을 트리거하는 라벨 이름 (예: "claude")                                                                                                                          | 아니요    | -             |
| `trigger_phrase`                 | 댓글, 이슈/PR 본문, 이슈 제목에서 찾을 트리거 문구                                                                                                                                     | 아니요    | `@claude`     |
| `branch_prefix`                  | Claude 브랜치에 사용할 접두사 (기본값 'claude/', 대시 형식은 'claude-' 사용)                                                                                                            | 아니요    | `claude/`     |
| `settings`                       | JSON 문자열 또는 설정 JSON 파일 경로로 된 Claude Code 설정                                                                                                                             | 아니요    | ""            |
| `additional_permissions`         | 활성화할 추가 권한. 현재 워크플로우 결과 조회를 위한 'actions: read'를 지원합니다                                                                                                       | 아니요    | ""            |
| `use_commit_signing`             | GitHub API를 사용한 커밋 서명을 활성화합니다. 간단하지만 리베이스와 같은 복잡한 git 작업은 수행할 수 없습니다. [보안](./security.ko.md#commit-signing) 참조                               | 아니요    | `false`       |
| `ssh_signing_key`                | 커밋 서명을 위한 SSH 개인 키. 전체 git CLI 지원(리베이스 등)이 가능한 서명된 커밋을 활성화합니다. [보안](./security.ko.md#commit-signing) 참조                                           | 아니요    | ""            |
| `bot_id`                         | git 작업에 사용할 GitHub 사용자 ID (기본값은 Claude의 봇 ID). 인증된 커밋을 위해 `ssh_signing_key`와 함께 필요합니다                                                                    | 아니요    | `41898282`    |
| `bot_name`                       | git 작업에 사용할 GitHub 사용자명 (기본값은 Claude의 봇 이름). 인증된 커밋을 위해 `ssh_signing_key`와 함께 필요합니다                                                                   | 아니요    | `claude[bot]` |
| `allowed_bots`                   | 허용할 봇 사용자명의 쉼표 구분 목록, 또는 모든 봇을 허용하려면 '\*'. 빈 문자열(기본값)은 봇을 허용하지 않습니다                                                                         | 아니요    | ""            |
| `allowed_non_write_users`        | **⚠️ 위험**: 쓰기 권한 없이 허용할 사용자명의 쉼표 구분 목록, 또는 모든 사용자에 대해 '\*'. `github_token` 입력값과 함께만 동작합니다. [보안](./security.ko.md) 참조                     | 아니요    | ""            |
| `path_to_claude_code_executable` | 커스텀 Claude Code 실행 파일의 선택적 경로. 자동 설치를 건너뜁니다. Nix, 커스텀 컨테이너 또는 특수 환경에 유용합니다                                                                    | 아니요    | ""            |
| `path_to_bun_executable`         | 커스텀 Bun 실행 파일의 선택적 경로. 자동 Bun 설치를 건너뜁니다. Nix, 커스텀 컨테이너 또는 특수 환경에 유용합니다                                                                        | 아니요    | ""            |
| `plugin_marketplaces`            | 설치할 Claude Code 플러그인 마켓플레이스 Git URL의 줄바꿈 구분 목록 (예: 위 워크플로우 예제 참조). 마켓플레이스는 플러그인 설치 전에 추가됩니다                                          | 아니요    | ""            |
| `plugins`                        | 설치할 Claude Code 플러그인 이름의 줄바꿈 구분 목록 (예: 위 워크플로우 예제 참조). 플러그인은 Claude Code 실행 전에 설치됩니다                                                          | 아니요    | ""            |

### 지원 종료 예정 입력값

이 입력값들은 지원 종료 예정이며 향후 버전에서 제거될 예정입니다:

| 입력값                | 설명                                                                                          | 마이그레이션 방법                                              |
| --------------------- | --------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| `mode`                | **지원 종료 예정**: 모드는 이제 워크플로우 컨텍스트를 기반으로 자동 감지됩니다                 | 이 입력값을 제거하세요. Action이 올바른 모드를 자동 감지합니다  |
| `direct_prompt`       | **지원 종료 예정**: 대신 `prompt`를 사용하세요                                                 | `prompt`로 대체하세요                                          |
| `override_prompt`     | **지원 종료 예정**: 템플릿 변수와 함께 `prompt`를 사용하거나 `claude_args`에 `--system-prompt`를 사용하세요 | 템플릿에는 `prompt`를, 시스템 프롬프트에는 `claude_args`를 사용하세요 |
| `custom_instructions` | **지원 종료 예정**: `claude_args`에 `--system-prompt`를 사용하거나 `prompt`에 포함하세요        | 지시 사항을 `prompt`로 이동하거나 `claude_args`를 사용하세요   |
| `max_turns`           | **지원 종료 예정**: 대신 `claude_args`에 `--max-turns`를 사용하세요                            | `claude_args: "--max-turns 5"` 사용                            |
| `model`               | **지원 종료 예정**: 대신 `claude_args`에 `--model`을 사용하세요                                | `claude_args: "--model claude-4-0-sonnet-20250805"` 사용       |
| `fallback_model`      | **지원 종료 예정**: `claude_args`에 폴백 구성을 사용하세요                                     | `claude_args` 또는 `settings`에서 폴백을 구성하세요            |
| `allowed_tools`       | **지원 종료 예정**: 대신 `claude_args`에 `--allowedTools`를 사용하세요                         | `claude_args: "--allowedTools Edit,Read,Write"` 사용           |
| `disallowed_tools`    | **지원 종료 예정**: 대신 `claude_args`에 `--disallowedTools`를 사용하세요                      | `claude_args: "--disallowedTools WebSearch"` 사용              |
| `mcp_config`          | **지원 종료 예정**: 대신 `claude_args`에 `--mcp-config`를 사용하세요                           | `claude_args: "--mcp-config '{...}'"` 사용                     |
| `claude_env`          | **지원 종료 예정**: `settings`에 환경 구성을 사용하세요                                        | `settings` JSON에서 환경을 구성하세요                          |

\*직접 Anthropic API 사용 시 필수 (기본값이며 Bedrock 또는 Vertex를 사용하지 않는 경우)

> **참고**: 이 Action은 현재 베타 단계입니다. 통합을 지속적으로 개선함에 따라 기능과 API가 변경될 수 있습니다.

## v0.x에서 업그레이드하시나요?

v0.x에서 v1.0으로의 마이그레이션에 대한 단계별 지침과 예제를 포함한 종합 가이드는 **[마이그레이션 가이드](./migration-guide.ko.md)**를 참조하세요.

### 빠른 마이그레이션 예제

#### 대화형 워크플로우 (@claude 멘션 사용)

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mode: "tag"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    custom_instructions: "Focus on security"
    max_turns: "10"
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --max-turns 10
      --system-prompt "Focus on security"
```

#### 자동화 워크플로우

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mode: "agent"
    direct_prompt: "Update the API documentation"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    model: "claude-4-0-sonnet-20250805"
    allowed_tools: "Edit,Read,Write"
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: |
      REPO: ${{ github.repository }}
      PR NUMBER: ${{ github.event.pull_request.number }}

      Update the API documentation to reflect changes in this PR
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --model claude-4-0-sonnet-20250805
      --allowedTools Edit,Read,Write
```

#### 커스텀 템플릿

**이전 (v0.x):**

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    override_prompt: |
      Analyze PR #$PR_NUMBER for security issues.
      Focus on: $CHANGED_FILES
```

**이후 (v1.0):**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: |
      Analyze PR #${{ github.event.pull_request.number }} for security issues.
      Focus on the changed files in this PR.
```

## 구조화된 출력

Claude로부터 검증된 JSON 결과를 받아 자동으로 GitHub Action 출력값으로 사용할 수 있습니다. 이를 통해 Claude가 데이터를 분석하고 후속 단계에서 그 결과를 사용하는 복잡한 자동화 워크플로우를 구축할 수 있습니다.

### 기본 예제

```yaml
- name: Detect flaky tests
  id: analyze
  uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: |
      Check the CI logs and determine if this is a flaky test.
      Return: is_flaky (boolean), confidence (0-1), summary (string)
    claude_args: |
      --json-schema '{"type":"object","properties":{"is_flaky":{"type":"boolean"},"confidence":{"type":"number"},"summary":{"type":"string"}},"required":["is_flaky"]}'

- name: Retry if flaky
  if: fromJSON(steps.analyze.outputs.structured_output).is_flaky == true
  run: gh workflow run CI
```

### 작동 방식

1. **스키마 정의**: `claude_args`의 `--json-schema` 플래그를 통해 JSON 스키마를 제공합니다
2. **Claude 실행**: Claude가 도구를 사용하여 작업을 완료합니다
3. **검증된 출력**: 결과가 제공한 스키마에 대해 검증됩니다
4. **JSON 출력**: 모든 필드가 단일 `structured_output` JSON 문자열로 반환됩니다

### 구조화된 출력 접근하기

모든 구조화된 출력 필드는 `structured_output` 출력값에서 JSON 문자열로 사용할 수 있습니다:

**GitHub Actions 표현식에서:**

```yaml
if: fromJSON(steps.analyze.outputs.structured_output).is_flaky == true
run: |
  CONFIDENCE=${{ fromJSON(steps.analyze.outputs.structured_output).confidence }}
```

**bash에서 jq 사용:**

```yaml
- name: Process results
  run: |
    OUTPUT='${{ steps.analyze.outputs.structured_output }}'
    IS_FLAKY=$(echo "$OUTPUT" | jq -r '.is_flaky')
    SUMMARY=$(echo "$OUTPUT" | jq -r '.summary')
```

**참고**: GitHub Actions의 제한 사항으로 인해 복합 액션은 동적 출력값을 노출할 수 없습니다. 모든 필드는 단일 `structured_output` JSON 문자열에 묶여 있습니다.

### 전체 예제

작동하는 예제는 `examples/test-failure-analysis.yml`을 참조하세요. 이 예제는 다음을 수행합니다:

- 불안정한 테스트 실패 감지
- 조건문에서 신뢰도 임계값 사용
- 워크플로우 자동 재시도
- PR에 댓글 작성

### 문서

JSON Schema 구문 및 Agent SDK 구조화된 출력에 대한 전체 내용은 다음을 참조하세요:
https://docs.claude.com/en/docs/agent-sdk/structured-outputs

## @claude를 태그하는 방법

다음 예제들은 PR과 이슈에서 댓글을 사용하여 Claude와 상호작용하는 방법을 보여줍니다. 기본적으로 `@claude`를 멘션하면 Claude가 트리거되지만, 워크플로우에서 `trigger_phrase` 입력값을 사용하여 정확한 트리거 문구를 커스터마이즈할 수 있습니다.

Claude는 댓글을 포함한 전체 PR 컨텍스트를 볼 수 있습니다.

### 질문하기

PR 또는 이슈에 댓글을 추가하세요:

```
@claude What does this function do and how could we improve it?
```

Claude가 코드를 분석하고 개선 제안과 함께 상세한 설명을 제공합니다.

### 수정 요청

Claude에게 특정 변경 사항을 구현하도록 요청하세요:

```
@claude Can you add error handling to this function?
```

### 코드 리뷰

철저한 리뷰를 받으세요:

```
@claude Please review this PR and suggest improvements
```

Claude가 변경 사항을 분석하고 피드백을 제공합니다.

### 스크린샷으로 버그 수정

버그 스크린샷을 업로드하고 Claude에게 수정을 요청하세요:

```
@claude Here's a screenshot of a bug I'm seeing [upload screenshot]. Can you fix it?
```

Claude는 이미지를 보고 분석할 수 있어 시각적 버그나 UI 문제를 쉽게 수정할 수 있습니다.
