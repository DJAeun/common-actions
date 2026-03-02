# 🚀 Central GitHub Actions (common-actions)

이 저장소는 조직(`skn-ai22` 등) 내 다른 프로젝트 저장소에서 공통으로 사용할 수 있는 **재사용 가능한 워크플로우(Reusable Workflows)** 를 모아둔 중앙 저장소입니다.

다른 저장소에서 아래 액션들을 손쉽게 호출하여 사용할 수 있으며, 중앙 저장소의 코드를 업데이트하는 것만으로 모든 연결된 저장소의 파이프라인 정책을 통일할 수 있습니다.

---

## 📖 사용 방법 가이드

### 1. 🤖 PR 자동 요약 및 리뷰 (AI)
ChatGPT 모델을 활용하여 PR 생성/수정 시 자동으로 내용을 요약하고 코드 리뷰를 남기는 액션입니다.

**호출 방법(`.github/workflows/ai-pr-review.yml`)**:
```yaml
name: AI PR 요약 및 리뷰

on:
  pull_request:
    types: [opened, synchronize] 

jobs:
  ai-review:
    # uses: <소유자>/<저장소명>/.github/workflows/<파일명>@<참조>
    uses: DJAeun/common-actions/.github/workflows/pr-ai-summary.yml@main
    secrets:
      # 대상 리포지토리(또는 조직) 설정의 Secrets에 등록된 OpenAI API Key를 전달
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

---

### 2. 🔍 PR 안전벨트 (Lint & Test)
Python 프로젝트 전용으로 문법(Flake8), 테스트(pytest) 및 포맷팅(Black)을 검토하는 필수 안전벨트 액션입니다.

**호출 방법(`.github/workflows/pr-lint-test.yml`)**:
```yaml
name: PR Lint and Test

on:
  pull_request:
    branches: [ "main", "develop" ]
  push:
    branches: [ "main", "develop" ]

jobs:
  lint-and-test:
    uses: DJAeun/common-actions/.github/workflows/pr-lint-test.yml@main
    with:
      # (옵션) 파이썬 버전 지정 (디폴트: "3.10")
      python-version: "3.11"
```

---

### 3. 📋 Project Board 자동 동기화
새로운 이슈나 PR이 생성되면, 지정한 GitHub Project Board (Projects V2)로 항목을 자동으로 추가합니다.

**호출 방법(`.github/workflows/project-sync.yml`)**:
```yaml
name: Project Board Sync

on:
  issues:
    types: [opened]
  pull_request:
    types: [opened]

jobs:
  board-sync:
    uses: DJAeun/common-actions/.github/workflows/project-board-sync.yml@main
    with:
      # (필수) 동기화할 프로젝트 URL
      project-url: "https://github.com/orgs/skn-ai22-251029/projects/38"
    secrets:
      # (필수) Project V2 접근 등가 권한을 지닌 PAT 토큰
      ADD_TO_PROJECT_PAT: ${{ secrets.ADD_TO_PROJECT_PAT }}
```

---

### 4. 📦 스마트 릴리스 (Release Please)
PR 병합을 감지하여 커밋 컨벤션(Conventional Commits)에 따라 버전을 계산하고 CHANGELOG.md 생성 및 GitHub Release 배포를 수행합니다.

**호출 방법(`.github/workflows/release-please.yml`)**:
```yaml
name: Release Please

on:
  push:
    branches:
      - main 

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    uses: DJAeun/common-actions/.github/workflows/release-please.yml@main
    with:
      # (옵션) 릴리스 언어 유형 지정 (디폴트: "python")
      # "node", "rust", "java" 등 사용 가능
      release-type: "python"
```

---

### 5. 🔐 시크릿 키 유출 방지 (TruffleHog)
소스코드 내에 실수로 포함된 민감한 정보(API Secret, 개인키, 토큰 등)가 커밋/푸시되는 것을 막는 역할을 수행합니다.

**호출 방법(`.github/workflows/secret-scan.yml`)**:
```yaml
name: Secret Scanner

on:
  push:
    branches: [ "main", "develop" ]
  pull_request:
    branches: [ "main", "develop" ]

jobs:
  scan:
    uses: DJAeun/common-actions/.github/workflows/secret-scan.yml@main
```

---

### 💡 활용 팁
* 위 모든 YAML 파일을 새로운 리포지토리의 `.github/workflows/` 디렉터리에 복사해서 사용하기만 하면 연동이 완료됩니다!
* 필요할 경우, 본 중앙 저장소의 `uses`의 태그를 `@main` 대신 `@v1.0.0` 같은 특정 버전을 사용해 안전성을 높일 수 있습니다.
