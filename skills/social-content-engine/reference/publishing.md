# 投稿の準備（公開しない）

経路は優先順に3つ。どの経路でも投稿は **準備されるだけで、公開されない** —
公開のタイミングは経営者が握り、取り消しも編集もできる。

| 経路 | いつ | 経営者が受け取るもの |
|---|---|---|
| HubSpot ソーシャル | Marketing Hub Professional 以上 | HubSpot のキャンペーンキューに予約済み投稿 |
| 予約 CSV | HubSpot なし、または Professional 未満のプラン | Buffer、Later、Hootsuite、SocialDog（X）など経営者のツールに取り込む CSV |
| 手動投稿用コピー | LINE公式アカウント、または `build-connector` で接続していない媒体 | 管理画面に貼るだけの、日付・媒体ごとにまとめた本文と画像 URL |

LINE公式アカウント、Instagram、X への直接の予約は、HubSpot の対応外の媒体
（LINE）や HubSpot 未接続のときは `build-connector` を提案し、接続がなければ
手動投稿用コピーを渡す。

---

## 準備する前に

1. **すべての予約時刻が未来（JST）であること。** 数週間前に作ったカレンダーは
   黙って古くなる。各時刻を現在と比べ、過ぎているものを示す：「6月8日の投稿
   日はもう過ぎています。飛ばしますか、動かしますか？」
2. **すべてのアセット URL が恒久エクスポート URL であること。** オートフィル
   応答のサムネイルではない。あれは数分で失効し、壊れた画像として添付される。
3. **キャプションは承認済みのものであること。** チェックポイント3を飛ばした
   後からの編集ではない。

---

## HubSpot ソーシャルへの準備

前提: **HubSpot Marketing Hub Professional**（以上）。  
ベース URL: `https://api.hubapi.com`  
認証: Bearer トークン（OAuth 2.0 またはプライベートアプリのトークン）。

---

## プランの確認

ユーザーが Marketing Hub Professional かどうか分からないとき：

```
GET /crm/v3/objects/companies?limit=1
```

アカウントの `subscriptionType` を確認する。ソーシャルのエンドポイントで API が
403 を返したら、ユーザーに伝える：「HubSpot でのキャンペーンの予約には
Marketing Hub Professional が必要です。現在のプランには含まれていません。
代わりに予約 CSV を出せますが、それでよいですか？」

---

## キャンペーンの作成

```
POST /marketing/v3/campaigns
{
  "name": "梅雨前エアコン点検キャンペーン 2026 — SNS",
  "startDate": "2026-06-01",
  "endDate":   "2026-06-30",
  "currencyCode": "<Business context ブロックの通貨。通常は JPY>",
  "utm": {
    "source": "social",
    "medium": "owned",
    "campaign": "tsuyu-tenken-2026"
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
  "scheduledAt": "2026-06-03T12:00:00+09:00",
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
`type`（`INSTAGRAM`、`FACEBOOK`、`TWITTER`、`LINKEDIN`）で照合し、`id` を使う。
LINE公式アカウントと Threads は HubSpot のチャネルにない — それらの行は
CSV か手動投稿用コピーへ回す。

**`scheduledAt`** — 未来の時刻でなければならない（API コール時点に対して）。
経営者の指定がなければ JST の正午（`+09:00` を必ず付ける）。BtoB なら平日の
昼休み前、店舗客向けなら夕方 18 時が反応の取れる時間帯として一般的だ。

---

## 予約キューの確認

全投稿を準備した後：

```
GET /marketing/v3/social/posts?status=SCHEDULED&campaignId=<campaign_id>
```

結果を表にする：日付、媒体、キャプションの先頭 40 字、状態。
HubSpot のキャンペーン URL を直接示す：
`https://app.hubspot.com/content/{portalId}/social/campaigns/{campaign_id}`

ユーザーに伝える：「投稿は予約済みで、時刻になると自動で公開されます。
公開前なら HubSpot 上でどの投稿も取り消し・編集できます。」

---

## CSV の代替（Professional 未満のユーザー）

HubSpot での準備が使えないときは、Buffer、Later、Hootsuite、SocialDog など
ユーザーの予約ツールに取り込める CSV を出す。

列見出し：
```
Date,Time,Channel,Caption,ImageURL,Status
```

行の例：
```
2026-06-03,12:00 JST,Instagram,"梅雨前のエアコン点検、6月の枠を受付中 ...",https://canva.com/export/...,Scheduled
```

ユーザーに伝える：「HubSpot のプランにソーシャルの予約が含まれていないので、
予約 CSV を用意しました。Buffer や Later に取り込めます。[ファイルパス]」

---

## 手動投稿用コピー（LINE公式アカウント・Instagram・X）

`build-connector` での接続がない媒体は、管理画面に貼るだけの形で渡す。
日付・媒体ごとにまとめ、本文、画像の恒久 URL、投稿時刻（JST）を並べる：

```
6月2日 12:00 — LINE公式アカウント（メッセージ配信、リッチメッセージ 1040×1040）
  本文: <承認済みの本文、200字以内、リンク1つ>
  画像: https://canva.com/export/...
  配信: LINE Official Account Manager → メッセージ配信 → 予約配信

6月2日 12:00 — Instagram フィード
  本文: <承認済みのキャプション＋ハッシュタグ>
  画像: https://canva.com/export/...
  投稿: Meta Business Suite の予約投稿
```

ユーザーに伝える：「LINE と Instagram はここから予約できないので、貼るだけの
形にしました。各管理画面の予約機能で時刻を指定してください。」

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

---

## メールの引き渡し

メールの文章はどの経路でもこのスキルからは準備しない。承認済みの件名、
プレヘッダー、本文を配信日ごとにまとめてインラインで示し、経営者が自分の
送信ツールに貼れるようにする：

```
6月5日  — 「保守契約は7月1日に更新となります」
7月3日  — 「7月の点検枠、残り2件のご案内」
```

引き渡しはこれで全部だ。送信を申し出ない。Canva でデザインを作らない。
Mailchimp 接続時は、本文を下書きキャンペーンとして保存できる（SKILL.md の
ステップ9）が、予約と送信は経営者が Mailchimp 上で行う。
