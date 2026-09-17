# コネクタ対応表

スキルごとの必須／任意コネクタと、コネクタがゼロのときの代替経路。ルーターはこれを使って、一部が失敗するものを推奨しないようにし、代わりに代替経路を提示する。

**下の一覧は検証済みの経路であって、壁ではない。** 経営者が、あるスキルに一覧にないツールを使わせたいなら、まず `build-connector` へ案内する — コネクタディレクトリを確認し、なければ Zapier 経由で接続する。接続ができれば、そのツールは他の任意コネクタと同じ承認ゲートのもとでそのスキルに加わる。

すべてのスキルはコネクタがゼロでも動く。下の「必須」は *接続済み経路* に必要という意味で、代替経路の列は常に使える。

**呼び出しの形。** コネクタが要求するのにツールの説明からは分かりにくいパラメータの形は [`../../../shared/connector-call-shapes.md`](../../../shared/connector-call-shapes.md) にまとめてある。スキルはコネクタへの最初の呼び出しの前に、そのコネクタの行を読む。

**一つの製品、二つの登録があり得る。** 経営者自身のコネクタと、プラグインのマニフェストによる同じ製品の登録（`small-business-jp:<name>`）は一つと数える。どちらかが認可されていればゲートは開く（`../../../shared/connector-neutrality.md`「一つのコネクタ、二つの登録」）。

**ゲートはカテゴリであって、ベンダーではない。** 行に「会計ソフト」とあれば、freee会計、マネーフォワード クラウド会計、弥生会計（CSV）のいずれでもゲートが開く（Xero、Zoho Books、NetSuite など海外製の会計ソフトを使う事業者も同格）。「決済サービス」は PayPal、Square、Stripe のいずれか。「CRM」は HubSpot、kintone、Monday.com、Salesforce、Zoho CRM のいずれか（Salesforce は経営者が追加するカスタムコネクタでマニフェスト外、kintone は `build-connector` 経由）。「ネットショップ・POS」は Shopify または Square（注文と在庫を持つコネクタ）。「メール」は Gmail または Microsoft 365。「ファイル」は Google Drive または Microsoft 365（Microsoft 365 は両カテゴリを一つで満たすコネクタで、経営者が追加するカスタムコネクタ、マニフェスト外）。いずれも他より上位に立たない（`../../../shared/connector-neutrality.md`）。ベンダーの並びは五十音／アルファベット順。ゲートに関わる機能の限界は各表の下の注記に、「そのコネクタが何を持っているか」として書き、順位付けとしては書かない。

---

## 会社を回す

| スキル | 必須 | 任意 | コネクタゼロの代替 |
|---|---|---|---|
| `report-builder` | データソースを一つ | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 CSV）、Expensify、HubSpot、PayPal、Shopify、Square、Stripe | CSV/XLSX のアップロード — 完全に動く |
| `business-pulse` | なし | 会計ソフト、カレンダー、DocuSign、Expensify、Gmail または M365、給与計算ソフト（CSV）、HubSpot、PayPal、Shopify、Slack（Chatwork は `build-connector` 経由）、Stripe、TikTok広告、Zoho Desk | 接続済みのものだけで組む。なければ貼り付けたデータ |
| `cash-flow-snapshot` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など）または決済サービス（PayPal、Square、Stripe） | 給与計算ソフト（CSV）、法人カード明細（会計ソフトのカード連携）、Shopify | CSV のアップロード |
| `month-end-prep` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など） | Expensify、給与計算ソフト（CSV）、PayPal、Shopify、Square、Stripe | 銀行明細／CSV のアップロード |
| `invoice-chase` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など） | Airwallex、Gmail、Microsoft 365、PayPal、Shopify、Square、Stripe | 売掛金一覧 CSV を入れ、督促文の下書きを出す |
| `ap-processor` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など）+ メール（Gmail または M365） | Expensify、法人カード明細（会計ソフトのカード連携） | 請求書の PDF／写真をアップロード、仕訳をエクスポート |
| `payroll-prep` | 給与計算ソフト（freee人事労務、マネーフォワード クラウド給与、SmartHR、ジョブカン など。CSV エクスポートで可） | 書き込み経路のある会計ソフト（給与仕訳の計上用） | 勤怠データのアップロード → 検証済みの給与一覧 |
| `inventory-planner` | Shopify または Square | カレンダー、会計ソフト | 売上＋在庫の CSV |
| `inbox-manager` | Gmail または M365 | Slack | 貼り付け／転送されたメール本文 |
| `hiring-screener` | メール（Gmail または M365）+ カレンダー | DocuSign、Drive または M365、給与計算ソフト（CSV）、Trello | 履歴書のアップロード、手動送信用の下書き |
| `job-post-builder` | なし | DocuSign、Drive または M365 | 完全に単独で動く |
| `contract-review` | なし（ファイルのアップロード） | DocuSign、Drive または M365、Gmail または M365 | ファイルのアップロードが主経路 |
| `tax-season-organizer` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など） | Expensify、給与計算ソフト（CSV）、PayPal、Square、Stripe | CSV／書類のアップロード。納税額の計算は日本の税制前提で、税理士確認を添える（`../../../shared/currency-and-locale.md`） |
| `ticket-deflector` | 決済サービス（PayPal、Square、Stripe）、CRM、またはメール | Atlassian、Chatwork（`build-connector` 経由）、Shopify、Zoho Desk | 貼り付けたテキスト |

会計ソフトの能力メモ（それぞれが何を持っているか。各参照ファイルに準拠）:
- **マネーフォワード クラウド会計** — 残高試算表（損益計算書・貸借対照表）、推移表（月次）、仕訳、銀行・カード連携の明細（未仕訳の絞り込み可）、取引先、勘定科目・補助科目、税区分、事業者情報と会計期間。売掛金の期日別滞留一覧レポートはない: 取引先別残高は補助科目付き試算表から、請求書単位の期日はマネーフォワード クラウド請求書または請求書 CSV から組み立てる。`../../month-end-prep/reference/moneyforward-reconcile.md` を参照。レポート機能の落とし穴は `../../../shared/ledger-report-traps.md`。
- **freee会計** — 取引（収入／支出）、明細（未処理の明細）、試算表（損益計算書・貸借対照表）、月次推移、請求書、取引先、税区分。書き込みの範囲（取引の登録、取引先の作成）は接続後に確認する。`../../month-end-prep/reference/freee-reconcile.md` を参照。
- **弥生会計** — CSV 前提: 仕訳日記帳 CSV、残高試算表 CSV、補助元帳 CSV のエクスポートを読む。銀行明細の未仕訳を直接読むことはできないので、エクスポートを求め、そう言う。`../../month-end-prep/reference/yayoi-csv-reconcile.md` を参照。
- **海外製の会計ソフト**（Xero、Zoho Books、NetSuite など）を使う事業者も同格に扱う。損益・売掛金・現預金の同じ指標を、そのコネクタが返す範囲で読み、返さないものはエクスポートを求める。
- 会計ソフトが二つ接続されているとき: 両方読み、合計の「正」を一つ名指しし、決して足し合わせない。

ネットショップ・POS の能力メモ（それぞれが何を持っているか。各参照ファイルに準拠。カテゴリの定義は `../../../shared/connector-neutrality.md`）:
- **Shopify** — 注文、顧客、画像付き商品、在庫数、ShopifyQL の分析。ショップ情報に国と通貨がある。入金（payout）はない: コネクタのスコープが Shopify ペイメントを含まないため、精算のタイミングは経営者に聞き、推測しない（`../../cash-flow-snapshot/reference/v2_sources.md`）。在庫の読み取りは `productId` 一つずつ（`../../../shared/connector-call-shapes.md`）。
- **Square** — 明細付きの注文、画像付きカタログ、在庫数、決済、入金。加盟店情報に国と通貨がある。決済一覧は呼び出しごとに 1 店舗なので、スキルは店舗を順に回す。本番アカウントでは入金が構造的に存在しないことがある。
- 「CRM、決済サービス、またはネットショップ・POS」と書かれたゲートは、ネットショップ・POS 単独で満たせる: 注文履歴は「誰が何をいつ買ったか」だ。
- ネットショップ・POS と決済サービスの両方が接続されているとき: 注文とその精算入金は一つの売上。売上高は経営者が「正」と名指しした方から取り、もう一方はタイミング、手数料、在庫を供給する。

## 会社を伸ばす

| スキル | 必須 | 任意 | コネクタゼロの代替 |
|---|---|---|---|
| `lead-finder` | Apollo または Clay（国内企業の網羅率は限定的） | HubSpot | Web 調査＋顧客 CSV |
| `outreach-composer` | Gmail または M365 | Apollo、Clay、HubSpot、Mailchimp | 下書きのみ。どこにでも貼り付け可 |
| `speed-to-lead` | HubSpot + メール（Gmail または M365） | カレンダー、Slack（Chatwork は `build-connector` 経由） | 転送／貼り付けた問い合わせ |
| `lead-triage` | HubSpot | Apollo、カレンダー、Clay、Gmail または M365 | 貼り付け／アップロードしたリード一覧 |
| `proposal-builder` | Drive または M365 | Apollo、Atlassian、Canva、DocuSign、Notion、Trello、Zoom。価格の履歴と着手金の請求書には会計ソフトまたは決済サービス | アップロードを入れ、DOCX/PDF を出す |
| `crm-autopilot` | CRM（HubSpot、kintone、Monday.com、Salesforce、Zoho CRM のいずれか） | カレンダー、Notion、Slack（Chatwork は `build-connector` 経由）、Trello、Zoom | 内蔵のスプレッドシート CRM |
| `growth-pulse` | HubSpot + 売上ソースを一つ | Mailchimp、PayPal、Shopify、Square、Stripe、TikTok広告 | CSV エクスポート |
| `content-strategy` | 会計ソフトまたは PayPal | Shopify、Square | 売上 CSV |
| `social-content-engine` | Canva | HubSpot、Mailchimp、Notion、Shopify、Trello | 投稿カレンダー＋原稿＋手動投稿用のブリーフ |
| `canva-creator` | Canva、HubSpot | Shopify、Square — 商品写真と価格は経営者に聞く前に店舗から読む | ブリーフを入れ、投稿カレンダー・原稿・素材ブリーフを出して手動でデザイン・投稿 |
| `review-reputation` | CRM、決済サービス（PayPal、Square、Stripe）、またはネットショップ・POS（Shopify または Square） | Gmail または M365、Zoho Desk | 貼り付け／エクスポートした口コミ＋Web |
| `ad-manager` | TikTok広告（ネイティブ）。Google 広告・Yahoo!広告・Meta広告・LINE広告は `build-connector` の Zapier 接続経由 | Canva | 成果の CSV エクスポート — 現時点で最も一般的な経路 |
| `seo-ai-visibility` | なし（Web ネイティブ） | Shopify、Wix | 公開サイトなら何でもクロール |
| `grant-rfp-writer` | Drive または M365 | DocuSign、カレンダー、Trello | 書類のアップロード＋Web 検索 |

## 作る・メタ

| スキル | 必須 | 任意 | 代替 |
|---|---|---|---|
| `build-agent` | なし | 作る対象に応じて何でも | ネイティブ |
| `build-connector` | なし | Emergent、Zapier | 代替経路そのものを作る |
| `smb-onboard` | なし | すべて | インタビュー形式。何を接続すべきかを提案 |
| `brand-style` | なし | WebFetch による Web サイト | 言葉でのブランド説明。スキップならハウスの既定 |

## コマンド（11 フォルダ＋スキル内チェーン 1）

11 個のコマンドフォルダを下に列挙する。12 番目の流れは、独立したフォルダではなく `speed-to-lead` スキルの中にある定期実行の問い合わせチェーンで、そのゲートは `speed-to-lead` と同じ（HubSpot + Gmail）。

コマンドは、連結するスキルの要件を引き継ぐ。先頭のスキルの要件でコマンドを開始できるかが決まり、任意の工程はスキルごとに縮退する。

| コマンド | 開始のゲート |
|---|---|
| `/report-pack` | データコネクタを一つ、または CSV |
| `/pay-the-bills` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など）+ メール（Gmail または M365） |
| `/restock` | Shopify、Square、または CSV |
| `/grow-pipeline` | Apollo または Clay + HubSpot |
| `/marketing-monday` | PayPal / Shopify / HubSpot のいずれか一つ。TikTok広告があれば広告費の層が加わる |
| `/reactivate` | CRM、決済サービス（PayPal、Square、Stripe）、またはネットショップ・POS（Shopify または Square）。会計ソフトとサポートデスクがあれば請求と対応の履歴が加わる |
| `/plan-payroll` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など）。勤怠の工程には給与計算ソフトの CSV エクスポート |
| `/close-month` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など） |
| `/tax-prep` | 会計ソフト（freee会計、マネーフォワード クラウド会計、弥生会計 など）。納税額の見積もりは日本の税制（法人税等・消費税・所得税）前提で、税理士確認を添える |
| `/monday-brief` | なし — 段階的に縮退する |
| `/call-list` | HubSpot。メール（Gmail または M365）でメールの文脈、Google カレンダーで予定枠、Apollo または Clay で企業の適合度（従量課金、一度だけ確認）が加わる |
