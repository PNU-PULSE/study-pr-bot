# AtCoder-CF-Study-PR-Bot

![Language](https://img.shields.io/badge/Language-Kotlin%202.2.21-7F52FF?style=flat-square&logo=kotlin&logoColor=white)
![JDK](https://img.shields.io/badge/JDK-21-000000?style=flat-square&logo=openjdk&logoColor=white)
![Framework](https://img.shields.io/badge/Framework-Spring%20Boot%204.0.6-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![Build](https://img.shields.io/badge/Build-Gradle%209.4.1-02303A?style=flat-square&logo=gradle&logoColor=white)
![CI](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)
![API](https://img.shields.io/badge/API-GitHub%20REST-181717?style=flat-square&logo=github&logoColor=white)
![Webhook](https://img.shields.io/badge/API-Discord%20Webhook-5865F2?style=flat-square&logo=discord&logoColor=white)

GitHub Actions에서 주기적으로 실행되는 Kotlin + Spring Boot 배치 앱입니다. 저장소의 open pull request를 조회한 뒤 PR 제목과 변경 파일 경로를 검사하고, 규칙을 만족하면 squash merge합니다. 규칙을 위반하면 PR에 실패 사유 댓글을 남깁니다.

## 기술 스택

| 구분 | 사용 |
| --- | --- |
| 언어 | ![Kotlin](https://img.shields.io/badge/Kotlin-2.2.21-7F52FF?style=flat-square&logo=kotlin&logoColor=white) ![JDK](https://img.shields.io/badge/JDK-21-000000?style=flat-square&logo=openjdk&logoColor=white) |
| 프레임워크 | ![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.0.6-6DB33F?style=flat-square&logo=springboot&logoColor=white) |
| 기술 스택 | ![Gradle](https://img.shields.io/badge/Gradle-9.4.1-02303A?style=flat-square&logo=gradle&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-Scheduled%20Batch-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![Spring Web RestClient](https://img.shields.io/badge/Spring%20Web-RestClient-6DB33F?style=flat-square&logo=spring&logoColor=white) |
| 사용 API | ![GitHub REST API](https://img.shields.io/badge/GitHub%20REST%20API-PR%20%2F%20Issue%20Comment%20%2F%20Merge-181717?style=flat-square&logo=github&logoColor=white) ![Discord Webhook](https://img.shields.io/badge/Discord-Webhook-5865F2?style=flat-square&logo=discord&logoColor=white) |

## PR 제목 규칙

PR 제목은 다음 형식이어야 합니다.

```text
대회이름 / AtCoder핸들
```

`/` 양쪽 공백은 있어도 되고 없어도 됩니다. AtCoder 핸들은 `[A-Za-z0-9_-]+` 형식만 허용합니다.

예:

```text
ABC350 / tourist
ABC350/tourist
ABC350 / tourist_123
```

## 파일 경로 규칙

변경 파일은 PR 제목에서 파싱한 `대회이름`과 `AtCoder핸들`을 사용해 다음 경로 아래에 있어야 합니다.

```text
대회이름/AtCoder핸들/**
```

예를 들어 PR 제목이 `ABC350 / tourist`라면 다음 경로는 허용됩니다.

```text
ABC350/tourist/A.kt
ABC350/tourist/src/Main.kt
```

다음 경로는 허용되지 않습니다.

```text
ABC350/other/A.kt
README.md
```

## 환경변수

| 변수 | 설명 | 기본값 |
| --- | --- | --- |
| `REPO_OWNER` | 검사할 GitHub 저장소 owner 또는 organization | 없음 |
| `REPO_NAME` | 검사할 GitHub 저장소 이름 | 없음 |
| `GITHUB_TOKEN` | 검사할 저장소에 접근 가능한 GitHub API 토큰 | 없음 |
| `DRY_RUN` | 실제 댓글/머지/Discord 전송 없이 로그만 출력 | `true` |
| `DISCORD_ENABLED` | Discord webhook 전송 활성화 | `false` |
| `DISCORD_WEBHOOK_URL` | Discord webhook URL | 빈 값 |

## 로컬 실행

JDK 21이 필요합니다.

```bash
export REPO_OWNER=your-owner
export REPO_NAME=your-repo
export GITHUB_TOKEN=your-token
export DRY_RUN=true

./gradlew bootRun
```

Windows PowerShell에서는 다음처럼 실행할 수 있습니다.

```powershell
$env:REPO_OWNER="your-owner"
$env:REPO_NAME="your-repo"
$env:GITHUB_TOKEN="your-token"
$env:DRY_RUN="true"

./gradlew bootRun
```

## GitHub Actions 설정

워크플로는 `.github/workflows/pr-bot.yml`에 있습니다.

- 매주 금요일 19:00 KST 실행
- GitHub Actions cron 기준: `0 10 * * 5`
- `workflow_dispatch`로 수동 실행 가능
- 수동 실행 시 `dry_run`, `target_owner_1`, `target_repo_1`, `target_owner_2`, `target_repo_2` input 지원

봇 레포와 검사할 레포가 달라도 됩니다. 최대 2개 레포까지 순차적으로 검사할 수 있습니다.

### 1. 검사 대상 레포 정하기

봇 레포의 `Settings -> Secrets and variables -> Actions -> Variables`에 값을 추가합니다.

| 종류 | 이름 | 설명 |
| --- | --- | --- |
| Repository 또는 Organization variable | `TARGET_REPO_OWNER_1` | 첫 번째 검사 레포의 owner 또는 organization |
| Repository 또는 Organization variable | `TARGET_REPO_NAME_1` | 첫 번째 검사 레포 이름 |
| Repository 또는 Organization variable | `TARGET_REPO_OWNER_2` | 두 번째 검사 레포의 owner 또는 organization. 선택 |
| Repository 또는 Organization variable | `TARGET_REPO_NAME_2` | 두 번째 검사 레포 이름. 선택 |
| Repository 또는 Organization variable | `DISCORD_ENABLED` | Discord 알림 사용 여부. 선택 |

레포를 하나만 검사하려면 `TARGET_REPO_OWNER_1`, `TARGET_REPO_NAME_1`만 설정하면 됩니다. 기존 `TARGET_REPO_OWNER`, `TARGET_REPO_NAME`도 첫 번째 레포 설정으로 사용할 수 있습니다.

예를 들어 `pulse-club/algorithm-study`와 `pulse-club/coding-test-study`를 검사하려면 다음처럼 설정합니다.

```text
TARGET_REPO_OWNER_1=pulse-club
TARGET_REPO_NAME_1=algorithm-study
TARGET_REPO_OWNER_2=pulse-club
TARGET_REPO_NAME_2=coding-test-study
```

### 2. 봇 토큰 등록하기

봇 레포의 `Settings -> Secrets and variables -> Actions -> Secrets`에 값을 추가합니다.

| 종류 | 이름 | 설명 |
| --- | --- | --- |
| Repository secret | `BOT_GITHUB_TOKEN` | 모든 검사 대상 레포에 접근 가능한 토큰 |
| Repository secret | `DISCORD_WEBHOOK_URL` | Discord webhook URL. 선택 |

봇 레포와 검사할 레포가 다르면 기본 `secrets.GITHUB_TOKEN`으로는 부족합니다. `BOT_GITHUB_TOKEN`에는 fine-grained PAT 또는 GitHub App 토큰을 넣어야 합니다.

fine-grained PAT를 쓴다면 대상 레포 1개 또는 2개를 선택하고, Repository permissions를 다음처럼 줍니다.

| 권한 | 필요 수준 | 이유 |
| --- | --- | --- |
| `Contents` | Read and write | 조건을 만족한 PR을 squash merge |
| `Pull requests` | Read and write | 열린 PR과 변경 파일 조회 |
| `Issues` | Read and write | 규칙 위반 PR에 댓글 작성 |

### 3. 수동 실행으로 확인하기

수동 실행할 때 `target_owner_1`, `target_repo_1`, `target_owner_2`, `target_repo_2`를 입력하면 Actions 변수보다 우선합니다.

처음에는 `Actions -> PR Guardian Bot -> Run workflow`에서 `dry_run=true`로 실행해 로그만 확인합니다. 문제가 없으면 `dry_run=false`로 다시 실행합니다.

스케줄 실행은 `dry_run=false`로 동작합니다. 실제 댓글 작성, squash merge, Discord 전송이 수행될 수 있으므로 토큰과 대상 레포를 먼저 확인해야 합니다.

## dry-run

`DRY_RUN=true`이면 실제 side effect를 수행하지 않습니다.

- PR 댓글 작성 안 함
- PR merge 안 함
- Discord webhook 전송 안 함

대신 로그에 `Would comment`, `Would merge`, `Would send Discord notification` 형태로 출력합니다. 기본값은 `true`입니다.

## Discord Webhook

Discord 알림은 선택 기능입니다. `DISCORD_ENABLED=true`이고 `DISCORD_WEBHOOK_URL` 값이 있을 때만 invalid PR 요약을 전송합니다. `DRY_RUN=true`이면 전송하지 않고 로그만 출력합니다.

## GitHub Copilot Automatic Code Review

Copilot automatic code review는 GitHub 저장소 설정에서 별도로 활성화해야 합니다. 이 봇은 Copilot 리뷰 결과를 직접 처리하지 않고, PR 제목/파일 경로 검증과 댓글/merge 처리만 담당합니다.
