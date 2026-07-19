# afropunx.com 公開手順書

GitHub Pages + カスタムドメイン（afropunx.com）でこのサイトを公開するための手順と実施記録。

- 対象リポジトリ: `zrockafropunx/afropunx-site`（Public）
- ドメイン: `afropunx.com`（レジストラ: GoDaddy）
- 最終更新: 2026-07-19（初回公開作業の実測結果を反映）

## 実施状況（2026-07-19 時点）

| Step | 内容 | 状態 |
|---|---|---|
| 0 | 事前確認 | ✅ 完了 |
| 1 | プレースホルダの実値記入 | ✅ 完了 |
| 2 | GitHub Pages の有効化 | ✅ 完了 |
| 3 | GoDaddy で DNS 変更 | ✅ 完了 |
| 4 | Search Console にドメイン登録 | ✅ 完了 |
| 5 | GitHub verified domain | ✅ 完了 |
| 6 | HTTPS の有効化 | 🔄 **証明書の発行待ち** |
| 7 | WHOIS プライバシー | ✅ 元から有効・作業不要だった |
| 8 | 動作確認 | 🔄 HTTPS 化後 |
| 9 | Play Console にウェブサイト登録 | ⬜ 未着手 |
| 追加 | `.nojekyll` / SPF / DKIM / DMARC | ✅ 完了 |

**サイトは HTTP で公開済み（`http://afropunx.com` が 200 OK）。残るは HTTPS 証明書のみで、GitHub の自動発行待ち。**

---

> **⚠️ 最重要注意（メール停止事故防止）**
>
> afropunx.com の **MX レコード 7 件（`aspmx.l.google.com` ほか Google 系）と、送信ドメイン認証の TXT（SPF / `google._domainkey` の DKIM / `_dmarc` の DMARC）は絶対に変更・削除しない**。触ると support@afropunx.com のメールが停止する、または迷惑メール判定されるようになる。
>
> 触ってよいのは **A / AAAA / www の CNAME と、手順内で明示する TXT の追加のみ**。
>
> ※ 補足: 2026-07-19 の作業開始時点では **TXT レコードは実測 0 件で、SPF も DKIM も DMARC も存在しなかった**（旧版の本書はこれらが存在する前提で書かれていた）。同日に新規追加したため、以降はこの警告がそのまま有効。

---

## 前提: `.nojekyll` は必須（削除禁止）

リポジトリ直下の **`.nojekyll` を削除してはいけない**。

GitHub Pages は既定で Jekyll を通し、Jekyll は `.md` ファイルを Liquid テンプレートとして解釈する。本書や `README.md` には `{{` を含む記述（置換漏れ確認の grep 手順）があり、Liquid がこれを「変数の開始」と解釈して構文エラーになる:

```
Liquid syntax error (line 23): Variable '{{' was not properly terminated with regexp: /\}\}/ in PUBLISH.md
```

その結果 `pages build and deployment` が failure で終了し、**サイトが一切配信されない**（GitHub は "There isn't a GitHub Pages site here." を返す）。証明書の発行もサイトが配信されるまで始まらないため、公開が完全に止まる。

2026-07-19 の初回公開時に実際にこれが発生した。**手順書自身の記述が公開を阻害していた**ことになる。commit `f304f48` で `.nojekyll` を追加して解消済み。

本サイトは `index.html` + `style.css` の静的 HTML のみで Jekyll の機能を一切使わないため、`.nojekyll` で処理をスキップするのが正しい対処。

---

## Step 0: 事前確認

- 2026-07-19 の作業前実測: A レコードは旧 Google 系 IP（216.239.32/34/36/38.21）、www は CNAME → `ghs.google.com`。ただし HTTP 404 / TLS 失敗で**実サイトは配信されていなかった**（旧 Google Workspace 設定の残骸）
- afropunx.com で公開中のもの（旧 Google Sites 等）に心当たりがないか確認する。あれば消える前提でよいか判断してから進む

## Step 1: プレースホルダの実値記入

1. `index.html` の `{{OFFICE_ADDRESS}}` / `{{PHONE}}` を実値（事業所住所・事業用電話番号）に置換して main に反映する
2. 置換漏れ確認: リポジトリルートで `grep -rn "{{" .` → `index.html` に 0 件
   （`README.md` / 本書の `{{` は手順の説明であり残ってよい。`.nojekyll` があるためビルドは壊れない）
3. **登録情報の一致（重要）**: サイトに掲載する「afropunx」表記・住所・電話番号は、**D-U-N-S（東京商工リサーチ / D&B）の登録内容および Google お支払いプロファイルと一致**させる。Play の組織確認は D&B プロファイルとお支払いプロファイルの一致を要求するため、表記揺れ（英字/カナ・番地の書式・ハイフン等）を作らない

> **表記の上流関係**: `開業届 → D-U-N-S → Google お支払いプロファイル → サイト` の順で上流が下流を縛る。**開業届の記載を確定させてからサイトを合わせる**のが正しい順序。サイトは後からいつでも修正できるが、D-U-N-S の登録内容の変更は面倒。

## Step 2: GitHub Pages の有効化

1. リポジトリ → **Settings → Pages**
2. **Build and deployment → Source**: `Deploy from a branch`
3. **Branch**: `main` / フォルダ `/ (root)` → Save
4. **Custom domain** は `CNAME` ファイル（同梱済み・中身 `afropunx.com`）を GitHub が読んで**自動設定される**。空欄なら `afropunx.com` を手入力して Save

> **⚠️ `https://zrockafropunx.github.io/afropunx-site/` での表示確認はできない**
> `CNAME` ファイルがあるため github.io は `afropunx.com` へ 301 リダイレクトする。DNS 切替（Step 3）が済むまでブラウザでは確認できず、エラーに見えるが**故障ではない**。
> この段階でビルドの成否を確認したい場合は Actions タブの `pages build and deployment` の結果を見る（`gh api repos/zrockafropunx/afropunx-site/pages/builds/latest` でも可）。

> **⚠️ DNS 切替前は `DNS check unsuccessful` / `NotServedByPagesError` が表示される**
> これも想定どおり。Step 3 完了後に「Check again」を押せば解消する。

## Step 3: GoDaddy で DNS 変更

GoDaddy → My Products → afropunx.com → **DNS**（直リンク: <https://dcc.godaddy.com/control/afropunx.com/dns>）

### 3-1. apex（@）の A レコードを置換

既存の A レコード 4 件（216.239.32.21 / 216.239.34.21 / 216.239.36.21 / 216.239.38.21）を GitHub Pages の 4 件に置き換える。**削除＋新規作成ではなく、各行の編集（鉛筆アイコン）で値だけ書き換える**のが誤削除を防げて安全:

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

- 「**Parked**」を指す A レコードが残っていれば削除する（2026-07-19 時点では存在しなかった）
- **Domain Forwarding（転送）は完全に OFF**。転送が残っていると GoDaddy が A レコードを勝手に再追加する既知問題がある
  - 確認場所は **DNS レコード一覧ではなく「Forwarding」タブ**。Domain と Subdomains の両方が `Not set up` であること
  - 「Forwarding over HTTPS is now available」という水色バナーは宣伝表示であり、転送が有効という意味ではない
- 繰り返し: **MX と送信ドメイン認証の TXT は触らない**

### 3-4. 実測メモ（伝播時間）

- afropunx.com の TTL は **3600 秒**。実測では**約1時間以内**、レコードによっては数分で伝播した。本書旧版の「最大 24 時間」より大幅に速い
- 新規レコードのデフォルト TTL は 600 秒（SOA 記載値）
- ローカル PC の DNS キャッシュは TTL 切れまで旧 IP を返し続けるため、切替直後の検証は権威ネームサーバー（`ns77.domaincontrol.com`）や `8.8.8.8` を明示して照会するか、`curl --resolve` で DNS を迂回する

## Step 4: Google Search Console にドメイン登録（Play の「ウェブサイトの確認」の実体・必須）

1. **Google Play Console のアカウント所有者と同一の Google アカウント**で <https://search.google.com/search-console> を開く
   - 右上のアカウントアイコンでメールアドレスを必ず確認する。**ここを間違えると Step 9 の確認リクエストが自動承認されず、やり直しになる**
2. プロパティ追加 → **左の「ドメイン」**（右の「URL プレフィックス」ではない）→ `afropunx.com`（`https://` も `www.` も付けない）
3. 所有権の確認
   - **2026-07-19 の実測: GoDaddy が Google のドメイン名プロバイダとして連携しているため、TXT を追加せず「所有権を自動確認しました」で完了した**（本書旧版は手動 TXT 追加の前提で書かれていた）
   - 自動確認されない場合は、表示される TXT（Type: TXT / Name: @ / Value: `google-site-verification=...`）を GoDaddy に追加し「確認」を押す。反映待ちで失敗したら数分置いて再試行
4. 左上のプロパティ表示が `afropunx.com`（URL なし）になっていればドメインプロパティとして正常

> Play Console のウェブサイト確認は Search Console との関連付けで行われ、**同一アカウントで登録済みなら確認リクエストが自動承認**される（[Play Console ヘルプ](https://support.google.com/googleplay/android-developer/answer/13205715)）。

> **⚠️ 既知の弱点**: 自動確認は **DNS 上に痕跡を残さない**。GoDaddy ↔ Google の連携のみに依存するため、レジストラ移管や連携仕様の変更で確認が外れる可能性がある。ドメインプロパティは DNS 確認しか方式が無く、確認済みの状態では追加の確認方法が提示されないため、**現状では冗長化できない**。確認が外れると Play 側の確認にも波及するので、レジストラ移管時は要注意。

## Step 5: （推奨）GitHub の verified domain 設定

ドメイン乗っ取り防止のため、GitHub 側でもドメイン検証しておく。

**目的**: カスタムドメイン設定を外した／リポジトリを消したのに DNS が GitHub を向いたままだと、**第三者が自分の Pages で `afropunx.com` を名乗れてしまう**（dangling DNS によるドメイン乗っ取り）。verified domain を設定すると、このドメインを Pages のカスタムドメインとして使えるのは自分のアカウントだけになる。事業者確認サイトを乗っ取られると Play アカウントごと危険に晒されるため、掛けておくべき鍵。

1. GitHub の **個人 Settings → Pages → Add a verified domain** → `afropunx.com`
   （リポジトリの Settings ではなくアカウントの個人設定）
2. 表示される TXT（Name: `_github-pages-challenge-zrockafropunx` / Value: 表示されたコード）を GoDaddy に追加
   - GoDaddy の「名前」欄には `_github-pages-challenge-zrockafropunx` だけを入れる（`.afropunx.com` は付けない）
3. **Verify を押す**（TXT を入れただけでは完了しない）。`Verified` バッジが付けば完了

## Step 6: HTTPS の有効化

1. DNS 伝播後、リポジトリ **Settings → Pages** に戻る
2. Custom domain の DNS チェックが成功していることを確認（赤いエラーが出ていれば「Check again」）
3. **Enforce HTTPS** にチェック

> DNS チェックが通ってから GitHub が証明書の発行を開始する。**発行まで最大 24 時間**かかり、それまで Enforce HTTPS のチェックボックスは押せない。押せない間は待つしかない。
>
> 発行状況は `gh api repos/zrockafropunx/afropunx-site/pages` の `https_certificate.state` で確認できる（未発行なら `null`）。
>
> 24 時間を大きく超えても発行されない場合は、Custom domain を一度 Remove して再設定すると発行がやり直される。

## Step 7: WHOIS プライバシーの確認

1. GoDaddy → Domain Settings → afropunx.com → **Domain privacy が ON** であることを確認
2. 実地確認: <https://lookup.icann.org> で `afropunx.com` を検索し、登録者の氏名・住所・メールが**開示されていない**ことを確認

> **2026-07-19 の実測: 既に有効で作業不要だった。** 登録者は `Registration Private` / `Domains By Proxy, LLC`（Tempe, Arizona の代理住所・代理電話）で、氏名・所在地・電話は完全に非開示。あわせて `client transfer / update / delete prohibited` のロックも掛かっている。
>
> なお `.com` レジストリ（Verisign）の RDAP は thin 仕様で登録者情報を持たないため、確認にはレジストラ側（`https://rdap.godaddy.com/v1/domain/afropunx.com`）を参照する必要がある。

## Step 8: 動作確認

- [ ] `https://afropunx.com` がこのサイトを表示し、鍵マーク（有効な証明書）が出る
- [ ] `http://afropunx.com` → `https://afropunx.com` にリダイレクトされる
- [ ] `https://www.afropunx.com` → `https://afropunx.com` にリダイレクトされる
- [ ] 配信されているページにプレースホルダ（`{{`）が残っていない
- [ ] メール送受信が正常（support@afropunx.com 宛てにテストメール）
- [ ] **送信認証の確認**: support@afropunx.com から Gmail 宛てに送信し、「メッセージのソースを表示」で `SPF: PASS` と `DKIM: 'PASS' with domain afropunx.com` が出る

## Step 9: Play Console にウェブサイトを登録

1. Play Console → **あなたの情報**（デベロッパー アカウント情報）→ ウェブサイト欄に `https://afropunx.com` を入力して保存
2. **確認リクエストを送信** → 承認されたことを確認（Step 4 が同一アカウントなら自動承認）
3. ウェブサイト確認の完了後に「アカウントの種類を変更」オプションが表示される（[Play Console ヘルプ](https://support.google.com/googleplay/android-developer/answer/16260648)）

---

## 送信ドメイン認証（SPF / DKIM / DMARC）

2026-07-19 の作業開始時点で afropunx.com には **TXT レコードが 1 件も無く、SPF・DKIM・DMARC のすべてが未設定**だった。MX は Google Workspace を向いていたため、support@ は長期間 SPF なしで運用されていたことになる（過去の送信メールが迷惑メール判定されていた可能性がある）。同日に以下を追加した。

| 用途 | Type | Name | Value |
|---|---|---|---|
| SPF | TXT | `@` | `v=spf1 include:_spf.google.com ~all` |
| DKIM | TXT | `google._domainkey` | `v=DKIM1; k=rsa; p=...`（2048bit・Google 管理コンソールで生成） |
| DMARC | TXT | `_dmarc` | `v=DMARC1; p=none; rua=mailto:support@afropunx.com` |

### 注意点

- **SPF は 1 ドメインに 1 本だけ**。別の送信サービスを追加する場合は 2 本目を作らず、既存の 1 本に `include:` を書き足してマージする。2 本になると `permerror` で機能停止する
- **`~all`（ソフトフェイル）を使う**。`-all`（ハードフェイル）にすると、見落とした送信経路があったときにメールが消える
- **DKIM は DNS に入れただけでは有効にならない**。Google 管理コンソール（admin.google.com → アプリ → Google Workspace → Gmail → メールの認証）で**「認証を開始」を押す**必要がある。反映には最大 48 時間
- 2048bit の DKIM 値は 400 文字超になる。GoDaddy は通常自動で分割するが、保存時にエラーが出る場合は 1024bit で生成し直す
- **DMARC は `p=none`（監視のみ・配送影響なし）から始める**。SPF/DKIM が実際に PASS することを確認してから、数週間かけて `quarantine` → `reject` へ段階的に引き上げる。**DKIM が有効化されていない状態で `reject` にすると正規のメールが拒否され、support@ が機能停止する**

---

## 所要時間の目安

- **DNS 伝播は実測で約 1 時間**（TTL 3600）。本書旧版の「最大 24 時間」は保守的すぎた
- **証明書の発行は最大 24 時間**かかる。ここだけは短縮手段がないため、期日がある場合は前倒しする
- Step 9 のウェブサイト確認は後続手続きの前提になるため、期日がある場合は余裕をもって（1 週間前までに）完了させる

> **2026-07-19 の実績**: Step 1〜5 と Step 7、および `.nojekyll` / SPF / DKIM / DMARC の追加を**1 日で完了**した。ボトルネックは DNS 伝播ではなく証明書の発行のみ。

---

## トラブルシューティング

| 症状 | 対処 |
|---|---|
| **サイトが配信されない / "There isn't a GitHub Pages site here."** | `pages build and deployment` が失敗していないか Actions タブを確認。`{{` を含む Markdown が Liquid 構文エラーを起こしていないか。**`.nojekyll` が消えていないか**（本書冒頭の「前提」を参照） |
| Custom domain の DNS チェックが失敗する | 伝播待ち。`nslookup -type=A afropunx.com 8.8.8.8` で 185.199.108〜111.153 が返るか確認。実測 TTL は 3600 秒 |
| Enforce HTTPS が押せない | DNS チェック成功後、証明書発行待ち（最大 24h）。`gh api repos/zrockafropunx/afropunx-site/pages` の `https_certificate.state` で確認。24h 超なら Custom domain を再設定 |
| github.io の URL で 404 / リダイレクトされる | `CNAME` ファイルがあるため正常な挙動。DNS 切替前はブラウザで確認できない（Step 2 参照） |
| ブラウザやローカルでは旧 IP が返る | ローカル DNS キャッシュ。権威 NS や 8.8.8.8 を明示して照会するか `curl --resolve` で迂回して確認する |
| A レコードが勝手に増えている | GoDaddy の Domain Forwarding が ON になっていないか確認（Step 3-3。DNS レコード一覧ではなく Forwarding タブ） |
| メールが届かなくなった | MX を変更していないか即確認。`nslookup -type=MX afropunx.com` で Google 系 MX が **7 件**返ること |
| 送信メールが迷惑メール判定される | SPF/DKIM が PASS しているか受信側のソース表示で確認。DKIM は管理コンソールで「認証を開始」を押したか、反映（最大 48h）が済んでいるかを確認 |
