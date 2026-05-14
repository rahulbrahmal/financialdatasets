# RAHUL-39 Bursa Malaysia Equivalence Map

Prepared 2026-05-14 for a Malaysia/Bursa clone of the Financial Datasets API. This document maps the upstream API shape to Bursa-compatible identifiers, data domains, and response semantics. It complements the endpoint/schema inventory in `rahul-39-financialdatasets-api-inventory.md`.

## Clone Compatibility Principles

1. **Keep paths and response envelopes compatible first.** Existing clients should be able to call the same Financial Datasets paths and parse the same top-level JSON keys.
2. **Use Bursa-native identifiers internally.** Store Bursa stock code, short name, ISIN, market, sector, subsector, announcement id, attachment id, and source URL separately even when the public response has legacy names such as `ticker`, `cik`, or `accession_number`.
3. **Prefer explicit `null` over fabricated data.** Where the US source has a SEC/GAAP concept with no Malaysia equivalent, return `null` or omit optional fields in the same style as upstream earnings/KPI docs, and document the gap.
4. **Preserve provenance.** Every normalized financial, filing item, KPI, insider-trade row, corporate action, and price should retain `source_url`, source category, source timestamp, and parser version internally. Public fields like `filing_url`, `accession_number`, and `source_text` should point back to the Bursa announcement or attachment.
5. **Normalize currency and timezone.** Default reporting currency is `MYR`; some Bursa issuers report in other currencies and must retain the reported ISO currency. Bursa event times are `Asia/Kuala_Lumpur`; compatibility `time` fields may be UTC, but raw exchange-local timestamps should be retained.
6. **Route both slash forms.** Official docs often link trailing slash URLs while OpenAPI omits them. Clone should accept both and canonicalize.
7. **Separate official-disclosure data from editorial news.** Bursa announcements are issuer disclosures; news endpoints may combine announcements and licensed editorial feeds but should label the source.

## Identifier and Field Equivalence

| Financial Datasets concept | Bursa/Malaysia equivalent | Public compatibility recommendation | Implementation notes |
| --- | --- | --- | --- |
| `ticker` | Bursa stock/security code (`1818`, `5250`, `0800EA`, warrant/right codes) plus alias table for vendor symbols (`1818.KL`, `BURSA MK`, short names). | Use canonical Bursa stock code as `ticker`; accept aliases at request boundary. | Bursa pages expose `Stock Code` and company/security names. Codes can include suffix letters for ETFs, warrants, rights, and loan stocks, so treat as string, not integer. |
| `cik` | No SEC CIK exists. Closest stable public key is Bursa stock code/security code; company registration number may exist but is not a traded security id. | For `/company/facts/ciks` and `cik` params, either return stock-code aliases under `ciks` for drop-in compatibility or document `cik` unsupported. Recommended v1: accept `cik=<stock_code>` as an alias. | Avoid presenting the alias as a real SEC CIK. Internally name it `issuer_id` or `stock_code`. |
| `accession_number` | Bursa announcement id/reference id/message id, or deterministic synthetic id from announcement URL/date/category if an official id is unavailable. | Populate `accession_number` with the Bursa announcement id/reference used to retrieve the attachment. | Must be stable across re-ingestion. Keep `bursa_announcement_id` internally. |
| `filing_type` / `source_type` | Bursa announcement category/subcategory, e.g. `Financial Results`, `Annual Report`, `Annual Audited Account`, `General Announcement`, `Entitlements`, `Changes in Shareholdings`, `Shares Buy Back`. | Keep a Bursa-specific filing-type vocabulary in `/filings/types`; for strict SEC compatibility optionally map `10-Q -> Financial Results`, `10-K -> Annual Report/Annual Audited Account`, `8-K -> General Announcement`. | Do not force Bursa data into SEC forms without retaining original category. |
| `filing_url` / `url` | Bursa company announcement URL or attachment PDF URL. | Return the direct announcement/attachment URL in `filing_url`/`url`. | Store both landing-page and attachment URLs. |
| SEC `Item-*` | No direct item taxonomy. Closest equivalents are Bursa announcement categories, annual report sections, financial-result sections, and PDF headings. | For `/filings/items`, support a Bursa item taxonomy (`financial_result`, `management_discussion`, `risk`, `corporate_governance`, `entitlement_terms`) while optionally accepting SEC item names as aliases where meaningful. | Requires text extraction and section classifier. |
| `exchange` | Bursa Malaysia Securities market: Main Market, ACE Market, LEAP Market, ETF, Bond, Structured Warrants. | Use `exchange="BURSA"` or `exchange="BURSA MALAYSIA"`; expose `market` internally and possibly in `category`. | Company profile and listing directory expose markets. |
| `sector` / `industry` | Bursa sector/subsector/classification. | Return Bursa sector in `sector`; return Bursa subsector/classification in `industry`. | Do not call it GICS unless a licensed GICS mapping is added. |
| `currency` | Usually `MYR`; issuer-reported currency may differ. | Return ISO 4217 currency code exactly as reported/normalized. | Store monetary units and scaling from source attachments. |
| `report_period` | Financial period end date from Bursa Financial Results/Annual Report. | Use `YYYY-MM-DD` period end date. | Annual and quarterly periods can follow non-calendar fiscal years. |
| `fiscal_period` | `Q1`-`Q4`, `FY`, or upstream-like `2026-Q1`/`2025-FY`. | Mirror upstream labels where possible. | Bursa Financial Results titles include the financial period ended date; quarter number appears in structured announcement feeds. |
| `period` | `annual`, `quarterly`, `ttm`. | Support same enum. | TTM is computed, not filed. |
| `volume` | Bursa website displays volume in board lots/`'00`; feeds may provide shares/lots. | Return shares/units in the API to match upstream stock-price semantics; internally retain raw lot units and multiplier. | Bursa standard board lot is commonly 100 shares, but securities can have exceptions. |
| `time` | Market-data timestamp. | Upstream schema says UTC human-readable string; convert Bursa timestamps to UTC for public field and retain MYT internally. | Trading sessions are Monday-Friday 09:00-12:30 and 14:30-17:00 MYT per Bursa securities market page. |

## Bursa Source Model Needed for the Clone

| Data area | Primary Bursa/Malaysia source candidates | Fields unlocked | Caveats |
| --- | --- | --- | --- |
| Reference data | Bursa Listing Directory, company profile pages, ISIN page/NNA process, Bursa sectors/indices pages. | Stock code, company name, market, sector, subsector/classification, Shariah flag, website, ISIN, active/inactive status. | Public pages may be Cloudflare-protected and JS-rendered; obtain licensed/reference files where possible. |
| Announcements/filings | Bursa Company Announcements, Bursa LINK/announcement feeds, Announcements Message Structure v1.6, RSS where available. | Announcement id, category, subcategory, date/time, issuer, title, attachments, structured fields for financial results, entitlements, boardroom changes, shareholder/director interest, share buybacks. | Bursa disclaimer says announcements are provided by PLCs and not verified by Bursa. Need legal review for redistribution and automated extraction. |
| Financial results | Bursa `Financial Results`, `Annual Audited Account`, `Annual Report` categories; announcement message structure fields for financial results. | Revenue, profit/loss before/after tax, EPS, NTAB/share, dividends, quarter indicator, period start/end, year-to-date columns. | Full balance sheet and cash flow often require parsing PDF/XBRL-like tables from attachments; not all fields in Financial Datasets schema are available in the announcement feed. |
| Market data | Bursa website delayed data, Historical Data Packages, Information Services Vendors, BTS2 FIX market-data specification. | OHLCV, snapshots, market-wide snapshots, trading session/security status, real-time depth where licensed. | Bursa public website says prices are delayed by 15 minutes; real-time/display/non-display use requires licensed arrangements. |
| Corporate actions | Bursa `Entitlements`, `Important Relevant Dates for Renounceable Rights`, `Shares Buy Back`, `Additional Listing Announcement/Subdivision of Shares`. | Ex-date, entitlement date, payment date, entitlement type, numerator/denominator, dividend sen/percentage, share buyback and treasury-share changes. | Need security-code lifecycle model for rights/warrants/bonus issues. |
| Insider/director/shareholder changes | Bursa categories for director interest and substantial shareholder interest under Companies Act 2016 sections 137-139 and 219. | Name, address/nationality where public, registered holder, direct/indirect holdings, shares changed, price, date of change, action, total holdings. | Not identical to SEC Form 4; some officers who are not directors/substantial shareholders may not be covered. |
| Institutional ownership | Substantial shareholder notices, annual report top shareholders, possible share registrar/paid datasets. | Major shareholder holdings and changes. | No US 13F equivalent; investor-portfolio endpoint is a derived best-effort dataset, not a primary regulatory feed. |
| News | Bursa media releases/RSS, issuer announcements, licensed Malaysian business-news feeds. | `title`, `source`, `date`, `url`, optional ticker. | Publisher licensing is required for full-text/news redistribution. |
| Macro/rates | Bank Negara Malaysia FMIP, BNM Open API/portal, data.gov.my interest-rate dashboard and APIs. | OPR, MYOR/MYOR-i, KLIBOR transition data, government yields, FX reference rates. | Endpoint naming should include `BNM`; historical API availability must be verified. |
| Calendars | Bursa securities market page, Bursa holiday/trading calendar, announcements for special closures. | Trading days, sessions, market holidays, half days, session status. | Derivatives calendar is separate from securities calendar. |

## Endpoint-by-Endpoint Bursa Mapping

### Company Information

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /company/facts` | Accept `ticker=<stock_code-or-alias>`; accept `cik=<stock_code>` only as compatibility alias. | `company_facts.ticker` = Bursa stock code; `name` = issuer/security name; `cik` = stock-code alias or `null`; `industry` = Bursa subsector/classification; `sector` = Bursa sector; `category` = market/security type; `exchange` = `BURSA`; `is_active` from listing status; `location` from issuer address if available; `sec_filings_url` = Bursa announcements URL; `sic_*` = `null` unless a mapping is licensed. | Need exact active/delisted source and security vs issuer handling for warrants, ETFs, rights, REITs, ETBS. |
| `GET /company/facts/tickers` | No params. | `resource="company_facts"`; `tickers[]` = stock codes/accepted aliases. | Decide whether to include inactive/delisted securities; recommend active default, with internal status dimension. |
| `GET /company/facts/ciks` | No params. | Compatibility `ciks[]` = stock codes or synthetic issuer ids. | No real CIK; document alias behavior in public docs. |

### Earnings

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /earnings` company mode | `ticker` = stock code; `limit` = number of most recent report periods. | `earnings[]` rows from Bursa Financial Results and Annual Report/Annual Audited Account announcements. `source_type` = original category (`Financial Results`, `Annual Report`) or compatibility enum; `filing_date/datetime/window` = Bursa announcement timestamp in MYT-derived market window; `filing_url/accession_number` = Bursa source; `quarterly`/`annual` blocks from structured financial-result fields and normalized statements. | Analyst estimates, surprises, and `signals` require consensus/news analytics; return optional fields only when a licensed source exists. Malaysia filing sequence is not 8-K then 10-Q/10-K, but preliminary vs audited updates can still occur. |
| `GET /earnings` feed mode | Omit `ticker`; optional `limit`. | Latest financial-result/earnings-like Bursa announcements across all issuers, sorted by announcement date/time descending; dedupe by `(ticker, report_period)` if later audited/annual report supersedes quarterly result. | Human docs and OpenAPI disagree on mode/limits; support both. |
| `GET /earnings/tickers` | No params. | `tickers[]` = issuers with earnings/financial-result coverage. | Coverage begins only after parser is reliable for each issuer/category. |

### Market Data

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /prices` | `ticker` = stock code; `interval` = day/week/month/year; `start_date`, `end_date` in local trade dates. | `ticker`; `prices[]` with split-adjusted or raw OHLCV policy selected explicitly. `time` should be period timestamp in UTC for compatibility; store MYT trade date. | Bursa public delayed data is not a durable historical source. Need Historical Data Package or licensed vendor. Corporate action adjustment policy must be defined. |
| `GET /prices/snapshot` | `ticker` = stock code. | `snapshot.price` = last done; `day_change` and percent vs prior close/LACP; `time`/`time_milliseconds` from licensed feed or delayed website. | Public website data is delayed 15 minutes; real-time redistribution needs license. |
| `GET /prices/snapshot/tickers` | No params. | `tickers[]` = securities eligible for snapshots. | Include/exclude warrants, ETFs, ETBS by product policy. |
| `GET /prices/snapshot/market` | No params. | `snapshots[]` for all eligible Bursa securities. | Large response; subscription/licensing; may need pagination even if upstream does not. |
| `GET /prices/tickers` | No params. | `tickers[]` = securities with EOD historical price coverage. | Historical coverage differs by listed date and vendor. |

### Financial Statements and Search

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `POST /financials/search/screener` | Same JSON filters; fields come from normalized Bursa financials, metrics, and company sector/industry. | Return `search_results` or `results` after live-compat verification. Values in MYR unless field is ratio/percentage decimal. | Screener field list must exclude unsupported US-only fields or return clear validation errors. Sector values should use Bursa sectors, not GICS. |
| `POST /financials/search/line-items` | Same body with Bursa stock codes and canonical line items. | Rows include requested line-item keys, `ticker`, `report_period`, `period`, `currency`. | Need line-item dictionary mapping MFRS/Bursa labels to Financial Datasets canonical names. |
| `GET /financials/income-statements` | `ticker` or `cik` alias; `period`; `limit`; report-period filters. | Normalize Bursa/MFRS income statements to upstream `IncomeStatement` fields. `accession_number/filing_url` from source announcement/annual report. | Bursa financial-result feed gives core income lines; complete SG&A/R&D/preferred-dividend fields may require annual report notes and may be `null`. |
| `GET /financials/balance-sheets` | Same as upstream. | Normalize statement of financial position to `BalanceSheet` fields. | Many fields require PDF table extraction from quarterly/annual reports. Bank/insurer balance sheets need sector-specific mapping. |
| `GET /financials/cash-flow-statements` | Same as upstream. | Normalize cash-flow statements to `CashFlowStatement` fields. | Quarterly financial-result announcements may not include full cash flow; annual/interim PDFs needed. |
| `GET /financials` | Same params as component endpoints. | Bundle `{income_statements, balance_sheets, cash_flow_statements}` by period. | One statement may be unavailable for a period; preserve upstream nullable/missing behavior. |
| `GET /financials/income-statements/as-reported` | Same params, annual/quarterly only. | Recursive `AsReportedNode` tree preserving original Bursa labels, order, value, and children. | Requires robust PDF/table parser and hierarchy reconstruction. |
| `GET /financials/balance-sheets/as-reported` | Same. | As-reported balance sheet hierarchy. | Same parser risk; multi-column/year tables need period selection. |
| `GET /financials/cash-flow-statements/as-reported` | Same. | As-reported cash-flow hierarchy. | Same parser risk. |
| `GET /financials/as-reported` | Same. | Bundle three as-reported statement trees per period; any component can be null. | Annual reports may contain parent/company columns; choose group/consolidated by default. |
| `GET /financials/income-statements/segments` | Same params. | Segment categories from MFRS 8 notes or financial-result segment tables: product/segment arrays plus metadata. | Segment names and dimensions are issuer-specific; no guaranteed product/business distinction. |
| `GET /financials/balance-sheets/segments` | Same params. | Segment assets/liabilities/PPE/goodwill where disclosed. | Many issuers disclose segment assets annually only. |
| `GET /financials/cash-flow-statements/segments` | Same params. | Segment capex/cash-flow where disclosed. | Often absent; return empty arrays/nulls rather than infer. |
| `GET /financials/segments` | Same params. | Bundle segment disclosures across statements. | Period alignment and null handling required. |
| `GET /financials/search/screener/filters` | No params. | Return supported fields, operators, and enumerated Bursa sectors/industries. | Must dynamically reflect implemented fields. |
| Docs-only financial `/tickers` endpoints | No params. | `TickersResponse` for coverage by statement/search domain. | Not in OpenAPI; verify live before matching exact envelope. |

### Financial Metrics

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /financial-metrics` | `ticker`/`cik` alias, `period`, `limit`, report-period filters. | Compute valuation, profitability, liquidity, leverage, growth, and per-share metrics from Bursa price, market cap/shares, financials, and corporate actions. | Enterprise value requires debt/cash plus market cap; TTM and growth need restatement/corporate-action policy. |
| `GET /financial-metrics/snapshot` | `ticker`/`cik` alias. | Latest metrics snapshot using latest price and latest financials. | Real-time price licensing and stale financials need timestamp/provenance. |
| `GET /financial-metrics/snapshot/tickers` | No params. | `tickers[]` with snapshot coverage. | OpenAPI-free endpoint. |
| Docs-only `GET /financial-metrics/tickers` | No params. | `tickers[]` with historical metrics coverage. | Absent from OpenAPI. |

### SEC Filings / Bursa Announcements

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /filings` | `ticker` or `cik` alias; `filing_type` = Bursa category/subcategory or compatibility SEC alias; `limit`. | `filings[]`: `cik` = stock code alias; `accession_number` = announcement id; `filing_type` = Bursa category/subcategory; `report_date` = period end/effective date when available; `filing_date` = announcement date; `ticker`; `url`. | Need multi-category mapping and stable announcement ids. Support repeated `filing_type`. |
| `GET /filings/items` | `ticker`, Bursa filing/category, year/quarter, optional item/announcement id, `include_exhibits`. | `items[]` with item number/name/text from parsed announcement/attachment sections; `exhibits[]` for PDFs/HTML attachments. | SEC `Item-1A` etc. do not map directly. Build Bursa section taxonomy and alias SEC items only where useful. |
| `GET /filings/tickers` | No params. | `tickers[]` with announcement coverage. | Include active and historical issuers? choose active default. |
| `GET /filings/ciks` | No params. | `ciks[]` = stock-code/issuer-id aliases. | No real CIK. |
| `GET /filings/types` | No params. | Bursa category/subcategory list, seeded from Company Announcements and Announcements Message Structure. | Upstream enum is SEC forms; clone should expose Bursa types while optionally accepting SEC aliases. |
| `GET /filings/items/types` | Optional `filing_type`. | Supported Bursa item/section names by category. | Must reflect parser capabilities, not just theoretical categories. |

### Insider Trades

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /insider-trades` | `ticker`, `limit`, optional `name`, `transaction_type`, filing-date filters. | `insider_trades[]`: `issuer`, `name`, `title`/directorate, `is_board_director`, transaction date, shares, price, value, holdings before/after when computable, security title, transaction type, filing date. | Bursa sources are director/substantial-shareholder notices, not SEC Form 4. Some values (before holdings, title, indirect holdings) may be missing or require roll-forward calculations. |
| Docs-only `GET /insider-trades/tickers` | No params. | `tickers[]` with director/shareholder interest coverage. | Absent OpenAPI. |
| Docs-only `GET /insider-trades/names` | `ticker` required. | `names[]` for Bursa directors/substantial shareholders. | Normalize aliases and name changes. |
| Docs-only `GET /insider-trades/transaction-types` | No params. | `transaction_types[]` from Bursa action taxonomy. | Include acquisition/disposal/transfer/direct/indirect/treasury categories. |

### Institutional Ownership

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /institutional-ownership` | Either `ticker` or `investor`, but not both. | `institutional_ownership[]` or OpenAPI hyphen key after verification. For ticker mode: major/substantial shareholders and annual-report top shareholders. For investor mode: aggregate holdings across issuers by normalized investor name. | No 13F equivalent or 45-day standardized lag. Coverage and freshness depend on disclosures and annual reports. Security types include common stock, warrants, preferred/loan stock where relevant. |
| Docs-only `GET /institutional-ownership/investors` | No params. | `investors[]` from normalized shareholder/investor names. | May include individuals, nominees, funds, EPF/KWAP, and nominee accounts; identity resolution is hard. |
| Docs-only `GET /institutional-ownership/tickers` | No params. | `tickers[]` with ownership coverage. | Absent OpenAPI. |

### News

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /news` | Optional `ticker`; `limit` default/max as upstream after verification. | If `ticker` provided, issuer announcements plus licensed company news. If omitted, broad Bursa/Malaysia market news. `news[]` fields map to ticker, title, source, date, url. | Bursa announcements are not editorial news; label `source` clearly. Publisher full text/licensing not covered by Bursa data. |

### Macroeconomics

| Endpoint | Bursa/Malaysia request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /macro/interest-rates` | `bank=BNM` primary; optional `start_date`, `end_date`. | `interest_rates[]`: `bank="BNM"`, `name="Bank Negara Malaysia"`, `rate`, `date`. Additional codes could include `MYOR`, `MYOR-I`, `KLIBOR`, `MGS10Y` if the bank-code model is extended. | Upstream endpoint is central-bank policy rates; Malaysia rates include policy rates, transaction-based MYOR, yields, and reference FX. Keep codes explicit. |
| `GET /macro/interest-rates/snapshot` | `bank=BNM`. | Latest OPR/MYOR-like rate under same envelope. | Decide whether snapshot returns only OPR or multiple BNM rates; upstream requires one bank code. |
| `GET /macro/interest-rates/banks` | No params. | `banks[]` includes `BNM`; optionally aliases `BANK_NEGARA_MALAYSIA`. | If global banks remain supported, keep upstream codes too. |

### KPIs, Guidance, and Non-GAAP

| Endpoint | Bursa request semantics | Response mapping | Gaps/edge cases |
| --- | --- | --- | --- |
| `GET /kpi/metrics` | `ticker`, optional metric/period/report-period filters, `limit`. | `kpi_metrics[]` from sector-specific Bursa disclosures: plantation FFB/CPO, REIT NPI/occupancy, banks CET1/NIM, airlines load factor, telco ARPU/subscribers, utilities capacity, property sales/unbilled sales, etc. | Requires sector dictionaries, unit normalization, source text spans, and human QA. |
| `GET /kpi/metrics/tickers` | No params. | `tickers[]` with KPI coverage. | OpenAPI requires auth; docs do not state free. |
| `GET /kpi/metrics/sectors` | No params. | `sectors[]` using Bursa sectors and internal KPI sector groups. | Sector names should not be GICS unless mapped/licensed. |
| `GET /kpi/guidance` | Same filters as upstream. | `kpi_guidance[]` from guidance statements in Bursa announcements, annual reports, investor presentations, or results briefings. | Guidance may be qualitative; `low/high/point_estimate` often null. |
| Docs-only `GET /kpi/guidance/tickers` | No params. | `tickers[]` with guidance coverage. | Absent OpenAPI. |
| `GET /kpi/non-gaap` | Same filters as upstream. | `kpi_non_gaap[]` for non-MFRS/non-GAAP metrics, adjusted earnings, core profit, EBITDA, distributable income, with `gaap_equivalent` and `key_adjustments`. | Malaysia issuers may use non-MFRS measures inconsistently; require source-text evidence and reconciliation. |
| Docs-only `GET /kpi/non-gaap/tickers` | No params. | `tickers[]` with non-GAAP coverage. | Absent OpenAPI. |

## Endpoint Implementation Priority

1. **Reference + announcements foundation:** `/company/facts`, `/company/facts/tickers`, `/filings`, `/filings/tickers`, `/filings/types`, `/filings/items/types`.
2. **Market data with licensing decision:** `/prices`, `/prices/snapshot`, `/prices/tickers`, `/prices/snapshot/tickers`.
3. **Financial statements:** normalized and as-reported income/balance/cash-flow endpoints, then `/financials` bundle and screener.
4. **Corporate actions and insider/shareholder changes:** `/insider-trades`, `/institutional-ownership`, entitlement-derived internal models even though Financial Datasets has no dedicated corporate-action endpoint.
5. **Metrics and search:** `/financial-metrics`, snapshots, screeners, line-items once statements and price/share data are stable.
6. **Macro/news/KPI:** BNM interest rates, licensed news, and LLM/extraction-heavy KPI/guidance/non-GAAP endpoints.

## Open Implementation Questions

- Should public `ticker` use bare Bursa stock code (`1818`) or vendor-style suffix (`1818.KL`)? Recommendation: bare stock code canonical, alias suffixes accepted.
- Should `cik` compatibility endpoints return stock codes or be unsupported? Recommendation: return stock-code aliases with prominent docs warning.
- Should `/filings/types` expose Bursa categories only, SEC aliases only, or both? Recommendation: Bursa categories plus `sec_aliases` internally; accept both in request validation.
- Should public price `volume` be shares or Bursa lots? Recommendation: shares/units for compatibility, raw lots stored separately.
- Should screener response use `results` or `search_results`? Requires live upstream verification.
- Can Bursa official feeds be licensed for redistribution/non-display use? This is a commercial/legal blocker for production market data and announcement feeds.
