# 보안

## 접근 제어

- **저장소 접근**: 이 액션은 저장소에 쓰기 권한이 있는 사용자만 트리거할 수 있습니다
- **봇 사용자 제어**: 기본적으로 GitHub Apps 및 봇은 보안상의 이유로 이 액션을 트리거할 수 없습니다. `allowed_bots` 파라미터를 사용하여 특정 봇 또는 모든 봇을 허용할 수 있습니다
- **⚠️ 쓰기 권한이 없는 사용자 접근 (위험)**: `allowed_non_write_users` 파라미터를 사용하면 쓰기 권한 요구사항을 우회할 수 있습니다. **이는 상당한 보안 위험을 수반하며, 극도로 제한된 권한을 가진 워크플로우에서만 사용해야 합니다** (예: `issues: write` 권한만 가진 이슈 라벨링 워크플로우). 이 기능은:
  - `github_token`이 입력으로 제공된 경우에만 작동합니다 (GitHub App 인증으로는 사용할 수 없습니다)
  - 쉼표로 구분된 특정 사용자명 목록 또는 모든 사용자를 허용하는 `*`를 입력값으로 받습니다
  - 이 액션의 주요 보안 메커니즘을 우회하므로 **극도의 주의를 기울여 사용해야 합니다**
  - 워크플로우의 권한 범위에 의해 사용자 권한이 이미 제한되어 있는 자동화 워크플로우를 위해 설계되었습니다
- **토큰 권한**: GitHub 앱은 작동 중인 저장소에만 한정된 단기 토큰만 발급받습니다
- **교차 저장소 접근 불가**: 각 액션 호출은 트리거된 저장소로만 제한됩니다
- **제한된 범위**: 토큰은 다른 저장소에 접근하거나 설정된 권한을 초과하는 작업을 수행할 수 없습니다

## Pull Request 생성

기본 설정에서 **Claude는 `@claude` 멘션에 응답할 때 자동으로 Pull Request를 생성하지 않습니다**. 대신:

- Claude는 코드 변경사항을 새 브랜치에 커밋합니다
- Claude는 응답에 **GitHub PR 생성 페이지로의 링크**를 제공합니다
- **사용자가 직접 링크를 클릭하여 PR을 생성해야 합니다**, 이를 통해 코드가 병합 제안되기 전에 사람의 검토가 보장됩니다

이 설계는 사용자가 어떤 Pull Request가 생성되는지 완전히 통제하고, PR 워크플로우를 시작하기 전에 변경사항을 검토할 수 있도록 보장합니다.

## ⚠️ 프롬프트 인젝션 위험

**신뢰할 수 없는 콘텐츠에 Claude를 태그할 때 숨겨진 마크다운에 주의하세요.** 외부 기여자가 HTML 주석, 보이지 않는 문자, 숨겨진 속성 또는 기타 기법을 통해 숨겨진 지시사항을 포함할 수 있습니다. 이 액션은 HTML 주석, 보이지 않는 문자, 마크다운 이미지 대체 텍스트, 숨겨진 HTML 속성, HTML 엔티티를 제거하여 콘텐츠를 정화하지만, 새로운 우회 기법이 등장할 수 있습니다. 외부 기여자로부터 들어오는 모든 입력의 원본 콘텐츠를 Claude에게 처리를 허용하기 전에 검토하는 것을 권장합니다.

## GitHub App 권한

[Claude Code GitHub 앱](https://github.com/apps/claude)은 다음 권한을 요청합니다:

### 현재 사용 중인 권한

- **Contents** (읽기 및 쓰기): 저장소 파일 읽기 및 브랜치 생성용
- **Pull Requests** (읽기 및 쓰기): PR 데이터 읽기 및 Pull Request 생성/업데이트용
- **Issues** (읽기 및 쓰기): 이슈 데이터 읽기 및 이슈 댓글 업데이트용

### 향후 기능을 위한 권한

다음 권한은 요청되지만 아직 활발하게 사용되지 않습니다. 향후 릴리스에서 계획된 기능을 위해 사용될 예정입니다:

- **Discussions** (읽기 및 쓰기): GitHub Discussions와의 상호작용용
- **Actions** (읽기): 워크플로우 실행 데이터 및 로그 접근용
- **Checks** (읽기): 체크 실행 결과 읽기용
- **Workflows** (읽기 및 쓰기): GitHub Actions 워크플로우 트리거 및 관리용

## 커밋 서명

기본적으로 Claude가 만든 커밋은 서명되지 않습니다. 다음 두 가지 방법 중 하나를 사용하여 커밋 서명을 활성화할 수 있습니다:

### 옵션 1: GitHub API 커밋 서명 (use_commit_signing)

GitHub의 API를 사용하여 커밋을 생성하며, GitHub App에서 검증된 것으로 자동 서명됩니다:

```yaml
- uses: anthropics/claude-code-action@main
  with:
    use_commit_signing: true
```

가장 간단한 옵션이며 추가 설정이 필요하지 않습니다. 다만, git CLI 대신 GitHub API를 사용하기 때문에 리베이스, 체리픽, 대화형 히스토리 조작과 같은 복잡한 git 작업을 수행할 수 없습니다.

### 옵션 2: SSH 서명 키 (ssh_signing_key)

SSH 키를 사용하여 git CLI를 통해 커밋에 서명합니다. 서명된 커밋과 표준 git 작업(리베이스, 체리픽 등)이 모두 필요한 경우 이 옵션을 사용하세요:

```yaml
- uses: anthropics/claude-code-action@main
  with:
    ssh_signing_key: ${{ secrets.SSH_SIGNING_KEY }}
    bot_id: "YOUR_GITHUB_USER_ID"
    bot_name: "YOUR_GITHUB_USERNAME"
```

커밋은 서명 키를 소유한 GitHub 계정으로 검증 및 귀속됩니다.

**설정 단계:**

1. 서명용 SSH 키 쌍을 생성합니다:

   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/signing_key -N "" -C "commit signing key"
   ```

2. **공개 키**를 GitHub 계정에 추가합니다:

   - GitHub → Settings → SSH and GPG keys로 이동합니다
   - "New SSH key"를 클릭합니다
   - **Key type: Signing Key**를 선택합니다 (중요)
   - `~/.ssh/signing_key.pub`의 내용을 붙여넣습니다

3. **비밀 키**를 저장소 시크릿에 추가합니다:

   - 저장소 → Settings → Secrets and variables → Actions로 이동합니다
   - `SSH_SIGNING_KEY`라는 이름의 새 시크릿을 생성합니다
   - `~/.ssh/signing_key`의 내용을 붙여넣습니다

4. GitHub 사용자 ID를 확인합니다:

   ```bash
   gh api users/YOUR_USERNAME --jq '.id'
   ```

5. 서명 키를 추가한 계정과 일치하도록 워크플로우의 `bot_id`와 `bot_name`을 업데이트합니다.

**참고:** `ssh_signing_key`와 `use_commit_signing`이 모두 제공된 경우, `ssh_signing_key`가 우선 적용됩니다.

## ⚠️ 인증 보호

**중요: Anthropic API 키 또는 OAuth 토큰을 워크플로우 파일에 절대 하드코딩하지 마세요!**

무단 접근을 방지하기 위해 인증 자격 증명은 반드시 GitHub Secrets에 저장해야 합니다:

```yaml
# 올바른 방법 ✅
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
# 또는
claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}

# 절대 이렇게 하지 마세요 ❌
anthropic_api_key: "sk-ant-api03-..." # 노출되어 취약합니다!
claude_code_oauth_token: "oauth_token_..." # 노출되어 취약합니다!
```

## ⚠️ 전체 출력 보안 경고

`show_full_output` 옵션은 보안상의 이유로 **기본적으로 비활성화**되어 있습니다. 활성화하면 다음을 포함한 모든 Claude Code 메시지가 출력됩니다:

- 도구 실행의 전체 출력 (예: `ps`, `env`, 파일 읽기)
- 토큰이나 자격 증명이 포함될 수 있는 API 응답
- 시크릿이 포함될 수 있는 파일 내용
- 민감한 시스템 정보가 노출될 수 있는 명령 출력

**이러한 로그는 공개 저장소의 GitHub Actions에서 공개적으로 확인할 수 있습니다!**

### 디버그 모드에서의 자동 활성화

GitHub Actions 디버그 모드가 활성화된 경우(`ACTIONS_STEP_DEBUG` 시크릿이 `true`로 설정된 경우) 전체 출력이 **자동으로 활성화됩니다**. 디버깅에 도움이 되지만 동일한 보안 위험이 따릅니다.

### 전체 출력을 활성화해야 하는 경우

다음 조건에 해당하는 경우에만 `show_full_output: true` 또는 GitHub Actions 디버그 모드를 활성화하세요:

- 접근이 통제된 비공개 저장소에서 작업하는 경우
- 프로덕션이 아닌 환경에서 문제를 디버깅하는 경우
- 출력에 시크릿이 노출되지 않음을 확인한 경우
- 보안에 미치는 영향을 이해하고 있는 경우

### 권장 사항

디버깅 시에는 `show_full_output: false`(기본값)를 사용하고, 민감한 데이터를 노출하지 않으면서 오류 및 완료 상태와 같은 필수 정보만 표시하는 Claude Code의 정화된 출력에 의존하는 것을 권장합니다.
