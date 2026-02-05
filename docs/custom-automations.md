# 커스텀 자동화

이 예제들은 GitHub 이벤트를 기반으로 Claude가 자동으로 동작하도록 설정하는 방법을 보여줍니다. `prompt` 입력을 제공하면 액션이 수동 @멘션 없이 자동으로 에이전트 모드로 실행됩니다. `prompt`가 없으면 인터랙티브 모드로 실행되어 @claude 멘션에 응답합니다.

## 모드 감지 및 추적 코멘트

액션은 설정에 따라 사용할 모드를 자동으로 감지합니다:

- **인터랙티브 모드** (`prompt` 입력 없음): @claude 멘션에 응답하고, 진행 상황 표시가 포함된 추적 코멘트를 생성합니다
- **자동화 모드** (`prompt` 입력 있음): 즉시 실행되며, **추적 코멘트를 생성하지 않습니다**

> **참고**: v1에서 자동화 모드는 자동화된 워크플로우의 노이즈를 줄이기 위해 기본적으로 추적 코멘트를 생성하지 않습니다. 진행 상황 추적이 필요한 경우 `track_progress: true` 입력 파라미터를 사용하세요.

## 지원되는 GitHub 이벤트

이 액션은 다음 GitHub 이벤트를 지원합니다 ([GitHub 이벤트 트리거 자세히 알아보기](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows)):

- `pull_request` 또는 `pull_request_target` - PR이 열리거나 동기화될 때
- `issue_comment` - 이슈 또는 PR에 코멘트가 작성될 때
- `pull_request_comment` - PR diff에 코멘트가 작성될 때
- `issues` - 이슈가 열리거나 할당될 때
- `pull_request_review` - PR 리뷰가 제출될 때
- `pull_request_review_comment` - PR 리뷰에 코멘트가 작성될 때
- `repository_dispatch` - API를 통해 트리거되는 커스텀 이벤트
- `workflow_dispatch` - 수동 워크플로우 트리거 (지원 예정)

## 자동 문서 업데이트

특정 파일이 변경되면 자동으로 문서를 업데이트합니다 ([`examples/claude-pr-path-specific.yml`](../examples/claude-pr-path-specific.yml) 참조):

```yaml
on:
  pull_request:
    paths:
      - "src/api/**/*.ts"

steps:
  - uses: anthropics/claude-code-action@v1
    with:
      prompt: |
        Update the API documentation in README.md to reflect
        the changes made to the API endpoints in this PR.
      anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

API 파일이 수정되면 액션이 자동으로 `prompt`가 제공된 것을 감지하고 에이전트 모드로 실행됩니다. Claude는 최신 엔드포인트 문서로 README를 업데이트하고 변경 사항을 PR에 다시 푸시하여, 문서가 코드와 항상 동기화된 상태를 유지합니다.

## 작성자별 코드 리뷰

특정 작성자 또는 외부 기여자의 PR을 자동으로 리뷰합니다 ([`examples/claude-review-from-author.yml`](../examples/claude-review-from-author.yml) 참조):

```yaml
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review-by-author:
    if: |
      github.event.pull_request.user.login == 'developer1' ||
      github.event.pull_request.user.login == 'external-contributor'
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          prompt: |
            Please provide a thorough review of this pull request.
            Pay extra attention to coding standards, security practices,
            and test coverage since this is from an external contributor.
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

새로운 팀원, 외부 기여자, 또는 추가 가이드가 필요한 특정 개발자의 PR을 자동으로 리뷰하는 데 적합합니다. `prompt`가 제공되면 액션이 자동으로 에이전트 모드로 실행됩니다.

## 커스텀 프롬프트 템플릿

동적 자동화를 위해 GitHub 컨텍스트 변수와 함께 `prompt` 입력을 사용하세요:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: |
      Analyze PR #${{ github.event.pull_request.number }} in ${{ github.repository }} for security vulnerabilities.

      Focus on:
      - SQL injection risks
      - XSS vulnerabilities
      - Authentication bypasses
      - Exposed secrets or credentials

      Provide severity ratings (Critical/High/Medium/Low) for any issues found.
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

표준 GitHub Actions 구문을 사용하여 모든 GitHub 컨텍스트 변수에 접근할 수 있습니다:

- `${{ github.repository }}` - 리포지토리 이름
- `${{ github.event.pull_request.number }}` - PR 번호
- `${{ github.event.issue.number }}` - 이슈 번호
- `${{ github.event.pull_request.title }}` - PR 제목
- `${{ github.event.pull_request.body }}` - PR 설명
- `${{ github.event.comment.body }}` - 코멘트 내용
- `${{ github.actor }}` - 워크플로우를 트리거한 사용자
- `${{ github.base_ref }}` - PR의 베이스 브랜치
- `${{ github.head_ref }}` - PR의 헤드 브랜치

## claude_args를 활용한 고급 설정

Claude의 동작을 더 세밀하게 제어하려면 `claude_args` 입력을 사용하여 CLI 인수를 직접 전달하세요:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: "Review this PR for performance issues"
    claude_args: |
      --max-turns 15
      --model claude-4-0-sonnet-20250805
      --allowedTools Edit,Read,Write,Bash
      --system-prompt "You are a performance optimization expert. Focus on identifying bottlenecks and suggesting improvements."
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

간소화된 액션 인터페이스를 유지하면서도 Claude Code CLI의 모든 기능에 접근할 수 있습니다.
