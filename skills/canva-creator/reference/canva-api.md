# Canva Connect API リファレンス

ベース URL: `https://api.canva.com/rest/v1`  
認証: Bearer トークン（OAuth 2.0）。必要なスコープ: `design:content:read`、`design:content:write`、`asset:read`、`asset:write`、`brandtemplate:content:read`（Enterprise のみ）。

---

## 目次

1. [プランごとの要件](#プランごとの要件)
2. [ブランドテンプレート（Enterprise）](#ブランドテンプレートenterprise)
3. [オートフィル（Enterprise）](#オートフィルenterprise)
4. [デザインのコピー（Pro／Teams）](#デザインのコピーproteams)
5. [アセットのアップロード](#アセットのアップロード)
6. [エクスポート](#エクスポート)
7. [エラーコード](#エラーコード)

---

## プランごとの要件

| 機能 | Free | Pro | Teams | Enterprise |
|---------|------|-----|-------|------------|
| デザインの作成 | ✓ | ✓ | ✓ | ✓ |
| 自分のデザインの一覧 | ✓ | ✓ | ✓ | ✓ |
| ブランドテンプレート（読み取り） | — | — | — | ✓ |
| ブランドテンプレートのオートフィル | — | — | — | ✓ |
| アセットのアップロード（ブランドキット） | — | — | — | ✓ |
| アセットのアップロード（ユーザー） | — | ✓ | ✓ | ✓ |

---

## ブランドテンプレート（Enterprise）

**ブランドテンプレートの一覧:**
```
GET /brand-templates?query={keyword}&ownership=organization
```
注目する応答項目:
- `id` — オートフィルに渡す
- `title` — 人が読める名前
- `thumbnail.url` — プレビュー画像
- `dataset[].label` — オートフィルの項目ラベル（例: "Headline"、"ProductName"）

`title` にアセット種別のキーワード（例: 「正方形投稿」「ストーリー」「リッチメッセージ」）が含まれるもので絞り込む。

テンプレートの文字スロットに日本語を流し込む前に、テンプレート側のフォントが日本語対応（Noto Sans JP、BIZ UDPGothic など）であることを確認する。欧文フォントだけのテンプレートは、日本語が文字化け（□）になるか、意図しない代替フォントで描画される。縦書きのテキストボックスを持つテンプレートは SNS 用には選ばない。

---

## オートフィル（Enterprise）

ブランドテンプレートの可変項目を埋めて新しいデザインを作る：

```
POST /autofills
{
  "brand_template_id": "<template_id>",
  "title": "秋の新作キャンペーン — 投稿1",
  "data": {
    "Headline": { "type": "text", "text": "秋の新作、届きました — ウールニット 9,900円（税込）" },
    "ProductImage": { "type": "image", "asset_id": "<uploaded_asset_id>" }
  }
}
```

応答: `{ "job": { "id": "<job_id>", "status": "queued" } }`

`GET /autofills/{job_id}` を `status == "success"` になるまでポーリングする。応答に
`result.design.id` が含まれる — これをエクスポートに使う。

価格を文字項目に入れるときは、ネットショップかブリーフが返した円の税込価格を
そのまま使う（総額表示義務）。記憶や概算の値を入れない。

---

## デザインのコピー（Pro／Teams）

Pro／Teams ユーザーには「テンプレートをコピーする」直接のエンドポイントがない。手順：

1. `GET /designs?ownership=any&query={template name}` — 名前でデザインを一覧
2. 上位3件をユーザーに示し、確認を取る
3. `POST /designs` に `asset_type` を付けて適切な寸法の空のデザインを作り、
   ユーザーが Canva 上で手動更新すべき点を説明する。

Pro／Teams のアセット生成は半手動だ：Claude がデザインの器を作り、API で
できる範囲を埋める。ユーザーが Canva 上でブランド固有の編集をして、エクスポート
用のデザイン ID を返す。

---

## アセットのアップロード

ブリーフがユーザーのデスクトップやファイルシステムにある商品写真を
参照しているときに使う。

**ステップ1 — アップロードの初期化:**
```
POST /asset-uploads
{ "name_base64": "<base64(filename)>", "types": ["image/jpeg"] }
```
応答: `{ "job": { "id": "<job_id>" }, "upload_url": "<presigned_s3_url>" }` 

**ステップ2 — ファイルのアップロード:**
```
PUT <upload_url>
Content-Type: image/jpeg
Body: <raw file bytes>
```

**ステップ3 — 完了までポーリング:**
`GET /asset-uploads/{job_id}` → `status == "success"` を待つ → `asset.id` を取得。

最大ファイルサイズ: 100 MB。対応形式: `image/jpeg`、`image/png`、`image/webp`。

---

## エクスポート

**なぜ毎回エクスポートするのか:** オートフィル応答のサムネイル
（`design.canva.ai/...`）は短命な認証付き CDN URL で、数分で失効し、失効後に
Markdown 画像として埋め込むと壊れた「Show Image」のプレースホルダーになる。
`POST /exports` の出力は **恒久 URL** で、チャットのプレビューへの埋め込み、
HubSpot 投稿への添付、経営者への共有に安全に使える。

生成後、プレビューを見せる前に、必ず各デザインをエクスポートする。

```
POST /exports
{
  "design_id": "<design_id>",
  "format": {
    "type": "png",
    "export_quality": "regular",
    "pages": [1]
  }
}
```

`GET /exports/{job_id}` を `status == "success"` までポーリングする。応答に
`urls[]` が含まれる — `urls[0]` をプレビューリンクと HubSpot 添付 URL に使う。

**Canva MCP の対応表**（直接 REST ではなく Cowork の Canva コネクタを使うとき）：

| 目的 | MCP ツール | 返すもの |
|------|---------|---------|
| 恒久プレビュー URL | `export-design` | 恒久ダウンロード URL（埋め込み可） |
| ページごとのサムネイル（オートフィル応答より安定、ただし恒久ではない） | `get-design-thumbnail` | ページのサムネイル URL |
| デザインのメタデータのみ | `get-design` | タイトル、所有者、ページ数、サムネイル |
| 経営者の端末からのアップロード | `upload-asset-from-url` | `asset_id`（1コール1 URL） |

**媒体別の形式:**

| 媒体 | 形式 | 寸法 |
|---------|------|-----------|
| Instagram フィード | `png` | 1080×1080（正方形） |
| Instagram ストーリーズ／リール表紙 | `png` | 1080×1920 |
| X | `png` | 1200×675 |
| LINE リッチメッセージ | `png` | 1040×1040 |
| Facebook フィード | `png` | 1200×630 |
| メールヘッダー | `png` | 600×200 |

---

## レート制限と生成予算

**上限:** トークンごとに毎分 100 リクエスト。

**1デザインのコスト:** 約5 API コール — `POST /autofills` 1回 + `POST /exports`
1回 + ポーリング約3回（`GET /autofills/{job_id}`、`GET /exports/{job_id}`）。
ポーリング間隔は 3〜5 秒。速くしても完了は早まらず、上限を消費するだけ。

**安全な上限:** 毎分 15〜20 デザインなら、毎分 100 リクエストの上限に対して
十分な余裕がある。

**推奨ペース（ステージ3で使う）:**

```
1行あたり候補3つ × 一度に1行 × 行間 30 秒
= 30 秒あたり 6 デザイン（約 30 API コール）
= 毎分 12 デザイン、上限のおよそ半分
```

**事前確認の予算の式:**

```
total_designs = (Canva 行数) × (1行あたりの候補数、既定 3)
total_api_calls ≈ total_designs × 5
generation_time ≈ (total_designs / 12) 分
```

**`RATE_LIMIT_EXCEEDED` のバックオフ:**

1. セッション中の最初のヒット → 60 秒待ち、その候補1つを再試行。一時的な
   スパイクとして扱う。
2. 同じセッションで2回目のヒット、または `quota_exceeded` ／日次上限の
   エラー → 生成を止め、経営者に進捗を示す。再試行しない。スキルは経営者に
   (a) 残りの行を候補1つに減らす、(b) 60 分休んで再開する、(c) ここで止めて
   生成済みのものでキャプション作成に進む、のどれにするか聞く。

---

## エラーコード

| コード | 意味 | 対処 |
|------|---------|-----------|
| `PERMISSION_DENIED` | スコープ不足かプランが違う | プランを確認。正しいスコープで Canva を再接続してもらう |
| `DESIGN_NOT_FOUND` | デザイン ID が違う | デザインを再一覧して ID を確認 |
| `AUTOFILL_FIELD_NOT_FOUND` | テンプレートの項目名が不一致 | `GET /brand-templates/{id}` で `dataset[].label` を読み直す |
| `RATE_LIMIT_EXCEEDED` | リクエスト過多 | 初回: 60 秒待って1回だけ再試行。2回目: 止まって経営者に聞く（上のレート制限を参照） |
| `JOB_FAILED` | 非同期ジョブの失敗 | `job.error.message` を確認。よくある原因はアセットのサイズ超過 |
