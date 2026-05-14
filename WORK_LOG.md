# 작업 로그

youtube_notes 외부 프로젝트 작업 메모. 여러 기기에서 처음 열 때 "내가 마지막에 뭐 하다 멈췄지?" 보려는 용도. 상세 컨텍스트는 각 프로젝트의 자체 문서를 참조.

---

## 2026-05-14 — Yorg_project: `/add <url>` 파이프라인

**상태**: 일시 중단. 코드는 양 repo main에 push 완료, Cloud Run 봇 redeploy 1단계만 남음.

- 작업 위치: `~/projects/Yorg_project/`, `~/projects/auto_project/`
- 상세·재개 절차: [`Yorg_project/PROGRESS.md`](https://github.com/Sweet-Butters/Yorg_project/blob/main/PROGRESS.md) 의 "2026-05-14 세션" 섹션
- 한 줄 요약: URL 한 줄로 영상 자동 등록 (oembed + youtube-transcript-api) → 기존 summarize/categorize 파이프라인이 처리
- 재개 시 첫 액션: 폰 브라우저 → https://shell.cloud.google.com → `gcloud run deploy auto-bot --source bot/ --region asia-northeast3` (auto_project clone 후)
