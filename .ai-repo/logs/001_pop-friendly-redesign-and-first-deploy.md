# 001 — Pop Friendly redesign and first AI-driven deploy

**Date**: 2026-05-10 JST
**Branch**: master
**Commit**: 2da15fc `feat(design): Pop Friendly redesign with new assets`
**Production**: https://aicrew.c-h.co.jp (Vercel auto-deploy via GitHub)
**Backup**: `index.html.bak.20260510` (preserved locally, gitignored)

## Summary

aicrew-lp の全面リデザイン（Slack/Aubergine系 → Pop Friendly系）+ AI生成アセット15枚の差し替え + Vercel本番反映。1セッションで企画→モック→アセット→codex実装→検証→デプロイまで完遂。

## What changed

### Files modified
- `index.html`: 1744行 → 2738行（+994行）。CSS全面書き換え、画像参照を `assets/v2-pop-friendly/` に切替。

### Files added
- `.gitignore`: 新規（`.DS_Store`, `.vercel`, `.playwright-mcp/`, `*.bak.*`, `mocks/`）
- `assets/v2-pop-friendly/*.png`: 15枚（hero 2, pain 4, capabilities 6, testimonials 3）
- `FORkota.md`: プロジェクト説明書（kota向け、非エンジニア向け）

### Image generation
- ツール: OpenAI `gpt-image-2 (low)`
- 合計: 21枚（モック6 + アセット15）
- 実コスト: $0.146（モック $0.046 + アセット $0.10）
- 生成済モックは `mocks/2026-05-10/` `mocks/2026-05-10-v2-light/` にローカル保存（gitignored）

## Process flow

1. **モック生成3案 × ダーク** → kota が全却下（"ダークモードじゃない、もっとカジュアル"）
2. **モック再生成3案 × 明るい/カジュアル** → Variation A (Pop Friendly) 採用
3. **アセット15枚生成**（OpenAI billing hard limit に2回ブロック → kota がチャージで解決）
4. **codex に全面リデザイン依頼**（最初は cwd=bot-manager で sandbox 拒否、約24分使ってキャンセル）
5. **codex を `--cd /Users/kotaiwama/projects/aicrew-lp --full-auto` で再起動** → 約16分で実装完了
6. **Playwright で検証** → 画像 aspect-ratio が効かないバグ発見、`height: auto` で解決
7. **reveal アニメーション安全タイマー追加**（IntersectionObserver 初動失敗対策、1.2秒で強制 visible）
8. **commit + push** → Vercel 自動デプロイ → 約1分で aicrew.c-h.co.jp 反映

## Critical bugs found and fixed

### Bug 1. CSS aspect-ratio が画像のHTML属性に上書きされる
**症状**: `.cap-card__media img { width: 100%; aspect-ratio: 1/1 }` を書いても、画像がHTML属性 `width="1024" height="1024"` の自然サイズで表示され、カードが異常に縦長になる（特に Pain Points / Capabilities / Results）。

**原因**: HTML属性の `height="1024"` が CSS の `aspect-ratio` の計算を阻害。`aspect-ratio` は両方のdimensionが auto のときのみ機能。

**修正**: CSS に `height: auto` を明示追加。3箇所（`.pain-card__media img`, `.cap-card__media img`, `.result-highlight img`）に適用。

**学び**: 今後 codex / 自分で `aspect-ratio` を使うときは `height: auto` を必ずセットで書く。HTML属性はそのまま残してOK（browser に intrinsic size hint として有効、CLS 改善に貢献）。

### Bug 2. codex の sandbox restriction
**症状**: codex を bot-manager の cwd から起動して aicrew-lp に書き込ませようとすると拒否される。

**原因**: codex のサンドボックスが cwd 外のファイルへの書き込みを拒否。

**回避策**: `codex exec --cd <target-dir> --full-auto` で対象ディレクトリを cwd に明示。

**今後の運用**: 兄弟プロジェクトを codex に触らせるときは必ず `--cd` を付ける。Skill `codex:rescue` の引数だけだと cwd 制御できないので、複数プロジェクト跨ぎは `codex exec` 直接コールが安全。

### Bug 3. IntersectionObserver の初動失敗
**症状**: ページロード直後、ファーストビューに入る要素以外は opacity:0 のまま動かないことがある（Playwright スクショで再現）。

**原因**: `.reveal { opacity: 0 }` で初期非表示にし、observer 発火で `.visible` クラス追加。observer が何らかの理由で発火しないと永続非表示。

**修正**: `setTimeout(() => reveals.forEach(el => el.classList.add('visible')), 1200)` の安全タイマーを追加。observer が正常動作すれば 100ms 以内に visible になり、フォールバックは効かない（unobserve 済みなので無害）。

## Cost & timing

| フェーズ | 時間 | コスト |
|---|---|---|
| モック生成（2回 × 6枚） | 約3分 | $0.087 |
| アセット生成（15枚） | 約1分（並列） | $0.10 |
| codex 実装（最終） | 約16分 | 243k tokens |
| バグ修正 + 検証 | 約20分 | （主にClaude） |
| デプロイ + 検証 | 約2分 | $0 |
| **合計** | **約1時間** | **$0.187 (LP直接)** + Claude usage |

## Open issues / TODO

1. **テスティモニアル本物化**: 現状の `testi-01〜03.png` はAI生成。実テスト導入企業3社のクオート + 写真 or ロゴに差し替え必要
2. **Formspree ID 未設定**: `index.html` 内に `REPLACE_WITH_FORMSPREE_ID` のままなのでフォーム送信不可。本番運用前に Formspree でフォーム作成 → ID埋める
3. **Vercel CLI不可視**: 現Vercelデプロイは別アカウント管理（kota本人）。kota-elife アカウントの CLI からは見えない。緊急時は kota がブラウザで直接Vercel ログインする必要
4. **CRO改善案 8件**: 別途評価済（FORkota.md 内 §7参照）。フォーム削減・希少性バー・ROI比較ボックス等、優先度別に実装計画あり

## References

- Mock images (gitignored, local only): `~/projects/aicrew-lp/mocks/2026-05-10-v2-light/variation-a-pop-friendly-{pc,sp}.png`
- Backup: `~/projects/aicrew-lp/index.html.bak.20260510`
- FORkota.md: `~/projects/aicrew-lp/FORkota.md` (project overview for non-engineer)
- Memory: `feedback_design_aicrew_lp.md`（kota の Pop Friendly 路線記録）
