# RAHUL-39 Financial Datasets API Inventory

Prepared for cloning the Financial Datasets API response structure for Bursa Malaysia. Access date for live sources: 2026-05-14.

## Crawl and Source Coverage

- Financial Datasets documentation index: <https://docs.financialdatasets.ai/llms.txt>.
- Human API pages downloaded from every `llms.txt` link: 31 endpoint/example pages plus guides/support/coverage pages.
- Machine-readable specs reviewed: <https://docs.financialdatasets.ai/openapi-spec.md>, <https://docs.financialdatasets.ai/api/openapi.json>, and <https://financialdatasets.ai/openapi.json>.
- OpenAPI version `3.0.1`, API title `Financial Datasets API`, operations found: **45** across **45** paths.
- Important reconciliation: the docs navigation exposes narrative pages for the main business endpoints, while the OpenAPI contains additional free availability/list endpoints. This inventory treats all OpenAPI operations as endpoints to clone.

## Global Protocol, Auth, Errors, and Pagination

- Base URL in the official spec: `https://api.financialdatasets.ai/`.
- Default auth: `X-API-KEY` header (`apiKey` security scheme). Endpoints with operation-level `security: []` are documented as free and do not require authentication.
- Do not copy examples with real keys; examples in this repo use `<api-key>` placeholders only.
- Error schema where referenced: `ErrorResponse` = `{ "error": string, "message": string }`. Documented status codes include `400`, `401`, `402`, `403`, `404`, and `503` depending on endpoint. Empty OpenAPI descriptions mean the docs identify the status code but not body-specific semantics.
- No numeric rate-limit policy was found in the docs or OpenAPI. Treat rate limits, retry headers, and quota reset behavior as **unknown** until confirmed with the upstream provider.
- No cursor/offset pagination was found. Collection endpoints use bounded `limit` parameters and/or date ranges. Availability endpoints return complete lists in one response unless implementation-specific limits are later introduced.
- Dates are ISO `YYYY-MM-DD` unless noted. Financial Datasets earnings filing times are in SEC Eastern Time; price schema timestamps are UTC. Bursa clone should store exchange-local `Asia/Kuala_Lumpur` timestamps internally and decide whether public compatibility responses remain UTC for `time` fields.

## Docs/OpenAPI Reconciliation Notes

The current documentation set has several compatibility-relevant discrepancies. Implementers should add regression tests for whichever behavior is confirmed against the live upstream API.

| Area | Human docs | OpenAPI | Clone recommendation |
| --- | --- | --- | --- |
| `GET /earnings` | Two modes are documented: company mode with `ticker`, and feed mode when `ticker` is omitted. Feed mode defaults `limit=10`, max `100`, sorts by `filing_date` descending, and deduplicates `(ticker, report_period)`. | `ticker` is required; `limit` defaults to `1` and maxes at `40`. | Support both modes. For Bursa, company mode returns issuer-specific Financial Results/Annual Report entries; feed mode returns latest Bursa earnings/financial-result announcements across issuers. |
| `POST /financials/search/screener` | Example response uses top-level `results`. | `FinancialsSearchResponse` uses top-level `search_results`. | Choose one canonical response after live verification. For strict OpenAPI compatibility use `search_results`; for docs/client compatibility consider returning both during migration. |
| Statement defaults | Narrative pages say `period=annual`, `limit=4`, `report_period=null` defaults for statement endpoints. | OpenAPI marks `period` required and does not encode defaults for every endpoint. | Require `period` for deterministic clone behavior, but support documented defaults if upstream-compatible clients depend on them. |
| Availability endpoints | Many parent pages link to domain-specific `/tickers`, `/names`, `/transaction-types`, or `/investors` endpoints. | Some are present (`/prices/tickers`, `/earnings/tickers`, etc.); others are absent. | Treat human-docs links as discovered endpoints until live verification proves otherwise. They are inventoried below as docs-only endpoints. |
| Trailing slashes | Human docs often include trailing slash URLs. | OpenAPI paths omit trailing slashes. | Route both forms or redirect trailing slash to canonical non-slash paths. |
| `GET /institutional-ownership` envelope | Human examples parse `institutional_ownership`. | OpenAPI property is `institutional-ownership` with a hyphen. | Verify live behavior. Prefer `institutional_ownership` for idiomatic JSON/client usability unless upstream strict compatibility requires the hyphen. |
| Insider-trade `limit` | Narrative docs say default `100`, max `1000`. | OpenAPI says default `10` and no max. | Verify live behavior; clone should explicitly set and validate a max to avoid unbounded scans. |


## Canonical OpenAPI Endpoint Inventory

| Domain | Endpoint | operationId | Auth | 200 response | Bursa clone equivalent |
| --- | --- | --- | --- | --- | --- |
| Financial Statements | `POST /financials/search/screener` | `searchFinancials` | X-API-KEY | FinancialsSearchResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `POST /financials/search/line-items` | `searchLineItems` | X-API-KEY | FinancialsSearchResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/income-statements/segments` | `getIncomeStatementSegments` | X-API-KEY | IncomeStatementSegmentsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/balance-sheets/segments` | `getBalanceSheetSegments` | X-API-KEY | BalanceSheetSegmentsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/cash-flow-statements/segments` | `getCashFlowStatementSegments` | X-API-KEY | CashFlowStatementSegmentsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/income-statements/as-reported` | `getIncomeStatementAsReported` | X-API-KEY | AsReportedIncomeStatementsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/balance-sheets/as-reported` | `getBalanceSheetAsReported` | X-API-KEY | AsReportedBalanceSheetsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/cash-flow-statements/as-reported` | `getCashFlowStatementAsReported` | X-API-KEY | AsReportedCashFlowStatementsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/as-reported` | `getAllAsReportedFinancials` | X-API-KEY | AsReportedFinancialsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/segments` | `getAllSegmentedFinancials` | X-API-KEY | SegmentedFinancialsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/income-statements` | `getIncomeStatements` | X-API-KEY | IncomeStatementResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/balance-sheets` | `getBalanceSheets` | X-API-KEY | BalanceSheetResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/cash-flow-statements` | `getCashFlowStatements` | X-API-KEY | CashFlowStatementResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials` | `getAllFinancialStatements` | X-API-KEY | FinancialsResponse | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Financial Statements | `GET /financials/search/screener/filters` | `getScreenerFilters` | free/no auth | object | Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction. |
| Market Data | `GET /prices` | `getPrices` | X-API-KEY | PricesResponse | Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds. |
| Market Data | `GET /prices/snapshot` | `getPriceSnapshot` | X-API-KEY | PriceSnapshotResponse | Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds. |
| Market Data | `GET /prices/snapshot/tickers` | `getPriceSnapshotTickers` | free/no auth | TickersResponse | Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds. |
| Market Data | `GET /prices/snapshot/market` | `getPriceSnapshotMarket` | X-API-KEY | PriceSnapshotMarketResponse | Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds. |
| Market Data | `GET /prices/tickers` | `getPricesTickers` | free/no auth | TickersResponse | Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds. |
| Company Information | `GET /company/facts` | `getCompanyFacts` | X-API-KEY | CompanyFactsResponse | Bursa listing directory/company profile, stock code, short name, ISIN, market, sector, Shariah flag. |
| Company Information | `GET /company/facts/tickers` | `getCompanyFactsTickers` | free/no auth | TickersResponse | Bursa listing directory/company profile, stock code, short name, ISIN, market, sector, Shariah flag. |
| Company Information | `GET /company/facts/ciks` | `getCompanyFactsCiks` | free/no auth | CiksResponse | Bursa listing directory/company profile, stock code, short name, ISIN, market, sector, Shariah flag. |
| Earnings | `GET /earnings` | `getEarnings` | X-API-KEY | EarningsResponse | Bursa Financial Results and Annual Report announcements; estimates/surprises require a separate consensus vendor. |
| Earnings | `GET /earnings/tickers` | `getEarningsTickers` | free/no auth | TickersResponse | Bursa Financial Results and Annual Report announcements; estimates/surprises require a separate consensus vendor. |
| News | `GET /news` | `getNews` | X-API-KEY | NewsResponse | Bursa announcements/media plus licensed Malaysian publisher/RSS feeds; announcements are official-disclosure data, not editorial news. |
| SEC Filings | `GET /filings` | `getFilings` | X-API-KEY | FilingsResponse | Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id. |
| SEC Filings | `GET /filings/items` | `getFilingItems` | X-API-KEY | FilingItemsResponse | Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id. |
| SEC Filings | `GET /filings/tickers` | `getFilingsTickers` | free/no auth | TickersResponse | Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id. |
| SEC Filings | `GET /filings/ciks` | `getFilingsCiks` | free/no auth | CiksResponse | Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id. |
| SEC Filings | `GET /filings/types` | `getFilingsTypes` | free/no auth | object | Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id. |
| SEC Filings | `GET /filings/items/types` | `getFilingsItemsTypes` | free/no auth | object | Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id. |
| Insider Trades | `GET /insider-trades` | `getInsiderTrades` | X-API-KEY | InsiderTradesResponse | Director interest and substantial shareholder announcements under Companies Act 2016 sections 137-139/219. |
| Institutional Ownership | `GET /institutional-ownership` | `getInstitutionalOwnership` | X-API-KEY | InstitutionalOwnershipResponse | No 13F equivalent; approximate from substantial shareholder notices, annual reports, and licensed shareholder data. |
| Financial Metrics | `GET /financial-metrics` | `getFinancialMetrics` | X-API-KEY | FinancialMetricsResponse | Derived from Bursa prices, normalized Bursa/MFRS statements, shares outstanding, corporate actions, and FX/MYR metadata. |
| Financial Metrics | `GET /financial-metrics/snapshot` | `getFinancialMetricsSnapshot` | X-API-KEY | FinancialMetricSnapshotResponse | Derived from Bursa prices, normalized Bursa/MFRS statements, shares outstanding, corporate actions, and FX/MYR metadata. |
| Financial Metrics | `GET /financial-metrics/snapshot/tickers` | `getFinancialMetricsSnapshotTickers` | free/no auth | TickersResponse | Derived from Bursa prices, normalized Bursa/MFRS statements, shares outstanding, corporate actions, and FX/MYR metadata. |
| Macroeconomics | `GET /macro/interest-rates` | `getInterestRates` | X-API-KEY | InterestRatesResponse | Bank Negara Malaysia/data.gov.my OPR/MYOR/KLIBOR/rates and FX sources; use BNM as primary bank code. |
| Macroeconomics | `GET /macro/interest-rates/snapshot` | `getInterestRatesSnapshot` | X-API-KEY | InterestRatesResponse | Bank Negara Malaysia/data.gov.my OPR/MYOR/KLIBOR/rates and FX sources; use BNM as primary bank code. |
| Macroeconomics | `GET /macro/interest-rates/banks` | `getInterestRatesBanks` | free/no auth | object | Bank Negara Malaysia/data.gov.my OPR/MYOR/KLIBOR/rates and FX sources; use BNM as primary bank code. |
| KPIs | `GET /kpi/metrics` | `getKPIMetrics` | X-API-KEY | KPIMetricsResponse | Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics. |
| KPIs | `GET /kpi/metrics/tickers` | `getKPIMetricsTickers` | X-API-KEY | object | Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics. |
| KPIs | `GET /kpi/metrics/sectors` | `getKPIMetricsSectors` | X-API-KEY | object | Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics. |
| KPIs | `GET /kpi/guidance` | `getKPIGuidance` | X-API-KEY | KPIGuidanceResponse | Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics. |
| KPIs | `GET /kpi/non-gaap` | `getKPINonGAAP` | X-API-KEY | KPINonGAAPResponse | Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics. |

## Additional Human-Docs-Only Endpoints Absent From OpenAPI

These endpoints were linked from narrative pages or guides but were not present as operations in the downloaded OpenAPI JSON on 2026-05-14. They should be verified live before implementation, but they matter for clone completeness because official examples tell users to call them.

| Method/path | Source page | Auth documented? | Parameters | Expected response shape / field semantics | Edge cases | Bursa mapping |
| --- | --- | --- | --- | --- | --- | --- |
| `GET /financial-metrics/tickers` | `api/financial-metrics/historical.md` | Not explicitly stated on the link; parent endpoint requires `X-API-KEY`. | none | Inferred `TickersResponse`: `resource`, `tickers[]`. | OpenAPI has `/financial-metrics/snapshot/tickers` but not this historical variant. | Bursa stock codes with enough normalized financials and price data to compute historical metrics. |
| `GET /financials/tickers` | `api/financials/all-financial-statements.md`; `market-coverage.md` | Not explicitly stated. | none | Inferred `TickersResponse`: `resource`, `tickers[]`. | Parent docs advertise available tickers for all statements. | Bursa issuers with at least one parsed financial-statement package. |
| `GET /financials/income-statements/tickers` | `api/financials/income-statements.md`; guide | Not explicitly stated. | none | Inferred `TickersResponse`. | Trailing slash appears in examples. | Bursa issuers with normalized income statement coverage. |
| `GET /financials/balance-sheets/tickers` | `api/financials/balance-sheets.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | Trailing slash appears in examples. | Bursa issuers with normalized balance sheet coverage. |
| `GET /financials/cash-flow-statements/tickers` | `api/financials/cash-flow-statements.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | Trailing slash appears in examples. | Bursa issuers with normalized cash-flow statement coverage. |
| `GET /financials/segments/tickers` | `api/financials/all-segments.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | OpenAPI has `/financials/segments` but not this list endpoint. | Bursa issuers with parsed segment note coverage. |
| `GET /financials/income-statements/segments/tickers` | `api/financials/income-statement-segments.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | OpenAPI has the data endpoint only. | Bursa issuers with revenue/profit segment disclosures. |
| `GET /financials/balance-sheets/segments/tickers` | `api/financials/balance-sheet-segments.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | OpenAPI has the data endpoint only. | Bursa issuers with asset/goodwill/PPE segment disclosures. |
| `GET /financials/cash-flow-statements/segments/tickers` | `api/financials/cash-flow-statement-segments.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | OpenAPI has the data endpoint only. | Bursa issuers with capex/cash-flow segment disclosures. |
| `GET /insider-trades/tickers` | `api/insider-trades/insider-trades.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | Needed to discover Form 4 coverage upstream. | Bursa issuers with director/substantial-shareholder interest announcements. |
| `GET /insider-trades/names` | `api/insider-trades/insider-trades.md` | Not explicitly stated. | `ticker` required in example. | Inferred `{ "names": [string] }`; names are available insiders for a ticker. | Mentioned by the OpenAPI parameter description but no operation exists. | Directors/substantial shareholders for a Bursa stock code, normalized with aliases. |
| `GET /insider-trades/transaction-types` | `api/insider-trades/insider-trades.md` | Not explicitly stated. | none | Inferred `{ "transaction_types": [string] }`. | Mentioned by the OpenAPI parameter description but no operation exists. | Bursa action/type taxonomy such as acquisition, disposal, direct/indirect interest change, treasury/share buyback where applicable. |
| `GET /institutional-ownership/investors` | `api/institutional-ownership/investor.md` | Not explicitly stated. | none | Inferred `{ "investors": [string] }`. | No OpenAPI operation. | Investors/shareholders discovered from substantial-shareholder notices and annual reports; not equivalent to US 13F managers. |
| `GET /institutional-ownership/tickers` | `api/institutional-ownership/ticker.md` | Not explicitly stated. | none | Inferred `TickersResponse`. | No OpenAPI operation. | Bursa issuers with ownership/shareholder coverage. |
| `GET /kpi/guidance/tickers` | `api/kpi/guidance.md` | Not explicitly stated; parent is Pro+ in OpenAPI. | none | Inferred `TickersResponse`. | No OpenAPI operation. | Bursa issuers with extracted forward guidance from announcements/presentations. |
| `GET /kpi/non-gaap/tickers` | `api/kpi/non-gaap.md` | Not explicitly stated; parent is Pro+ in OpenAPI. | none | Inferred `TickersResponse`. | No OpenAPI operation. | Bursa issuers with non-MFRS/non-GAAP metrics and reconciliation text. |

## Financial Statements

Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `POST /financials/search/screener`

- **Operation ID:** `searchFinancials`
- **Summary:** Search financial statements
- **Description:** Search for stocks by filtering across financial metrics from income statements, balance sheets, and cash flow statements.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/search-screener.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Request body**

| Content type | Schema | Required |
| --- | --- | --- |
| application/json | object | yes |

Body field semantics (`SearchFiltersRequest`):

| Field | Type | Meaning |
| --- | --- | --- |
| limit | integer | The maximum number of results to return. |
| filters | array<object> | An array of filter objects to apply to the search. |

**Response schema and example shape**

- 200 schema: `FinancialsSearchResponse`

```json
{
  "search_results": [
    {
      "ticker": "...",
      "report_period": "...",
      "period": "...",
      "currency": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| search_results | array<object> | No field description in OpenAPI. |

Nested item field semantics (`item/object under `search_results`` from `search_results`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| report_period | string/date | The reporting period of the financial data. |
| period | string enum=annual\|quarterly\|ttm | The time period of the financial data. |
| currency | string | The currency of the financial data. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Successful search response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** POST body filters are AND-combined; string company fields support eq/in and are case-insensitive; ratios/margins are decimals, not percentages; default limit 10.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `POST /financials/search/line-items`

- **Operation ID:** `searchLineItems`
- **Summary:** Search specific financial metrics
- **Description:** Search for specific financial metrics across income statements, balance sheets, and cash flow statements for a list of tickers.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/search-line-items.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Request body**

| Content type | Schema | Required |
| --- | --- | --- |
| application/json | object | yes |

Body field semantics (`SearchLineItemsRequest`):

| Field | Type | Meaning |
| --- | --- | --- |
| line_items | array<string> | An array of line items to apply to the search. |
| tickers | array<string> | An array of tickers to apply to the search. |
| period | string enum=annual\|quarterly\|ttm | The time period for the financial data. |
| limit | integer | The maximum number of results to return. |

**Response schema and example shape**

- 200 schema: `FinancialsSearchResponse`

```json
{
  "search_results": [
    {
      "ticker": "...",
      "report_period": "...",
      "period": "...",
      "currency": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| search_results | array<object> | No field description in OpenAPI. |

Nested item field semantics (`item/object under `search_results`` from `search_results`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| report_period | string/date | The reporting period of the financial data. |
| period | string enum=annual\|quarterly\|ttm | The time period of the financial data. |
| currency | string | The currency of the financial data. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Successful search response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Searches requested line_items across requested tickers and period; returns search_results rows with requested fields.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/income-statements/segments`

- **Operation ID:** `getIncomeStatementSegments`
- **Summary:** Get income statement segments
- **Description:** Get income statement segment breakdowns (revenue, operating income, depreciation) by product and business segment.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/income-statement-segments.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `IncomeStatementSegmentsResponse`

```json
{
  "segmented_financials": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Income statement segments response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Segment categories include product and business/operating segment arrays plus SEC provenance.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/balance-sheets/segments`

- **Operation ID:** `getBalanceSheetSegments`
- **Summary:** Get balance sheet segments
- **Description:** Get balance sheet segment breakdowns (assets, goodwill, long-lived assets) by business segment.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/balance-sheet-segments.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `BalanceSheetSegmentsResponse`

```json
{
  "segmented_financials": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Balance sheet segments response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Segment categories focus on assets/goodwill/long-lived-assets style disclosures where available.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/cash-flow-statements/segments`

- **Operation ID:** `getCashFlowStatementSegments`
- **Summary:** Get cash flow statement segments
- **Description:** Get cash flow statement segment breakdowns (capital expenditure) by business segment.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/cash-flow-statement-segments.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `CashFlowStatementSegmentsResponse`

```json
{
  "segmented_financials": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Cash flow statement segments response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Segment categories focus on capex/cash-flow segment disclosures where available.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/income-statements/as-reported`

- **Operation ID:** `getIncomeStatementAsReported`
- **Summary:** Get income statement (as-reported)
- **Description:** Get the as-filed income statement hierarchy. Each line item is returned exactly as it appears on the face of the 10-K or 10-Q, with parent-child relationships preserved in the `children` field.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/income-statements.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `AsReportedIncomeStatementsResponse`

```json
{
  "as_reported_income_statements": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_income_statements | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | As-reported income statements response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Preserves filed labels/order via recursive children hierarchy; period supports annual/quarterly only.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/balance-sheets/as-reported`

- **Operation ID:** `getBalanceSheetAsReported`
- **Summary:** Get balance sheet (as-reported)
- **Description:** Get the as-filed balance sheet hierarchy. Each line item is returned exactly as it appears on the face of the 10-K or 10-Q, with parent-child relationships preserved in the `children` field.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/balance-sheets.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `AsReportedBalanceSheetsResponse`

```json
{
  "as_reported_balance_sheets": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_balance_sheets | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | As-reported balance sheets response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Preserves filed labels/order via recursive children hierarchy; period supports annual/quarterly only.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/cash-flow-statements/as-reported`

- **Operation ID:** `getCashFlowStatementAsReported`
- **Summary:** Get cash flow statement (as-reported)
- **Description:** Get the as-filed cash flow statement hierarchy. Each line item is returned exactly as it appears on the face of the 10-K or 10-Q, with parent-child relationships preserved in the `children` field.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/cash-flow-statements.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `AsReportedCashFlowStatementsResponse`

```json
{
  "as_reported_cash_flow_statements": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_cash_flow_statements | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | As-reported cash flow statements response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Preserves filed labels/order via recursive children hierarchy; period supports annual/quarterly only.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/as-reported`

- **Operation ID:** `getAllAsReportedFinancials`
- **Summary:** Get all as-reported financials
- **Description:** Get the as-filed hierarchies for income statement, balance sheet, and cash flow statement in a single API call. Each period contains three nested objects; any one may be null if data is missing.
- **Docs/source page:** OpenAPI only; as-reported view described from statement pages
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `AsReportedFinancialsResponse`

```json
{
  "as_reported_financials": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_financials | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | All as-reported financials response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Bundles as-filed hierarchies; any of the three statement objects may be null for a period.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/segments`

- **Operation ID:** `getAllSegmentedFinancials`
- **Summary:** Get all segmented financials
- **Description:** Get segment breakdowns from all three financial statement types (income statement, balance sheet, cash flow) in a single API call.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/all-segments.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly | The time period of the data. (enum `annual`, `quarterly`) |
| limit | query | no | integer/int32 | The maximum number of periods to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `SegmentedFinancialsResponse`

```json
{
  "segmented_financials": [
    null
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | All segmented financials response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Bundles segment breakdowns from income, balance sheet, and cash flow statements.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/income-statements`

- **Operation ID:** `getIncomeStatements`
- **Summary:** Get income statements
- **Description:** Get income statements for a ticker.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/income-statements.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly\|ttm | The time period of the income statements. (enum `annual`, `quarterly`, `ttm`) |
| limit | query | no | integer/int32 | The maximum number of income statements to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `IncomeStatementResponse`

```json
{
  "income_statements": [
    {
      "ticker": "...",
      "report_period": "...",
      "fiscal_period": "...",
      "period": "...",
      "currency": "...",
      "accession_number": "...",
      "filing_url": "...",
      "revenue": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| income_statements | array<object> | No field description in OpenAPI. |

Nested item field semantics (`IncomeStatement` from `income_statements`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| report_period | string/date | The reporting period of the income statement. |
| fiscal_period | string | The fiscal period of the income statement. |
| period | string enum=quarterly\|ttm\|annual | The time period of the income statement. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| revenue | number | The total revenue of the company. |
| cost_of_revenue | number | The cost of revenue of the company. |
| gross_profit | number | The gross profit of the company. |
| operating_expense | number | The operating expenses of the company. |
| selling_general_and_administrative_expenses | number | The selling, general, and administrative expenses of the company. |
| research_and_development | number | The research and development expenses of the company. |
| operating_income | number | The operating income of the company. |
| interest_expense | number | The interest expenses of the company. |
| ebit | number | The earnings before interest and taxes of the company. |
| income_tax_expense | number | The income tax expenses of the company. |
| net_income_discontinued_operations | number | The net income from discontinued operations of the company. |
| net_income_non_controlling_interests | number | The net income from non-controlling interests of the company. |
| net_income | number | The net income of the company. |
| net_income_common_stock | number | The net income available to common stockholders of the company. |
| preferred_dividends_impact | number | The impact of preferred dividends on the net income of the company. |
| consolidated_income | number | The consolidated income of the company. |
| earnings_per_share | number | The earnings per share of the company. |
| earnings_per_share_diluted | number | The diluted earnings per share of the company. |
| dividends_per_common_share | number | The dividends per common share of the company. |
| weighted_average_shares | number | The weighted average shares of the company. |
| weighted_average_shares_diluted | number | The diluted weighted average shares of the company. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Income statements response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Normalized schema, annual/quarterly/ttm; docs state normalized coverage from the 1990s onward and as-reported coverage from 2010 onward.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/balance-sheets`

- **Operation ID:** `getBalanceSheets`
- **Summary:** Get balance sheets
- **Description:** Get balance sheets for a ticker.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/balance-sheets.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly\|ttm | The time period of the balance sheets. (enum `annual`, `quarterly`, `ttm`) |
| limit | query | no | integer/int32 | The maximum number of balance sheets to return |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `BalanceSheetResponse`

```json
{
  "balance_sheets": [
    {
      "ticker": "...",
      "report_period": "...",
      "fiscal_period": "...",
      "period": "...",
      "currency": "...",
      "accession_number": "...",
      "filing_url": "...",
      "total_assets": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| balance_sheets | array<object> | No field description in OpenAPI. |

Nested item field semantics (`BalanceSheet` from `balance_sheets`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| report_period | string/date | The reporting period of the balance sheet. |
| fiscal_period | string | The fiscal period of the balance sheet. |
| period | string enum=quarterly\|ttm\|annual | The time period of the balance sheet. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| total_assets | number | The total assets of the company. |
| current_assets | number | The current assets of the company. |
| cash_and_equivalents | number | The cash and equivalents of the company. |
| inventory | number | The inventory of the company. |
| current_investments | number | The current investments of the company. |
| trade_and_non_trade_receivables | number | The trade and non-trade receivables of the company. |
| non_current_assets | number | The non-current assets of the company. |
| property_plant_and_equipment | number | The property, plant, and equipment of the company. |
| goodwill_and_intangible_assets | number | The goodwill and intangible assets of the company. |
| investments | number | The investments of the company. |
| non_current_investments | number | The non-current investments of the company. |
| outstanding_shares | number | The outstanding shares of the company. |
| tax_assets | number | The tax assets of the company. |
| total_liabilities | number | The total liabilities of the company. |
| current_liabilities | number | The current liabilities of the company. |
| current_debt | number | The current debt of the company. |
| trade_and_non_trade_payables | number | The trade and non-trade payables of the company. |
| deferred_revenue | number | The deferred revenue of the company. |
| deposit_liabilities | number | The deposit liabilities of the company. |
| non_current_liabilities | number | The non-current liabilities of the company. |
| non_current_debt | number | The non-current debt of the company. |
| tax_liabilities | number | The tax liabilities of the company. |
| shareholders_equity | number | The shareholders' equity of the company. |
| retained_earnings | number | The retained earnings of the company. |
| accumulated_other_comprehensive_income | number | The accumulated other comprehensive income of the company. |
| total_debt | number | The total debt of the company. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Balance sheets response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Normalized schema, annual/quarterly/ttm; ticker required unless cik is provided.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/cash-flow-statements`

- **Operation ID:** `getCashFlowStatements`
- **Summary:** Get cash flow statements
- **Description:** Get cash flow statements for a ticker.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/cash-flow-statements.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly\|ttm | The time period of the cash flow statements. (enum `annual`, `quarterly`, `ttm`) |
| limit | query | no | integer/int32 | The maximum number of cash flow statements to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `CashFlowStatementResponse`

```json
{
  "cash_flow_statements": [
    {
      "ticker": "...",
      "report_period": "...",
      "fiscal_period": "...",
      "period": "...",
      "currency": "...",
      "accession_number": "...",
      "filing_url": "...",
      "net_income": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| cash_flow_statements | array<object> | No field description in OpenAPI. |

Nested item field semantics (`CashFlowStatement` from `cash_flow_statements`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| report_period | string/date | The reporting period of the cash flow statement. |
| fiscal_period | string | The fiscal period of the cash flow statement. |
| period | string enum=quarterly\|ttm\|annual | The time period of the cash flow statement. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| net_income | number | The net income of the company. |
| depreciation_and_amortization | number | The depreciation and amortization of the company. |
| share_based_compensation | number | The share-based compensation of the company. |
| net_cash_flow_from_operations | number | The net cash flow from operations of the company. |
| capital_expenditure | number | The capital expenditure of the company. |
| business_acquisitions_and_disposals | number | The business acquisitions and disposals of the company. |
| investment_acquisitions_and_disposals | number | The investment acquisitions and disposals of the company. |
| net_cash_flow_from_investing | number | The net cash flow from investing of the company. |
| issuance_or_repayment_of_debt_securities | number | The issuance or repayment of debt securities of the company. |
| issuance_or_purchase_of_equity_shares | number | The issuance or purchase of equity shares of the company. |
| dividends_and_other_cash_distributions | number | The dividends and other cash distributions of the company. |
| net_cash_flow_from_financing | number | The net cash flow from financing of the company. |
| change_in_cash_and_equivalents | number | The change in cash and equivalents of the company. |
| effect_of_exchange_rate_changes | number | The effect of exchange rate changes of the company. |
| ending_cash_balance | number | The ending cash balance of the company. |
| free_cash_flow | number | The free cash flow of the company. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Cash flow statements response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Normalized schema, annual/quarterly/ttm; ticker required unless cik is provided.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials`

- **Operation ID:** `getAllFinancialStatements`
- **Summary:** Get all financial statements
- **Description:** Get all financial statements for a ticker.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financials/all-financial-statements.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. Required if cik is not provided. |
| period | query | yes | string enum=annual\|quarterly\|ttm | The time period of the financial statements. (enum `annual`, `quarterly`, `ttm`) |
| limit | query | no | integer/int32 | The maximum number of financial statements to return. |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `FinancialsResponse`

```json
{
  "financials": {
    "income_statements": [
      "..."
    ],
    "balance_sheets": [
      "..."
    ],
    "cash_flow_statements": [
      "..."
    ]
  }
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| financials | object | No field description in OpenAPI. |

Nested item field semantics (`item/object under `financials`` from `financials`):

| Field | Type | Meaning |
| --- | --- | --- |
| income_statements | array<object> | No field description in OpenAPI. |
| balance_sheets | array<object> | No field description in OpenAPI. |
| cash_flow_statements | array<object> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Financial statements response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Bundles income statement, balance sheet, and cash flow statement arrays in one response.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

### `GET /financials/search/screener/filters`

- **Operation ID:** `getScreenerFilters`
- **Summary:** Available screener filter fields
- **Description:** Returns a list of available filter fields, valid values, and operators for the stock screener endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `object`

```json
{
  "metrics": {},
  "operators": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| metrics | object | No field description in OpenAPI. |
| operators | array<string> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Metrics response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results, Annual Audited Accounts, Annual Reports, and as-reported PDF/XHTML extraction.

## Market Data

Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds.

### `GET /prices`

- **Operation ID:** `getPrices`
- **Summary:** Get historical stock price data
- **Description:** Get end-of-day (EOD) historical price data for stocks.
- **Docs/source page:** https://docs.financialdatasets.ai/api/prices/historical.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The stock ticker symbol (e.g. AAPL, MSFT). |
| interval | query | yes | string enum=day\|week\|month\|year | The time interval for the price data. (enum `day`, `week`, `month`, `year`) |
| start_date | query | yes | string/date | The start date for the price data (format: YYYY-MM-DD). |
| end_date | query | yes | string/date | The end date for the price data (format: YYYY-MM-DD). |

**Response schema and example shape**

- 200 schema: `PricesResponse`

```json
{
  "ticker": "string",
  "prices": [
    {
      "open": "...",
      "close": "...",
      "high": "...",
      "low": "...",
      "volume": "...",
      "time": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| prices | array<object> | No field description in OpenAPI. |

Nested item field semantics (`Price` from `prices`):

| Field | Type | Meaning |
| --- | --- | --- |
| open | number | The open price of the ticker in the given time period. |
| close | number | The close price of the ticker in the given time period. |
| high | number | The high price of the ticker in the given time period. |
| low | number | The low price of the ticker in the given time period. |
| volume | integer | The trading volume of the ticker in the given time period. |
| time | string | The human-readable time format of the price in UTC. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Price data response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Requires ticker, interval, start_date, end_date; no pagination. Open/high/low/close are per requested interval and time is UTC in the schema.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds.

### `GET /prices/snapshot`

- **Operation ID:** `getPriceSnapshot`
- **Summary:** Price Snapshot (Real-Time)
- **Description:** Get the real-time price snapshot for a stock, including the current price, day change, and day change percent.
- **Docs/source page:** https://docs.financialdatasets.ai/api/prices/snapshot.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The stock ticker symbol (e.g. AAPL, MSFT). |

**Response schema and example shape**

- 200 schema: `PriceSnapshotResponse`

```json
{
  "snapshot": {
    "price": 0,
    "ticker": "string",
    "day_change": 0,
    "day_change_percent": 0,
    "time": "string",
    "time_milliseconds": 0
  }
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| snapshot | object | No field description in OpenAPI. |

Nested item field semantics (`PriceSnapshot` from `snapshot`):

| Field | Type | Meaning |
| --- | --- | --- |
| price | number | The current price of the stock. |
| ticker | string | The ticker symbol. |
| day_change | number | The price change since the previous trading day's close. |
| day_change_percent | number | The percentage price change since the previous trading day's close. |
| time | string | The timestamp of the price snapshot in human-readable format in UTC. |
| time_milliseconds | number | The timestamp of the price snapshot in milliseconds since epoch. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Price snapshot response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds.

### `GET /prices/snapshot/tickers`

- **Operation ID:** `getPriceSnapshotTickers`
- **Summary:** Available tickers for price snapshots
- **Description:** Returns a list of available tickers for the price snapshot endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `TickersResponse`

```json
{
  "resource": "string",
  "tickers": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| tickers | array<string> | List of available ticker symbols. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Tickers response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds.

### `GET /prices/snapshot/market`

- **Operation ID:** `getPriceSnapshotMarket`
- **Summary:** Market Snapshot (Real-Time)
- **Description:** Get the real-time price snapshot for the entire market. Requires an active subscription.
- **Docs/source page:** https://docs.financialdatasets.ai/api/prices/market-snapshot.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Response schema and example shape**

- 200 schema: `PriceSnapshotMarketResponse`

```json
{
  "snapshots": [
    {
      "price": "...",
      "ticker": "...",
      "day_change": "...",
      "day_change_percent": "...",
      "time": "...",
      "time_milliseconds": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| snapshots | array<object> | No field description in OpenAPI. |

Nested item field semantics (`PriceSnapshot` from `snapshots`):

| Field | Type | Meaning |
| --- | --- | --- |
| price | number | The current price of the stock. |
| ticker | string | The ticker symbol. |
| day_change | number | The price change since the previous trading day's close. |
| day_change_percent | number | The percentage price change since the previous trading day's close. |
| time | string | The timestamp of the price snapshot in human-readable format in UTC. |
| time_milliseconds | number | The timestamp of the price snapshot in milliseconds since epoch. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Market snapshot response | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Subscription-required market-wide real-time snapshot.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds.

### `GET /prices/tickers`

- **Operation ID:** `getPricesTickers`
- **Summary:** Available tickers for price history
- **Description:** Returns a list of available tickers for the prices endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `TickersResponse`

```json
{
  "resource": "string",
  "tickers": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| tickers | array<string> | List of available ticker symbols. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Tickers response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa delayed website data, historical packages, licensed real-time Information Services/BTS2 FIX feeds.

## Company Information

Bursa listing directory/company profile, stock code, short name, ISIN, market, sector, Shariah flag.

### `GET /company/facts`

- **Operation ID:** `getCompanyFacts`
- **Summary:** Get company facts
- **Description:** Get company facts for a ticker.
- **Docs/source page:** https://docs.financialdatasets.ai/api/company/facts/ticker.md and https://docs.financialdatasets.ai/api/company/facts/cik.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol. |
| cik | query | no | string | The CIK of the company. |

**Response schema and example shape**

- 200 schema: `CompanyFactsResponse`

```json
{
  "company_facts": {
    "ticker": "string",
    "name": "string",
    "cik": "string",
    "industry": "string",
    "sector": "string",
    "category": "string",
    "exchange": "string",
    "is_active": false,
    "...": "truncated"
  }
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| company_facts | object | No field description in OpenAPI. |

Nested item field semantics (`CompanyFacts` from `company_facts`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| name | string | The name of the company. |
| cik | string | The Central Index Key (CIK) of the company. |
| industry | string | The industry of the company. |
| sector | string | The sector of the company. |
| category | string | The category of the company. |
| exchange | string | The exchange of the company. |
| is_active | boolean | Whether the company is currently active. |
| location | string | The location of the company. |
| sec_filings_url | string/uri | The URL of the company's SEC filings. |
| sic_code | string | The Standard Industrial Classification (SIC) code of the company. |
| sic_industry | string | The industry of the company based on the SIC code. |
| sic_sector | string | The sector of the company based on the SIC code. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Company facts response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 404 | The specified resource was not found | object |

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa listing directory/company profile, stock code, short name, ISIN, market, sector, Shariah flag.

### `GET /company/facts/tickers`

- **Operation ID:** `getCompanyFactsTickers`
- **Summary:** Available tickers for company facts
- **Description:** Returns a list of available tickers for the company facts endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `TickersResponse`

```json
{
  "resource": "string",
  "tickers": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| tickers | array<string> | List of available ticker symbols. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Tickers response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa listing directory/company profile, stock code, short name, ISIN, market, sector, Shariah flag.

### `GET /company/facts/ciks`

- **Operation ID:** `getCompanyFactsCiks`
- **Summary:** Available CIKs for company facts
- **Description:** Returns a list of available CIKs for the company facts endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `CiksResponse`

```json
{
  "resource": "string",
  "ciks": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| ciks | array<string> | List of available CIK codes. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | CIKs response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa listing directory/company profile, stock code, short name, ISIN, market, sector, Shariah flag.

## Earnings

Bursa Financial Results and Annual Report announcements; estimates/surprises require a separate consensus vendor.

### `GET /earnings`

- **Operation ID:** `getEarnings`
- **Summary:** Get earnings snapshot
- **Description:** Get the most recent earnings snapshot for a ticker. Optional estimate/surprise and change fields are returned only when available.
- **Docs/source page:** https://docs.financialdatasets.ai/api/earnings/earnings.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The ticker symbol (e.g. `AAPL`). |
| limit | query | no | integer | Number of most-recent **report periods** worth of filings to return, sorted by `(report_period DESC, filing_date ASC)`. The number of entries returned may exceed `limit` when a recent period has both an 8-K and a 10-Q / 10-K. Values above 40 are clamped to 40. Non-positive or non-integer values return 400. (default `1`; min 1; max 40) |

**Response schema and example shape**

- 200 schema: `EarningsResponse`

```json
{
  "earnings": [
    {
      "ticker": "...",
      "report_period": "...",
      "fiscal_period": "...",
      "currency": "...",
      "source_type": "...",
      "filing_date": "...",
      "filing_datetime": "...",
      "filing_window": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| earnings | array<object> | Flat list of SEC filings for the ticker, sorted by (report_period DESC, filing_date ASC). When `limit=N`, up to N report_periods worth of filings are returned; entry count may exceed N when a recent period has both an 8-K and a 10-Q/10-K. |

Nested item field semantics (`EarningsRecord` from `earnings`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The requested ticker symbol. |
| report_period | string/date | Report period this filing covers. |
| fiscal_period | string | Fiscal period label (e.g. 2026-Q1 or 2025-FY). |
| currency | string | ISO currency code (e.g. USD). |
| source_type | string enum=8-K\|10-Q\|10-K\|20-F | The SEC form type backing this filing. |
| filing_date | string/date | Calendar day SEC accepted the filing, in Eastern Time (the SEC's operating timezone). Calendar-day pair to `filing_datetime`. |
| filing_datetime | string/date-time | Sub-day timestamp recording when SEC accepted the filing, in Eastern Time. Sourced from SEC's `acceptanceDateTime` field on the EDGAR submissions API. Pairs with `filing_date` (calendar-day precision of the same event). |
| filing_window | string enum=PRE_MARKET\|DURING_MARKET\|AFTER_HOURS\|WEEKEND | Market session at the moment SEC accepted the filing, evaluated in Eastern Time. PRE_MARKET = weekday 04:00–09:30 ET. DURING_MARKET = weekday 09:30–16:00 ET. AFTER_HOURS = weekday outside regular trading hours. WEEKEND = Saturday or Sunday. Null when filing_datetime is unavailable. |
| signals | array<object> | Curated, market-moving insights distilled from this filing. Each item has a pre-rendered `headline`, structured `details`, and an `importance` ranking. Returned only when at least one signal fires for this entry; field is omitted otherwise. Capped at 8 per entry, ordered HIGH → MEDIUM → LOW. |
| filing_url | string/uri | URL to the SEC filing. |
| accession_number | string | SEC accession number identifying the filing. |
| quarterly | object | Quarterly figures from this filing. Returned only when available. |
| annual | object | Annual figures from this filing. Returned only when available. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Earnings response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Limit is by report-period, not row count; entries can exceed limit when a period has both preliminary 8-K and later 10-Q/10-K. Values above 40 are clamped; non-positive/non-integer returns 400. Times are SEC Eastern Time.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results and Annual Report announcements; estimates/surprises require a separate consensus vendor.

### `GET /earnings/tickers`

- **Operation ID:** `getEarningsTickers`
- **Summary:** Available tickers for earnings
- **Description:** Returns a list of available tickers for the earnings endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `TickersResponse`

```json
{
  "resource": "string",
  "tickers": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| tickers | array<string> | List of available ticker symbols. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Tickers response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Financial Results and Annual Report announcements; estimates/surprises require a separate consensus vendor.

## News

Bursa announcements/media plus licensed Malaysian publisher/RSS feeds; announcements are official-disclosure data, not editorial news.

### `GET /news`

- **Operation ID:** `getNews`
- **Summary:** Get news articles
- **Description:** Get recent news articles for a specific company or the broad market. Pass a ticker for company-specific news, or omit the ticker for general market news. Articles are sourced from RSS feeds of publishers like The Motley Fool, Investing.com, Reuters, and more.
- **Docs/source page:** https://docs.financialdatasets.ai/api/news/company.md and https://docs.financialdatasets.ai/api/news/market.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol of the company. Omit for broad market news. |
| limit | query | no | integer | The maximum number of news articles to return (default: 5, max: 10). (default `5`; max 10) |

**Response schema and example shape**

- 200 schema: `NewsResponse`

```json
{
  "news": [
    {
      "ticker": "...",
      "title": "...",
      "source": "...",
      "date": "...",
      "url": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| news | array<object> | No field description in OpenAPI. |

Nested item field semantics (`News` from `news`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| title | string | The title of the news article. |
| source | string | The source of the news article. |
| date | string/date | The date the news article was published. |
| url | string/uri | The URL of the news article. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | News articles response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Omitting ticker returns broad market news. limit defaults to 5 and is documented with max 10.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa announcements/media plus licensed Malaysian publisher/RSS feeds; announcements are official-disclosure data, not editorial news.

## SEC Filings

Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id.

### `GET /filings`

- **Operation ID:** `getFilings`
- **Summary:** Get SEC filings
- **Description:** Get SEC filings for a company.
- **Docs/source page:** https://docs.financialdatasets.ai/api/filings/ticker.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| cik | query | no | string | The Central Index Key (CIK) of the company. |
| ticker | query | no | string | The ticker symbol. |
| filing_type | query | no | array<string enum=10-K\|10-Q\|8-K\|20-F\|6-K> | Filter by one or more filing types. Repeat the query parameter to pass multiple values (e.g. filing_type=10-Q&filing_type=10-K). |
| limit | query | no | integer | The maximum number of filings to return (default: 10). (default `10`; min 1) |

**Response schema and example shape**

- 200 schema: `FilingsResponse`

```json
{
  "filings": [
    {
      "cik": "...",
      "accession_number": "...",
      "filing_type": "...",
      "report_date": "...",
      "filing_date": "...",
      "ticker": "...",
      "url": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| filings | array<object> | No field description in OpenAPI. |

Nested item field semantics (`Filing` from `filings`):

| Field | Type | Meaning |
| --- | --- | --- |
| cik | integer | The Central Index Key (CIK) of the company. |
| accession_number | string | The accession number of the filing. |
| filing_type | string | The type of the SEC filing (e.g., 10-Q, 8-K). |
| report_date | string/date | The date of the report. |
| filing_date | string/date | The date the filing was submitted to the SEC. |
| ticker | string | The ticker symbol. |
| url | string/uri | The URL of the SEC filing. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | SEC filings response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** filing_type can be repeated. Documented enum includes 10-K, 10-Q, 8-K, 20-F, 6-K.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id.

### `GET /filings/items`

- **Operation ID:** `getFilingItems`
- **Summary:** Get SEC filing items
- **Description:** Get the raw text Items from an SEC filing.
- **Docs/source page:** https://docs.financialdatasets.ai/api/filings/items.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The ticker symbol. |
| filing_type | query | yes | string enum=10-K\|10-Q\|8-K | The type of filing. (enum `10-K`, `10-Q`, `8-K`) |
| year | query | yes | integer | The year of the filing. |
| quarter | query | no | integer | The quarter of the filing if 10-Q. |
| item | query | no | string enum=Item-1\|Item-1A\|Item-1B\|Item-2\|Item-3\|Item-4\|Item-5\|Item-6\|Item-7\|Item-7A\|Item-8\|Item-9\|Item-9A\|Item-9B\|Item-10\|Item-11\|Item-12\|Item-13\|Item-14\|Item-15\|Item-16\|Item-1.01\|Item-1.02\|Item-1.03\|Item-1.04\|Item-2.01\|Item-2.02\|Item-2.03\|Item-2.04\|Item-2.05\|Item-2.06\|Item-3.01\|Item-3.02\|Item-3.03\|Item-4.01\|Item-4.02\|Item-5.01\|Item-5.02\|Item-5.03\|Item-5.04\|Item-5.05\|Item-5.06\|Item-5.07\|Item-5.08\|Item-6.01\|Item-6.02\|Item-6.03\|Item-6.04\|Item-6.05\|Item-7.01\|Item-8.01\|Item-9.01 | The item to get. (enum `Item-1`, `Item-1A`, `Item-1B`, `Item-2`, `Item-3`, `Item-4`, `Item-5`, `Item-6`, `Item-7`, `Item-7A`, `Item-8`, `Item-9`, `Item-9A`, `Item-9B`, `Item-10`, `Item-11`, `Item-12`, `Item-13`, `Item-14`, `Item-15`, `Item-16`, `Item-1.01`, `Item-1.02`, `Item-1.03`, `Item-1.04`, `Item-2.01`, `Item-2.02`, `Item-2.03`, `Item-2.04`, `Item-2.05`, `Item-2.06`, `Item-3.01`, `Item-3.02`, `Item-3.03`, `Item-4.01`, `Item-4.02`, `Item-5.01`, `Item-5.02`, `Item-5.03`, `Item-5.04`, `Item-5.05`, `Item-5.06`, `Item-5.07`, `Item-5.08`, `Item-6.01`, `Item-6.02`, `Item-6.03`, `Item-6.04`, `Item-6.05`, `Item-7.01`, `Item-8.01`, `Item-9.01`) |
| accession_number | query | no | string | The accession number of the filing if 8-K. |
| include_exhibits | query | no | boolean | Whether to include the raw text from linked exhibits. Only applicable for 8-K filings. When true, exhibit objects will include the 'text' field containing the full exhibit content. (default `False`) |

**Response schema and example shape**

- 200 schema: `FilingItemsResponse`

```json
{
  "resource": "string",
  "ticker": "string",
  "cik": "string",
  "filing_type": "string",
  "accession_number": "string",
  "year": 0,
  "quarter": 0,
  "items": [
    {
      "number": "...",
      "name": "...",
      "text": "...",
      "exhibits": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| ticker | string | The ticker symbol of the company. |
| cik | string | The Central Index Key (CIK) of the company. |
| filing_type | string | The type of filing. |
| accession_number | string | The accession number of the filing. |
| year | integer | The year of the filing. |
| quarter | integer | The quarter of the filing. |
| items | array<object> | No field description in OpenAPI. |

Nested item field semantics (`FilingItem` from `items`):

| Field | Type | Meaning |
| --- | --- | --- |
| number | string | The item number. |
| name | string | The item name. |
| text | string | The item text. |
| exhibits | array<object> | An array of exhibits linked to this item. Only present for 8-K filings when exhibits exist. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | SEC filing items response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |
| 503 | Filing parse in progress. The request reached the deadline before the parse completed; the parse continues in the background and will be cached. Retry after the duration in the Retry-After header. | object |

**Documented behavior and edge cases:** Only supports 10-K, 10-Q, and 8-K; quarter applies to 10-Q; accession_number applies to 8-K; include_exhibits adds full exhibit text; 503 is documented for extraction failure.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id.

### `GET /filings/tickers`

- **Operation ID:** `getFilingsTickers`
- **Summary:** Available tickers for SEC filings
- **Description:** Returns a list of available tickers for the filings endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `TickersResponse`

```json
{
  "resource": "string",
  "tickers": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| tickers | array<string> | List of available ticker symbols. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Tickers response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id.

### `GET /filings/ciks`

- **Operation ID:** `getFilingsCiks`
- **Summary:** Available CIKs for SEC filings
- **Description:** Returns a list of available CIKs for the filings endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `CiksResponse`

```json
{
  "resource": "string",
  "ciks": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| ciks | array<string> | List of available CIK codes. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | CIKs response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id.

### `GET /filings/types`

- **Operation ID:** `getFilingsTypes`
- **Summary:** Available SEC filing types
- **Description:** Returns a sorted list of valid SEC filing types (e.g., 10-K, 10-Q, 8-K). This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `object`

```json
{
  "filing_types": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| filing_types | array<string> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Filing types response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id.

### `GET /filings/items/types`

- **Operation ID:** `getFilingsItemsTypes`
- **Summary:** Available filing item types
- **Description:** Returns a list of extractable item sections for 10-K, 10-Q, and 8-K filings. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| filing_type | query | no | string | Optional filter by filing type (e.g., 10-K, 10-Q). |

**Response schema and example shape**

- 200 schema: `object`

```json
{}
```

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Filing item types response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bursa Company Announcements and attachments; accession_number maps to Bursa announcement/reference id.

## Insider Trades

Director interest and substantial shareholder announcements under Companies Act 2016 sections 137-139/219.

### `GET /insider-trades`

- **Operation ID:** `getInsiderTrades`
- **Summary:** Get insider trades
- **Description:** Get insider trades like buys and sells for a ticker by a company insider.
- **Docs/source page:** https://docs.financialdatasets.ai/api/insider-trades/insider-trades.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The ticker symbol of the company. |
| limit | query | no | integer | The maximum number of transactions to return (default: 10). (default `10`) |
| name | query | no | string | Filter by insider name (e.g., 'Jen Hsun Huang'). Use the /insider-trades/names endpoint to get available names for a ticker. |
| transaction_type | query | no | string | Filter by transaction type (e.g., 'Open market sale', 'Gift'). Use the /insider-trades/transaction-types endpoint to get available types. |
| filing_date | query | no | string/date | Filter by exact filing date in YYYY-MM-DD format. |
| filing_date_gte | query | no | string/date | Filter by filing date greater than or equal to this date (YYYY-MM-DD). |
| filing_date_lte | query | no | string/date | Filter by filing date less than or equal to this date (YYYY-MM-DD). |
| filing_date_gt | query | no | string/date | Filter by filing date greater than this date (YYYY-MM-DD). |
| filing_date_lt | query | no | string/date | Filter by filing date less than this date (YYYY-MM-DD). |

**Response schema and example shape**

- 200 schema: `InsiderTradesResponse`

```json
{
  "insider_trades": [
    {
      "ticker": "...",
      "issuer": "...",
      "name": "...",
      "title": "...",
      "is_board_director": "...",
      "transaction_date": "...",
      "transaction_shares": "...",
      "transaction_price_per_share": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| insider_trades | array<object> | No field description in OpenAPI. |

Nested item field semantics (`InsiderTrade` from `insider_trades`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| issuer | string | The name of the issuing company. |
| name | string | The name of the insider. |
| title | string | The title of the insider. |
| is_board_director | boolean | Whether the insider is a board director. |
| transaction_date | string/date | The date of the transaction. |
| transaction_shares | number | The number of shares involved in the transaction. |
| transaction_price_per_share | number | The price per share in the transaction. |
| transaction_value | number | The total value of the transaction. |
| shares_owned_before_transaction | number | The number of shares owned before the transaction. |
| shares_owned_after_transaction | number | The number of shares owned after the transaction. |
| security_title | string | The title of the security involved in the transaction. |
| transaction_type | string | A human-readable description of the transaction type. |
| filing_date | string/date | The date the transaction was filed. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Insider trades response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Supports name, transaction_type, exact filing_date, and filing_date comparison operators.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Director interest and substantial shareholder announcements under Companies Act 2016 sections 137-139/219.

## Institutional Ownership

No 13F equivalent; approximate from substantial shareholder notices, annual reports, and licensed shareholder data.

### `GET /institutional-ownership`

- **Operation ID:** `getInstitutionalOwnership`
- **Summary:** Get the equity holdings of an investment manager
- **Description:** Get institutional ownership by investor or ticker. Requires either investor or ticker parameter, but not both.
- **Docs/source page:** https://docs.financialdatasets.ai/api/institutional-ownership/investor.md and https://docs.financialdatasets.ai/api/institutional-ownership/ticker.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| investor | query | no | string | The name of the investment manager |
| ticker | query | no | string | The ticker symbol, if queried by investor. |
| limit | query | no | integer | The maximum number of holdings to return (default: 10). (default `10`) |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `InstitutionalOwnershipResponse`

```json
{
  "institutional-ownership": [
    {
      "ticker": "...",
      "investor": "...",
      "report_period": "...",
      "security_type": "...",
      "price": "...",
      "shares": "...",
      "market_value": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| institutional-ownership | array<object> | No field description in OpenAPI. |

Nested item field semantics (`InstitutionalOwnership` from `institutional-ownership`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol, if queried by investor. |
| investor | string | The investor name, if queried by ticker. |
| report_period | string/date | The reporting period of the institutional ownership. |
| security_type | string enum=common_stock\|put_option\|call_option\|warrant\|preferred_stock\|debt\|fund\|other | The type of security held by the investment manager. |
| price | number | The estimated purchase price of the equity position. |
| shares | number | The number of shares held by the investment manager. |
| market_value | number | The market value of the equity position. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Institutional ownership response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Requires investor or ticker, but not both. OpenAPI description says ticker is "if queried by investor" but endpoint-level docs cover both modes.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: No 13F equivalent; approximate from substantial shareholder notices, annual reports, and licensed shareholder data.

## Financial Metrics

Derived from Bursa prices, normalized Bursa/MFRS statements, shares outstanding, corporate actions, and FX/MYR metadata.

### `GET /financial-metrics`

- **Operation ID:** `getFinancialMetrics`
- **Summary:** Get financial metrics
- **Description:** Get financial metrics for a ticker, including valuation, profitability, efficiency, liquidity, leverage, growth, and per share metrics.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financial-metrics/historical.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol of the company. Required if cik is not provided. |
| cik | query | no | string | The Central Index Key (CIK) of the company. Can be used instead of ticker. |
| period | query | yes | string enum=annual\|quarterly\|ttm | The time period for the financial data. (enum `annual`, `quarterly`, `ttm`) |
| limit | query | no | integer | The maximum number of results to return. |
| report_period | query | no | string/date | Filter by exact report period date in YYYY-MM-DD format. |
| report_period_gte | query | no | string/date | Filter by report period greater than or equal to date in YYYY-MM-DD format. |
| report_period_lte | query | no | string/date | Filter by report period less than or equal to date in YYYY-MM-DD format. |
| report_period_gt | query | no | string/date | Filter by report period greater than date in YYYY-MM-DD format. |
| report_period_lt | query | no | string/date | Filter by report period less than date in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `FinancialMetricsResponse`

```json
{
  "ticker": "string",
  "report_period": "2026-03-31",
  "fiscal_period": "string",
  "period": "quarterly",
  "currency": "string",
  "accession_number": "string",
  "filing_url": "https://example.com/resource",
  "enterprise_value": 0,
  "...": "truncated"
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| report_period | string/date | The reporting period of the financial metrics. |
| fiscal_period | string | The fiscal period of the financial metrics. |
| period | string enum=quarterly\|ttm\|annual | The time period of the financial metrics. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| enterprise_value | number | The total value of the company (market cap + debt - cash). |
| price_to_earnings_ratio | number | Price to earnings ratio. |
| price_to_book_ratio | number | Price to book ratio. |
| price_to_sales_ratio | number | Price to sales ratio. |
| enterprise_value_to_ebitda_ratio | number | Enterprise value to EBITDA ratio. |
| enterprise_value_to_revenue_ratio | number | Enterprise value to revenue ratio. |
| free_cash_flow_yield | number | Free cash flow yield. |
| peg_ratio | number | Price to earnings growth ratio. |
| gross_margin | number | Gross profit as a percentage of revenue. |
| operating_margin | number | Operating income as a percentage of revenue. |
| net_margin | number | Net income as a percentage of revenue. |
| return_on_equity | number | Net income as a percentage of shareholders' equity. |
| return_on_assets | number | Net income as a percentage of total assets. |
| return_on_invested_capital | number | Net operating profit after taxes as a percentage of invested capital. |
| asset_turnover | number | Revenue divided by average total assets. |
| inventory_turnover | number | Cost of goods sold divided by average inventory. |
| receivables_turnover | number | Revenue divided by average accounts receivable. |
| days_sales_outstanding | number | Average accounts receivable divided by revenue over the period. |
| operating_cycle | number | Inventory turnover + receivables turnover. |
| working_capital_turnover | number | Revenue divided by average working capital. |
| current_ratio | number | Current assets divided by current liabilities. |
| quick_ratio | number | Quick assets divided by current liabilities. |
| cash_ratio | number | Cash and cash equivalents divided by current liabilities. |
| operating_cash_flow_ratio | number | Operating cash flow divided by current liabilities. |
| debt_to_equity | number | Total debt divided by shareholders' equity. |
| debt_to_assets | number | Total debt divided by total assets. |
| interest_coverage | number | EBIT divided by interest expense. |
| revenue_growth | number | Year-over-year growth in revenue. |
| earnings_growth | number | Year-over-year growth in earnings. |
| book_value_growth | number | Year-over-year growth in book value. |
| earnings_per_share_growth | number | Growth in earnings per share over the period. |
| free_cash_flow_growth | number | Growth in free cash flow over the period. |
| operating_income_growth | number | Growth in operating income over the period. |
| ebitda_growth | number | Growth in EBITDA over the period. |
| payout_ratio | number | Dividends paid as a percentage of net income. |
| earnings_per_share | number | Net income divided by weighted average shares outstanding. |
| book_value_per_share | number | Shareholders' equity divided by shares outstanding. |
| free_cash_flow_per_share | number | Free cash flow divided by shares outstanding. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | The historical financial metrics and ratios for a ticker | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Derived from Bursa prices, normalized Bursa/MFRS statements, shares outstanding, corporate actions, and FX/MYR metadata.

### `GET /financial-metrics/snapshot`

- **Operation ID:** `getFinancialMetricsSnapshot`
- **Summary:** Financial Metrics Snapshot (Real-Time)
- **Description:** Get the real-time financial metrics snapshot for a stock, including valuation ratios, profitability, efficiency, liquidity, leverage, growth, and per share metrics.
- **Docs/source page:** https://docs.financialdatasets.ai/api/financial-metrics/snapshot.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | no | string | The ticker symbol of the company. |
| cik | query | no | string | The Central Index Key (CIK) of the company. Can be used instead of ticker. |

**Response schema and example shape**

- 200 schema: `FinancialMetricSnapshotResponse`

```json
{
  "snapshot": {
    "ticker": "string",
    "market_cap": 0,
    "enterprise_value": 0,
    "price_to_earnings_ratio": 0,
    "price_to_book_ratio": 0,
    "price_to_sales_ratio": 0,
    "enterprise_value_to_ebitda_ratio": 0,
    "enterprise_value_to_revenue_ratio": 0,
    "...": "truncated"
  }
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| snapshot | object | No field description in OpenAPI. |

Nested item field semantics (`FinancialMetricSnapshot` from `snapshot`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| market_cap | number | The total market capitalization (stock price × shares outstanding). |
| enterprise_value | number | The total value of the company (market cap + debt - cash). |
| price_to_earnings_ratio | number | Price to earnings ratio. |
| price_to_book_ratio | number | Price to book ratio. |
| price_to_sales_ratio | number | Price to sales ratio. |
| enterprise_value_to_ebitda_ratio | number | Enterprise value to EBITDA ratio. |
| enterprise_value_to_revenue_ratio | number | Enterprise value to revenue ratio. |
| free_cash_flow_yield | number | Free cash flow yield. |
| peg_ratio | number | Price to earnings growth ratio. |
| gross_margin | number | Gross profit as a percentage of revenue. |
| operating_margin | number | Operating income as a percentage of revenue. |
| net_margin | number | Net income as a percentage of revenue. |
| return_on_equity | number | Net income as a percentage of shareholders' equity. |
| return_on_assets | number | Net income as a percentage of total assets. |
| return_on_invested_capital | number | Net operating profit after taxes as a percentage of invested capital. |
| asset_turnover | number | Revenue divided by average total assets. |
| inventory_turnover | number | Cost of goods sold divided by average inventory. |
| receivables_turnover | number | Revenue divided by average accounts receivable. |
| days_sales_outstanding | number | Average accounts receivable divided by revenue over the period. |
| operating_cycle | number | Inventory turnover + receivables turnover. |
| working_capital_turnover | number | Revenue divided by average working capital. |
| current_ratio | number | Current assets divided by current liabilities. |
| quick_ratio | number | Quick assets divided by current liabilities. |
| cash_ratio | number | Cash and cash equivalents divided by current liabilities. |
| operating_cash_flow_ratio | number | Operating cash flow divided by current liabilities. |
| debt_to_equity | number | Total debt divided by shareholders' equity. |
| debt_to_assets | number | Total debt divided by total assets. |
| interest_coverage | number | EBIT divided by interest expense. |
| revenue_growth | number | Year-over-year growth in revenue. |
| earnings_growth | number | Year-over-year growth in earnings. |
| book_value_growth | number | Year-over-year growth in book value. |
| earnings_per_share_growth | number | Growth in earnings per share over the period. |
| free_cash_flow_growth | number | Growth in free cash flow over the period. |
| operating_income_growth | number | Growth in operating income over the period. |
| ebitda_growth | number | Growth in EBITDA over the period. |
| payout_ratio | number | Dividends paid as a percentage of net income. |
| earnings_per_share | number | Net income divided by weighted average shares outstanding. |
| book_value_per_share | number | Shareholders' equity divided by shares outstanding. |
| free_cash_flow_per_share | number | Free cash flow divided by shares outstanding. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Financial metrics snapshot response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Derived from Bursa prices, normalized Bursa/MFRS statements, shares outstanding, corporate actions, and FX/MYR metadata.

### `GET /financial-metrics/snapshot/tickers`

- **Operation ID:** `getFinancialMetricsSnapshotTickers`
- **Summary:** Available tickers for financial metrics snapshots
- **Description:** Returns a list of available tickers for the financial metrics snapshot endpoint. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `TickersResponse`

```json
{
  "resource": "string",
  "tickers": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| tickers | array<string> | List of available ticker symbols. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Tickers response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Derived from Bursa prices, normalized Bursa/MFRS statements, shares outstanding, corporate actions, and FX/MYR metadata.

## Macroeconomics

Bank Negara Malaysia/data.gov.my OPR/MYOR/KLIBOR/rates and FX sources; use BNM as primary bank code.

### `GET /macro/interest-rates`

- **Operation ID:** `getInterestRates`
- **Summary:** Interest Rates (Historical)
- **Description:** Historical interest rates for all major central banks in the world.
- **Docs/source page:** https://docs.financialdatasets.ai/api/macro/interest-rates/historical.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| bank | query | yes | string | The bank whose interest rates to return. Use the /macro/interest-rates/banks endpoint to get a list of available banks. |
| start_date | query | no | string | The start date of the interest rates to return in YYYY-MM-DD format. |
| end_date | query | no | string | The end date of the interest rates to return in YYYY-MM-DD format. |

**Response schema and example shape**

- 200 schema: `InterestRatesResponse`

```json
{
  "interest_rates": [
    {
      "bank": "...",
      "name": "...",
      "rate": "...",
      "date": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| interest_rates | array<object> | No field description in OpenAPI. |

Nested item field semantics (`InterestRate` from `interest_rates`):

| Field | Type | Meaning |
| --- | --- | --- |
| bank | string | The symbol of the central bank. |
| name | string | The name of the central bank. |
| rate | number | The interest rate of the central bank. |
| date | string | The date of the interest rate in YYYY-MM-DD format. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Interest rates response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Requires bank code. Use /macro/interest-rates/banks for codes. Historical range via start_date/end_date.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bank Negara Malaysia/data.gov.my OPR/MYOR/KLIBOR/rates and FX sources; use BNM as primary bank code.

### `GET /macro/interest-rates/snapshot`

- **Operation ID:** `getInterestRatesSnapshot`
- **Summary:** Interest Rates (Real-Time)
- **Description:** Get the current interest rates from all major central banks in the world.
- **Docs/source page:** https://docs.financialdatasets.ai/api/macro/interest-rates/snapshot.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| bank | query | yes | string | The central bank code (e.g., FED, ECB, BOJ). Use the /macro/interest-rates/banks endpoint to get a list of available banks. |

**Response schema and example shape**

- 200 schema: `InterestRatesResponse`

```json
{
  "interest_rates": [
    {
      "bank": "...",
      "name": "...",
      "rate": "...",
      "date": "..."
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| interest_rates | array<object> | No field description in OpenAPI. |

Nested item field semantics (`InterestRate` from `interest_rates`):

| Field | Type | Meaning |
| --- | --- | --- |
| bank | string | The symbol of the central bank. |
| name | string | The name of the central bank. |
| rate | number | The interest rate of the central bank. |
| date | string | The date of the interest rate in YYYY-MM-DD format. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Interest rates snapshot response | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 404 | The specified resource was not found | object |

**Documented behavior and edge cases:** Requires bank code. Returns same interest_rates envelope as historical.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bank Negara Malaysia/data.gov.my OPR/MYOR/KLIBOR/rates and FX sources; use BNM as primary bank code.

### `GET /macro/interest-rates/banks`

- **Operation ID:** `getInterestRatesBanks`
- **Summary:** Available central banks
- **Description:** Returns a list of available central bank codes for the interest rates endpoints. This endpoint is free and does not require authentication.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** No auth required (`security: []`; documented free endpoint).

**Response schema and example shape**

- 200 schema: `object`

```json
{
  "resource": "string",
  "banks": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | No field description in OpenAPI. |
| banks | array<string> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Banks response | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Bank Negara Malaysia/data.gov.my OPR/MYOR/KLIBOR/rates and FX sources; use BNM as primary bank code.

## KPIs

Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics.

### `GET /kpi/metrics`

- **Operation ID:** `getKPIMetrics`
- **Summary:** Get KPI metrics
- **Description:** Get sector-specific operational KPIs for a ticker. Includes metrics like load factor, CET1 ratio, same-store sales, FFO per share, and more. Sourced from SEC 8-K earnings releases.
- **Docs/source page:** https://docs.financialdatasets.ai/api/kpi/metrics.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The ticker symbol. |
| metric_name | query | no | string | Filter to a specific metric (e.g., load_factor, cet1_ratio). |
| period | query | no | string enum=quarterly\|annual | Filter by period type: quarterly or annual. (default `quarterly`; enum `quarterly`, `annual`) |
| report_period_gte | query | no | string/date | Only return metrics on or after this date (YYYY-MM-DD). |
| report_period_lte | query | no | string/date | Only return metrics on or before this date (YYYY-MM-DD). |
| limit | query | no | integer | Number of periods to return. Returns all metrics for the N most recent periods. (default `4`; max 50) |

**Response schema and example shape**

- 200 schema: `KPIMetricsResponse`

```json
{
  "kpi_metrics": [
    {
      "ticker": "...",
      "metric_name": "...",
      "value": "...",
      "unit": "...",
      "period": "...",
      "period_type": "...",
      "segment": "...",
      "yoy_value": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| kpi_metrics | array<object> | No field description in OpenAPI. |

Nested item field semantics (`KPIMetric` from `kpi_metrics`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | Stock ticker symbol. |
| metric_name | string | Canonical metric name. |
| value | number | Extracted numeric value. |
| unit | string | Unit of measure (e.g., %, cents, dollars, miles, count). |
| period | string | Reporting period (e.g., Q4 2025, FY 2025, or a date like 2025-04-30 for non-standard fiscal years). |
| period_type | string enum=quarterly\|annual | Whether this is a quarterly or annual figure. |
| segment | string | Business segment, geography, or sub-category. Only present when the metric is reported for a sub-unit rather than the consolidated entity. |
| yoy_value | number | Year-over-year comparison value from the prior period. |
| yoy_change_pct | number | Year-over-year percentage change. |
| source_text | string | The source text span from the filing where this value was found. |
| source_url | string/uri | Direct link to the SEC filing. When available, includes a text fragment that highlights the source in the browser. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | KPI metrics response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 402 | The request requires a paid subscription | object |
| 403 | Enterprise access required | object |

**Documented behavior and edge cases:** Enterprise access required is documented as 403; returns all metrics for N most recent periods.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics.

### `GET /kpi/metrics/tickers`

- **Operation ID:** `getKPIMetricsTickers`
- **Summary:** Get available KPI tickers
- **Description:** Returns a list of tickers that have KPI metrics available.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Response schema and example shape**

- 200 schema: `object`

```json
{
  "resource": "string",
  "tickers": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | No field description in OpenAPI. |
| tickers | array<string> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Available tickers | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics.

### `GET /kpi/metrics/sectors`

- **Operation ID:** `getKPIMetricsSectors`
- **Summary:** Get available KPI sectors
- **Description:** Returns a list of sectors that have KPI metrics available.
- **Docs/source page:** OpenAPI availability/list endpoint; referenced from parent domain docs where applicable.
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Response schema and example shape**

- 200 schema: `object`

```json
{
  "resource": "string",
  "sectors": [
    "string"
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | No field description in OpenAPI. |
| sectors | array<string> | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Available sectors | object |

**Availability/list behavior:** returns supported identifiers or metadata for the parent endpoint; no pagination or auth requirement is documented unless noted otherwise.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics.

### `GET /kpi/guidance`

- **Operation ID:** `getKPIGuidance`
- **Summary:** Get forward guidance
- **Description:** Get structured forward guidance from earnings releases. Returns ranges, point estimates, and directional signals.
- **Docs/source page:** https://docs.financialdatasets.ai/api/kpi/guidance.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The ticker symbol. |
| metric_name | query | no | string | Filter to a specific metric. |
| period | query | no | string enum=quarterly\|annual | Filter by period type: quarterly or annual. (default `quarterly`; enum `quarterly`, `annual`) |
| report_period_gte | query | no | string/date | Only return guidance on or after this date (YYYY-MM-DD). |
| report_period_lte | query | no | string/date | Only return guidance on or before this date (YYYY-MM-DD). |
| limit | query | no | integer | Number of periods to return. (default `4`; max 50) |

**Response schema and example shape**

- 200 schema: `KPIGuidanceResponse`

```json
{
  "kpi_guidance": [
    {
      "ticker": "...",
      "metric_name": "...",
      "value": "...",
      "unit": "...",
      "period": "...",
      "period_type": "...",
      "segment": "...",
      "low": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| kpi_guidance | array<object> | No field description in OpenAPI. |

Nested item field semantics (`KPIGuidanceItem` from `kpi_guidance`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | No field description in OpenAPI. |
| metric_name | string | No field description in OpenAPI. |
| value | number | No field description in OpenAPI. |
| unit | string | No field description in OpenAPI. |
| period | string | No field description in OpenAPI. |
| period_type | string enum=quarterly\|annual | No field description in OpenAPI. |
| segment | string | No field description in OpenAPI. |
| low | number | Low end of guidance range. |
| high | number | High end of guidance range. |
| point_estimate | number | Single-point guidance when no range is given. |
| prior_value | number | Prior guidance value for revisions. |
| raw_text | string | Original guidance text from the filing. |
| change_direction | string | Direction: raised, lowered, maintained, initiated, withdrawn. |
| source_text | string | No field description in OpenAPI. |
| source_url | string/uri | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Guidance response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 403 | Pro subscription required |  |

**Documented behavior and edge cases:** Pro subscription required is documented as 403.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics.

### `GET /kpi/non-gaap`

- **Operation ID:** `getKPINonGAAP`
- **Summary:** Get non-GAAP metrics
- **Description:** Get non-GAAP financial metrics with GAAP equivalents and key adjustments.
- **Docs/source page:** https://docs.financialdatasets.ai/api/kpi/non-gaap.md
- **Authentication:** `X-API-KEY: <api-key>` header required unless the upstream plan has a free-tier exception for selected sample tickers.

**Query parameters**

| Name | In | Required | Type | Semantics / constraints |
| --- | --- | --- | --- | --- |
| ticker | query | yes | string | The ticker symbol. |
| metric_name | query | no | string | Filter to a specific metric. |
| period | query | no | string enum=quarterly\|annual | Filter by period type: quarterly or annual. (default `quarterly`; enum `quarterly`, `annual`) |
| report_period_gte | query | no | string/date | Only return metrics on or after this date (YYYY-MM-DD). |
| report_period_lte | query | no | string/date | Only return metrics on or before this date (YYYY-MM-DD). |
| limit | query | no | integer | Number of periods to return. (default `4`; max 50) |

**Response schema and example shape**

- 200 schema: `KPINonGAAPResponse`

```json
{
  "kpi_non_gaap": [
    {
      "ticker": "...",
      "metric_name": "...",
      "value": "...",
      "unit": "...",
      "period": "...",
      "period_type": "...",
      "gaap_equivalent": "...",
      "key_adjustments": "...",
      "...": "truncated"
    }
  ]
}
```

Top-level response fields:

| Field | Type | Meaning |
| --- | --- | --- |
| kpi_non_gaap | array<object> | No field description in OpenAPI. |

Nested item field semantics (`KPINonGAAPMetric` from `kpi_non_gaap`):

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | No field description in OpenAPI. |
| metric_name | string | No field description in OpenAPI. |
| value | number | No field description in OpenAPI. |
| unit | string | No field description in OpenAPI. |
| period | string | No field description in OpenAPI. |
| period_type | string enum=quarterly\|annual | No field description in OpenAPI. |
| gaap_equivalent | string | The GAAP line item this metric adjusts. |
| key_adjustments | string | Description of adjustments made. |
| source_text | string | No field description in OpenAPI. |
| source_url | string/uri | No field description in OpenAPI. |

**Documented responses / errors**

| HTTP | Meaning | Schema |
| --- | --- | --- |
| 200 | Non-GAAP metrics response | object |
| 400 | Bad request | object |
| 401 | Unauthorized | object |
| 403 | Pro subscription required |  |

**Documented behavior and edge cases:** Pro subscription required is documented as 403.

**Bursa mapping pointer:** See `docs/specs/rahul-39-bursa-equivalence-map.md` for the endpoint-specific Malaysia clone mapping. Short form: Extract from Bursa financial-result narratives, annual reports, investor presentations, and sector-specific operating statistics.

## Shared Schema Catalog

This catalog captures common reusable schema semantics from the OpenAPI so implementers can generate database models and compatibility tests. Some schemas are not directly attached to a current path but appear in the upstream spec and should be tracked for drift.

### `AsReportedBalanceSheetsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_balance_sheets | array<allOf(object, object)> | No field description in OpenAPI. |

### `AsReportedCashFlowStatementsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_cash_flow_statements | array<allOf(object, object)> | No field description in OpenAPI. |

### `AsReportedFinancialsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_financials | array<allOf(object, object)> | No field description in OpenAPI. |

### `AsReportedIncomeStatementsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| as_reported_income_statements | array<allOf(object, object)> | No field description in OpenAPI. |

### `AsReportedNode`

| Field | Type | Meaning |
| --- | --- | --- |
| label | string | The label as printed on the face of the filing. |
| full_label | string | The canonical taxonomy label for the underlying concept. |
| value | number | The numeric value as reported. Null for section headers and subtotals with no direct value. |
| children | array<object> | Nested line items beneath this row. |

### `BalanceSheet`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| report_period | string/date | The reporting period of the balance sheet. |
| fiscal_period | string | The fiscal period of the balance sheet. |
| period | string enum=quarterly\|ttm\|annual | The time period of the balance sheet. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| total_assets | number | The total assets of the company. |
| current_assets | number | The current assets of the company. |
| cash_and_equivalents | number | The cash and equivalents of the company. |
| inventory | number | The inventory of the company. |
| current_investments | number | The current investments of the company. |
| trade_and_non_trade_receivables | number | The trade and non-trade receivables of the company. |
| non_current_assets | number | The non-current assets of the company. |
| property_plant_and_equipment | number | The property, plant, and equipment of the company. |
| goodwill_and_intangible_assets | number | The goodwill and intangible assets of the company. |
| investments | number | The investments of the company. |
| non_current_investments | number | The non-current investments of the company. |
| outstanding_shares | number | The outstanding shares of the company. |
| tax_assets | number | The tax assets of the company. |
| total_liabilities | number | The total liabilities of the company. |
| current_liabilities | number | The current liabilities of the company. |
| current_debt | number | The current debt of the company. |
| trade_and_non_trade_payables | number | The trade and non-trade payables of the company. |
| deferred_revenue | number | The deferred revenue of the company. |
| deposit_liabilities | number | The deposit liabilities of the company. |
| non_current_liabilities | number | The non-current liabilities of the company. |
| non_current_debt | number | The non-current debt of the company. |
| tax_liabilities | number | The tax liabilities of the company. |
| shareholders_equity | number | The shareholders' equity of the company. |
| retained_earnings | number | The retained earnings of the company. |
| accumulated_other_comprehensive_income | number | The accumulated other comprehensive income of the company. |
| total_debt | number | The total debt of the company. |

### `BalanceSheetResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| balance_sheets | array<object> | No field description in OpenAPI. |

### `BalanceSheetSegmentsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

### `Beta`

| Field | Type | Meaning |
| --- | --- | --- |
| value | number | Beta coefficient of stock monthly returns vs benchmark monthly returns. |
| ticker | string | The requested ticker symbol. |
| benchmark | string | Benchmark ticker used for beta calculation. |
| requested_lookback | string | Requested lookback window. |
| interval | string | Interval used for returns alignment. |
| start_date | string/date | First aligned month-end date used. |
| end_date | string/date | Last aligned month-end date used. |
| n_observations | integer | Number of overlapping monthly observations used. |
| r_squared | number | Coefficient of determination for the OLS fit. |
| method | string | Computation method used. |
| price_type | string | Price type used to compute returns. |
| reason | string | Present when value is null due to insufficient or invalid input data. |

### `BetaResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| beta | object | No field description in OpenAPI. |

### `CashFlowStatement`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| report_period | string/date | The reporting period of the cash flow statement. |
| fiscal_period | string | The fiscal period of the cash flow statement. |
| period | string enum=quarterly\|ttm\|annual | The time period of the cash flow statement. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| net_income | number | The net income of the company. |
| depreciation_and_amortization | number | The depreciation and amortization of the company. |
| share_based_compensation | number | The share-based compensation of the company. |
| net_cash_flow_from_operations | number | The net cash flow from operations of the company. |
| capital_expenditure | number | The capital expenditure of the company. |
| business_acquisitions_and_disposals | number | The business acquisitions and disposals of the company. |
| investment_acquisitions_and_disposals | number | The investment acquisitions and disposals of the company. |
| net_cash_flow_from_investing | number | The net cash flow from investing of the company. |
| issuance_or_repayment_of_debt_securities | number | The issuance or repayment of debt securities of the company. |
| issuance_or_purchase_of_equity_shares | number | The issuance or purchase of equity shares of the company. |
| dividends_and_other_cash_distributions | number | The dividends and other cash distributions of the company. |
| net_cash_flow_from_financing | number | The net cash flow from financing of the company. |
| change_in_cash_and_equivalents | number | The change in cash and equivalents of the company. |
| effect_of_exchange_rate_changes | number | The effect of exchange rate changes of the company. |
| ending_cash_balance | number | The ending cash balance of the company. |
| free_cash_flow | number | The free cash flow of the company. |

### `CashFlowStatementResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| cash_flow_statements | array<object> | No field description in OpenAPI. |

### `CashFlowStatementSegmentsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

### `CiksResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| ciks | array<string> | List of available CIK codes. |

### `CompanyFacts`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| name | string | The name of the company. |
| cik | string | The Central Index Key (CIK) of the company. |
| industry | string | The industry of the company. |
| sector | string | The sector of the company. |
| category | string | The category of the company. |
| exchange | string | The exchange of the company. |
| is_active | boolean | Whether the company is currently active. |
| location | string | The location of the company. |
| sec_filings_url | string/uri | The URL of the company's SEC filings. |
| sic_code | string | The Standard Industrial Classification (SIC) code of the company. |
| sic_industry | string | The industry of the company based on the SIC code. |
| sic_sector | string | The sector of the company based on the SIC code. |

### `CompanyFactsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| company_facts | object | No field description in OpenAPI. |

### `EarningsRecord`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The requested ticker symbol. |
| report_period | string/date | Report period this filing covers. |
| fiscal_period | string | Fiscal period label (e.g. 2026-Q1 or 2025-FY). |
| currency | string | ISO currency code (e.g. USD). |
| source_type | string enum=8-K\|10-Q\|10-K\|20-F | The SEC form type backing this filing. |
| filing_date | string/date | Calendar day SEC accepted the filing, in Eastern Time (the SEC's operating timezone). Calendar-day pair to `filing_datetime`. |
| filing_datetime | string/date-time | Sub-day timestamp recording when SEC accepted the filing, in Eastern Time. Sourced from SEC's `acceptanceDateTime` field on the EDGAR submissions API. Pairs with `filing_date` (calendar-day precision of the same event). |
| filing_window | string enum=PRE_MARKET\|DURING_MARKET\|AFTER_HOURS\|WEEKEND | Market session at the moment SEC accepted the filing, evaluated in Eastern Time. PRE_MARKET = weekday 04:00–09:30 ET. DURING_MARKET = weekday 09:30–16:00 ET. AFTER_HOURS = weekday outside regular trading hours. WEEKEND = Saturday or Sunday. Null when filing_datetime is unavailable. |
| signals | array<object> | Curated, market-moving insights distilled from this filing. Each item has a pre-rendered `headline`, structured `details`, and an `importance` ranking. Returned only when at least one signal fires for this entry; field is omitted otherwise. Capped at 8 per entry, ordered HIGH → MEDIUM → LOW. |
| filing_url | string/uri | URL to the SEC filing. |
| accession_number | string | SEC accession number identifying the filing. |
| quarterly | object | Quarterly figures from this filing. Returned only when available. |
| annual | object | Annual figures from this filing. Returned only when available. |

### `EarningsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| earnings | array<object> | Flat list of SEC filings for the ticker, sorted by (report_period DESC, filing_date ASC). When `limit=N`, up to N report_periods worth of filings are returned; entry count may exceed N when a recent period has both an 8-K and a 10-Q/10-K. |

### `EarningsTimeDimension`

| Field | Type | Meaning |
| --- | --- | --- |
| revenue | number | No field description in OpenAPI. |
| revenue_chg | number | Percentage change in revenue versus the prior period as a decimal (e.g. 0.10 = +10%). Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| revenue_yoy_chg | number | Year-over-year change in revenue versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| estimated_revenue | number | Consensus analyst estimate for revenue. Returned only when available. |
| revenue_surprise | string enum=BEAT\|MISS\|MEET | Revenue surprise classification versus estimate. Returned only when available. |
| revenue_surprise_pct | number | Signed magnitude of the revenue surprise as a decimal (e.g. 0.0196 = revenue came in 1.96% above estimate). Returned only when an estimate is available. |
| earnings_per_share | number | No field description in OpenAPI. |
| earnings_per_share_chg | number | Percentage change in EPS versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| earnings_per_share_yoy_chg | number | Year-over-year change in EPS versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| estimated_earnings_per_share | number | Consensus analyst estimate for diluted earnings per share. Returned only when available. |
| eps_surprise | string enum=BEAT\|MISS\|MEET | EPS surprise classification versus estimate. Returned only when available. |
| eps_surprise_pct | number | Signed magnitude of the EPS surprise as a decimal (e.g. 0.5652 = EPS came in 56.52% above estimate). Returned only when an estimate is available. |
| gross_profit | number | No field description in OpenAPI. |
| gross_profit_chg | number | Percentage change in gross profit versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| gross_profit_yoy_chg | number | Year-over-year change in gross profit versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| gross_margin | number | Gross margin as a decimal (gross_profit / revenue). Returned only when calculable. |
| gross_margin_chg_bps | number | Absolute change in gross margin versus the prior period in basis points (e.g. 200.0 = margin expanded 200 bps). Returned only when calculable. |
| gross_margin_chg_pct | number | Percentage change in gross margin versus the prior period as a decimal. Returned only when calculable. |
| gross_margin_yoy_chg_bps | number | Year-over-year change in gross margin versus the same fiscal quarter one year prior, in basis points. Quarterly payload only. Returned only when calculable. |
| gross_margin_yoy_chg_pct | number | Year-over-year percentage change in gross margin versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| operating_income | number | No field description in OpenAPI. |
| operating_income_chg | number | Percentage change in operating income versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| operating_income_yoy_chg | number | Year-over-year change in operating income versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| operating_margin | number | Operating margin as a decimal (operating_income / revenue). Returned only when calculable. |
| operating_margin_chg_bps | number | Absolute change in operating margin versus the prior period in basis points. Returned only when calculable. |
| operating_margin_chg_pct | number | Percentage change in operating margin versus the prior period as a decimal. Returned only when calculable. |
| operating_margin_yoy_chg_bps | number | Year-over-year change in operating margin versus the same fiscal quarter one year prior, in basis points. Quarterly payload only. Returned only when calculable. |
| operating_margin_yoy_chg_pct | number | Year-over-year percentage change in operating margin versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| net_income | number | No field description in OpenAPI. |
| net_income_chg | number | Percentage change in net income versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| net_income_yoy_chg | number | Year-over-year change in net income versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| net_margin | number | Net margin as a decimal (net_income / revenue). Returned only when calculable. |
| net_margin_chg_bps | number | Absolute change in net margin versus the prior period in basis points. Returned only when calculable. |
| net_margin_chg_pct | number | Percentage change in net margin versus the prior period as a decimal. Returned only when calculable. |
| net_margin_yoy_chg_bps | number | Year-over-year change in net margin versus the same fiscal quarter one year prior, in basis points. Quarterly payload only. Returned only when calculable. |
| net_margin_yoy_chg_pct | number | Year-over-year percentage change in net margin versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| weighted_average_shares | number | No field description in OpenAPI. |
| weighted_average_shares_diluted | number | No field description in OpenAPI. |
| cash_and_equivalents | number | No field description in OpenAPI. |
| change_in_cash_and_equivalents | number | No field description in OpenAPI. |
| total_debt | number | No field description in OpenAPI. |
| total_assets | number | No field description in OpenAPI. |
| total_liabilities | number | No field description in OpenAPI. |
| shareholders_equity | number | No field description in OpenAPI. |
| net_cash_flow_from_operations | number | No field description in OpenAPI. |
| net_cash_flow_from_operations_chg | number | Percentage change in net cash flow from operations versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| net_cash_flow_from_operations_yoy_chg | number | Year-over-year change in net cash flow from operations versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| net_cash_flow_from_investing | number | No field description in OpenAPI. |
| net_cash_flow_from_investing_chg | number | Percentage change in net cash flow from investing versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| net_cash_flow_from_investing_yoy_chg | number | Year-over-year change in net cash flow from investing versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| net_cash_flow_from_financing | number | No field description in OpenAPI. |
| net_cash_flow_from_financing_chg | number | Percentage change in net cash flow from financing versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| net_cash_flow_from_financing_yoy_chg | number | Year-over-year change in net cash flow from financing versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| capital_expenditure | number | No field description in OpenAPI. |
| capital_expenditure_chg | number | Percentage change in capital expenditure versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| capital_expenditure_yoy_chg | number | Year-over-year change in capital expenditure versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |
| free_cash_flow | number | No field description in OpenAPI. |
| free_cash_flow_chg | number | Percentage change in free cash flow versus the prior period as a decimal. Sequential in the quarterly payload, year-over-year in the annual payload. Returned only when calculable. |
| free_cash_flow_yoy_chg | number | Year-over-year change in free cash flow versus the same fiscal quarter one year prior, as a decimal. Quarterly payload only. Returned only when calculable. |

### `ErrorResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| error | string | A short error message. |
| message | string | A more detailed error message. |

### `Exhibit`

| Field | Type | Meaning |
| --- | --- | --- |
| number | string | The exhibit number (e.g., '99.1'). |
| description | string | The description of the exhibit. |
| url | string/uri | The URL to the exhibit document on the SEC website. |
| text | string | The raw text content of the exhibit. Only included when 'include_exhibits' parameter is set to true. |

### `Filing`

| Field | Type | Meaning |
| --- | --- | --- |
| cik | integer | The Central Index Key (CIK) of the company. |
| accession_number | string | The accession number of the filing. |
| filing_type | string | The type of the SEC filing (e.g., 10-Q, 8-K). |
| report_date | string/date | The date of the report. |
| filing_date | string/date | The date the filing was submitted to the SEC. |
| ticker | string | The ticker symbol. |
| url | string/uri | The URL of the SEC filing. |

### `FilingItem`

| Field | Type | Meaning |
| --- | --- | --- |
| number | string | The item number. |
| name | string | The item name. |
| text | string | The item text. |
| exhibits | array<object> | An array of exhibits linked to this item. Only present for 8-K filings when exhibits exist. |

### `FilingItemsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| ticker | string | The ticker symbol of the company. |
| cik | string | The Central Index Key (CIK) of the company. |
| filing_type | string | The type of filing. |
| accession_number | string | The accession number of the filing. |
| year | integer | The year of the filing. |
| quarter | integer | The quarter of the filing. |
| items | array<object> | No field description in OpenAPI. |

### `FilingsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| filings | array<object> | No field description in OpenAPI. |

### `FinancialMetricSnapshot`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| market_cap | number | The total market capitalization (stock price × shares outstanding). |
| enterprise_value | number | The total value of the company (market cap + debt - cash). |
| price_to_earnings_ratio | number | Price to earnings ratio. |
| price_to_book_ratio | number | Price to book ratio. |
| price_to_sales_ratio | number | Price to sales ratio. |
| enterprise_value_to_ebitda_ratio | number | Enterprise value to EBITDA ratio. |
| enterprise_value_to_revenue_ratio | number | Enterprise value to revenue ratio. |
| free_cash_flow_yield | number | Free cash flow yield. |
| peg_ratio | number | Price to earnings growth ratio. |
| gross_margin | number | Gross profit as a percentage of revenue. |
| operating_margin | number | Operating income as a percentage of revenue. |
| net_margin | number | Net income as a percentage of revenue. |
| return_on_equity | number | Net income as a percentage of shareholders' equity. |
| return_on_assets | number | Net income as a percentage of total assets. |
| return_on_invested_capital | number | Net operating profit after taxes as a percentage of invested capital. |
| asset_turnover | number | Revenue divided by average total assets. |
| inventory_turnover | number | Cost of goods sold divided by average inventory. |
| receivables_turnover | number | Revenue divided by average accounts receivable. |
| days_sales_outstanding | number | Average accounts receivable divided by revenue over the period. |
| operating_cycle | number | Inventory turnover + receivables turnover. |
| working_capital_turnover | number | Revenue divided by average working capital. |
| current_ratio | number | Current assets divided by current liabilities. |
| quick_ratio | number | Quick assets divided by current liabilities. |
| cash_ratio | number | Cash and cash equivalents divided by current liabilities. |
| operating_cash_flow_ratio | number | Operating cash flow divided by current liabilities. |
| debt_to_equity | number | Total debt divided by shareholders' equity. |
| debt_to_assets | number | Total debt divided by total assets. |
| interest_coverage | number | EBIT divided by interest expense. |
| revenue_growth | number | Year-over-year growth in revenue. |
| earnings_growth | number | Year-over-year growth in earnings. |
| book_value_growth | number | Year-over-year growth in book value. |
| earnings_per_share_growth | number | Growth in earnings per share over the period. |
| free_cash_flow_growth | number | Growth in free cash flow over the period. |
| operating_income_growth | number | Growth in operating income over the period. |
| ebitda_growth | number | Growth in EBITDA over the period. |
| payout_ratio | number | Dividends paid as a percentage of net income. |
| earnings_per_share | number | Net income divided by weighted average shares outstanding. |
| book_value_per_share | number | Shareholders' equity divided by shares outstanding. |
| free_cash_flow_per_share | number | Free cash flow divided by shares outstanding. |

### `FinancialMetricSnapshotResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| snapshot | object | No field description in OpenAPI. |

### `FinancialMetricsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| report_period | string/date | The reporting period of the financial metrics. |
| fiscal_period | string | The fiscal period of the financial metrics. |
| period | string enum=quarterly\|ttm\|annual | The time period of the financial metrics. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| enterprise_value | number | The total value of the company (market cap + debt - cash). |
| price_to_earnings_ratio | number | Price to earnings ratio. |
| price_to_book_ratio | number | Price to book ratio. |
| price_to_sales_ratio | number | Price to sales ratio. |
| enterprise_value_to_ebitda_ratio | number | Enterprise value to EBITDA ratio. |
| enterprise_value_to_revenue_ratio | number | Enterprise value to revenue ratio. |
| free_cash_flow_yield | number | Free cash flow yield. |
| peg_ratio | number | Price to earnings growth ratio. |
| gross_margin | number | Gross profit as a percentage of revenue. |
| operating_margin | number | Operating income as a percentage of revenue. |
| net_margin | number | Net income as a percentage of revenue. |
| return_on_equity | number | Net income as a percentage of shareholders' equity. |
| return_on_assets | number | Net income as a percentage of total assets. |
| return_on_invested_capital | number | Net operating profit after taxes as a percentage of invested capital. |
| asset_turnover | number | Revenue divided by average total assets. |
| inventory_turnover | number | Cost of goods sold divided by average inventory. |
| receivables_turnover | number | Revenue divided by average accounts receivable. |
| days_sales_outstanding | number | Average accounts receivable divided by revenue over the period. |
| operating_cycle | number | Inventory turnover + receivables turnover. |
| working_capital_turnover | number | Revenue divided by average working capital. |
| current_ratio | number | Current assets divided by current liabilities. |
| quick_ratio | number | Quick assets divided by current liabilities. |
| cash_ratio | number | Cash and cash equivalents divided by current liabilities. |
| operating_cash_flow_ratio | number | Operating cash flow divided by current liabilities. |
| debt_to_equity | number | Total debt divided by shareholders' equity. |
| debt_to_assets | number | Total debt divided by total assets. |
| interest_coverage | number | EBIT divided by interest expense. |
| revenue_growth | number | Year-over-year growth in revenue. |
| earnings_growth | number | Year-over-year growth in earnings. |
| book_value_growth | number | Year-over-year growth in book value. |
| earnings_per_share_growth | number | Growth in earnings per share over the period. |
| free_cash_flow_growth | number | Growth in free cash flow over the period. |
| operating_income_growth | number | Growth in operating income over the period. |
| ebitda_growth | number | Growth in EBITDA over the period. |
| payout_ratio | number | Dividends paid as a percentage of net income. |
| earnings_per_share | number | Net income divided by weighted average shares outstanding. |
| book_value_per_share | number | Shareholders' equity divided by shares outstanding. |
| free_cash_flow_per_share | number | Free cash flow divided by shares outstanding. |

### `FinancialsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| financials | object | No field description in OpenAPI. |

### `FinancialsSearchResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| search_results | array<object> | No field description in OpenAPI. |

### `IncomeStatement`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| report_period | string/date | The reporting period of the income statement. |
| fiscal_period | string | The fiscal period of the income statement. |
| period | string enum=quarterly\|ttm\|annual | The time period of the income statement. |
| currency | string | The currency in which the financial data is reported. |
| accession_number | string | The SEC accession number of the filing. |
| filing_url | string/uri | URL to the SEC filing. |
| revenue | number | The total revenue of the company. |
| cost_of_revenue | number | The cost of revenue of the company. |
| gross_profit | number | The gross profit of the company. |
| operating_expense | number | The operating expenses of the company. |
| selling_general_and_administrative_expenses | number | The selling, general, and administrative expenses of the company. |
| research_and_development | number | The research and development expenses of the company. |
| operating_income | number | The operating income of the company. |
| interest_expense | number | The interest expenses of the company. |
| ebit | number | The earnings before interest and taxes of the company. |
| income_tax_expense | number | The income tax expenses of the company. |
| net_income_discontinued_operations | number | The net income from discontinued operations of the company. |
| net_income_non_controlling_interests | number | The net income from non-controlling interests of the company. |
| net_income | number | The net income of the company. |
| net_income_common_stock | number | The net income available to common stockholders of the company. |
| preferred_dividends_impact | number | The impact of preferred dividends on the net income of the company. |
| consolidated_income | number | The consolidated income of the company. |
| earnings_per_share | number | The earnings per share of the company. |
| earnings_per_share_diluted | number | The diluted earnings per share of the company. |
| dividends_per_common_share | number | The dividends per common share of the company. |
| weighted_average_shares | number | The weighted average shares of the company. |
| weighted_average_shares_diluted | number | The diluted weighted average shares of the company. |

### `IncomeStatementResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| income_statements | array<object> | No field description in OpenAPI. |

### `IncomeStatementSegmentsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

### `InsiderTrade`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol of the company. |
| issuer | string | The name of the issuing company. |
| name | string | The name of the insider. |
| title | string | The title of the insider. |
| is_board_director | boolean | Whether the insider is a board director. |
| transaction_date | string/date | The date of the transaction. |
| transaction_shares | number | The number of shares involved in the transaction. |
| transaction_price_per_share | number | The price per share in the transaction. |
| transaction_value | number | The total value of the transaction. |
| shares_owned_before_transaction | number | The number of shares owned before the transaction. |
| shares_owned_after_transaction | number | The number of shares owned after the transaction. |
| security_title | string | The title of the security involved in the transaction. |
| transaction_type | string | A human-readable description of the transaction type. |
| filing_date | string/date | The date the transaction was filed. |

### `InsiderTradesResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| insider_trades | array<object> | No field description in OpenAPI. |

### `InstitutionalOwnership`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol, if queried by investor. |
| investor | string | The investor name, if queried by ticker. |
| report_period | string/date | The reporting period of the institutional ownership. |
| security_type | string enum=common_stock\|put_option\|call_option\|warrant\|preferred_stock\|debt\|fund\|other | The type of security held by the investment manager. |
| price | number | The estimated purchase price of the equity position. |
| shares | number | The number of shares held by the investment manager. |
| market_value | number | The market value of the equity position. |

### `InstitutionalOwnershipResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| institutional-ownership | array<object> | No field description in OpenAPI. |

### `InterestRate`

| Field | Type | Meaning |
| --- | --- | --- |
| bank | string | The symbol of the central bank. |
| name | string | The name of the central bank. |
| rate | number | The interest rate of the central bank. |
| date | string | The date of the interest rate in YYYY-MM-DD format. |

### `InterestRatesResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| interest_rates | array<object> | No field description in OpenAPI. |

### `KPIGuidanceItem`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | No field description in OpenAPI. |
| metric_name | string | No field description in OpenAPI. |
| value | number | No field description in OpenAPI. |
| unit | string | No field description in OpenAPI. |
| period | string | No field description in OpenAPI. |
| period_type | string enum=quarterly\|annual | No field description in OpenAPI. |
| segment | string | No field description in OpenAPI. |
| low | number | Low end of guidance range. |
| high | number | High end of guidance range. |
| point_estimate | number | Single-point guidance when no range is given. |
| prior_value | number | Prior guidance value for revisions. |
| raw_text | string | Original guidance text from the filing. |
| change_direction | string | Direction: raised, lowered, maintained, initiated, withdrawn. |
| source_text | string | No field description in OpenAPI. |
| source_url | string/uri | No field description in OpenAPI. |

### `KPIGuidanceResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| kpi_guidance | array<object> | No field description in OpenAPI. |

### `KPIMetric`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | Stock ticker symbol. |
| metric_name | string | Canonical metric name. |
| value | number | Extracted numeric value. |
| unit | string | Unit of measure (e.g., %, cents, dollars, miles, count). |
| period | string | Reporting period (e.g., Q4 2025, FY 2025, or a date like 2025-04-30 for non-standard fiscal years). |
| period_type | string enum=quarterly\|annual | Whether this is a quarterly or annual figure. |
| segment | string | Business segment, geography, or sub-category. Only present when the metric is reported for a sub-unit rather than the consolidated entity. |
| yoy_value | number | Year-over-year comparison value from the prior period. |
| yoy_change_pct | number | Year-over-year percentage change. |
| source_text | string | The source text span from the filing where this value was found. |
| source_url | string/uri | Direct link to the SEC filing. When available, includes a text fragment that highlights the source in the browser. |

### `KPIMetricsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| kpi_metrics | array<object> | No field description in OpenAPI. |

### `KPINonGAAPMetric`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | No field description in OpenAPI. |
| metric_name | string | No field description in OpenAPI. |
| value | number | No field description in OpenAPI. |
| unit | string | No field description in OpenAPI. |
| period | string | No field description in OpenAPI. |
| period_type | string enum=quarterly\|annual | No field description in OpenAPI. |
| gaap_equivalent | string | The GAAP line item this metric adjusts. |
| key_adjustments | string | Description of adjustments made. |
| source_text | string | No field description in OpenAPI. |
| source_url | string/uri | No field description in OpenAPI. |

### `KPINonGAAPResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| kpi_non_gaap | array<object> | No field description in OpenAPI. |

### `News`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| title | string | The title of the news article. |
| source | string | The source of the news article. |
| date | string/date | The date the news article was published. |
| url | string/uri | The URL of the news article. |

### `NewsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| news | array<object> | No field description in OpenAPI. |

### `Price`

| Field | Type | Meaning |
| --- | --- | --- |
| open | number | The open price of the ticker in the given time period. |
| close | number | The close price of the ticker in the given time period. |
| high | number | The high price of the ticker in the given time period. |
| low | number | The low price of the ticker in the given time period. |
| volume | integer | The trading volume of the ticker in the given time period. |
| time | string | The human-readable time format of the price in UTC. |

### `PriceSnapshot`

| Field | Type | Meaning |
| --- | --- | --- |
| price | number | The current price of the stock. |
| ticker | string | The ticker symbol. |
| day_change | number | The price change since the previous trading day's close. |
| day_change_percent | number | The percentage price change since the previous trading day's close. |
| time | string | The timestamp of the price snapshot in human-readable format in UTC. |
| time_milliseconds | number | The timestamp of the price snapshot in milliseconds since epoch. |

### `PriceSnapshotMarketResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| snapshots | array<object> | No field description in OpenAPI. |

### `PriceSnapshotResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| snapshot | object | No field description in OpenAPI. |

### `PricesResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| prices | array<object> | No field description in OpenAPI. |

### `SearchFiltersRequest`

| Field | Type | Meaning |
| --- | --- | --- |
| limit | integer | The maximum number of results to return. |
| filters | array<object> | An array of filter objects to apply to the search. |

### `SearchLineItemsRequest`

| Field | Type | Meaning |
| --- | --- | --- |
| line_items | array<string> | An array of line items to apply to the search. |
| tickers | array<string> | An array of tickers to apply to the search. |
| period | string enum=annual\|quarterly\|ttm | The time period for the financial data. |
| limit | integer | The maximum number of results to return. |

### `SegmentBreakdown`

| Field | Type | Meaning |
| --- | --- | --- |
| label | string | The label for the segment (e.g., 'iPhone', 'Americas'). |
| value | number | The value for the segment. |

### `SegmentCategory`

| Field | Type | Meaning |
| --- | --- | --- |
| product | array<object> | Breakdowns by product line. |
| segment | array<object> | Breakdowns by business or operating segment. |

### `SegmentMetadata`

| Field | Type | Meaning |
| --- | --- | --- |
| ticker | string | The ticker symbol. |
| report_period | string/date | The reporting period. |
| fiscal_period | string | The fiscal period (e.g., Q1, Q2, Q3, Q4, FY). |
| period | string enum=quarterly\|annual | The time period. |
| currency | string | The reporting currency (e.g., USD, EUR, GBP). |
| accession_number | string | The SEC filing accession number. |
| filing_url | string | URL to the SEC filing. |

### `SegmentedFinancialsResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| segmented_financials | array<allOf(object, object)> | No field description in OpenAPI. |

### `Signal`

| Field | Type | Meaning |
| --- | --- | --- |
| type | string enum=EPS_BEAT\|EPS_MISS\|REVENUE_BEAT\|REVENUE_MISS\|MARGIN_EXPANSION\|MARGIN_CONTRACTION\|GUIDANCE_RAISED\|GUIDANCE_LOWERED\|GUIDANCE_REAFFIRMED\|GUIDANCE_INITIATED\|SEGMENT_OUTPERFORMANCE\|SEGMENT_UNDERPERFORMANCE\|NON_GAAP_DIVERGENCE\|BACKLOG_SURGE\|BACKLOG_DECLINE\|REVENUE_INCREASE\|REVENUE_DECLINE\|STRONG_REVENUE_INCREASE\|STRONG_REVENUE_DECLINE\|NET_INCOME_INCREASE\|NET_INCOME_DECLINE\|STRONG_NET_INCOME_INCREASE\|STRONG_NET_INCOME_DECLINE\|EPS_INCREASE\|EPS_DECLINE\|STRONG_EPS_INCREASE\|STRONG_EPS_DECLINE\|CAPEX_INCREASE\|CAPEX_DECLINE\|STRONG_CAPEX_INCREASE\|STRONG_CAPEX_DECLINE | Signal category. EPS_BEAT/EPS_MISS: actual EPS vs analyst estimate. REVENUE_BEAT/REVENUE_MISS: actual revenue vs estimate. MARGIN_EXPANSION/MARGIN_CONTRACTION: any of gross/operating/net margin moved year-over-year by ≥100bps. GUIDANCE_RAISED/GUIDANCE_LOWERED/GUIDANCE_REAFFIRMED/GUIDANCE_INITIATED: forward-looking guidance change reported in the filing. SEGMENT_OUTPERFORMANCE/SEGMENT_UNDERPERFORMANCE: best- or worst-performing reportable segment by organic revenue growth. NON_GAAP_DIVERGENCE: an adjusted/non-GAAP figure differs materially from its GAAP counterpart on the same period. BACKLOG_SURGE/BACKLOG_DECLINE: backlog or remaining performance obligations moved year-over-year by ≥20%. REVENUE_INCREASE/STRONG_REVENUE_INCREASE and DECLINE variants: tiered year-over-year revenue change (≥10% / ≥20% magnitude). NET_INCOME_INCREASE/STRONG_NET_INCOME_INCREASE and DECLINE variants: tiered year-over-year net income change (≥15% / ≥30%). EPS_INCREASE/STRONG_EPS_INCREASE and DECLINE variants: tiered year-over-year EPS change (≥15% / ≥30%); independent from net income because share buybacks can amplify EPS movement. CAPEX_INCREASE/STRONG_CAPEX_INCREASE and DECLINE variants: tiered year-over-year capital expenditure change in spending magnitude (≥15% / ≥30%). |
| metric | string | The metric this signal references, when applicable (e.g. `revenue`, `earnings_per_share`, `operating_margin`, or a guidance/KPI metric name). |
| period | string | The fiscal period the signal references (e.g. `2026-Q1`, `FY 2026`). Forward guidance signals reference the guided period; financial signals reference the reporting period of this filing. |
| headline | string | Pre-rendered, human-readable summary suitable for direct display. |
| details | object | Structured fields specific to the signal type. Contents vary: surprise signals carry `actual`, `estimate`, `surprise_pct`; guidance signals carry `low`, `high`, `point_estimate`, `prior_value`, `revision_pct`; segment signals carry `best`, `worst`, `spread_pp`; non-GAAP signals carry `gaap_field`, `non_gaap_value`, `gaap_value`, `divergence_pct`; growth signals carry `current`, `yoy_chg`. Null values are omitted. Refer to type-specific examples. |

### `TickersResponse`

| Field | Type | Meaning |
| --- | --- | --- |
| resource | string | The resource type identifier. |
| tickers | array<string> | List of available ticker symbols. |

