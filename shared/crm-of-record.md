# 正とする CRM は一つ

会計ソフトが従うのと同じルールを CRM に当てはめる: **事業者の CRM は一つで、
スキルはそれを読む — 二つを混ぜない。** HubSpot、kintone、Monday.com、
Salesforce、Zoho CRM は同格（`connector-neutrality.md`）。接続されている
ものが正とする CRM である。二つ接続されていれば、経営者が一つを名指しし
（ルーターのステップ5の選択肢）、スキルはどちらかを書き、リードの母集団と
パイプラインの合計はその一つだけから読む。二つの CRM を合併すると、重複した
連絡先、二重計上のパイプライン、同じ人が二回載った架電リストができる。

**レコードが答えに寄与する前に、組織を確認する。** Zoho の `getOrganization`
は会社名を返す。働いている事業者と一致しなければ、他社のパイプラインを
報告するのではなく、止まってそう言う。

**認証バナーは障害ではない。** CRM を使えないと宣言して CSV に流れる前に、
安価な読み取りを一回試す。

---

## フィールド対応 — HubSpot、Salesforce、Zoho CRM

この三つはリードのモデルが違い、その違いはフィールド名以上に重要である。
Monday.com にはリードのオブジェクトがない。ボードを CRM として使う形は
crm-autopilot のスキル本体にある。kintone はアプリ設計が事業者ごとに異なる
ため、フィールドは接続後に `build-connector` で確認する。Salesforce の名前は
標準オブジェクトとフィールドの API 名で、Headless 360 の四つのツール
（`discover`、`describe`、`dispatch_readonly`、`dispatch`）を通して到達する。
それ自体がツールになることはない — `connector-call-shapes.md` を参照。

| スキルが必要とするもの | HubSpot | Salesforce | Zoho CRM |
|---|---|---|---|
| リードの母集団 | `lifecyclestage` = `Lead`/`MQL` で絞った Contacts | **`Lead` オブジェクト**で `IsConverted = false` — 変換済みリードの行は残るので除外が必要 | **`Leads` モジュール** — 連絡先のステージではなく独立したモジュール |
| リードのステータス | `hs_lead_status`（`Unqualified` を除外） | `Status`（その組織の「不適格」の値を除外。選択リストはオブジェクト describe の操作を `discover` で見つけ、`describe` してから `dispatch_readonly` で実行して得る — `describe` 自体は操作を説明するもので、オブジェクトを説明するものではない） | `Lead_Status`（`Not Qualified` / `Junk Lead` を除外） |
| 会社 | **関連付けられた会社レコード** — 二段目のホップ | リード上の素のフィールド `Company`。連絡先では `AccountId` → `Account.Name` | **リード自体の素のフィールド `Company`** — ホップなし |
| 流入元 | `hs_analytics_source` | `LeadSource` | `Lead_Source` |
| 最終接触 | `notes_last_contacted` | `LastActivityDate` — 完了した Task と Event で設定される。何も記録されていないレコードでは null | `Last_Activity_Time` — **Deals には入るが Leads では null が多い**（下記） |
| 作成日 | `createdate` | `CreatedDate` | `Created_Time` |
| 商談 | Deals: `dealname`、`dealstage`、`amount`、`closedate` | `Opportunity`: `Name`、`StageName`、`Amount`、`CloseDate`、`NextStep`、`IsClosed` | Deals: `Deal_Name`、`Stage`、`Amount`、`Closing_Date` |
| 商談の会社 | 関連付けられた会社 | `AccountId` → `Account.Name` | `Account_Name`（ルックアップ — `.name` を読む） |
| 活動の記録 | 連絡先のタイムラインへのメモ | `WhoId` = 連絡先、`WhatId` = 商談とした `Task`（電話、メール）または `Event`（会議） | `Calls` / `Tasks` のレコード、または Note |
| 全文検索 | `search_crm_objects` | `discover` に「レコード検索」、`describe`、`dispatch_readonly` の順。選択的なものは同じ三拍子でクエリ操作を通した SOQL | `searchRecords`。選択的なものは `executeCOQLQuery` |

### 知っておく価値のある構造上の違いは一つ

**Zoho では、リードが自分の会社名を持っている。** HubSpot は会社適合度を
採点するのに連絡先に関連付けられた会社レコードが必要で、その関連付けが
ないと適合度の次元が黙って平坦になる。Zoho の素の `Company` フィールドなら
会社適合度はリードから直接読める。Zoho が正とする CRM のとき、会社適合度は
*より* 信頼できる。劣るのではない。

### null か同一値になりうる二つのもの

フィールドを信じると、どちらも次元のスコアを平坦にする:

- **`Last_Activity_Time` は、すべての商談に入っていながら、すべての未対応
  リードで null になりうる。** そうなっていると、接触の新しさと反応の次元に
  順位付けの材料がなく、スコアが崩れる — HubSpot が別方向から踏んだのと
  同じ平坦スコアの失敗。順位のように見せるのではなく、シグナルが空だと言い、
  メールとの突合に寄せる。
- **すべての未対応リードが秒単位まで同じ `Created_Time` を持ちうる** — 一括
  インポートの時刻であって、12件のリードが同時に来たのではない。それから
  計算したリードの経過日数は無意味。HubSpot の `createdate` や Shopify の
  `createdAt` と同じ罠: **レコードがインポートされた日は、関係が始まった日
  ではない。** 経過日数で何かを採点する前に、時刻が不自然に同一でないかを
  確認する。

### COQL には WHERE 句が要る

`select Deal_Name, Amount from Deals limit 12` は `SYNTAX_ERROR: missing
clause` で失敗する。COQL のクエリには必ず `where` が要るので、緩い条件
（`where Amount > 0`、`where Lead_Status != 'Not Qualified'`）を与える。
クエリが壊れていると思い込まない。

### 変換済みリード

Zoho はリードを連絡先＋取引先＋商談に変換し、元のリードは未対応の母集団
から抜ける。作業用のリード一覧を引くときは `converted=false`（既定）を渡す。
でないと受注済みの顧客が架電対象として再登場する。

### Zoho が代わりにならないもの

Zoho CRM が持つのは連絡先、リード、商談、活動。メールのコネクタ、カレンダー、
売上のデータ源の**代わりにはならない**。メールの文脈や空き時間枠が必要な
スキルは、背後の CRM が何であれ、Gmail とカレンダーを別に必要とする。
