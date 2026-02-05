# CLAUDE.md

## 명령어

```bash
bun test                # 테스트 실행
bun run typecheck       # TypeScript 타입 검사
bun run format          # Prettier로 포맷팅
bun run format:check    # 포맷팅 검사
```

## 프로젝트 개요

GitHub 이슈/PR에서 `@claude` 멘션에 응답하거나(태그 모드), `prompt` 입력을 통해 작업을 수행하는(에이전트 모드) GitHub Action입니다. 모드는 자동으로 감지됩니다: `prompt`가 제공되면 에이전트 모드, `@claude` 멘션이 포함된 댓글/이슈 이벤트로 트리거되면 태그 모드입니다. `src/modes/registry.ts`를 참고하세요.

## 실행 구조

단일 진입점: `src/entrypoints/run.ts`가 모든 것을 조율합니다 — 준비(인증, 권한, 트리거 확인, 브랜치/댓글 생성), Claude Code CLI 설치, `base-action/` 함수를 통한 Claude 실행(서브프로세스가 아닌 직접 임포트), 그리고 정리(추적 댓글 업데이트, 스텝 요약 작성). SSH 서명 정리와 토큰 해지는 `action.yml`에서 별도의 `always()` 스텝으로 처리됩니다.

`base-action/`은 `@anthropic-ai/claude-code-base-action`으로 단독 배포되기도 합니다. 공개 API를 깨뜨리지 마세요. 이 모듈은 액션 입력이 아닌 `INPUT_` 접두사가 붙은 환경 변수(`action.yml`에서 설정)에서 설정을 읽습니다.

## 핵심 개념

**인증 우선순위**: `github_token` 입력(사용자 제공) > GitHub App OIDC 토큰(기본값). `claude_code_oauth_token`과 `anthropic_api_key`는 GitHub용이 아니라 Claude API용입니다. 토큰 설정은 `src/github/token.ts`에 있습니다.

**모드 생명주기**: 모드는 `shouldTrigger()` → `prepare()` → `prepareContext()` → `getSystemPrompt()`를 구현합니다. `src/modes/registry.ts`의 레지스트리가 이벤트 타입과 입력에 따라 모드를 선택합니다. 새로운 모드를 추가하려면 `src/modes/types.ts`의 `Mode` 타입을 구현하고 등록하세요.

**프롬프트 구성**: `src/prepare/`가 GitHub 데이터를 가져오고(`src/github/data/fetcher.ts`), 마크다운으로 포맷팅한 후(`src/github/data/formatter.ts`), 임시 파일에 기록하여 프롬프트를 빌드합니다. 프롬프트에는 이슈/PR 본문, 댓글, 디프, CI 상태가 포함됩니다. 이것이 액션에서 가장 중요한 부분입니다 — Claude가 실제로 보는 내용이기 때문입니다.

## 주의 사항

- **엄격한 TypeScript**: `noUnusedLocals`와 `noUnusedParameters`가 활성화되어 있습니다. 미사용 변수가 있으면 타입 검사가 실패합니다.
- **GitHub 컨텍스트의 구별된 유니온 타입**: `GitHubContext`는 유니온 타입입니다 — `context.issue`나 `context.pullRequest` 같은 엔티티별 필드에 접근하기 전에 반드시 `isEntityContext(context)`를 호출하세요.
- **토큰 생명주기가 중요합니다**: GitHub App 토큰은 초기에 획득되며 `action.yml`의 별도 `always()` 스텝에서 해지됩니다. 토큰 해지를 `run.ts`로 옮기면 프로세스 크래시 시 실행되지 않습니다. SSH 서명 정리도 마찬가지입니다.
- **오류 단계 구분**: `run.ts`의 catch 블록은 `prepareCompleted`를 사용하여 준비 실패와 실행 실패를 구분합니다. 추적 댓글은 각각에 대해 다른 메시지를 표시합니다.
- **`action.yml` 출력이 스텝 ID를 참조합니다**: `execution_file`, `branch_name`, `github_token` 같은 출력은 `steps.run.outputs.*`를 참조합니다. 스텝 ID를 변경하면 출력 섹션도 함께 업데이트하세요.
- **통합 테스트**는 이 저장소가 아닌 별도의 저장소(`install-test`)에서 수행됩니다. 이 저장소의 테스트는 단위 테스트입니다.

## 코드 규칙

- 런타임은 Node가 아닌 Bun입니다. `jest`가 아닌 `bun test`를 사용하세요.
- `moduleResolution: "bundler"` — 임포트에 `.js` 확장자가 필요 없습니다.
- GitHub API 호출 시 재시도 로직을 사용해야 합니다 (`src/utils/retry.ts`).
- MCP 서버는 런타임에 `~/.claude/mcp/github-{type}-server/`에 자동 설치됩니다.
