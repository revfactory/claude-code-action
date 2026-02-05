# 자주 묻는 질문 (FAQ)

이 FAQ는 Claude Code GitHub Action을 사용할 때 자주 발생하는 질문과 주의사항을 다룹니다.

## 트리거 및 인증

### 자동화된 워크플로에서 @claude를 태그해도 작동하지 않는 이유는 무엇인가요?

`github-actions` 사용자는 후속 GitHub Actions 워크플로를 트리거할 수 없습니다. 이는 무한 루프를 방지하기 위한 GitHub의 보안 기능입니다. 이 문제를 해결하려면 일반 사용자로 동작하는 Personal Access Token(PAT)을 사용하거나, 별도의 앱 토큰을 사용해야 합니다. 워크플로에서 이슈나 PR에 댓글을 작성할 때, 워크플로에서 생성된 `GITHUB_TOKEN` 대신 PAT를 사용하세요.

### Claude가 트리거 권한이 없다고 표시하는 이유는 무엇인가요?

저장소에 **쓰기 권한**이 있는 사용자만 Claude를 트리거할 수 있습니다. 이는 무단 사용을 방지하기 위한 보안 기능입니다. 댓글을 작성하는 사용자가 저장소에 대해 최소한 쓰기 접근 권한을 가지고 있는지 확인하세요.

### 저장소의 이슈에 @claude를 담당자로 지정할 수 없는 이유는 무엇인가요?

공개 저장소에서는 문제 없이 Claude를 담당자로 지정할 수 있습니다. 비공개 조직 저장소의 경우, 자신의 조직에 속한 사용자만 담당자로 지정할 수 있으며 Claude는 조직에 속하지 않습니다. 이 경우에는 커스텀 사용자를 별도로 만들어야 합니다.

### OIDC 인증 오류가 발생하는 이유는 무엇인가요?

기본 GitHub App 인증을 사용하는 경우, 워크플로에 `id-token: write` 권한을 추가해야 합니다:

```yaml
permissions:
  contents: read
  id-token: write # OIDC 인증에 필요합니다
```

OIDC 토큰은 Claude GitHub 앱이 작동하는 데 필요합니다. GitHub 앱을 사용하지 않으려면 대신 `github_token` 입력을 액션에 제공하여 Claude가 해당 토큰으로 동작하도록 할 수 있습니다. 자세한 내용은 [Claude Code 권한 문서][perms]를 참조하세요.

### '403 Resource not accessible by integration' 오류가 발생하는 이유는 무엇인가요?

이 오류는 액션이 GitHub App 설치 토큰을 사용하여 인증된 사용자 정보를 가져오려고 할 때 발생합니다. GitHub App 토큰은 제한된 접근 권한을 가지며 `/user` 엔드포인트에 접근할 수 없어 이 403 오류가 발생합니다.

**해결 방법**: 현재 액션에는 Claude의 봇 자격 증명으로 기본값이 설정된 `bot_id`와 `bot_name` 입력이 포함되어 있습니다. 이를 통해 API에서 사용자 정보를 가져올 필요가 없습니다.

기본 claude[bot]의 경우:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    # bot_id와 bot_name은 적절한 기본값이 설정되어 있으므로 지정할 필요가 없습니다
```

커스텀 봇의 경우 두 값을 모두 지정하세요:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    bot_id: "12345678" # 봇의 GitHub 사용자 ID
    bot_name: "my-bot" # 봇의 사용자 이름
```

이 문제는 일반적으로 에이전트/자동화 모드 워크플로에서만 발생합니다. 대화형 워크플로(@claude 멘션 사용)에서는 댓글 작성자의 정보를 사용하므로 이 문제가 발생하지 않습니다.

## Claude의 기능 및 제한 사항

### Claude가 워크플로 파일을 수정하지 않는 이유는 무엇인가요?

Claude용 GitHub App은 보안상의 이유로 워크플로 쓰기 접근 권한이 없습니다. 이는 Claude가 의도하지 않은 결과를 초래할 수 있는 CI/CD 설정을 수정하는 것을 방지합니다. 이 부분은 향후 재검토될 수 있습니다.

### Claude가 브랜치를 리베이스하지 않는 이유는 무엇인가요?

기본적으로 Claude는 브랜치에 대해 비파괴적인 변경만을 위해 커밋 도구를 사용합니다. Claude는 다음과 같이 설정되어 있습니다:

- 호출된 브랜치(자체 브랜치 또는 PR 브랜치) 이외의 브랜치에 절대 푸시하지 않습니다
- 강제 푸시나 파괴적인 작업을 절대 수행하지 않습니다

필요한 경우 `claude_args` 입력을 통해 추가 도구를 허용할 수 있습니다:

```yaml
claude_args: |
  --allowedTools "Bash(git rebase:*)"  # 주의하여 사용하세요
```

### Claude가 Pull Request를 생성하지 않는 이유는 무엇인가요?

Claude는 기본적으로 PR을 생성하지 않습니다. 대신 브랜치에 커밋을 푸시하고 미리 채워진 PR 제출 페이지 링크를 제공합니다. 이 방식은 저장소의 브랜치 보호 규칙이 여전히 준수되도록 하며, PR 생성에 대한 최종 제어권을 사용자에게 부여합니다.

### Claude가 GitHub Actions CI 결과를 볼 수 있나요?

네! Claude는 태그된 PR의 GitHub Actions 워크플로 실행, 작업 로그, 테스트 결과에 접근할 수 있습니다. 이를 활성화하려면:

1. 워크플로에 `actions: read` 권한을 추가하세요:

   ```yaml
   permissions:
     contents: write
     pull-requests: write
     issues: write
     actions: read
   ```

2. 추가 권한으로 액션을 설정하세요:
   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       additional_permissions: |
         actions: read
   ```

그러면 Claude가 CI 실패를 분석하고 워크플로 문제를 디버깅하는 데 도움을 줄 수 있습니다. 커밋 전에 로컬에서 테스트를 실행하려면 요청 시 Claude에게 직접 지시할 수 있습니다.

### Claude가 새 댓글을 작성하지 않고 하나의 댓글만 업데이트하는 이유는 무엇인가요?

Claude는 PR/이슈 토론이 복잡해지는 것을 방지하기 위해 단일 댓글을 업데이트하도록 설정되어 있습니다. 진행 상황 업데이트와 최종 결과를 포함한 Claude의 모든 응답은 작업 진행 상황을 보여주는 체크박스와 함께 동일한 댓글에 표시됩니다.

## 브랜치 및 커밋 동작

### 닫힌 PR에 댓글을 달 때 Claude가 새 브랜치를 생성하는 이유는 무엇인가요?

Claude의 브랜치 동작은 상황에 따라 달라집니다:

- **열린 PR**: 기존 PR 브랜치에 직접 푸시합니다
- **닫힌/병합된 PR**: 새 브랜치를 생성합니다 (닫힌 PR 브랜치에는 푸시할 수 없습니다)
- **이슈**: 항상 타임스탬프가 포함된 새 브랜치를 생성합니다

### 커밋이 얕거나 히스토리가 누락되는 이유는 무엇인가요?

성능을 위해 Claude는 얕은 클론을 사용합니다:

- PR: `--depth=20` (최근 20개 커밋)
- 새 브랜치: `--depth=1` (단일 커밋)

전체 히스토리가 필요한 경우, Claude를 호출하기 전에 `actions/checkout` 단계에서 워크플로를 설정할 수 있습니다.

```
- uses: actions/checkout@v6
  depth: 0 # 전체 저장소 히스토리를 가져옵니다
```

## 설정 및 도구

### 자동 모드 감지는 어떻게 작동하나요?

액션은 대화형 모드와 자동화 모드 중 어떤 것으로 실행할지 지능적으로 감지합니다:

- **`prompt` 입력이 있는 경우**: 자동화 모드로 실행 - @claude 멘션을 기다리지 않고 즉시 실행합니다
- **`prompt` 입력이 없는 경우**: 대화형 모드로 실행 - 댓글에서 @claude 멘션을 기다립니다

이 자동 감지 기능으로 모드를 수동으로 설정할 필요가 없습니다.

예시:

```yaml
# 자동화 모드 - 자동으로 실행됩니다
prompt: "Review this PR for security vulnerabilities"
# 대화형 모드 - @claude 멘션을 기다립니다
# (prompt가 제공되지 않음)
```

### `direct_prompt`와 `custom_instructions`는 어떻게 되었나요?

**이 입력들은 v1.0에서 더 이상 사용되지 않습니다(deprecated):**

- **`direct_prompt`** → `prompt`를 대신 사용하세요
- **`custom_instructions`** → `claude_args`에 `--system-prompt`를 사용하세요

마이그레이션 예시:

```yaml
# 이전 (v0.x)
direct_prompt: "Review this PR"
custom_instructions: "Focus on security"

# 현재 (v1.0)
prompt: "Review this PR"
claude_args: |
  --system-prompt "Focus on security"
```

### Claude가 bash 명령어를 실행하지 않는 이유는 무엇인가요?

Bash 도구는 보안을 위해 **기본적으로 비활성화**되어 있습니다. `claude_args`를 사용하여 개별 bash 명령어를 활성화할 수 있습니다:

```yaml
claude_args: |
  --allowedTools "Bash(npm:*),Bash(git:*)"  # npm과 git 명령어만 허용합니다
```

### Claude가 여러 저장소에서 작업할 수 있나요?

아니요, Claude의 GitHub 앱 토큰은 현재 저장소에만 샌드박스 처리되어 있습니다. 다른 저장소에 푸시할 수 없습니다. 다만 공개 저장소는 읽을 수 있지만, 이를 위해서는 해당 도구에 대한 접근 권한을 설정해야 합니다.

### 댓글이 claude[bot]으로 게시되지 않는 이유는 무엇인가요?

액션이 내장 인증을 사용할 때 댓글은 claude[bot]으로 표시됩니다. 그러나 워크플로에서 `github_token`을 제공하면 액션이 해당 토큰의 인증을 사용하므로 댓글이 다른 사용자 이름으로 표시됩니다.

**해결 방법**: 커스텀 GitHub App을 사용하지 않는 한 워크플로 파일에서 `github_token`을 제거하세요.

**참고**: `use_sticky_comment` 기능은 claude[bot] 인증에서만 작동합니다. 커스텀 `github_token`을 사용하는 경우, 고정 댓글은 claude[bot] 사용자 이름을 기대하므로 올바르게 업데이트되지 않습니다.

## MCP 서버 및 확장 기능

### 기본적으로 사용 가능한 MCP 서버는 무엇인가요?

Claude Code Action은 두 개의 MCP 서버를 자동으로 설정합니다:

1. **GitHub MCP 서버**: GitHub API 작업용
2. **파일 작업 서버**: 고급 파일 조작용

다만, 이 서버의 도구들은 `claude_args`의 `--allowedTools`를 통해 명시적으로 허용해야 합니다.

## 문제 해결

### Claude가 무엇을 하고 있는지 어떻게 디버깅할 수 있나요?

Claude 실행에 대한 GitHub Action 로그에서 전체 실행 추적을 확인하세요.

### `@claude-mention`이나 `claude!`로 Claude를 트리거할 수 없는 이유는 무엇인가요?

트리거는 단어 경계를 사용하므로 `@claude`는 완전한 단어여야 합니다. `@claude-bot`, `@claude!`, `claude@mention` 같은 변형은 `trigger_phrase`를 커스터마이즈하지 않는 한 작동하지 않습니다.

### 특수 환경에서 커스텀 실행 파일을 어떻게 사용할 수 있나요?

Nix, NixOS 또는 커스텀 컨테이너 설정처럼 자체 실행 파일을 제공해야 하는 특수 환경의 경우:

**커스텀 Claude Code 실행 파일 사용:**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    path_to_claude_code_executable: "/path/to/custom/claude"
    # ... 기타 입력
```

**커스텀 Bun 실행 파일 사용:**

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    path_to_bun_executable: "/path/to/custom/bun"
    # ... 기타 입력
```

**주요 사용 사례:**

- 패키지가 다르게 관리되는 Nix/NixOS 환경
- 실행 파일이 사전 설치된 Docker 컨테이너
- 특정 버전 요구사항이 있는 커스텀 빌드 환경
- 특정 버전의 문제를 디버깅하는 경우

**중요 참고 사항:**

- 이전 버전의 Claude Code를 사용하면 액션이 최신 기능을 사용할 때 문제가 발생할 수 있습니다
- 호환되지 않는 Bun 버전을 사용하면 런타임 오류가 발생할 수 있습니다
- 커스텀 경로가 제공되면 액션은 자동 설치를 건너뜁니다
- GitHub Actions 환경에서 커스텀 실행 파일을 사용할 수 있는지 확인하세요

## 모범 사례

1. **워크플로 파일에 항상 권한을 명시적으로 지정하세요**
2. **API 키에는 GitHub Secrets를 사용하세요** - 절대 하드코딩하지 마세요
3. **도구 권한은 구체적으로 설정하세요** - `claude_args`를 통해 필요한 것만 활성화하세요
4. **중요한 PR에 사용하기 전에 별도의 브랜치에서 테스트하세요**
5. **Claude의 토큰 사용량을 모니터링하여** API 제한에 도달하지 않도록 하세요
6. **병합 전에 Claude의 변경 사항을 신중하게 검토하세요**

## 도움 받기

여기에서 다루지 않은 문제가 발생한 경우:

1. [GitHub Issues](https://github.com/anthropics/claude-code-action/issues)를 확인하세요
2. [예시 워크플로](https://github.com/anthropics/claude-code-action#examples)를 검토하세요

[perms]: https://docs.anthropic.com/en/docs/claude-code/settings#permissions
