# コネクタ呼び出しの形 — 間違えやすいパラメータ

**プラグイン全体で共有。** 下の各行は、素直な形では失敗し、正しい形で動く
呼び出しである。どれもバグではない。すべてコネクタが要求する形で、ツールの
説明文からは自明でないもの。コネクタへの最初の呼び出しの前に、その行を読む。

**どの行より先に: どの項目を呼ぶか。** コネクタは二重に登録されうる —
経営者自身の項目と、プラグインのマニフェスト側コピー
（`small-business-jp:<マニフェストのキー>`、ツール名の接頭辞は
`mcp__plugin_small-business-jp_`）。認可されている方を呼び、両方なら経営者の
ものを呼ぶ。経営者の項目が生きているのにプラグイン側コピーの呼び出しが
拒否されても、それは「未接続」ではなく、スキルの縮退経路を取る理由にも
エクスポートを求める理由にもならない。ルールは `connector-neutrality.md` の
「一つのコネクタ、二つの登録」。

| コネクタ | ツール | 動く形 |
|---|---|---|
| Airwallex | `list_billing_invoices` | 「未回収は何か」には `status: FINALIZED` と `payment_status: UNPAID` の両方を渡す。`VOIDED` の請求書も `UNPAID` と報告される。行が持つのは `billing_customer_id` で顧客名ではない — 名前とメールは `retrieve_billing_customer` で。Airwallex の `number` は Airwallex 独自の採番で、会計ソフト側の請求書番号があるなら `metadata` に入っている。日付フィルタ（`from_created_at`、`to_created_at`）は `+0000` のオフセット形式で、末尾 `Z` は請求書一覧で拒否される（Airwallex 自身の罠一覧）。ツール名は接続済みサーバーのツール一覧から取る。 |
| HubSpot | `get_crm_objects` | 明示的なオブジェクト ID しか受け付けない。一覧や絞り込みの問い合わせ（「進行中の商談」「今月作成された連絡先」）はフィルタ付きの `search_crm_objects` を使い、必要なら全レコードを `get_crm_objects` で取る。 |
| HubSpot | `query_crm_data` | 必須パラメータは `sql` 一つで、HubSpot 方言のクエリ（1クエリに1オブジェクト種別、JOIN なし、ID は `hs_object_id`）。自由文や他のパラメータ名は、項目名を示さない生の「Missing required field」エラーで失敗する。まず `tool_guidance` を呼び（それ自体のルール）、プロパティ名は `search_properties` で確認する。単純な絞り込みなら `search_crm_objects` の方が簡単。 |
| Mailchimp | `get_capabilities` | `user_request`（経営者がやろうとしていることを言葉で）と、enum からの `category` の両方が要る。返答はコネクタ自身の指示どおり、そのまま提示する。 |
| マネーフォワード クラウド会計 | すべてのツール | 経営者が API キー認証なら `identification_code`（事業者コード `XXXX-XXXX`）が必須。OAuth2 認証では無視される。最初に `currentOffice` を呼び、返った事業者名を事業コンテキストと照合する（`tenant-scope.md`）。会計期間が一つもない事業者ではエラーになる。 |
| マネーフォワード クラウド会計 | `getTransactions` | `start_date` と `end_date` は必須で、差は **366日以内**。1年を超える期間は分割して呼ぶ。未仕訳の明細（銀行・カード明細で仕訳がまだ付いていないもの）は `journalizing_statuses: ["none"]` で絞る。省略すると `excluded`（対象外）まで全ステータスが返る。`per_page` は既定 50・最大 500 なので、件数を言ってからページを回す。`connected_account_id` と `connected_sub_account_id` は同時に指定できない。`value_min`／`value_max` を使うなら `side`（`INCOME`／`EXPENSE`）が必須。 |
| マネーフォワード クラウド会計 | `getReportsTrialBalanceProfitLoss` | 期間は `fiscal_year` ＋ `start_month`／`end_month`、または `start_date`／`end_date`。**全項目が 0 の勘定科目・補助科目は返らない**: 返ってこないのは「無い」であって「0」ではない（`absent-is-not-zero.md`）。`ratio` は分母が 0 のとき `null` で、0 として扱わない。`include_tax` は経理方式が「税抜(内税)」のときだけ有効で、「税込」「税抜(別記)」では無視される — どちらの基準で集計したかを出力に書く。常に全部門合計で、未実現仕訳は含まない。取引先別の売掛金・買掛金残高は `with_sub_accounts: true` で補助科目を出す。期日別の滞留はこのレポートにはない（`ledger-report-traps.md`）。 |
| マネーフォワード クラウド会計 | `postJournals` | `journal` オブジェクトに `journal_type`（`journal_entry` 通常仕訳／`adjusting_entry` 決算整理仕訳）、`transaction_date`、`branches` の三つが必須。`branches` は最大 300 行で、各行の `debitor`／`creditor` に `account_id` と `value`（円の整数）が必須。消費税は `tax_id`、インボイス区分は `invoice_kind`（`INVOICE_KIND_QUALIFIED`、免税事業者からの仕入れは経過措置に応じて `INVOICE_KIND_UNQUALIFIED_80`／`INVOICE_KIND_UNQUALIFIED_50`、対象外は `INVOICE_KIND_NOT_TARGET`）、取引先は `trade_partner_code`。書き込みなので必ず承認後に呼び、呼ぶ前に仕訳を表で見せる。 |
| Salesforce | `discover`、`describe`、`dispatch_readonly`、`dispatch` | 四つのツール、一つの流れ、毎回: `discover` にやりたいことを平文で渡すと順位付けされた操作が返る。選んだものを `describe` し、呼ぶ前にパラメータを読む。読み取り（件数、SOQL クエリ、検索）はすべて `dispatch_readonly` で実行する。`dispatch` は経営者が承認した書き込みにだけ使い、サインイン中のユーザーのアクセス権が適用される。読み取りを `dispatch` しない、`describe` していない操作を `dispatch` しない、削除は存在しない。Headless 360 サーバーは経営者が追加するもの（カスタムコネクタ、組織ごとのアドレス）。 |
| Shopify | `get-inventory-levels` | 1回の呼び出しに `productId` 一つ。一括読み取りはない。実行に必要な商品をループし、カタログが大きければ何件読んだかを言う。 |
| Square | `get_service_info` と `get_type_info` | まず `service` 文字列（例: `payments`、`refunds`、`inventory`、`customers`）が要る。裸の呼び出しは失敗する。その後、そのサービスに対して `make_api_request` を呼ぶ。 |

**行を足す。** 呼び出しが形で失敗し、再試行で成功したら、一つのスキルではなく
ここに行を足す: そのコネクタに触るすべてのスキルが同時に直る。形の話に留める。
挙動の罠はコネクタ自身の罠ファイル（`ledger-report-traps.md`、
`gmail-inbox-traps.md`）に書く。
