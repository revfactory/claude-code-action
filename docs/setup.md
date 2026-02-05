# 설정 가이드

## 수동 설정 (Direct API)

**요구 사항**: 이 단계를 완료하려면 저장소 관리자여야 합니다.

1. Claude GitHub 앱을 저장소에 설치하세요: https://github.com/apps/claude
2. 저장소 시크릿에 인증 정보를 추가하세요 ([GitHub Actions에서 시크릿을 사용하는 방법 알아보기](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)):
   - API 키 인증을 위한 `ANTHROPIC_API_KEY`
   - 또는 OAuth 토큰 인증을 위한 `CLAUDE_CODE_OAUTH_TOKEN` (Pro 및 Max 사용자는 로컬에서 `claude setup-token`을 실행하여 생성할 수 있습니다)
3. [`examples/claude.yml`](../examples/claude.yml)의 워크플로우 파일을 저장소의 `.github/workflows/` 디렉토리에 복사하세요

## 커스텀 GitHub App 사용

공식 Claude 앱을 설치하지 않으려면, 이 Action과 함께 사용할 자체 GitHub App을 만들 수 있습니다. 이를 통해 권한과 접근을 완전히 제어할 수 있습니다.

**커스텀 GitHub App을 사용하면 좋은 경우:**

- 공식 앱보다 더 제한적인 권한이 필요한 경우
- 조직 정책상 서드파티 앱 설치가 제한되는 경우
- AWS Bedrock 또는 Google Vertex AI를 사용하는 경우

### 옵션 1: 앱 매니페스트를 활용한 빠른 설정 (권장)

커스텀 GitHub App을 만드는 가장 빠른 방법은 사전 구성된 매니페스트를 사용하는 것입니다. 한 번의 클릭으로 모든 권한이 올바르게 설정됩니다.

**단계:**

1. **앱 생성:**

   **[빠른 설정 도구 다운로드](./create-app.html)** (마우스 오른쪽 클릭 → "다른 이름으로 링크 저장" 또는 "링크된 파일 다운로드")

   다운로드 후, 웹 브라우저에서 `create-app.html`을 여세요:

   - **개인 계정의 경우:** "Create App for Personal Account" 버튼을 클릭하세요
   - **조직의 경우:** 조직 이름을 입력하고 "Create App for Organization" 버튼을 클릭하세요

   이 도구는 필요한 모든 권한을 자동으로 구성하고 매니페스트를 제출합니다.

   또는 매니페스트 파일을 직접 사용할 수도 있습니다:

   - 이 저장소의 [`github-app-manifest.json`](../github-app-manifest.json) 파일을 사용하세요
   - https://github.com/settings/apps/new (개인용) 또는 조직의 앱 설정 페이지를 방문하세요
   - "Create from manifest" 옵션을 찾아 JSON 내용을 붙여넣으세요

2. **생성 과정 완료:**

   - GitHub에서 앱 구성 미리보기가 표시됩니다
   - 앱 이름을 확인하세요 (원하는 대로 변경할 수 있습니다)
   - "Create GitHub App"을 클릭하세요
   - 필요한 모든 권한이 자동으로 구성된 상태로 앱이 생성됩니다

3. **프라이빗 키 생성 및 다운로드:**

   - 앱 생성 후 앱 설정 페이지로 이동됩니다
   - "Private keys"까지 스크롤하세요
   - "Generate a private key"를 클릭하세요
   - `.pem` 파일을 다운로드하세요 (안전하게 보관하세요!)

4. **설치 진행** - 아래 수동 설정의 3단계로 건너뛰어 앱을 설치하고 워크플로우를 구성하세요.

### 옵션 2: 수동 설정

앱을 수동으로 구성하거나 커스텀 권한이 필요한 경우:

1. **새 GitHub App 생성:**

   - https://github.com/settings/apps (개인 앱의 경우) 또는 조직 설정으로 이동하세요
   - "New GitHub App"을 클릭하세요
   - 다음의 최소 권한으로 앱을 구성하세요:
     - **Repository permissions:**
       - Contents: Read &amp; Write
       - Issues: Read &amp; Write
       - Pull requests: Read &amp; Write
     - **Account permissions:** 필요 없음
   - "Where can this GitHub App be installed?"을 원하는 대로 설정하세요
   - 앱을 생성하세요

2. **프라이빗 키 생성 및 다운로드:**

   - 앱 생성 후 "Private keys"까지 스크롤하세요
   - "Generate a private key"를 클릭하세요
   - `.pem` 파일을 다운로드하세요 (안전하게 보관하세요!)

3. **저장소에 앱 설치:**

   - 앱의 설정 페이지로 이동하세요
   - "Install App"을 클릭하세요
   - Claude를 사용할 저장소를 선택하세요

4. **저장소 시크릿에 앱 자격 증명 추가:**

   - 저장소의 Settings → Secrets and variables → Actions로 이동하세요
   - 다음 시크릿을 추가하세요:
     - `APP_ID`: GitHub App의 ID (앱 설정에서 확인할 수 있습니다)
     - `APP_PRIVATE_KEY`: 다운로드한 `.pem` 파일의 내용

5. **커스텀 앱을 사용하도록 워크플로우 업데이트:**

   ```yaml
   name: Claude with Custom App
   on:
     issue_comment:
       types: [created]
     # ... other triggers

   jobs:
     claude-response:
       runs-on: ubuntu-latest
       steps:
         # Generate a token from your custom app
         - name: Generate GitHub App token
           id: app-token
           uses: actions/create-github-app-token@v1
           with:
             app-id: ${{ secrets.APP_ID }}
             private-key: ${{ secrets.APP_PRIVATE_KEY }}

         # Use Claude with your custom app's token
         - uses: anthropics/claude-code-action@v1
           with:
             anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
             github_token: ${{ steps.app-token.outputs.token }}
             # ... other configuration
   ```

**중요 사항:**

- 커스텀 앱은 Issues, Pull Requests, Contents에 대한 읽기/쓰기 권한이 있어야 합니다
- 앱의 토큰은 구성한 권한만 가지며, 그 이상의 권한은 부여되지 않습니다

GitHub App 생성에 대한 자세한 내용은 [GitHub 문서](https://docs.github.com/en/apps/creating-github-apps)를 참고하세요.

## 보안 모범 사례

**중요: API 키를 저장소에 직접 커밋하지 마세요! 항상 GitHub Actions 시크릿을 사용하세요.**

Anthropic API 키를 안전하게 사용하려면:

1. API 키를 저장소 시크릿으로 추가하세요:

   - 저장소의 Settings로 이동하세요
   - "Secrets and variables" → "Actions"로 이동하세요
   - "New repository secret"을 클릭하세요
   - 이름을 `ANTHROPIC_API_KEY`로 지정하세요
   - API 키를 값으로 붙여넣으세요

2. 워크플로우에서 시크릿을 참조하세요:
   ```yaml
   anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
   ```

**절대 이렇게 하지 마세요:**

```yaml
# WRONG - API 키가 노출됩니다
anthropic_api_key: "sk-ant-..."
```

**항상 이렇게 하세요:**

```yaml
# CORRECT - GitHub 시크릿을 사용합니다
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

이 원칙은 API 키, 액세스 토큰, 자격 증명을 포함한 모든 민감한 값에 적용됩니다.
또한 가능하면 항상 단기 토큰을 사용하는 것을 권장합니다.

## GitHub 시크릿 설정

1. 저장소의 Settings로 이동하세요
2. "Secrets and variables" → "Actions"를 클릭하세요
3. "New repository secret"을 클릭하세요
4. 인증 방식을 선택하세요:
   - API 키: 이름: `ANTHROPIC_API_KEY`, 값: Anthropic API 키 (`sk-ant-`로 시작)
   - OAuth 토큰: 이름: `CLAUDE_CODE_OAUTH_TOKEN`, 값: Claude Code OAuth 토큰 (Pro 및 Max 사용자는 로컬에서 `claude setup-token`을 실행하여 생성할 수 있습니다)
5. "Add secret"을 클릭하세요

### 인증 모범 사례

1. 워크플로우에서는 항상 `${{ secrets.ANTHROPIC_API_KEY }}` 또는 `${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}`을 사용하세요
2. API 키나 토큰을 버전 관리 시스템에 커밋하지 마세요
3. API 키와 토큰을 정기적으로 교체하세요
4. 조직 전체에서 사용할 경우 환경 시크릿을 활용하세요
5. API 키나 토큰을 PR이나 이슈에 공유하지 마세요
6. 키가 포함될 수 있는 워크플로우 변수를 로깅하지 마세요
