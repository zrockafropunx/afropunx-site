# afropunx.com 公開手順書

GitHub Pages + カスタムドメイン（afropunx.com）でこのサイトを公開するための手順。上から順に実施する。

- 対象リポジトリ: `zrockafropunx/afropunx-site`（Public）
- ドメイン: `afropunx.com`（レジストラ: GoDaddy）
- 目安時間: 作業 30 分 + DNS 伝播/証明書発行の待ち（それぞれ最大 24 時間）

> **⚠️ 最重要注意（メール停止事故防止）**
> afropunx.com の **MX レコード（`aspmx.l.google.com` ほか Google 系）と SPF/DKIM の TXT レコードは絶対に変更・削除しない**。
> ここを触ると support@afropunx.com を含む既存のメールがすべて止まる。本手順で触ってよいのは **A / AAAA / www の CNAME と、手順内で明示する確認用 TXT の追加のみ**。

---

## Step 0: 事前確認

- 2026-07-19 時点の実測: afropunx.com の A レコードは旧 Google 系 IP（216.239.32/34/36/38.21）に向いているが、HTTP 404 / TLS 失敗で**実サイトは配信されていない**（旧 Google Workspace 設定の残骸とみられる）。www は CNAME → `ghs.google.com`
- 念のため、afropunx.com で公開中のもの（旧 Google Sites 等）に心当たりがないか確認する。あれば消える前提でよいか判断してから進む

## Step 1: プレースホルダの実値記入

1. `index.html` の `{{OFFICE_ADDRESS}}` / `{{PHONE}}` を実値（事業所住所・事業用電話番号）に置換して main に反映する
2. 置換漏れ確認: リポジトリルートで `grep -rn "{{" .` → 0 件
3. **登録情報の一致（重要）**: サイトに掲載する「afropunx」表記・住所・電話番号は、**D-U-N-S（東京商工リサーチ / D&B）の登録内容および Google お支払いプロファイルと一致**させる。Google Play の組織確認は D&B プロファイルとお支払いプロファイルの一致を要求するため、表記揺れ（英字/カナ・番地の書式・ハイフン等）を作らない

## Step 2: GitHub Pages の有効化

1. リポジトリ → **Settings → Pages**
2. **Build and deployment → Source**: `Deploy from a branch`
3. **Branch**: `main` / フォルダ `/ (root)` → Save
4. 数分後に `https://zrockafropunx.github.io/afropunx-site/` で表示されることを確認
5. **Custom domain** に `afropunx.com` を入力 → Save（`CNAME` ファイルは同梱済みのため整合する）

## Step 3: GoDaddy で DNS 変更

GoDaddy → My Products → afropunx.com → **DNS** で以下を設定する。

### 3-1. apex（@）の A レコードを置換

既存の A レコード 4 件（216.239.32.21 / 216.239.34.21 / 216.239.36.21 / 216.239.38.21）を**削除**し、GitHub Pages の 4 件に置き換える:

| Type | Name | Value |
|---|---|---|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |

（推奨）IPv6 も追加:

| Type | Name | Value |
|---|---|---|
| AAAA | @ | 2606:50c0:8000::153 |
| AAAA | @ | 2606:50c0:8001::153 |
| AAAA | @ | 2606:50c0:8002::153 |
| AAAA | @ | 2606:50c0:8003::153 |

### 3-2. www の CNAME を変更

| Type | Name | 変更前 | 変更後 |
|---|---|---|---|
| CNAME | www | ghs.google.com | `zrockafropunx.github.io` |

（値はユーザー名のみ。`afropunx-site` などリポジトリ名は含めない）

### 3-3. GoDaddy 特有の注意

- 「**Parked**」を指す A レコードが残っていれば削除する
- **Domain Forwarding（転送）設定は完全に OFF** にする（転送設定が残っていると GoDaddy が A レコードを勝手に再追加する既知問題がある）
- 繰り返し: **MX と SPF/DKIM TXT は触らない**

## Step 4: Google Search Console にドメイン登録（Play の「ウェブサイトの確認」の実体・必須）

1. **Google Play Console のアカウント所有者と同一の Google アカウント**で <https://search.google.com/search-console> を開く
2. プロパティ追加 → **ドメイン**（URL プレフィックスではなくドメインプロパティ）→ `afropunx.com`
3. 表示される確認用 **TXT レコード**を GoDaddy の DNS に追加（Type: TXT / Name: @ / Value: `google-site-verification=...`）
4. Search Console で「確認」→ 完了になることを確認（TXT の反映には数分〜数時間かかる場合がある。失敗したら時間を置いて再試行）

> Play Console のウェブサイト確認は Search Console との関連付けで行われ、**同一アカウントで登録済みなら確認リクエストが自動承認**される（[Play Console ヘルプ](https://support.google.com/googleplay/android-developer/answer/13205715)）。

## Step 5: （推奨）GitHub の verified domain 設定

ドメイン乗っ取り防止のため、GitHub 側でもドメイン検証しておく。

1. GitHub の **個人 Settings → Pages → Add a verified domain** → `afropunx.com`
2. 表示される TXT レコード（Name: `_github-pages-challenge-zrockafropunx` / Value: 表示されたコード）を GoDaddy に追加
   （Step 4 の Search Console 用 TXT とは**別物**。両方追加してよい）
3. Verify を押して完了を確認

## Step 6: HTTPS の有効化

1. DNS 伝播後（数分〜最大 24 時間）、リポジトリ **Settings → Pages** に戻る
2. Custom domain の DNS チェックが成功していることを確認
3. **Enforce HTTPS** にチェック（証明書発行まで最大 24 時間かかる。チェックボックスが押せない間は待つ）

## Step 7: WHOIS プライバシーの確認

1. GoDaddy → Domain Settings → afropunx.com → **Domain privacy が ON** であることを確認（OFF なら ON にする）
2. 実地確認: <https://lookup.icann.org> で `afropunx.com` を検索し、登録者の氏名・住所・メールアドレスが**開示されていない**こと（Redacted / プライバシー代理表記になっていること）を確認

## Step 8: 動作確認

- [ ] `https://afropunx.com` がこのサイトを表示し、鍵マーク（有効な証明書）が出る
- [ ] `http://afropunx.com` → `https://afropunx.com` にリダイレクトされる
- [ ] `https://www.afropunx.com` → `https://afropunx.com` にリダイレクトされる
- [ ] ページ内にプレースホルダ（`{{`）が残っていない
- [ ] メール送受信が正常（support@afropunx.com 宛てにテストメール）

## Step 9: Play Console にウェブサイトを登録

1. Play Console → **あなたの情報**（デベロッパー アカウント情報）→ ウェブサイト欄に `https://afropunx.com` を入力して保存
2. **確認リクエストを送信** → 承認されたことを確認（Step 4 が同一アカウントなら自動承認）
3. ウェブサイト確認の完了後に「アカウントの種類を変更」オプションが表示される（[Play Console ヘルプ](https://support.google.com/googleplay/android-developer/answer/16260648)）

## 所要時間の目安

- DNS 伝播と証明書発行はそれぞれ最大 24 時間かかるため、1 日で終わらせようとせず 2 日に分けると確実
- Step 9 のウェブサイト確認は後続手続きの前提になるため、期日がある場合は余裕をもって（1 週間前までに）完了させる

## トラブルシューティング

| 症状 | 対処 |
|---|---|
| Custom domain の DNS チェックが失敗する | 伝播待ち（最大 24h）。`nslookup afropunx.com` で 185.199.108〜111.153 が返るか確認 |
| Enforce HTTPS が押せない | DNS チェック成功後、証明書発行待ち（最大 24h） |
| A レコードが勝手に増えている | GoDaddy の Domain Forwarding が ON になっていないか確認（Step 3-3） |
| メールが届かなくなった | MX レコードを変更していないか即確認。`nslookup -type=MX afropunx.com` で Google 系 MX が返ること |
