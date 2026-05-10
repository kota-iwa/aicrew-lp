# AIcrew LP プロジェクト説明書

このプロジェクトは何で、どこにあるか、どう更新するかを kota（非エンジニア）向けに日本語で説明する取扱説明書。

## 1. このプロジェクトは何？

**AIcrew のランディングページ（LP）= 会社のお問い合わせ獲得ページ**

- 公開URL: https://aicrew.c-h.co.jp
- 目的: AIcrew（EC/D2C向けAIカスタマーサポート）の見込み顧客に「資料DL」「無料相談」のフォーム送信をしてもらう
- 構造: 1ファイル完結の `index.html`（インラインCSS + JS、合計 約2700行）
- フォーム送信先: **FormSubmit.co**（`https://formsubmit.co/ajax/aicrew@c-h.co.jp`）→ 受信は `aicrew@c-h.co.jp`

## 2. プロジェクトの場所

| | パス・URL |
|---|---|
| **ローカル（編集はここで）** | `~/projects/aicrew-lp/` |
| **GitHub** | https://github.com/kota-iwa/aicrew-lp |
| **Vercel デプロイ** | aicrew.c-h.co.jp（Vercel管理画面は別アカウント、CLI不可視） |
| **本番ドメイン** | aicrew.c-h.co.jp |

## 3. 構成ファイル

```
~/projects/aicrew-lp/
├── index.html                    # ★メイン。LPの中身ぜんぶ（HTML/CSS/JS）
├── vercel.json                   # Vercel設定（cleanUrls: true）
├── assets/                       # 旧素材（Slack風デザイン時代の画像）
│   ├── AIcrew-demo.mp4
│   ├── AIcrew-demo-poster.jpg
│   ├── favicon.png
│   ├── og-image.png（OG画像 — SNSシェア時の画像）
│   └── ...
├── assets/v2-pop-friendly/       # ★Pop Friendlyリデザイン用の新素材（15枚）
│   ├── hero-main.png             # ヒーロー右側の笑顔の女性
│   ├── hero-chat-mockup.png      # AIチャットUIの画像
│   ├── pain-01〜04.png           # 「こんな悩み」セクションの4枚の人物写真
│   ├── cap-01〜06.png            # 機能セクションの6つのイラストアイコン
│   └── testi-01〜03.png          # お客様の声の3枚の人物写真
├── mocks/                        # （Git管理外）デザイン検討用のモック画像
└── FORkota.md                    # このファイル
```

## 4. データ・更新フロー

### 自動でやってくれること
- **Vercelの自動デプロイ**: GitHub の master ブランチに push すると、1〜3分で aicrew.c-h.co.jp に自動反映される（GitHub連携設定あり）
- バックアップ: 大きな変更前に `index.html.bak.YYYYMMDD` 形式でバックアップが作られる（直近: `index.html.bak.20260510`）

### 手動でやる必要があること
- **FormSubmit 初回確認**: 初めて誰かがフォームから送信したとき、FormSubmit から `aicrew@c-h.co.jp` 宛に「確認メール」が届くので、その中のリンクを 1 度だけクリック → 以降は自動で受信できるようになる
- 画像差し替え: AI 生成画像（`assets/v2-pop-friendly/`）は仮素材。本番運用時は実テスティモニアル写真に差し替え推奨
- ドメイン管理: Vercel 側のチームアカウントは別。アクセスできない場合は kota が直接ログイン

## 5. よく使う操作

### ローカルでLPを見る
```bash
cd ~/projects/aicrew-lp
python3 -m http.server 8770
# ブラウザで http://localhost:8770/index.html を開く
```

### 本番に反映する
```bash
cd ~/projects/aicrew-lp
git add -A
git commit -m "メッセージ"
git push origin master
# 1〜3分後に aicrew.c-h.co.jp に自動反映
```

### デザインを大きく変える
1. モック画像を `gpt-image-2` で生成（`/banana` スキル）
2. 採用案を選ぶ
3. 必要なアセット（写真・イラスト）を `gpt-image-2` で生成 → `assets/v2-pop-friendly/` に保存
4. codex に「このモックに合わせて全面リデザイン」と依頼するか、Claude が直接 Edit
5. ローカルで確認 → push

## 6. デザインの方針（2026-05-10 確定）

- **Pop Friendly テイスト**（Notion / Slack.com / Mercari for Business 系）
- 白ベース + ピーチ #FFE5D9 / ミント #D1F0E0 / スカイ #E0F0FF / バター #FFF4D6 のソフトパステル
- アクセント: コーラル #FF6B6B、オーベルジン #4A154B
- 角丸 16-24px のカード、柔らかいドロップシャドウ
- 親しみやすく温かいトーン

**避けるべき方向**:
- ダークモード（OpenAI/Anthropic系の高級SaaS路線）
- Cyberpunk/ネオン系
- 過度に堅いBtoB高級SaaS路線

## 7. 既知の改善点（CRO観点）

CRO評価で以下の改善点が出てる（2026-05-10時点）:

| 優先度 | 内容 | 効果想定 |
|---|---|---|
| 高 | フォームを6→3フィールドに削減（名前/会社/メールのみ） | CV +40〜60% |
| 高 | Hero に「導入5社平均で月次CS人件費 -60% 実証済み」を追加 | 直帰率 -10〜15% |
| 高 | 「先着3社限定」の sticky bar を Above the fold に常時表示 | CTR +15〜25% |
| 中 | テスティモニアルを実在の3社に差し替え（名前・社名・クオート） | 信頼度 +大 |
| 中 | Pricing 横に「採用 vs AIcrew」ROI比較ボックス | 検討率 +10% |
| 中 | リスクリバーサル（返金保証、解約自由）を Pricing 直下に | CV +15〜20% |
| 中 | 中段（スクロール25/50/80%）に CTA を3箇所追加 | CV +10〜20% |
| 低 | CTA を3階層化（資料DL=メール1項目、料金見る=匿名、デモ=フル） | CV +20〜30% |

## 8. トラブルシューティング

### Q. push したのに本番に反映されない
- 1〜3分待つ
- GitHub Actionsを使ってないので、Vercelの自動デプロイのみ
- Vercel ダッシュボード（kota の別アカウント）で deploy ステータス確認
- 5分以上経っても反映されないなら：DNS or Vercel側の問題、kota が直接 Vercel ログインで確認

### Q. ローカルでブラウザで開くと画像が壊れて見える
- file:// プロトコルだと相対パスが効かない場合がある → `python3 -m http.server` でサーバー経由で開く
- Cmd+Shift+R でハードリロード（キャッシュクリア）

### Q. フォームが送信できない / メールが届かない
- 初回送信時は FormSubmit から `aicrew@c-h.co.jp` に **確認メール（Activate your form）** が届く。リンクを 1 度クリックすれば以降自動受信
- 確認メールが見当たらない: 迷惑メールフォルダもチェック
- それでも届かない: `https://formsubmit.co/aicrew@c-h.co.jp` を別タブで開いて再アクティベート

### Q. デザインを「ダークに戻したい」と言われたら
- メモリに「Pop Friendly路線」が記録されている。ダークに戻すには明示的な指示が必要
- 過去のダーク版モックは `mocks/2026-05-10/variation-{a,b,c}-*.png` に保存（gitignored、ローカルのみ）

## 9. 連絡先

- 問い合わせメール: aicrew@c-h.co.jp（LP内に記載・FormSubmit 受信先）
- 電話: 050-3033-0424（LP内に記載）
- 会社: C&H株式会社
