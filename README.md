# afropunx-site

[afropunx.com](https://afropunx.com) で公開する事業者情報サイト。ビルド不要の静的 HTML/CSS のみで構成する。

## 構成

| ファイル | 内容 |
|---|---|
| `index.html` | 1 枚サイト本体（事業者名・事業内容・アプリ紹介・連絡先） |
| `style.css` | スタイル（外部フォント・CDN・JS なし） |
| `CNAME` | GitHub Pages のカスタムドメイン設定（`afropunx.com`） |
| `PUBLISH.md` | 公開手順書（Pages 設定・DNS・各種確認） |

## プレースホルダ（公開前に実値へ置換）

| プレースホルダ | 置換する値 |
|---|---|
| `{{OFFICE_ADDRESS}}` | 事業所住所 |
| `{{PHONE}}` | 事業用電話番号 |

置換漏れの確認: `grep -rn "{{" .` が 0 件になること。

## 方針

- 素の静的 HTML/CSS のみ（フレームワーク・ビルド・JS なし）。ページ追加はルートに `*.html` を足すだけ
- 掲載する連絡先は事業用のもののみ（事業者名 / 事業所住所 / 事業用電話 / support@afropunx.com）
- 特定商取引法に基づく表記のページは、必要になった時点で `tokushoho.html` として追加する（現時点では未作成・リンクも置かない）
- 外部リンクは当面置かない（アプリ関連ページの URL 構成が確定してからリンクを検討する）

## 関連

- アプリ「推し録（oshiroku）」の法務ページ（プライバシーポリシー・利用規約）: <https://zrockafropunx.github.io/oshiroku/> — アプリ側リポジトリで管理しており、本リポジトリとは独立
