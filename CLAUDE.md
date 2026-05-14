# youtube_notes

Sweet-Butters의 유튜브 강의 학습 + C 실습 노트. 노트북 WSL을 상시 가동해두고 핸드폰·태블릿·다른 PC에서 원격 접속해 작업하는 멀티 디바이스 개인 워크플로의 단일 진실원(single source of truth).

## Quick Start (노트북)

```bash
cd ~/projects/youtube_notes
git pull --ff-only            # 세션 시작
# ... 작업 ...
git add -p && git commit -m "..." && git push   # 세션 종료
```

C 예제 실행:

```bash
gcc code_practice/hello_yonsei.c -o code_practice/hello_yonsei
./code_practice/hello_yonsei
```

## 환경

| 항목 | 값 |
| --- | --- |
| OS (노트북) | WSL2 Ubuntu 24.04 on Windows 11 |
| WSL 위치 | `D:\WSL_Storage` (`wsl --import`로 이전) — 홈이 물리적으로 D 드라이브. **`/mnt/d` 작업 금지** (9P 마운트는 느리고 C 컴파일 권한 문제). |
| 프로젝트 경로 | `/home/sweetbutters/projects/youtube_notes/` (native ext4) |
| 원격 저장소 | `git@github.com:Sweet-Butters/youtube_notes.git` (public, `main` 추적) |
| GitHub 인증 | ed25519 키 `~/.ssh/id_ed25519` (passphrase 없음, GitHub 등록 완료) |

## 작업 원칙

1. **세션 시작 = `git pull --ff-only`** — 다른 기기 커밋 먼저 받기.
2. **세션 종료 = `git add` → `commit` → `push`** — 미완성이라도 WIP 커밋으로.
3. **기기 간 직접 파일 복사 금지** — sync는 오직 GitHub 경유.
4. **Auto-Yes / 명시적 확인 구분**:
   - 일반 명령(편집, 빌드, 통상 git, 테스트) → Claude가 묻지 않고 즉시 실행.
   - Destructive (`git push --force`, `git reset --hard`, `rm -rf` 등) → 반드시 사용자 확인.

## 원격 접속 (폰 ↔ 노트북)

두 경로가 동시 운영. 목적에 따라 선택.

### A. VS Code Tunnel — 코드 시각 편집 / 파일 트리 검토

- 노트북 측 데몬: systemd user service `vscode-tunnel.service` (`~/.config/systemd/user/`).
- `loginctl enable-linger sweetbutters` 적용 → WSL 부팅 시 자동 기동, 크래시 시 5초 후 재시작.
- 접속 URL: `https://vscode.dev/tunnel/sb-laptop/home/sweetbutters/projects/youtube_notes`
- 한계: 모바일 화면에서 VS Code UI는 터치 조작이 답답함.

운영 명령:
```bash
systemctl --user status vscode-tunnel.service
journalctl --user -u vscode-tunnel.service -f
```

### B. Termius SSH over Tailscale — 터미널 / Claude Code 중심 (모바일 권장)

네트워크 (Tailscale mesh VPN, 외부망 어디서든 접속):

| 기기 | tailnet hostname | tailnet IPv4 |
| --- | --- | --- |
| 노트북 | `book-j6lo4pcd9k` | `100.96.71.125` |
| 폰 (Galaxy S22) | `s22` | `100.104.127.113` |

노트북 셋업:
- `openssh-server` 설치, `ssh.service` 활성 (`systemctl is-active ssh` → `active`).
- `~/.ssh/authorized_keys`에 Termius가 폰에서 생성한 ED25519 공개키 등록.

폰 셋업:
- Tailscale 앱 (Play Store) — 노트북과 **동일 계정** 로그인.
- Termius (Play Store) — Host: `100.96.71.125`, Port: `22`, User: `sweetbutters`, Auth: SSH Key.

권장 진입 시퀀스 — `tmux`로 연결 끊김 내성 확보:
```bash
tmux new -s work          # 최초 세션
tmux attach -t work       # 재접속 시
cd ~/projects/youtube_notes
claude
```

## 새 기기에서 이 repo 처음 열 때

기본(코드 작업만):
1. WSL2 + Ubuntu 24.04 (선택: `wsl --import`로 비-C 드라이브 이전)
2. `sudo apt install -y git gh jq build-essential tmux`
3. ed25519 SSH 키 생성 → GitHub `settings/keys` 등록
4. `gh auth login --hostname github.com --git-protocol ssh --web`
5. `git clone git@github.com:Sweet-Butters/youtube_notes.git`

원격 접속 추가 (선택):

6. **VS Code Tunnel**: standalone CLI 다운(`curl -sL https://update.code.visualstudio.com/latest/cli-linux-x64/stable | tar -xz -C ~/.local/bin/` → 원하면 `vscode`로 rename) 후 systemd user service로 등록.
7. **SSH over Tailscale**: `sudo apt install -y openssh-server` → `sudo systemctl enable --now ssh` → `curl -fsSL https://tailscale.com/install.sh \| sudo sh` → `sudo tailscale up` → 폰 공개키를 `~/.ssh/authorized_keys`에 추가.

## Claude Code 사용 시 참고

- 폰 접속 비중이 크므로 응답은 짧게, 긴 로그는 요약.
- 경로는 절대 경로 우선 — Bash 도구는 호출 간 cwd를 유지하지 않음.
- 권한 모드: `bypassPermissions` 기본 (`~/.claude/settings.json`). Shift+Tab으로 토글.

## Repo 구조

| 디렉터리 | 용도 |
| --- | --- |
| `code_practice/` | 강의에서 따라 친 C 코드 (`hw*.c`, 짧은 예제, 컴파일러 실험) |
| `lecture_summaries/` | 강의별 요약 노트 (Markdown) |
| `resources/` | 참고 자료, 링크 모음, 다이어그램, 외부 PDF 메모 |

## .gitignore 요지

C 컴파일 산출물(`*.o`, `*.a`, `*.so`, `*.out`, `*.exe`, `a.out`, 그리고 `code_practice/` 내 무확장 ELF 바이너리), 에디터/IDE 설정(`.vscode/`, `.idea/`, `*.swp`), 빌드 디렉터리(`build/`, `dist/`) 제외. `code_practice/`는 디렉터리 allowlist 패턴으로 소스(`*.c`, `*.h`, `Makefile`, `README*`, `.gitkeep`)만 트래킹.

## 멀티 디바이스 sync 검증 기록

### 2026-05-14 — Tailscale + Termius SSH 셋업 후 GitHub 왕복 sync 1회 검증

| 항목 | 내용 |
| --- | --- |
| 환경 | 노트북 `book-j6lo4pcd9k` (WSL2 Ubuntu 24.04, `100.96.71.125`) ↔ 폰 `s22` (Galaxy S22, Termius via Tailscale `100.104.127.113`). 양쪽 모두 동일 GitHub 계정(`Sweet-Butters`) SSH 키로 origin 접근. |
| 절차 | 1) 폰에서 `lecture_summaries/sync_test.md`에 `from phone <date>` 한 줄 commit → push. 2) 노트북에서 `git pull --ff-only` 후 내용 확인. 3) 노트북에서 `from laptop <date>` 한 줄 append → commit → push. 4) 폰에서 `git pull --ff-only` 후 두 줄 모두 확인. |
| 결과 | 폰 → 노트북 ✓ (`f4474d8`). 노트북 → 폰 ✓ (`717c514`). 양방향 fast-forward, 충돌·인증 이슈 없음. |
| 정리 | 본 절차의 임시 산출물(`lecture_summaries/sync_test.md`)은 검증 직후 제거 (해당 commit 함께). 이후 sync 의심 상황 시 동일 절차 반복. |
