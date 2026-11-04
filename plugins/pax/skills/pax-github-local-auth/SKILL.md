---
name: pax-github-local-auth
description: "PAX(sp) · 로컬에서 private GitHub repo를 clone/push 하기 위한 GitHub 인증 안내. \"GitHub 로그인\", \"clone 권한 없음\", \"push가 안 됨\" 등에 사용."
---
# 로컬 GitHub 인증

**Codex:** PAX 연결은 호스트가 자동 전달하는 현재 채팅의 `threadId`로 선택됩니다. 도구 인자에 채팅 ID를 만들거나 다른 채팅의 값을 넣지 마세요. 연결은 `connect`, 해제는 `disconnect`, 스킬 저장은 `sync_skills` 도구를 사용합니다. `session_required`면 Codex 업데이트·재시작을 안내하고 최근 연결이나 폴더 연결로 우회하지 않습니다. 로컬 파일을 바꾸기 전에는 연결 서비스와 작업 폴더가 맞는지 확인하세요. Claude Code의 기존 호출 방식은 유지합니다.

private repo는 **사용자 본인의 GitHub 인증**으로 clone/push 합니다. PAX 서버는 GitHub 토큰을 내려주지 않습니다.

- 확인: `git ls-remote <repoUrl>` 성공 = 인증 있음(여기서 끝). 실패 시에만 아래 진행.
- gh 미설치면 **AI가 직접 설치하세요**: macOS `brew install gh` / Windows `winget install --id GitHub.cli` / Linux 배포판 패키지(sudo 필요 시 명령 제시 후 사용자 실행). 패키지 매니저 없으면 https://cli.github.com 안내로 폴백.
- 로그인은 **AI가 디바이스 플로우를 돌려 채팅으로 유도**하세요 — "로그인하세요"로 끝내지 말 것:
  1. `gh auth login --hostname github.com --git-protocol https --web`을 출력을 파일로 받아 **백그라운드로 실행**(완료까지 블로킹되므로 포그라운드 금지).
  2. **코드부터 먼저 알리세요.** 그 파일에서 1회용 코드(XXXX-XXXX)가 보이면(몇 초 안에 나옵니다 — 최대 10초만 확인) 로그인 완료를 기다리거나 다른 명령·확인을 하기 **전에** 바로 사용자에게 전달합니다:
     "GitHub 로그인 코드는 **XXXX-XXXX** 예요. https://github.com/login/device 를 열어 이 코드를 넣고 승인해 주세요. 끝나면 알려 주세요."
     출력에 `copied to clipboard` 가 있으면 "코드는 이미 복사돼 있어 붙여 넣기만 하면 돼요" 를 덧붙이세요. 브라우저를 대신 열 수 있으면 코드를 알린 **뒤에** 여세요(브라우저가 먼저 뜨면 사용자는 코드 없이 기다리게 됩니다).
     코드를 알리기 전에 `gh auth status` 를 반복 확인하거나 다음 단계(clone 등)를 진행하지 마세요.
  3. 완료 답을 받으면 `gh auth status`로 검증한 뒤 **`gh auth setup-git`을 실행**(멱등). 백그라운드(비대화형) 로그인은 git 자격증명 연결 프롬프트를 건너뛰므로, 이 명령이 없으면 `gh`는 로그인돼 있어도 `git clone`/`push`가 인증 실패합니다. (코드는 약 15분 후 만료 — 만료 시 같은 명령 재실행.)
  - 백그라운드 실행이 안 되는 환경이면 같은 명령을 사용자가 터미널에 직접 붙여넣도록 안내(터미널 대화형 로그인은 git 연결을 물어보므로 `setup-git` 은 생략 가능). 편집기(VS Code 등)의 GitHub 로그인도 대안.
- PAX 연결(`/pax-sp:connect`)에는 gh 가 필요 없습니다 — 이 로그인은 clone/push 용입니다. 해당 repo 에 권한이 있는 GitHub 계정으로 로그인하세요. `git ls-remote` 가 성공하면 이 단계를 통째로 건너뛰세요.
- push는 **manifest(`get_project_manifest`)의 `workingBranch` 브랜치로만**. 다른 브랜치(main 등) 직접 push 금지(배포 정책 + 브랜치 보호 규칙이 거부).
- gh CLI는 **clone/push 인증 + 조회 전용**입니다. **조회는 허용** — `gh run list`/`gh run view`/`gh pr checks`/`gh pr view --comments` 로 빌드·게이트 로그를 더 깊이 볼 수 있습니다(인프라 디버깅용). 하지만 **PR 생성·머지(`gh pr create`/`gh pr merge`)는 금지** — workingBranch에 push 하면 워크플로우가 PR·보안 게이트·머지를 자동 처리합니다(수동 PR은 충돌·게이트 우회).
- 권한 오류(403/404)면 사용자의 GitHub 계정이 해당 org repo 멤버인지, SAML SSO 인가가 유효한지 확인하도록 안내.
- **PAX 서버의 GitHub 토큰을 요청하지 마세요.**
