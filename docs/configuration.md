# 고급 설정

## 커스텀 MCP 설정 사용하기

`claude_args`의 `--mcp-config` 플래그를 사용하여 커스텀 MCP (Model Context Protocol) 서버를 추가하면 Claude의 기능을 확장할 수 있습니다. 이러한 서버는 기본 제공되는 GitHub MCP 서버와 병합됩니다.

### 기본 예제: Sequential Thinking 서버 추가

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --mcp-config '{"mcpServers": {"sequential-thinking": {"command": "npx", "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]}}}'
      --allowedTools mcp__sequential-thinking__sequentialthinking
    # ... 기타 입력값
```

### MCP 서버에 시크릿 전달하기

API 키나 토큰과 같은 민감한 정보가 필요한 MCP 서버의 경우, GitHub Secrets를 사용하여 설정 파일을 생성할 수 있습니다:

```yaml
- name: Create MCP Config
  run: |
    cat > /tmp/mcp-config.json << 'EOF'
    {
      "mcpServers": {
        "custom-api-server": {
          "command": "npx",
          "args": ["-y", "@example/api-server"],
          "env": {
            "API_KEY": "${{ secrets.CUSTOM_API_KEY }}",
            "BASE_URL": "https://api.example.com"
          }
        }
      }
    }
    EOF

- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --mcp-config /tmp/mcp-config.json
    # ... 기타 입력값
```

### uv를 사용한 Python MCP 서버

`uv`로 관리되는 Python 기반 MCP 서버의 경우, 서버가 위치한 디렉터리를 지정해야 합니다:

```yaml
- name: Create MCP Config for Python Server
  run: |
    cat > /tmp/mcp-config.json << 'EOF'
    {
      "mcpServers": {
        "my-python-server": {
          "type": "stdio",
          "command": "uv",
          "args": [
            "--directory",
            "${{ github.workspace }}/path/to/server/",
            "run",
            "server_file.py"
          ]
        }
      }
    }
    EOF

- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --mcp-config /tmp/mcp-config.json
      --allowedTools my-python-server__<tool_name>  # <tool_name>을 서버의 도구 이름으로 교체하세요
    # ... 기타 입력값
```

예를 들어, Python MCP 서버가 `mcp_servers/weather.py`에 있는 경우 다음과 같이 사용합니다:

```yaml
"args":
  ["--directory", "${{ github.workspace }}/mcp_servers/", "run", "weather.py"]
```

### 다중 MCP 서버

여러 `--mcp-config` 플래그를 사용하여 다수의 MCP 서버를 추가할 수 있습니다:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --mcp-config /tmp/config1.json
      --mcp-config /tmp/config2.json
      --mcp-config '{"mcpServers": {"inline-server": {"command": "npx", "args": ["@example/server"]}}}'
    # ... 기타 입력값
```

**중요 사항**:

- API 키, 토큰, 비밀번호 등 민감한 값에는 반드시 GitHub Secrets (`${{ secrets.SECRET_NAME }}`)를 사용하세요. 워크플로 파일에 시크릿을 직접 하드코딩하지 마세요.
- 커스텀 서버가 기본 제공 서버와 동일한 이름을 가진 경우, 커스텀 서버가 기본 서버를 덮어씁니다.
- `claude_args`는 여러 `--mcp-config` 플래그를 지원하며, 이들은 함께 병합됩니다.

## CI/CD 통합을 위한 추가 권한

`additional_permissions` 입력값을 사용하면 필요한 권한을 부여하여 Claude가 GitHub Actions 워크플로 정보에 접근할 수 있습니다. 이는 CI/CD 실패 분석 및 워크플로 문제 디버깅에 특히 유용합니다.

### GitHub Actions 접근 활성화

Claude가 워크플로 실행 결과, 작업 로그, CI 상태를 확인할 수 있도록 하려면:

1. **GitHub 토큰에 필요한 권한을 부여하세요**:

   - 기본 `GITHUB_TOKEN`을 사용하는 경우, 워크플로에 `actions: read` 권한을 추가하세요:

   ```yaml
   permissions:
     contents: write
     pull-requests: write
     issues: write
     actions: read # 이 줄을 추가하세요
   ```

2. **추가 권한으로 액션을 설정하세요**:

   ```yaml
   - uses: anthropics/claude-code-action@v1
     with:
       anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
       additional_permissions: |
         actions: read
       # ... 기타 입력값
   ```

3. **Claude가 자동으로 CI/CD 도구에 접근할 수 있게 됩니다**:
   `actions: read`를 활성화하면 Claude는 다음 MCP 도구를 사용할 수 있습니다:
   - `mcp__github_ci__get_ci_status` - 워크플로 실행 상태 확인
   - `mcp__github_ci__get_workflow_run_details` - 상세 워크플로 정보 조회
   - `mcp__github_ci__download_job_log` - 작업 로그 다운로드 및 분석

### 예제: 실패한 CI 실행 디버깅

```yaml
name: Claude CI Helper
on:
  issue_comment:
    types: [created]

permissions:
  contents: write
  pull-requests: write
  issues: write
  actions: read # CI 접근에 필요

jobs:
  claude-ci-helper:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          additional_permissions: |
            actions: read
          # 이제 Claude가 "@claude why did the CI fail?"에 응답할 수 있습니다
```

**참고 사항**:

- GitHub 토큰은 워크플로에서 해당 권한을 보유하고 있어야 합니다
- 권한이 누락된 경우, Claude가 경고하고 권한 추가를 제안합니다
- 기본 권한 외에 다음과 같은 추가 권한을 요청할 수 있습니다:
  - `actions: read`
  - `checks: read`
  - `discussions: read` 또는 `discussions: write`
  - `workflows: read` 또는 `workflows: write`
- 표준 권한(`contents: write`, `pull_requests: write`, `issues: write`)은 항상 포함되므로 별도로 지정할 필요가 없습니다

## 커스텀 환경 변수

`settings` 입력값을 사용하여 Claude Code 실행에 커스텀 환경 변수를 전달할 수 있습니다. 이는 특정 환경 변수가 필요한 CI/테스트 설정에 유용합니다:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    settings: |
      {
        "env": {
          "NODE_ENV": "test",
          "CI": "true",
          "DATABASE_URL": "postgres://test:test@localhost:5432/test_db"
        }
      }
    # ... 기타 입력값
```

이러한 환경 변수는 Claude Code 실행 중에 사용할 수 있으므로, 특정 환경 설정에 의존하는 테스트, 빌드 프로세스 또는 기타 명령을 실행할 수 있습니다.

## 대화 턴 수 제한

`claude_args` 입력값을 사용하여 작업 실행 중 Claude가 수행할 수 있는 대화 턴 수를 제한할 수 있습니다. 이 기능은 다음과 같은 경우에 유용합니다:

- 과도한 대화를 방지하여 비용 관리
- 자동화된 워크플로에 시간 제한 설정
- CI/CD 파이프라인에서 예측 가능한 동작 보장

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --max-turns 5  # 대화 턴을 5회로 제한
    # ... 기타 입력값
```

턴 제한에 도달하면 Claude는 실행을 정상적으로 종료합니다. 일반적인 작업을 완료하기에 충분하면서도 과도한 사용을 방지할 수 있는 값을 선택하세요.

## 커스텀 도구

기본적으로 Claude는 다음 도구에만 접근할 수 있습니다:

- 파일 작업 (파일 읽기, 커밋, 편집, 읽기 전용 git 명령)
- 댓글 관리 (댓글 생성/수정)
- 기본 GitHub 작업

Claude는 기본적으로 임의의 Bash 명령을 실행할 수 **없습니다**. Claude가 특정 명령(예: npm install, npm test)을 실행하도록 하려면 `claude_args` 설정을 통해 명시적으로 허용해야 합니다:

**참고**: 리포지토리 루트 디렉터리에 `.mcp.json` 파일이 있는 경우, Claude는 해당 파일에 정의된 MCP 서버 도구를 자동으로 감지하여 사용합니다. 그러나 이러한 도구도 명시적으로 허용해야 합니다.

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    claude_args: |
      --allowedTools "Bash(npm install),Bash(npm run test),Edit,Replace,NotebookEditCell"
      --disallowedTools "TaskOutput,KillTask"
    # ... 기타 입력값
```

**참고**: 기본 GitHub 도구는 항상 포함됩니다. `--allowedTools`를 사용하여 추가 도구(특정 Bash 명령 포함)를 허용하고, `--disallowedTools`를 사용하여 특정 도구의 사용을 차단할 수 있습니다.

## 커스텀 모델

`claude_args`를 사용하여 Claude 모델을 지정하세요:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    claude_args: |
      --model claude-4-0-sonnet-20250805
    # ... 기타 입력값
```

제공자별 모델 사용 시:

```yaml
# AWS Bedrock
- uses: anthropics/claude-code-action@v1
  with:
    use_bedrock: "true"
    claude_args: |
      --model anthropic.claude-4-0-sonnet-20250805-v1:0
    # ... 기타 입력값

# Google Vertex AI
- uses: anthropics/claude-code-action@v1
  with:
    use_vertex: "true"
    claude_args: |
      --model claude-4-0-sonnet@20250805
    # ... 기타 입력값
```

## Claude Code 설정

`settings` 입력값을 사용하여 모델 선택, 환경 변수, 권한, 훅 등의 동작을 커스터마이징할 수 있습니다. 설정은 JSON 문자열 또는 설정 파일 경로로 제공할 수 있습니다.

### 옵션 1: 설정 파일

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    settings: "path/to/settings.json"
    # ... 기타 입력값
```

### 옵션 2: 인라인 설정

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    settings: |
      {
        "model": "claude-opus-4-1-20250805",
        "env": {
          "DEBUG": "true",
          "API_URL": "https://api.example.com"
        },
        "permissions": {
          "allow": ["Bash", "Read"],
          "deny": ["WebFetch"]
        },
        "hooks": {
          "PreToolUse": [{
            "matcher": "Bash",
            "hooks": [{
              "type": "command",
              "command": "echo Running bash command..."
            }]
          }]
        }
      }
    # ... 기타 입력값
```

설정은 다음을 포함한 모든 Claude Code 설정 옵션을 지원합니다:

- `model`: 기본 모델 재정의
- `env`: 세션용 환경 변수
- `permissions`: 도구 사용 권한
- `hooks`: 도구 실행 전/후 훅
- 기타 다양한 옵션...

사용 가능한 설정과 설명의 전체 목록은 [Claude Code 설정 문서](https://docs.anthropic.com/en/docs/claude-code/settings)를 참고하세요.

**참고 사항**:

- `enableAllProjectMcpServers` 설정은 MCP 서버가 올바르게 작동하도록 이 액션에 의해 항상 `true`로 설정됩니다.
- `claude_args` 입력값은 Claude Code CLI 인수에 직접 접근할 수 있으며, 설정보다 우선합니다.
- 간단한 설정에는 `claude_args`를, 훅과 환경 변수가 포함된 복잡한 설정에는 `settings`를 사용하는 것을 권장합니다.

## 더 이상 사용되지 않는 입력값에서 마이그레이션

많은 개별 입력 파라미터가 `claude_args` 또는 `settings`로 통합되었습니다. 마이그레이션 방법은 다음과 같습니다:

| 이전 입력값            | 새로운 방식                                              |
| --------------------- | -------------------------------------------------------- |
| `allowed_tools`       | `claude_args: "--allowedTools Tool1,Tool2"` 사용         |
| `disallowed_tools`    | `claude_args: "--disallowedTools Tool1,Tool2"` 사용      |
| `max_turns`           | `claude_args: "--max-turns 10"` 사용                     |
| `model`               | `claude_args: "--model claude-4-0-sonnet-20250805"` 사용 |
| `claude_env`          | `settings`에서 `"env"` 객체 사용                          |
| `custom_instructions` | `claude_args: "--system-prompt 'Your instructions'"` 사용 |
| `mcp_config`          | `claude_args: "--mcp-config '{...}'"` 사용               |
| `direct_prompt`       | `prompt` 입력값 사용                                      |
| `override_prompt`     | GitHub 컨텍스트 변수와 함께 `prompt` 사용                  |

## 특수 환경을 위한 커스텀 실행 파일

Nix, 커스텀 컨테이너 설정 또는 기타 패키지 관리 시스템과 같이 기본 설치가 작동하지 않는 특수 환경에서는 직접 실행 파일을 제공할 수 있습니다:

### 커스텀 Claude Code 실행 파일

자동 설치된 버전 대신 직접 준비한 Claude Code 바이너리를 사용하려면 `path_to_claude_code_executable`을 사용하세요:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    path_to_claude_code_executable: "/path/to/custom/claude"
    # ... 기타 입력값
```

### 커스텀 Bun 실행 파일

기본 설치 대신 직접 준비한 Bun 런타임을 사용하려면 `path_to_bun_executable`을 사용하세요:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    path_to_bun_executable: "/path/to/custom/bun"
    # ... 기타 입력값
```

**중요**: 호환되지 않는 버전을 사용하면 액션이 실패할 수 있습니다. 커스텀 실행 파일이 액션의 요구 사항과 호환되는지 확인하세요.
