# HubSpot キャンペーン準備リファレンス

前提: **HubSpot Marketing Hub Professional**（以上）。  
ベース URL: `https://api.hubapi.com`  
認証: Bearer トークン（OAuth 2.0 またはプライベートアプリのトークン）。

このスキルにスケジューラはない。ここでの「準備」は承認済みの投稿を HubSpot のキューに `SCHEDULED` として置くことで、時刻になって公開するのは HubSpot 側の機能だ。HubSpot のチャネルにない媒体（LINE公式アカウント、Threads、TikTok）は、`build-connector` で接続するか、各媒体の管理画面に貼る手動投稿用コピーを渡す。

---

## 目次

1. [プランの確認](#プランの確認)
2. [キャンペーンの作成](#キャンペーンの作成)
3. [ソーシャル投稿の作成](#ソーシャル投稿の作成)
4. [予約キューの確認](#予約キューの確認)
5. [CSV の代替（Professional 未満のユーザー）](#csv-の代替professional-未満のユーザー)
6. [手動投稿用コピー（LINE公式アカウント・Instagram・X）](#手動投稿用コピーline公式アカウントinstagramx)
7. [メールの引き渡し](#メールの引き渡し)
8. [項目リファレンス](#項目リファレンス)

---

## プランの確認

ユーザーが Marketing Hub Professional かどうか分からないとき：

```
GET /crm/v3/objects/companies?limit=1
```

アカウントの `subscriptionType` を確認する。ソーシャルのエンドポイントで API が
403 を返したら、ユーザーに伝える：「HubSpot でのキャンペーンの準備には Marketing
Hub Professional が必要です。現在のプランには含まれていません。代わりに予約 CSV
を出せますが、それでよいですか？」

---

## キャンペーンの作成

```
POST /marketing/v3/campaigns
{
  "name": "秋の新作キャンペーン 2026 — SNS",
  "startDate": "2026-09-01",
  "endDate":   "2026-09-30",
  "currencyCode": "<Business context ブロックの通貨。通常は JPY>",
  "utm": {
    "source": "social",
    "medium": "owned",
    "campaign": "aki-shinsaku-2026"
  }
}
```

応答: `{ "id": "<campaign_id>", ... }` — ソーシャル投稿の紐付けに保存する。

---

## ソーシャル投稿の作成

カレンダー1行につき1コール。`SCHEDULED` として準備し、`PUBLISHED` にはしない。

```
POST /marketing/v3/social/posts
{
  "campaignId": "<campaign_id>",
  "channelId":  "<hubspot_channel_id>",
  "content": {
    "body": "<承認済みのキャプション本文>"
  },
  "scheduledAt": "2026-09-03T12:00:00+09:00",
  "attachments": [
    {
      "url": "<canva_export_png_url>"
    }
  ],
  "status": "SCHEDULED"
}
```

**`channelId`** — HubSpot のソーシャルアカウント ID（媒体名ではない）。
接続済みアカウントの取得：
```
GET /marketing/v3/social/channels
```
`type`（`INSTAGRAM`、`FACEBOOK`、`TWITTER`、`LINKEDIN`）で照合し、`id` を使う。X は `TWITTER`。
LINE公式アカウント、Threads、TikTok は HubSpot のチャネルにない — それらの行は手動投稿用コピーか CSV へ回す。

**`scheduledAt`** — 未来の時刻でなければならない（API コール時点に対して）。経営者の
指定がなければ JST の正午（`+09:00` を必ず付ける。`Z` のまま送ると 9 時間ずれる）。

---

## 予約キューの確認

全投稿を準備した後：

```
GET /marketing/v3/social/posts?status=SCHEDULED&campaignId=<campaign_id>
```

結果を表にする：日付、媒体、キャプションの先頭 40 字、状態。
HubSpot のキャンペーン URL を直接示す：
`https://app.hubspot.com/content/{portalId}/social/campaigns/{campaign_id}`

ユーザーに伝える：「投稿は予約済みで、時刻になると自動で公開されます。公開前なら
HubSpot 上でどの投稿も取り消し・編集できます。」

---

## CSV の代替（Professional 未満のユーザー）

HubSpot での準備が使えないときは、Buffer、Later、Hootsuite、SocialDog（X）など、
ユーザーの予約ツールに取り込める CSV を出す。

列見出し：
```
Date,Time,Channel,Caption,ImageURL,Status
```

行の例：
```
2026-09-03,12:00 JST,Instagram,"秋の新作、届きました。やっと見つけた、秋の一枚 ...",https://canva.com/export/...,Scheduled
```

ユーザーに伝える：「HubSpot のプランにソーシャルの予約が含まれていないので、
予約 CSV を用意しました。Buffer や Later に取り込めます。[ファイルパス]」

---

## 手動投稿用コピー（LINE公式アカウント・Instagram・X）

`build-connector` での接続がない媒体、または HubSpot のチャネルにない媒体は、管理画面に
貼るだけの形で渡す。日付・媒体ごとにまとめ、本文、画像の恒久 URL、投稿時刻（JST）を並べる：

```
9月2日 12:00 — Instagram フィード
  本文: <承認済みのキャプション＋ハッシュタグ>
  画像: https://canva.com/export/...
  投稿: Meta Business Suite の予約投稿

9月4日 12:00 — LINE公式アカウント（メッセージ配信、リッチメッセージ 1040×1040）
  本文: <承認済みの本文、200字以内、リンク1つ>
  画像: https://canva.com/export/...
  配信: LINE Official Account Manager → メッセージ配信 → 予約配信
```

ユーザーに伝える：「LINE と Instagram はここから予約できないので、貼るだけの形にしました。
各管理画面の予約機能で時刻を指定してください。」同じ媒体に毎回手で貼ることになるなら、`build-connector` を提案する。

---

## メールの引き渡し

メールの文章はこの API では準備しない。承認済みの件名、プレヘッダー、本文を配信日ごとにまとめてインラインで示し、
経営者が HubSpot のマーケティングメール、Mailchimp、Gmail または Microsoft 365 に貼る。送信も予約もこのスキルからは行わない。

---

## 項目リファレンス

| 項目 | 型 | 備考 |
|-------|------|-------|
| `campaignId` | string | キャンペーン作成で得た UUID |
| `channelId` | string | `GET /social/channels` から |
| `content.body` | string | キャプション本文。最大 2,000 文字。X の行は 140 字以内に収める |
| `scheduledAt` | ISO 8601 | 未来の時刻。タイムゾーンのオフセット（`+09:00`）を含める |
| `attachments[].url` | string | 公開 URL（Canva のエクスポート URL が使える） |
| `status` | enum | 常に `"SCHEDULED"` — `"PUBLISHED"` にはしない |
