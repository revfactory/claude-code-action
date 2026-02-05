![Claude Code Action이 댓글에 응답하는 모습](https://github.com/user-attachments/assets/1d60c2e9-82ed-4ee5-b749-f9e021c85f4d)

# Claude Code Action

GitHub PR 및 이슈에서 질문에 답변하고 코드 변경을 구현할 수 있는 범용 [Claude Code](https://claude.ai/code) 액션입니다. 이 액션은 워크플로우 컨텍스트에 따라 활성화 시점을 자동으로 감지합니다. @claude 멘션에 대한 응답, 이슈 할당 처리, 명시적 프롬프트를 통한 자동화 작업 실행 등 다양한 상황에서 동작합니다. Anthropic 직접 API, Amazon Bedrock, Google Vertex AI, Microsoft Foundry를 포함한 다양한 인증 방식을 지원합니다.

## 주요 기능

- 🎯 **지능형 모드 감지**: 워크플로우 컨텍스트에 따라 적절한 실행 모드를 자동으로 선택하므로 별도의 설정이 필요 없습니다
- 🤖 **대화형 코드 어시스턴트**: Claude가 코드, 아키텍처, 프로그래밍에 관한 질문에 답변합니다
- 🔍 **코드 리뷰**: PR 변경 사항을 분석하고 개선 사항을 제안합니다
- ✨ **코드 구현**: 간단한 수정, 리팩터링은 물론 새로운 기능까지 구현할 수 있습니다
- 💬 **PR/이슈 통합**: GitHub 댓글 및 PR 리뷰와 원활하게 연동됩니다
- 🛠️ **유연한 도구 접근**: GitHub API 및 파일 작업에 대한 접근이 가능합니다 (설정을 통해 추가 도구를 활성화할 수 있습니다)
- 📋 **진행 상황 추적**: 체크박스 형태의 시각적 진행 표시기가 Claude의 작업 완료에 따라 실시간으로 업데이트됩니다
- 📊 **구조화된 출력**: 검증된 JSON 결과를 제공하며, 복잡한 자동화를 위해 자동으로 GitHub Action 출력으로 변환됩니다
- 🏃 **자체 인프라에서 실행**: 액션이 사용자의 GitHub 러너에서 전적으로 실행됩니다 (Anthropic API 호출은 선택한 제공자로 전송됩니다)
- ⚙️ **간소화된 설정**: 통합된 `prompt` 및 `claude_args` 입력으로 Claude Code SDK에 맞춘 깔끔하고 강력한 설정을 제공합니다

## 📦 v0.x에서 업그레이드하시나요?

워크플로우를 v1.0으로 업데이트하는 단계별 안내는 **[마이그레이션 가이드](./docs/migration-guide.md)**를 참고하세요. 새 버전은 대부분의 기존 설정과의 호환성을 유지하면서 설정을 간소화합니다.

## 빠른 시작

이 액션을 설정하는 가장 쉬운 방법은 터미널에서 [Claude Code](https://claude.ai/code)를 사용하는 것입니다. `claude`를 열고 `/install-github-app`을 실행하세요.

이 명령어가 GitHub 앱 설치 및 필요한 시크릿 설정 과정을 안내합니다.

**참고**:

- GitHub 앱을 설치하고 시크릿을 추가하려면 저장소 관리자 권한이 필요합니다
- 이 빠른 시작 방법은 Anthropic API 직접 사용자만 이용 가능합니다. AWS Bedrock, Google Vertex AI, Microsoft Foundry 설정은 [docs/cloud-providers.md](./docs/cloud-providers.md)를 참고하세요.

## 📚 솔루션 및 활용 사례

특정 자동화 패턴을 찾고 계신가요? **[솔루션 가이드](./docs/solutions.md)**에서 다음과 같은 완전한 동작 예제를 확인하세요:

- **🔍 자동 PR 코드 리뷰** - 리뷰 자동화 전체 구성
- **📂 경로별 리뷰** - 주요 파일 변경 시 트리거
- **👥 외부 기여자 리뷰** - 신규 기여자를 위한 특별 처리
- **📝 사용자 정의 리뷰 체크리스트** - 팀 표준 적용
- **🔄 정기 유지보수** - 자동화된 저장소 상태 점검
- **🏷️ 이슈 분류 및 라벨링** - 자동 분류
- **📖 문서 동기화** - 코드 변경에 맞춰 문서 자동 업데이트
- **🔒 보안 중심 리뷰** - OWASP 기반 보안 분석
- **📊 DIY 진행 상황 추적** - 자동화 모드에서 추적 댓글 생성

각 솔루션에는 완전한 동작 예제, 설정 세부 사항, 예상 결과가 포함되어 있습니다.

## 문서

- **[솔루션 가이드](./docs/solutions.md)** - **🎯 바로 사용 가능한 자동화 패턴**
- **[마이그레이션 가이드](./docs/migration-guide.md)** - **⭐ v0.x에서 v1.0으로 업그레이드**
- [설정 가이드](./docs/setup.md) - 수동 설정, 사용자 정의 GitHub 앱, 보안 모범 사례
- [사용 가이드](./docs/usage.md) - 기본 사용법, 워크플로우 설정, 입력 매개변수
- [사용자 정의 자동화](./docs/custom-automations.md) - 자동화 워크플로우 및 사용자 정의 프롬프트 예제
- [설정](./docs/configuration.md) - MCP 서버, 권한, 환경 변수, 고급 설정
- [실험적 기능](./docs/experimental.md) - 실행 모드 및 네트워크 제한
- [클라우드 제공자](./docs/cloud-providers.md) - AWS Bedrock, Google Vertex AI, Microsoft Foundry 설정
- [기능 및 제한 사항](./docs/capabilities-and-limitations.md) - Claude가 할 수 있는 것과 할 수 없는 것
- [보안](./docs/security.md) - 접근 제어, 권한, 커밋 서명
- [FAQ](./docs/faq.md) - 자주 묻는 질문 및 문제 해결

## 📚 FAQ

문제가 있거나 질문이 있으신가요? [자주 묻는 질문](./docs/faq.md)에서 일반적인 문제에 대한 해결 방법과 Claude의 기능 및 제한 사항에 대한 자세한 설명을 확인하세요.

## 라이선스

이 프로젝트는 MIT 라이선스에 따라 배포됩니다. 자세한 내용은 LICENSE 파일을 참고하세요.
