# アーティファクトのスタイル — ハウスルック

**プラグイン全体で共有。** スキルが生成するすべてのページが、46種類の別製品
ではなく、ひとつの製品として読めるようにするための唯一のビジュアル体系。
HTML アーティファクトを作る前に必ず読む。スキルごとに新しい配色を考案しない。

**目的は装飾ではない。** これは中小企業の経営者のための事務ツールだ — 架電
リスト、資金繰りスナップショット、発注リスト。派手さより、整っていて読みやす
いこと。ヒーローバナーも、マーケティング風のグラデーションも、大げさな理念の
掲示もいらない。ランディングページではなく、帳簿だ。

**金額は円で示す。** ページに載せる数値はすべて `1,240円` の書式（半角数字＋
3桁区切り＋「円」。小数は使わない）。外貨建ての取引だけ ISO コードを前置する
（`USD 1,200`、`EUR 850`）。通貨は `## Business context` ブロックの「通貨」行
から `shared/currency-and-locale.md` に従って読む。以下の例に出てくる金額は書式
の見本であり、既定値ではない。

---

## そもそもアーティファクトを使うべきか

すべてのスキル実行にページが必要なわけではない。ページを作るのは、出力が経営者
にとって **あとで見返す、印刷する、誰かに渡す、貼り付けて使う** ものであるとき
— ダッシュボード、架電リスト、チェックリスト、コピーして使う文面付きの提案など。
数字ひとつや一行の答えならページは不要。チャットの返答そのものが成果物であり、
一文のためのページはノイズでしかない。

**アーティファクトは追加であって、代替ではない。** 経営者はチャットで短い答え
を必ず受け取る。ページは全体像を見に行く場所だ。

**経営者が保存した設定は既定値に優先する。** `## Business context` ブロックの
「出力形式」行（`smb-onboard` で取得）を確認する。値は 6 種類：

- `visual artifacts`（または未設定）— ハウススタイルのアーティファクト。既定値。
- `docx` — ページの代わりに DOCX 文書を渡し、その理由を添える。
- `md` — ページの代わりに Markdown ファイルを渡し、その理由を添える。
- `notion` — Notion コネクタで、経営者が指定したワークスペース内の場所に
  Notion ページとして成果物を作成し、その理由を添える。ページ作成は書き込み
  なので、作成前に保存先を明示し、既存ページを決して上書きしない。この実行で
  Notion が接続されていなければ、そう伝えてビジュアルアーティファクトに戻す —
  成果物を止めない。
- `canva` — Canva コネクタで Canva Doc として成果物を作成する。手順は 2 回の
  呼び出し：`generate-design` をデザインタイプ `doc`、verbatim オン、クエリに
  全文を入れて呼ぶ（返るのは候補であってデザインではない）。次に最初の候補に
  対して `create-design-from-candidate` を呼ぶと、デザイン id とリンクが返る
  ので、それを経営者に渡す。コネクタが `create-design` を提供している場合は、
  この 2 回の代わりにそれを使う。方針は常に新規デザインで、既存デザインの編集
  はしない。名前を付けるには `read-design` でトランザクションを開き、
  `edit-design` で `update_title` 操作を 1 回適用（ページ名、事業者名、日付）
  してコミットし、実行同士が衝突しないようにする。Canva Doc に入るのは見出し、
  段落、太字、斜体、リストのみ — 表は 1 行 1 項目のリストに直し、グラフ・
  画像・リンクは入れない。ブランドキット：スキルが初めて Canva に出力するとき
  `list-brand-kits` を実行し、どのキットを適用するか（または適用しないか）を
  一度だけ聞き、キットの名前と id をプロファイルの「Canvaブランドキット」行に
  記録する。以降の実行はその id を `brand_kit_id` として渡し、聞き直さない。
  この実行で Canva が接続されていなければ、そう伝えてビジュアルアーティファクト
  に戻す — 成果物を止めない。
- `best for skill` — 成果物ごとにスキルが判断する：ダッシュボードや画面は
  アーティファクトとして描画し、提出・印刷・他所で編集されるもの（提案書、
  申請書の本文、修正案）はスキルがすでに指定している文書形式で出す。

単発の依頼（「これは Word で欲しい」）は、その実行に限り、保存された設定より
常に優先する。

**1 ページ＝1 成果物 — HTML を二度渡さない。** アーティファクトを作る過程で
先に HTML ファイルを書くことはあるが、そのファイルは足場であって成果物ではない。
アーティファクトを公開したら、そこで止める。同じ .html をダウンロード用に
出したり、添付したり、ページと並べて提示したりしない — アーティファクトと同一
の HTML ダウンロードを両方渡された経営者は、どちらが本物かを考えることになり、
答えは常にアーティファクトだ。アーティファクトと並べて出してよいファイルは、
スキルがもともと約束している本当に別の形式（XLSX のワークブック、DOCX の修正案、
PDF の資料一式）だけ。同じページを別の包みで出した二つ目のコピーは、それに
当たらない。

---

## ブランドトークン — ハウス既定、事業者ごとに差し替え可

以下の色とフォントはすべて CSS カスタムプロパティ。**事業者自身のブランドが
分かっている場合** — `smb-onboard` で Web サイトから取得し、セッションメモリの
`## Business context` ブロック（Webサイト／ブランドカラー／ロゴの各行）に保存
された色とロゴ — は、色トークンをそれで上書きする。**分かっていない場合**は
ハウス既定をそのまま使う。どちらの場合も、以下のコンポーネントのパターンは
変えない。変わるのはトークンの値だけだ。

**保存されたブランドは、すべてのアーティファクトに自動で適用される。**
`## Business context` ブロックにブランドカラー（と任意でロゴ）が入った時点で、
どのスキルが描画するページもそれを使う — 経営者は初期設定のときに一度だけ
カスタマイズし、ページごとには行わない。ブランドの取得は `smb-onboard` の
役目で、手間をかけさせてはならない：Web サイトのリンクの貼り付けか、言葉での
説明（「深い緑とクリーム色」）だけ。16進コード、カラーピッカー、アップロード
は決して求めない。

```css
:root {
  /* ハウス既定パレット — ブランドカラーが分かっていれば上書きする */
  --ink: #17211F;        /* 本文の文字色。暖かみのある黒に近い色 */
  --paper: #F4F3EE;      /* ページ背景。クリームではなく冷たい石色 */
  --paper-raised: #FFFFFF; /* --paper の上に載るカード面 */
  --line: #DEDCD1;       /* ヘアラインの罫線・境界線 */
  --accent: #0B6B5C;     /* 主アクセント — 落ち着いた帳簿の青緑 */
  --accent-soft: #E4EFEC; /* ピル背景やハイライト用のアクセント淡色 */
  --amber: #B9781F;      /* 副アクセント — 「要確認」のシグナル */
  --amber-soft: #F6EBDA;

  /* 意味色 — アクセントとは別。ブランド用途に転用しない */
  --good: #2E7D4F;
  --good-soft: #E4F0E8;
  --warn: #B9781F;
  --warn-soft: #F6EBDA;
  --critical: #B3392C;
  --critical-soft: #FBEAE7;

  /* 書体 — 事業者独自のフォントがあればファミリー名を上書きする */
  --font-display: 'Noto Serif JP', 'Zilla Slab', Georgia, serif;
  --font-body: 'BIZ UDPGothic', 'Noto Sans JP', -apple-system, sans-serif;
  --font-mono: 'IBM Plex Mono', 'Noto Sans JP', 'SF Mono', Consolas, monospace;
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --ink: #EDEAE2;
    --paper: #14201C;
    --paper-raised: #1B2A25;
    --line: #2C3A34;
    --accent: #4FBFA5;
    --accent-soft: #1E332C;
    --amber: #E0A559;
    --amber-soft: #362A16;
    --good: #6FCB94;
    --good-soft: #1D3226;
    --warn: #E0A559;
    --warn-soft: #362A16;
    --critical: #E37B6B;
    --critical-soft: #3A211C;
  }
}

:root[data-theme="dark"] {
  --ink: #EDEAE2;
  --paper: #14201C;
  --paper-raised: #1B2A25;
  --line: #2C3A34;
  --accent: #4FBFA5;
  --accent-soft: #1E332C;
  --amber: #E0A559;
  --amber-soft: #362A16;
  --good: #6FCB94;
  --good-soft: #1D3226;
  --warn: #E0A559;
  --warn-soft: #362A16;
  --critical: #E37B6B;
  --critical-soft: #3A211C;
}
```

**フォントは Google Fonts から読み込む** — アーティファクトのサンドボックスが
許可する唯一のホストだ：

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@500;600&family=BIZ+UDPGothic:wght@400;700&family=Noto+Sans+JP:wght@400;500;700&family=IBM+Plex+Mono:wght@500&display=swap" rel="stylesheet">
```

事業者のブランドフォントが Google Fonts にない場合は、CSP と格闘するよりハウス
既定を保つ — 近くて確実に動く代替のほうが、読み込みに黙って失敗するフォント
より良い。

**初期設定時のブランドトークン抽出。** `smb-onboard` が事業者の Web サイトを
読むとき（そのスキルのワークフローを参照）、主要な背景色・アクセント色と、
サイトが宣言している font-family を取り出す。見つかったものを経営者に示し —
*「御社のサイトは深い緑とセリフ体を使っていますね — レポートも合わせましょう
か？」* — 「はい」のときだけ適用する。確定した値は `## Business context`
ブロック（ブランドカラー／ロゴの行。使える font-family が見つかれば「ブランド
フォント」行を追加）に保存し、どのスキルのアーティファクトも再導出せずに同じ
トークンを読めるようにする。

---

## 書体

- **見出し用**（`--font-display`）はページタイトルとセクション見出しのみ。
  ウェイト 600、`text-wrap: balance`。本文には決して使わない。
- **本文用**（`--font-body`）は文章として読まれるすべてに。
- **等幅**（`--font-mono`）は列に並ぶすべての数字に — 金額、件数、表の中の
  日付。`font-variant-numeric: tabular-nums` と組み合わせる。

タイプスケールを決め、そこから外れない：

```css
--text-xs: 0.75rem;   --text-sm: 0.875rem;  --text-base: 1rem;
--text-lg: 1.125rem;  --text-xl: 1.375rem;  --text-2xl: 1.75rem;
--text-3xl: 2.25rem;  /* ダッシュボードの大きな数字ひとつ用。控えめに */
```

---

## レイアウト — 帳簿カードのパターン

コンテンツは枠付きパネルに置く（`--paper` の上に `--paper-raised`、`1px solid
var(--line)`、`border-radius: 6px` — わずかで意図的な角丸であって、既定で何に
でも `rounded-lg` ではない）。パネルは margin ではなく `gap` で積む。幅の広い
表には専用の `overflow-x: auto` ラッパーを付け、ページが横スクロールしないよう
にする。

```css
.panel {
  background: var(--paper-raised);
  border: 1px solid var(--line);
  border-radius: 6px;
  padding: 1.5rem;
}
.page {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  max-width: 840px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
}
```

---

## コンポーネント

### ヘッダー

すべてのアーティファクトは同じ形で始まる：事業者名（まだ分からなければプラグ
イン名）、そのページの役割を一行で、そして生成日。ヒーローもイラストもなし。

```html
<header class="page-header">
  <div class="brand-mark">岡</div>
  <div>
    <h1>資金繰りスナップショット</h1>
    <p class="meta">岡本設備工業 · 2026年8月21日 生成</p>
  </div>
</header>
```

`.brand-mark` はブランドマーク（モノグラム）。背景 `--accent-soft`、文字
`--accent`、書体 `--font-display` — どのページでも同じ形で、屋号の頭1文字
（漢字）またはローマ字2文字を入れる。

### 数値タイル

いちばん大切な数字ひとつを大きく。下にはラベルだけでなく、平易な一行を添える。

```html
<div class="stat-tile">
  <div class="stat-figure">6,240,000円</div>
  <div class="stat-label">手元資金</div>
  <p class="stat-context">現在の支出ペースで約11週間分の余裕。</p>
</div>
```

`.stat-figure` は `--font-mono`、`--text-3xl`、`tabular-nums`。役に立つのは
文脈の一行だ — 数字だけでは答えにならない。

### ステータスピル

丸い錠剤型のバッジではなく、小さなスタンプのようなタグ — 長方形にわずかな
角丸、英字は大文字、字間を広めに。意味色のみを使う。

```html
<span class="status status--good">順調</span>
<span class="status status--warn">要確認</span>
<span class="status status--critical">期限超過</span>
```

```css
.status {
  display: inline-block;
  padding: 0.15rem 0.55rem;
  border-radius: 3px;
  font: 600 var(--text-xs) var(--font-body);
  text-transform: uppercase;
  letter-spacing: 0.04em;
}
.status--good { background: var(--good-soft); color: var(--good); }
.status--warn { background: var(--warn-soft); color: var(--warn); }
.status--critical { background: var(--critical-soft); color: var(--critical); }
```

### 表

ヘアラインの行区切りだけで、重い格子線は使わない。数字は `--font-mono` で
右揃え。幅のために `.table-scroll` で包む。

```html
<div class="table-scroll">
  <table>
    <thead><tr><th>取引先</th><th>金額</th><th>超過日数</th></tr></thead>
    <tbody>
      <tr><td>株式会社コーウィン商会</td><td class="num">182,400円</td><td class="num">14</td></tr>
    </tbody>
  </table>
</div>
```

### コピーブロック

経営者が他所に貼り付けるための文面用 — SEO の修正案、メールの下書き、求人票。
ボタンは Clipboard API でコピーし、サンドボックスがそれをブロックした場合は
手動選択にフォールバックする（必ずテストする — 常に動くと決めつけない）。

```html
<div class="copy-block">
  <button class="copy-btn" onclick="copyBlock(this)">コピー</button>
  <pre>トップページの title タグにこれを追加: 「岡本設備工業 — さいたま市のエアコン修理・即日対応」</pre>
</div>
<script>
function copyBlock(btn) {
  const text = btn.nextElementSibling.textContent;
  const done = () => { btn.textContent = 'コピーしました'; setTimeout(() => btn.textContent = 'コピー', 1500); };
  if (navigator.clipboard?.writeText) {
    navigator.clipboard.writeText(text).then(done).catch(() => fallbackSelect(btn));
  } else {
    fallbackSelect(btn);
  }
}
function fallbackSelect(btn) {
  const range = document.createRange();
  range.selectNodeContents(btn.nextElementSibling);
  const sel = window.getSelection();
  sel.removeAllRanges();
  sel.addRange(range);
  btn.textContent = '選択しました — Cmd/Ctrl+C を押してください';
}
</script>
```

### 進捗付きチェックリスト

月次締めの手順や初期設定のステップ一覧用。件数は項目から算出し、決して手入力
しない。ずれようがないようにするためだ。

```html
<div class="checklist">
  <p class="checklist-progress">9件中3件完了</p>
  <label><input type="checkbox" checked disabled> 銀行明細を突合済み</label>
  <label><input type="checkbox" disabled> 給与仕訳を計上</label>
</div>
```

### フッター

すべてのアーティファクトは同じ形で終わる — 控えめに、小さく、`--text-xs`、
`--line` 系の色で：

```html
<footer class="page-footer"><strong>資金繰りスナップショット</strong> が生成 — 中小企業向けビジネスアシスタント</footer>
```

---

## 両テーマ、常に

上のトークン構造に正確に従う：ライトは素の `:root`、`prefers-color-scheme`
メディアクエリは `:not([data-theme="light"])` でガードし、`:root[data-theme="dark"]`
に同じ値を繰り返して、明示的な切り替えがどちらの方向にも勝つようにする。
すべてのコンポーネントは色をトークンから読む — コンポーネントのルールに 16進値
を直書きしない — ことで、ひとつの上書き点（ブランドカラー、またはダークモード
のブロック）だけでページ全体が正しく塗り替わる。

---

## やってはいけないこと

- **ページを二度渡さない。** アーティファクトを公開する。同じ内容をダウンロード
  用の .html ファイルとして重ねて渡さない。1 ページ＝1 成果物。
- **スキルごとに新しいパレットを考案しない。** すべてのアーティファクトはこの
  トークンを使う。一回限りの見た目は、ハウススタイルの意味そのものを壊す。
- **ヒーロー、イラスト、マーケティング文言を足さない。** これは業務ツールで
  あって、売り込みではない。
- **ダークモードを省かない。** 上のチェック項目の半分は、もう一方のテーマで
  読めないページを防ぐために存在する。
- **クリップボード API が動くと決めつけない。** サンドボックスがブロックする
  ことがある — テストし、フォールバックを込みで出す。
- **このルールで SKILL.md を太らせない。** スキルの出力ステップは一行：
  「結果はハウスのアーティファクトスタイル（`../../shared/artifact-style.md`）
  で描画する」に、使う具体的なコンポーネント名を添える。CSS とコンポーネント
  の詳細はすべてこの一ファイルに置く。

---

## 参照ビルド

このディレクトリの `reference/artifact-example.html` は、上のすべてのコンポー
ネントを使った完全に動くページで、構造をそのままコピーするためのもの。コピー
時の注意が一つ：例はブラウザで開けるよう doctype/head/body の骨組みを備えた
単体ファイルだ。公開先がコンテンツを自身の文書骨組みで包む場合（Artifact
ツールはそうする）、スタイルと body の中身だけをコピーし、doctype、html、
head、body のタグそのものは決してコピーしない。
