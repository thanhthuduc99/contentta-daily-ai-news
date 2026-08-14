# Contentta Shorts — Skill edit video ngắn (vertical, có mặt người)

Skill **portable** để edit video ngắn brand Contentta với motion graphics + karaoke caption + nhạc nền, **captions baked-in** (KHÔNG FCPXML, KHÔNG CapCut). Mang folder `contentta-shorts-skill/` sang máy/agent khác là chạy được — chỉ cần các phụ thuộc ở README.

**3 chế độ sản xuất** (chi tiết §3b):
- **Face dọc 1080×1920** (mặc định) — talking-head, giọng thu thật → Whisper. Mẫu: `video-projects/claude-intro-doc`.
- **No-face dọc 1080×1920** — "không mặt người" / bản tin / video tổng hợp, giọng **TTS tự sinh**. Mẫu: `video-projects/opus-48-daily-khong-face`.
- **Landscape 1920×1080** — "edit video ngang" / intro YouTube, face FULL ↔ dock phải. Mẫu: `video-projects/intro-google-io-ngang`.

> Đọc kèm: `WORKFLOW.md` (10 bước có lệnh), `scripts/README.md` (transcribe + cắt + gen caption), `MOTION_PHILOSOPHY.md` (gu thẩm mỹ — phần T1–T5 là legacy landscape, bỏ qua).

---

## 0. HARD RULES (bắt buộc — từ feedback thực tế)

1. **Dọc 1080×1920 mặc định.** Ngang 1920×1080 khi user nói "edit video ngang"; no-face + TTS khi "không mặt người" → §3b.
2. **Cắt đoạn vấp** trước khi build: dead-air + false-start + đoạn ấp úng (xem §1).
3. **Speed 1.1x** mặc định (user nói chậm).
4. **Tiếng Việt chuẩn**, không lẫn tiếng Anh, không thiếu chữ. Sửa lỗi Whisper qua `replacements.json`.
5. **Short-form ≠ poster:** graphics nửa trên, mặt nửa dưới, caption strip đáy, pacing nhanh, chữ vừa phải.
6. **Caption tách theo câu** (không gộp xuyên câu), trắng mờ→sáng, KHÔNG đỏ. Dùng `scripts/generate-captions.mjs`.
7. **Giãn dòng** title khi dòng trên dấu nặng + dòng dưới dấu sắc (Ậ/Ớ) → line-height ~1.34.
8. **YouTube outro:** user nói "edit video ngắn giới thiệu youtube" → chèn `youtube-outro` 3s cuối, dòng dưới "Channel: Thành Vũ Đức".
9. **Idle animation = CSS @keyframes** (không gsap repeat:-1). Determinism: không Date.now/Math.random.
10. **Visual-verify gate:** render draft → extract frames → **Read PNG** verify TRƯỚC final.

---

## 1. Pipeline cắt vấp (điểm cốt lõi)

Dùng **OpenAI Whisper API trực tiếp** (key `OPENAI_API_KEY` trong `.env`), KHÔNG phụ thuộc `hyperframes transcribe`.

```
source.mp4 → tách audio → transcribe (verbose_json, word+segment)
           → analyze-transcript.mjs + ffmpeg silencedetect → chốt đoạn giữ
           → ffmpeg trim+concat + setpts/1.1 + atempo=1.1 → face-final.mp4
           → re-transcribe face-final → transcript-final.json (timeline caption)
```

Cắt bỏ: lead-in im lặng, **dead-air** (gap > ~0.7s không cần thiết), **false-start** (lặp từ, "à ờ"), **đoạn ấp úng** — dấu hiệu: Whisper gán **1 từ kéo dài nhiều giây** (vd "hiểu" giữ 9s) = nói xong rồi khựng. Lệnh cụ thể: `scripts/README.md` §1–3. Source gốc giữ nguyên.

---

## 2. Hai hướng motion (đã duyệt)

- **COSMIC** — vũ trụ xoay vòng: orbital rings (CSS spin nhiều tốc độ) + particle stardust + solar flare burst + nebula gradient. Font Be Vietnam Pro 800.
- **EDITORIAL** — tạp chí: Playfair Display italic + số chương stroke-only + hairline draw + dấu nháy đôi oversized + stagger by line. Font Playfair + JetBrains Mono.

Trộn được trong 1 video (vd intro cosmic → giữa editorial → CTA cosmic). 8 mẫu xem nhanh: `templates/_catalog-preview/`. **KHÔNG** dùng HUD/terminal (đã bị loại).

---

## 3. Kiến trúc scene (index.html)

`index.html` là orchestrator. Mặt người **luôn BOTTOM** (talking-head); graphics nửa trên không đè mặt.

| Track | Nội dung |
|---|---|
| 0 | `ambient-bg` (nebula Deep Space, full canvas) |
| 1 | `#face-wrapper > video` (muted) — BOTTOM: `translate(0,1136) scale(0.5625)` |
| 2 | scenes (tuần tự, không overlap — shave 0.02 tránh float overlap) |
| 3 | `captions` (karaoke, full duration) |
| 4 | `face-audio` (`<audio>` src = face-final.mp4, volume 1) |
| 5 | `music` (`<audio>`, volume 0.14–0.15) |
| 99 | `grain-overlay` |

- **Face math:** `face-wrapper-pattern.md`. BOTTOM scale 0.5625 (1080/1920), y=1136. HIDDEN: opacity 0 (cho scene full-canvas). Seam line đỏ tại y≈1132. Ken Burns: face-video scale 1.0→1.04 over full.
- **Graphics zone:** y ~120–1050. Caption strip bottom 220 (đè phần dưới mặt — đúng chuẩn). Mỗi scene chừa sẵn "vùng mặt" phía dưới.
- **Fonts** (load ở index `<head>`, display=block): Be Vietnam Pro (display/number), Plus Jakarta Sans (body/chip), Playfair Display ital (editorial), JetBrains Mono (label/số).

## 3b. Biến thể: no-face (TTS) & landscape

§3 ở trên mô tả chế độ mặc định (face dọc, thu giọng thật). 2 biến thể dưới đây đổi pipeline + kiến trúc track.

### No-face dọc (giọng TTS) — mẫu `video-projects/opus-48-daily-khong-face`
Dùng khi user nói "không mặt người", làm bản tin / video tổng hợp không có người quay.
- **Bỏ B2–B5** (thu/cắt vấp). Sinh giọng bằng **Gemini 2.5 Flash Preview TTS** (`gemini-2.5-flash-preview-tts`, voice `Algenib`, style phát thanh viên) — key `GEMINI_API_KEY` trong `.env`. Gemini trả **PCM→WAV→mp3** (ffmpeg trong `tts.mjs`). Style/giọng điều khiển bằng câu chỉ đạo natural-language đầu prompt (không có param riêng). **`hyperframes tts` (Kokoro) KHÔNG hỗ trợ tiếng Việt**, đừng dùng.
- Pipeline: viết `assets/vo-script.txt` (phiên âm tên riêng, vd Contentta→"Còn Ten Ta") → `tts.mjs` → `voice.mp3` → `transcribe.mjs` (**OpenAI Whisper**, key `OPENAI_API_KEY`, word-timing) → `generate-captions.mjs` + `replacements.json` (fix Whisper nghe sai: Claude→"Plot", Anthropic→"Entropic", Contentta→"contenta") → canh `data-start` scene theo segment boundary của transcript. Script mẫu nằm trong `assets/` của project.
- **KHÔNG face-wrapper/seam.** Scene full-canvas trên `ambient-bg`. Track: `ambient` 0 · scenes 2 · `captions` 3 · `voice`(`<audio>`) 4 vol 1 · `music` 5 vol 0.14 · `grain` 99.
- Nhịp: câu chỉ đạo "tốc độ vừa phải" + script ~300 từ ≈ 88s. Muốn nhanh/chậm → sửa câu `style` trong `tts.mjs`. Đổi giọng → đổi `voiceName` (danh sách voice xem Google AI Studio).
- **⚠ DAILY NEWS PIPELINE override:** repo daily dùng `AGENT-RUNBOOK.md` ở root — target **55–62s, script 180–210 từ, TTS OpenAI onyx + atempo 1.15, form tùy biến 4–6 scene**. Số liệu 88s/300 từ ở trên là của project mẫu cũ, KHÔNG áp cho daily run.
- Pattern bổ sung (custom, tái dùng): badge số "THAY ĐỔI N" (liệt kê 1→5) · slider mốc (Low→Ultra) · số tương phản gạch (50→15) · split-flow (1 task → nhiều phần).

### Landscape 1920×1080 — mẫu `video-projects/intro-google-io-ngang`
Dùng khi user nói "edit video ngang" / intro YouTube landscape.
- Mặt **full-frame nền**: `#face-wrapper` 1920×1080, `#face-video` `height:100%` căn giữa (`left:50%;translateX(-50%)`). GSAP animate **width/height** trên wrapper (KHÔNG scale như dọc).
- 2 vị trí: `FULL {x:0,y:0,w:1920,h:1080}` ↔ `RIGHT dock {x:1180,y:60,w:680,h:960}` + class `.docked` (viền trắng 4px, bo góc 32px, shadow). Panel scene vào TRƯỚC, face dock sau ~0.15s.
- Track: `face-video` -1 · `voice`(`<audio>` tách) 1 vol 1 · `music` 2 vol 0.12 · scenes 4 (đặt `style="z-index:3"` inline trên div scene ở index). **KHÔNG** track `ambient`/`grain`/`captions` riêng — mỗi scene tự vẽ panel nền (vd panel trái 62%), không karaoke caption.
- Scene file đánh số `01-…html`, nghiêng **editorial** (hook-title · stat · questions · cta). Reveal nội bộ bằng nhiều clip con (`class="clip"` + `data-track-index` trong sub-comp) + thuộc tính `data-at` cho stagger.

## 4. Scene patterns — `templates/scene-patterns/`

Mỗi file là sub-composition `<template>` đăng ký 1 GSAP timeline paused vào `window.__timelines["<id>"]`. Copy vào project, sửa text + mốc reveal khớp lời.

**Cosmic:** `slideshow` (multi-slide + dots), `card-cascade` / `step-badge` (cards + badge BƯỚC N), `big-number-reveal`, `kinetic-type`, `step-flow`, `cta-supernova`, `full-canvas-intro`.
**Editorial:** `editorial-questions` (01/02 serif), `editorial-headline` (quote + Playfair), `editorial-stat` (% count-up).
**Outro:** `youtube-outro` (full-canvas, face HIDDEN).

Mỗi scene CẦN: entry whip (y/blur/opacity in), nội dung reveal **khớp lời** (mốc = timestamp transcript), exit whip (trừ scene cuối), ≥1 SVG icon, brand chip. Reveal **đúng lúc nói tới** từ khoá đó.

## 5. Caption rules

Dùng `scripts/generate-captions.mjs <transcript-final.json> compositions/captions.html [replacements.json]`:
- Chunk theo **từng câu Whisper**, gộp từ lẻ cuối câu vào dòng trước (không mồ côi "này…").
- Trắng mờ `rgba(250,247,245,0.55)` → sáng `#FAF7F5` theo lời (0.10s). **KHÔNG đỏ, không scale pop.**
- Be Vietnam Pro 600, 44px, outline 2px stack, bottom 220px. Fade out trước segment kế (SWAP_GUARD).

## 6. Typography
- Title nhiều dòng: nếu dòng trên dấu **nặng** + dòng dưới dấu **sắc** → line-height **~1.34** (per-line). Khác → ~1.15.
- Caption emphasis dùng trắng, không đỏ. Title accent = Cosmic Red. Text mặc định Stardust trên Deep Space.

## 7. Brand Contentta v2026.05 Orbital
| Token | Value | Vai trò |
|---|---|---|
| `--cosmic-red` | `#E10E1F` | accent (~25%) |
| `--deep-space` | `#070409` | bg (~60%) |
| `--stardust` | `#FAF7F5` | text (~15%) |
| `--dust` | `#A89BA1` | muted |
| `--maroon` | `#5A0410` | nebula/depth |

File: `assets/brand-tokens.contentta.css`. Đổi brand → `assets/brand-visual-guide.md`.

## 8. Render contract (compact)
1. Root `<div>`: id, data-composition-id, data-start="0", data-width, data-height.
2. Element có thời gian cần data-start/data-duration/data-track-index (trừ `<video>`/`<audio>`).
3. `<video>` phải `muted`; audio ở `<audio>` riêng.
4. Mỗi composition đăng ký 1 GSAP timeline **paused** trên `window.__timelines["<id>"]` (key = data-composition-id).
5. Pad timeline tới đúng data-duration: `tl.set({},{},DUR)`.
6. Scene cùng track không overlap → shave 0.02 nếu end == next start (tránh float).
7. KHÔNG .play()/.pause()/.currentTime trên media. KHÔNG animate width/height/top/left trên `<video>` (bọc div).
8. Idle = CSS @keyframes. Không Date.now/Math.random.

## 9. Visual-verify gate (BẮT BUỘC)
```bash
npx hyperframes render --quality draft --output renders/draft.mp4
# extract frame hero mỗi scene rồi Read PNG vào context:
for t in <scene-times>; do ffmpeg -y -ss $t -i renders/draft.mp4 -frames:v 1 -q:v 3 renders/frames/t$t.jpg; done
```
Read TỪNG PNG, verify: mặt không crop, caption sync đúng từ, brand color, KHÔNG va dấu, không overflow, jump-cut được transition che. Sai → sửa → re-render. Chỉ render `--quality standard` sau khi pass.

## 10. Commands
```bash
node scripts/analyze-transcript.mjs transcript.json     # tìm chỗ vấp
node scripts/generate-captions.mjs transcript-final.json compositions/captions.html [rep.json]
npx hyperframes lint                                     # phải 0 error
npx hyperframes preview                                  # Studio localhost:3002 (gate duyệt live)
npx hyperframes render --quality draft|standard --output renders/X.mp4
npx hyperframes doctor                                   # check env
```

Toàn bộ 10 bước có lệnh: `WORKFLOW.md`.
