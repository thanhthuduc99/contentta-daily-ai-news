# contentta-daily-ai-news

Pipeline tự động gen video AI news short 1080×1920, chạy bằng Claude Code. Mỗi run: đọc nguồn tin, chọn chủ đề, viết script tiếng Việt, dựng giọng đọc, render video.

Agent đọc `AGENT-RUNBOOK.md` rồi thực thi từng bước trong đó.

## Output

Mỗi lần chạy sinh ra `video-projects/daily-YYYYMMDD/` chứa các file HTML composition và `renders/final.mp4`. Thư mục `video-projects/` được gitignore, video không đẩy lên repo.

## Pipeline tóm tắt

1. Fetch YouTube RSS + GitHub trending tuần
2. Gen 3 topic candidates, reason, chọn 1
3. Viết VO script tiếng Việt khoảng 88 giây
4. OpenAI TTS `gpt-4o-mini-tts`, voice `onyx`, speed 1.1
5. OpenAI Whisper transcribe lấy word-timing
6. Build 8 scene HTML, reuse template `opus-48-daily-khong-face`
7. `hyperframes render --quality standard` ra `renders/final.mp4`

## Repo layout

- `AGENT-RUNBOOK.md` — full prompt agent đọc mỗi run
- `contentta-shorts-skill/` — skill: CLAUDE.md, template HTML, brand token, scripts
- `tools/` — script cho pipeline
  - `fetch-nateherk.mjs` — RSS feed YouTube
  - `fetch-yt-transcript.mjs` — auto-caption transcript
  - `fetch-github-trending.mjs` — scrape github.com/trending weekly
  - `tts-openai.mjs` — OpenAI TTS
  - `transcribe-openai.mjs` — Whisper word-timing
  - `ffmpeg-bin.mjs` — resolve binary ffmpeg-static

## Setup

```bash
npm install
cp .env.example .env
```

Mở `.env`, điền `OPENAI_API_KEY` của bạn. Lấy key ở https://platform.openai.com/api-keys

`.env` nằm trong `.gitignore`. Đừng commit file này.

## Nhạc nền

`contentta-shorts-skill/music/` để trống có chủ đích. Repo không kèm file nhạc vì lý do bản quyền. Bỏ file MP3 của bạn vào đó rồi tham chiếu bằng đường dẫn tương đối, xem `contentta-shorts-skill/music/README.md` để biết yêu cầu định dạng và volume.

## Yêu cầu

- Node 18+
- Claude Code CLI
- `hyperframes` (cài qua npm dependencies)

## License

MIT, xem `LICENSE`.
