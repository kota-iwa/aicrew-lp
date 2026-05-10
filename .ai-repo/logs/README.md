# Session Logs

AI agent reference for debugging and consistency tracking.
Read this file FIRST — it contains enough metadata to find the right log(s) without opening individual files.

## Log Index

| # | File | Date | Type | Scope | Summary | Key Files |
|---|------|------|------|-------|---------|-----------|
| 002 | [002_remotion-video-popfriendly-and-aicrew-audio.md](002_remotion-video-popfriendly-and-aicrew-audio.md) | 2026-05-10 | feat | remotion-video, audio-narration, lp-deploy | Pop Friendly に全面リペイントした Remotion 動画（86.7s/2600f）を再レンダー＋差し替え。さらに音声 n03/n06 の「CS-bot」を edge-tts (ja-JP-NanamiNeural) で「エーアイクルー」発音に再生成。本番反映済み。 | `design.ts`, `CsBotDemo.tsx`, `n03.mp3`, `n06.mp3` |
| 001 | [001_pop-friendly-redesign-and-first-deploy.md](001_pop-friendly-redesign-and-first-deploy.md) | 2026-05-10 | feat | lp-html, lp-assets, lp-deploy | LP `index.html` (1744行) を Slack/Aubergine 系から Notion/Slack 系の Pop Friendly テイストに全面リデザイン。AI 生成アセット 15 枚 (gpt-image-2 low, $0.10) を新規追加。codex に依頼してデザイン実装、画像 aspect-ratio バグと reveal アニメ初動失敗を修正、Vercel auto-deploy で本番反映。 | `index.html`, `assets/v2-pop-friendly/*.png` |

## Component Index

Quick lookup: which logs touched which component? Listed newest-first within each component.

### lp-html
001

### lp-assets
001

### lp-deploy
002, 001

### remotion-video
002

### audio-narration
002

## Topic Chains

Logs that form a logical sequence — read together for full context.

- **AIcrew Pop Friendly redesign** (LP + 動画 + 音声を世界観統一して本番リリース): 001 → 002

## Conventions

- Logs numbered sequentially: `001_`, `002_`, ...
- Written for AI agents: debugging reference + dependency tracking
- Always check Dependencies and State Changes when resuming work
- **Type values**: `feat`, `fix`, `refactor`, `audit`, `investigation`
- **Scope values**: 現状の Component Index にあるタグから選ぶ。新規が必要なら追加して Component Index も更新する
- **Backup naming**: `<file>.bak.YYYYMMDD`（JST 日付）
- **Production**: aicrew.c-h.co.jp は GitHub master push → Vercel auto-deploy で 1〜3 分後に反映
