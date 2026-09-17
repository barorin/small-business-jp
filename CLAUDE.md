# CLAUDE.md

このリポジトリは、Anthropic の [knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins)
にある `small-business/` プラグインを、日本の中小企業向けに全面的に書き換えたもの。
プラグイン名とマーケットプレイス名はどちらも `small-business-jp`。すべて Markdown と
JSON で、ビルドやテストはない。

## リモート

| リモート | URL | 用途 |
|---|---|---|
| `origin` | `https://github.com/barorin/small-business-jp.git` | このリポジトリ |
| `upstream` | `https://github.com/anthropics/knowledge-work-plugins.git` | 取得専用。`small-business/` だけを見る |

upstream とは履歴のつながりがなく、ファイルの配置（upstream の `small-business/` 以下が
こちらの直下）も中身（日本語化）も違う。**`git merge` / `cherry-pick` / `rebase` で
upstream を取り込まない。** 変更は差分を読んで手で移植する。

`upstream` リモートはクローンごとの設定なので、無ければ作る:

```bash
git remote add -t main --no-tags upstream https://github.com/anthropics/knowledge-work-plugins.git
git remote set-url --push upstream DISABLE
```

## upstream の追従

「upstream の変更を確認して」と頼まれたら、この手順で進める。

**移植済みの位置は `.upstream-base` に記録する**（upstream のコミット SHA を 1 行）。
タグは使わない: upstream のコミットを指すタグを origin にプッシュすると upstream の
全履歴が載ってしまい、プッシュしなければ他の環境に残らないため。

1. **取得と一覧。**
   ```bash
   git fetch upstream
   BASE=$(cat .upstream-base)
   git log --oneline --date=short --format='%h %ad %s' "$BASE"..upstream/main -- small-business/
   git diff -M --stat "$BASE" upstream/main -- small-business/
   ```
   何も出なければ「変更なし」と報告して終わる。`.upstream-base` は動かさない。
2. **差分を読んで仕分ける。** `git diff -M "$BASE" upstream/main -- small-business/<path>`
   でファイルごとに読み、下の対応表でこちらのファイルに当てはめたうえで、変更ごとに
   次のどれかに分類する:
   - **移植** — 国を問わない改善（手順・安全ゲート・バグ修正・新しいスキルの骨格など）。
   - **日本向けに書き換えて移植** — 趣旨は取り込むが、米国固有の制度・製品を日本の
     ものに置き換える必要があるもの。
   - **見送り** — 日本では意味がないもの（例: 1099、四半期予定納税、QuickBooks /
     Xero / MYOB / Gusto 専用の手順、日本版で削除したコネクタだけに関わる変更）。
3. **報告して了承を得る。** 編集の前に、コミット単位または変更単位で「何が変わったか /
   分類 / こちらのどのファイルをどう直すか / 見送る理由」を一覧で示す。
4. **移植する。** 了承された分だけ、下の「移植のルール」に従って編集する。
5. **`.upstream-base` を進める。** 移植（と見送りの判断）が終わったら、確認した範囲の
   末尾のコミット（通常は `git rev-parse upstream/main`）の SHA を書き込む。移植の
   変更と同じコミットに含める。途中までしか終わっていなければ動かさない。
6. **コミット・プッシュはユーザーの指示があってから。** コミットメッセージには、
   移植した upstream のコミット範囲（`<旧 BASE の短縮SHA>..<新 BASE の短縮SHA>`）を書く。

### ファイルの対応表

upstream の `small-business/<path>` は、基本的にこちらの `<path>` に対応する。
日本版で名前を変えた・置き換えたものは次のとおり。

| upstream（`small-business/` 以下） | こちら |
|---|---|
| `shared/quickbooks-report-traps.md` | `shared/ledger-report-traps.md` |
| `skills/month-end-prep/reference/quickbooks-reconcile.md`<br>`skills/month-end-prep/reference/xero-reconcile.md`<br>`skills/month-end-prep/reference/zoho-books-reconcile.md` | `skills/month-end-prep/reference/freee-reconcile.md`<br>`skills/month-end-prep/reference/moneyforward-reconcile.md`<br>`skills/month-end-prep/reference/yayoi-csv-reconcile.md`<br>（1 対 1 ではない。照合手順の共通部分の変更だけを 3 つに反映する） |
| `skills/social-content-engine/reference/examples/okonkwo-campaign.md` | `skills/social-content-engine/reference/examples/okamoto-campaign.md` |
| `skills/tax-season-organizer/reference/examples/quarterly-estimate.md` | `skills/tax-season-organizer/reference/examples/tax-reserve-estimate.md` |
| `skills/tax-season-organizer/reference/examples/year-end-1099.md` | `skills/tax-season-organizer/reference/examples/year-end-houtei-chosho.md` |

- **`.mcp.json`**: 日本版では `gusto`、`intuit-quickbooks`、`myob`、`ramp`、`ringex-chat` を
  削除している。これらのサーバーだけに関わる upstream の変更は見送る。それ以外の
  サーバーの URL 変更などは移植する。
- **こちらにしかないファイル**: `.claude-plugin/marketplace.json`（upstream ではリポジトリ
  直下にある別物）、`LICENSE`（同じくリポジトリ直下）、`CLAUDE.md`、`.upstream-base`。
- 移植でファイルの追加・改名が起きたら、この対応表も更新する。

### 移植のルール

- **日本語で書く。** 既存の文体に合わせる: `README.md` はです・ます調、`SKILL.md`・
  `reference/`・`shared/` はだ・である調。英語の原文をそのまま残さない。
- **応答言語の一行を残す。** すべての `SKILL.md` は frontmatter の直後に「**応答は常に日本語で書く。** …（`../../shared/response-language.md`）」の一行を持つ。upstream から新しいスキルを移植するときも必ず入れる。
- **制度と製品を日本のものに置き換える。** 例: QuickBooks / Xero / Zoho Books /
  MYOB → freee会計・マネーフォワード クラウド会計・弥生会計、1099 → 法定調書・
  支払調書、sales tax → 消費税・インボイス制度、quarterly estimated tax → 予定納税、
  Gusto → 給与計算ソフトの CSV 連携、$ → 円。置き換え先が無ければ見送りを提案する。
- **プラグイン名は `small-business-jp`。** upstream の `small-business:<name>` は
  `small-business-jp:<name>` に、`mcp__plugin_small-business_` は
  `mcp__plugin_small-business-jp_` にする。
- **バージョン。** upstream の `.claude-plugin/plugin.json` の `version` が上がっていたら
  こちらも同じ番号にし、`README.md` 末尾の「現在のバージョン: X.Y.Z（日本版） —
  <移植した日付>」も直す。各 `SKILL.md` の `version` も upstream に合わせる。
- **README の数と一覧。** スキルの追加・削除があれば、`README.md` の「44 のスキル」などの
  個数、カテゴリ別の一覧、「リポジトリの構成」を合わせて直す。
- **専門的な判断を創作しない。** 税務・労務・法務の日本の制度に置き換えるとき、
  根拠のない数字や期限を書かない。確かでなければ報告の中でユーザーに確認する。
