# 실험적 기능

**참고:** 실험적 기능은 불안정한 것으로 간주되며, 프로덕션 환경에서의 사용은 지원되지 않습니다. 이러한 기능은 언제든지 변경되거나 제거될 수 있습니다.

## 자동 모드 감지

이 액션은 워크플로우 컨텍스트에 따라 적절한 실행 모드를 지능적으로 감지하므로, 수동으로 모드를 설정할 필요가 없습니다.

### 대화형 모드 (Tag Mode)

명시적인 `prompt` 없이 Claude가 @멘션, 이슈 할당 또는 라벨을 감지하면 활성화됩니다.

- **트리거**: 댓글에서의 `@claude` 멘션, claude 사용자에게 이슈 할당, 라벨 적용
- **기능**: 진행 상황 체크박스가 포함된 추적 댓글 생성, 전체 구현 기능
- **사용 사례**: 대화형 코드 지원, 질의응답 및 구현 요청

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    # prompt 불필요 - @claude 멘션에 자동으로 응답합니다
```

### 자동화 모드 (Agent Mode)

`prompt` 입력을 제공하면 자동으로 활성화됩니다.

- **트리거**: `prompt` 입력이 제공되었을 때 모든 GitHub 이벤트
- **기능**: `@claude` 멘션 없이 직접 실행, 자동화에 최적화
- **사용 사례**: 자동 PR 리뷰, 예약된 작업, 워크플로우 자동화

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: |
      Check for outdated dependencies and create an issue if any are found.
    # prompt가 제공되면 자동으로 agent mode로 실행됩니다
```

### 동작 원리

이 액션은 다음 로직을 사용하여 모드를 결정합니다:

1. **`prompt`가 제공된 경우** → 자동화를 위해 **agent mode**로 실행됩니다
2. **`prompt`가 없지만 @claude가 멘션된 경우** → 상호작용을 위해 **tag mode**로 실행됩니다
3. **둘 다 해당하지 않는 경우** → 아무 동작도 수행하지 않습니다

이 자동 감지 기능을 통해 서로 다른 모드를 이해하거나 설정할 필요 없이, 워크플로우를 더 간단하고 직관적으로 구성할 수 있습니다.

### 고급 모드 제어

특수한 사용 사례의 경우, `claude_args`를 사용하여 동작을 세밀하게 조정할 수 있습니다:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: "Review this PR"
    claude_args: |
      --max-turns 20
      --system-prompt "You are a code review specialist"
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```
