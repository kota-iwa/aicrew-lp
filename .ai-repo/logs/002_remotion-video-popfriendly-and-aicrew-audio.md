---
log: 002
date: 2026-05-10
status: complete
type: feat
scope: [remotion-video, audio-narration, lp-deploy]
key_files: [design.ts, CsBotDemo.tsx, n03.mp3, n06.mp3]
related: [001]
---

# 002: Remotion video Pop Friendly repaint + AIcrew audio rebrand

**Date:** 2026-05-10
**Status:** Complete

## Summary

LP の Pop Friendly リデザイン (log 001) と世界観を揃えるため、Remotion 製デモ動画 (`AIcrew-demo.mp4`, 86.7s/2600f) を全面リペイント。さらに音声ナレーション 7 本のうち CS-bot 言及があった n03/n06 を edge-tts で再生成し「エーアイクルー」発音に差し替え。最終的に動画を 2 回レンダーして本番反映。

## Dependencies

- **Requires:**
  - log 001 で確定した Pop Friendly カラートークン（peach #FFE5D9 / mint #D1F0E0 / sky #E0F0FF / butter #FFF4D6 / coral #FF6B6B / aubergine #4A154B）
  - `~/projects/bot-manager/remotion/` の既存 CsBotDemo composition（fps=30、1920×1080、durationInFrames=2600）
  - 既存音声 `remotion/public/audio/{bgm,n01-n07,pop}.mp3`（n03/n06 以外は据え置き）
  - codex CLI（`/opt/homebrew/bin/codex`）— `--cd` でディレクトリ指定可
  - `edge-tts` (`/Users/kotaiwama/projects/comfy/venv/bin/edge-tts`) — Microsoft Neural TTS の Python ラッパー
  - OpenAI Whisper-1 API（既存ナレーション文字起こし用、`OPENAI_API_KEY` 必要）
- **Required by:**
  - LP の `<video>` タグから参照される `~/projects/aicrew-lp/assets/AIcrew-demo.mp4` と `AIcrew-demo-poster.jpg`
  - 将来の動画変更（再レンダー時に同じ design.ts と Pop Friendly トークンを起点に作業する）
- **Conflicts with:**
  - **bot-manager の `.gitignore` で `remotion/` が除外されている** — Remotion ソース変更は bot-manager 側に **commit されない**。再レンダーが必要になった場合、ローカルに残っている修正のみが頼り。バックアップ取得済（後述）

## State Changes

| Target | Before | After |
|--------|--------|-------|
| `aicrew-lp/assets/AIcrew-demo.mp4` | 4/24 ダーク版 9.34MB | 5/10 Pop Friendly + AIcrew 音声 8.47MB |
| `aicrew-lp/assets/AIcrew-demo-poster.jpg` | 4/24 ダーク版 79KB | 5/10 Pop Friendly Hero フレーム 58KB |
| `bot-manager/remotion/src/design.ts` | Cinematic Corporate (dark, #0A0A0F base, electric violet #8B5CF6) | Pop Friendly (white #FFFFFF base, coral #FF6B6B / aubergine #4A154B / pastels) |
| `bot-manager/remotion/src/CsBotDemo.tsx` | dark gradient orbs, white text on saturated bg | pastel orbs, aubergine text on light bg, "CS-bot 導入後" → "AIcrew 導入後" |
| `bot-manager/remotion/src/components/{HookScreen, BeforeAfterComparison, CatchCopy, ClockAnimation, SlackWindow, TitleCard, FlowDiagram, ProcessingSteps, TriageReport}.tsx` | dark theme + CS-bot 表記 | pastel theme + AIcrew 表記 |
| `bot-manager/remotion/public/audio/n03.mp3` | 14.97s「CS-Botなら、問い合わせを自動で…」 | 14.33s「エーアイクルーなら、問い合わせを自動で…」 |
| `bot-manager/remotion/public/audio/n06.mp3` | 14.23s「CS担当5名体制をCSボットと2名体制…」 | 13.66s「CS担当5名体制をエーアイクルーと2名体制に…」 |
| `aicrew-lp` HEAD | `7aa600c` (docs: FORkota.md + log 001) | `0163246` (feat(video): rebrand audio narration) |

## Steps Executed

### Step 1: Existing video diagnostic
- `ffprobe` で `AIcrew-demo.mp4` の素性確認 → 1920x1080 / 30fps / 86.6s
- `mem-search` で過去の cs-bot demo video セッションを発見（log 64949、2026-04-13、`bot-manager/remotion/` で制作）
- `grep CS-bot remotion/src/` で 6 ファイル + CsBotDemo.tsx 内 2 箇所のリブランド対象特定

### Step 2: Pop Friendly repaint via codex
- `/tmp/aicrew_remotion_repaint.md` に詳細指示書作成
- `cat /tmp/aicrew_remotion_repaint.md | codex exec --cd /Users/kotaiwama/projects/bot-manager --full-auto` で実行
- codex がコード修正を完遂（`design.ts` 全面書き換え、CsBotDemo.tsx パステル化、6 ファイルで CS-bot → AIcrew リネーム、`tsc --noEmit` 成功）
- codex のレンダリングは sandbox の Chrome/Crashpad permission denied で失敗 → ソース修正のみで終了

### Step 3: First render (Pop Friendly visual only)
```bash
cd /Users/kotaiwama/projects/bot-manager/remotion
npx remotion render CsBotDemo out/aicrew-popfriendly.mp4 --concurrency=4
```
- 約 17 分でレンダー完了（2600 フレーム / stitched 2600 / 8.5MB）
- `cp out/aicrew-popfriendly.mp4 ~/projects/aicrew-lp/assets/AIcrew-demo.mp4`
- ffmpeg で `-ss 00:00:01 -frames:v 1` を抽出 → 新 `AIcrew-demo-poster.jpg`
- commit `c5bfdbe` → push → Vercel auto-deploy → `last-modified: 2026-05-10 13:04 GMT` で反映確認

### Step 4: User feedback — audio still says CS-bot
- ユーザー指摘: 「cs-botじゃなくてAIcrewにしてほしいです。「エーアイクルー」」
- 音声ファイルにスクリプトテキストが保存されていなかったため、Whisper で全 7 本を文字起こし
```python
# /tmp/transcribe_aicrew.py
client.audio.transcriptions.create(model="whisper-1", file=af, language="ja")
```
- 結果: CS-bot/CSボット が登場するのは n03 と n06 の **2 本のみ**

### Step 5: TTS 再生成 (edge-tts)
```python
# /tmp/regen_aicrew_audio.py
EDGE_TTS = "/Users/kotaiwama/projects/comfy/venv/bin/edge-tts"
VOICE = "ja-JP-NanamiNeural"
```
- n03 (target 14.97s): rate +0% → 14.33s（許容内、-0.64s）
- n06 (target 14.23s): rate +0% → 15.14s だったので +11% に調整 → 13.66s（許容内、-0.57s）
- 新音声を Whisper で再文字起こしして「ai クルー」(=エーアイクルー) と発話されることを確認

### Step 6: 音声差し替え + 再レンダー
```bash
cp remotion/public/audio/n03.mp3 remotion/public/audio/n03.mp3.bak.20260510
cp remotion/public/audio/n06.mp3 remotion/public/audio/n06.mp3.bak.20260510
cp /tmp/n03_new.mp3 remotion/public/audio/n03.mp3
cp /tmp/n06_new.mp3 remotion/public/audio/n06.mp3
cd remotion && npx remotion render CsBotDemo out/aicrew-popfriendly-v2.mp4 --concurrency=4
```
- 再レンダー約 14 分で完了 / 8.47MB
- aicrew-lp/assets/ にコピー、ポスター再生成、commit `0163246`、push、Vercel 反映確認 (`content-length: 8473947`)

## Errors & Resolutions

| Error | Cause | Resolution |
|-------|-------|------------|
| codex がレンダー時に `mach_port_rendezvous` / `bootstrap_check_in ... Permission denied (1100)` で停止 | codex sandbox が macOS の Chrome/Crashpad の起動権限をブロック（Crashpad が `~/Library/Application Support/Google/Chrome/Crashpad/settings.dat` にアクセスできない） | 通常ターミナルから `npx remotion render` を直接実行（Bash run_in_background で背景化、polling で完了検知）|
| 1 回目の codex セッション（log 001 セッション内）で aicrew-lp 書き込み拒否 | codex の cwd が bot-manager で、aicrew-lp は cwd 外 | `codex exec --cd /Users/kotaiwama/projects/aicrew-lp` で対象ディレクトリを cwd に明示（log 001 で実証）|
| edge-tts n06 デフォルト rate +0% で 15.14s（target 14.23s より +0.91s 超過）| ja-JP-NanamiNeural のデフォルト発話速度 | rate を `+11%` に上げて 13.66s に短縮（target 比 -0.57s で許容範囲内）|

## Verification

```bash
# 本番動画反映確認
curl -sI https://aicrew.c-h.co.jp/assets/AIcrew-demo.mp4 | grep -iE "content-length|last-modified"
# Expected:
#   content-length: 8473947
#   last-modified: Sun, 10 May 2026 13:15:44 GMT  (or later)

# 音声内容確認
python3 -c "
from openai import OpenAI; c = OpenAI(api_key='...')
for f in ['/Users/kotaiwama/projects/bot-manager/remotion/public/audio/n03.mp3',
          '/Users/kotaiwama/projects/bot-manager/remotion/public/audio/n06.mp3']:
    with open(f, 'rb') as af:
        print(f, c.audio.transcriptions.create(model='whisper-1', file=af, language='ja').text)
"
# Expected: 'ai クルー' or 'エーアイクルー' in both transcripts
```

## Rollback

```bash
# Audio rollback
cp ~/projects/bot-manager/remotion/public/audio/n03.mp3.bak.20260510 ~/projects/bot-manager/remotion/public/audio/n03.mp3
cp ~/projects/bot-manager/remotion/public/audio/n06.mp3.bak.20260510 ~/projects/bot-manager/remotion/public/audio/n06.mp3

# Video rollback (LP)
cp ~/projects/aicrew-lp/assets/AIcrew-demo.mp4.bak.20260510 ~/projects/aicrew-lp/assets/AIcrew-demo.mp4
cd ~/projects/aicrew-lp && git add assets/AIcrew-demo.mp4 && git commit -m "rollback: revert to dark theme demo video"
git push origin master

# Source rollback (bot-manager remotion is gitignored, so use manual revert)
# design.ts や component を Cinematic Corporate に戻す場合は git stash があれば適用、なければ手動編集
```

## Known Issues & TODOs

1. **bot-manager 側の `remotion/` 修正が gitignored で履歴に残らない** — 同じ動画を別マシンで再レンダーするには Remotion ソースを bot-manager.gitignore から外すか、別 repo として管理する必要あり。**Severity: 中**（現環境では問題ないが、災害復旧時に再現不能）
2. **音声 n03/n06 はナレーターが Nanami（女性）に統一できているか未検証** — 既存 n01/n02/n04/n05/n07 が同じ Nanami 声か比較聴取していない。万一違う声優だと音声が違和感。**Severity: 軽微**（実聴で違和感あれば全 7 本再生成）
3. **「AIcrew」ロゴが Plus Jakarta Sans で「Alcrew」に見える I/l 識別問題** — LP/動画両方で発生。フォント変更 or 字間調整 or 別書体で対応。**Severity: 軽微**（CRO 的にはブランド認知のため要対応）
4. **音声中の他のナレーション (n01, n02, n04, n05, n07) は元のまま** — n01「CSチームに5人いませんか」は商品名言及なしで OK だが、「AI で60%削減」を「AIcrew で60%削減」にすると brand 強化可能。**Severity: 機会損失**（ROI は CRO 効果次第）
5. **Vercel CLI 不可視（log 001 の項目）は引き続き** — kota の別アカウント（kotai-8742）で aicrew.c-h.co.jp を管理。私の現 CLI トークン（kota-elife）からは見えないが、git push 経由の auto-deploy は問題なく動作する。**Severity: 運用上気にしない**

## File Inventory

| File | Created/Modified | Purpose |
|------|-----------------|---------|
| `aicrew-lp/assets/AIcrew-demo.mp4` | Modified | Pop Friendly + AIcrew 音声版動画（8.47MB） |
| `aicrew-lp/assets/AIcrew-demo-poster.jpg` | Modified | 新動画 1s 地点の Hero フレーム抽出 |
| `aicrew-lp/assets/AIcrew-demo.mp4.bak.20260510` | Created (Step 0) | 旧ダーク版バックアップ（9.34MB） |
| `bot-manager/remotion/src/design.ts` | Modified (codex) | Pop Friendly トークンに全面置換 |
| `bot-manager/remotion/src/CsBotDemo.tsx` | Modified (codex) | パステル化 + AIcrew リネーム |
| `bot-manager/remotion/src/components/HookScreen.tsx` | Modified (codex) | 白背景 + ピーチ/スカイ orb + aubergine 見出し |
| `bot-manager/remotion/src/components/BeforeAfterComparison.tsx` | Modified (codex) | パステル化 + "+ AIcrew" 表記 |
| `bot-manager/remotion/src/components/CatchCopy.tsx` | Modified (codex) | カラー調整 |
| `bot-manager/remotion/src/components/ClockAnimation.tsx` | Modified (codex) | パステル化 |
| `bot-manager/remotion/src/components/SlackWindow.tsx` | Modified (codex) | darkMode の意味再解釈 / lavender lite mode |
| `bot-manager/remotion/src/components/TitleCard.tsx` | Modified (codex) | AIcrew リネーム |
| `bot-manager/remotion/src/components/FlowDiagram.tsx` | Modified (codex) | AIcrew リネーム |
| `bot-manager/remotion/src/components/ProcessingSteps.tsx` | Modified (codex) | AIcrew リネーム |
| `bot-manager/remotion/src/components/TriageReport.tsx` | Modified (codex) | AIcrew リネーム |
| `bot-manager/remotion/public/audio/n03.mp3` | Modified | edge-tts 再生成「エーアイクルーなら…」14.33s |
| `bot-manager/remotion/public/audio/n06.mp3` | Modified | edge-tts 再生成「…エーアイクルーと2名体制に…」13.66s |
| `bot-manager/remotion/public/audio/n03.mp3.bak.20260510` | Created | 旧 n03 バックアップ |
| `bot-manager/remotion/public/audio/n06.mp3.bak.20260510` | Created | 旧 n06 バックアップ |
| `bot-manager/remotion/out/aicrew-popfriendly.mp4` | Created (1st render) | 中間生成物 |
| `bot-manager/remotion/out/aicrew-popfriendly-v2.mp4` | Created (2nd render) | 最終生成物 |
| `/tmp/aicrew_remotion_repaint.md` | Created | codex への指示書 |
| `/tmp/transcribe_aicrew.py` | Created | Whisper 文字起こしスクリプト |
| `/tmp/regen_aicrew_audio.py` | Created | edge-tts 再生成スクリプト |
