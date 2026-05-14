# RAHUL-39 Recommended Next Implementation Tickets

These tickets convert the research/specification work into implementable engineering slices. They are ordered to reduce data-source risk before endpoint work.

## P0 - Foundation and Commercial/Data Access

### 1. Bursa licensing and source-access decision

- **Goal:** Decide legal/commercial sources for Bursa announcements, real-time/delayed/EOD/historical market data, and non-display analytics.
- **Why:** Production `/prices`, `/filings`, `/financials`, metrics, and screeners are blocked without durable rights and feed access.
- **Acceptance criteria:** Written source matrix with provider, allowed uses, latency, redistribution/non-display terms, cost, contact, and fallback. Confirm whether Bursa Information Services, Bursa LINK, historical data packages, authorized vendors, or another provider will be used.

### 2. Bursa security master and symbology service

- **Goal:** Build canonical security/issuer reference data for Bursa.
- **Scope:** Stock code as string, short/long name, ISIN, market, sector, subsector/classification, product type, Shariah flag, listing status, listing/change-of-name history, vendor aliases (`1818.KL`, `BURSA MK`, etc.).
- **Acceptance criteria:** `/company/facts`, `/company/facts/tickers`, `/company/facts/ciks` can be backed by versioned reference data with no SEC CIK ambiguity.

### 3. Announcement ingestion and provenance store

- **Goal:** Ingest Bursa company announcements and attachments with stable ids.
- **Scope:** Category/subcategory, ticker/stock code, title, announcement datetime, URLs, attachment metadata, hashes, parser status, source snapshots.
- **Acceptance criteria:** `/filings`, `/filings/tickers`, `/filings/types` can return deterministic results; duplicate/update handling is tested.

## P1 - Core API Compatibility

### 4. OpenAPI compatibility harness

- **Goal:** Turn the Financial Datasets OpenAPI schemas and docs-only endpoints into tests for the Bursa clone.
- **Scope:** Response envelopes, auth errors, validation errors, trailing slash routing, `ticker`/`cik` alias behavior, `results` vs `search_results` decision, `institutional_ownership` vs hyphen key decision.
- **Acceptance criteria:** Golden JSON fixtures exist for every OpenAPI endpoint and docs-only availability endpoint.

### 5. Market data adapter

- **Goal:** Implement EOD and snapshot price ingestion with explicit latency/license policy.
- **Scope:** `/prices`, `/prices/tickers`, `/prices/snapshot`, `/prices/snapshot/tickers`, later `/prices/snapshot/market`; normalize volume units; define raw vs adjusted OHLCV.
- **Acceptance criteria:** At least 20 Bursa securities across products have verified OHLCV and snapshot fixtures with source timestamps.

### 6. Bursa filings/items parser

- **Goal:** Parse announcement landing pages and attachments into `/filings/items` text/exhibit structures.
- **Scope:** Financial Results, Annual Report, Annual Audited Account, Entitlements, Changes in Shareholdings, Director Interest, Shares Buy Back, General Announcement.
- **Acceptance criteria:** Item extraction fixtures for at least 5 issuers and 5 announcement categories; source text and URLs included.

## P2 - Financial Statements and Metrics

### 7. Financial Results structured parser

- **Goal:** Extract core quarterly/annual earnings fields from Bursa Financial Results announcements.
- **Scope:** report period, fiscal period, revenue, profit lines, EPS, NTAB/share, dividends/share, currency, filing date, source announcement.
- **Acceptance criteria:** `/earnings` company mode and feed mode return Bursa rows with optional fields omitted/null when unavailable.

### 8. As-reported statement extraction

- **Goal:** Build recursive statement trees from annual/quarterly report attachments.
- **Scope:** income statement, balance sheet, cash flow; group/consolidated column selection; period and scale detection; source spans/bounding boxes.
- **Acceptance criteria:** `/financials/*/as-reported` and `/financials/as-reported` pass fixtures for diversified issuers, banks, REITs, and plantations.

### 9. Normalized MFRS-to-Financial-Datasets mapping

- **Goal:** Normalize Bursa/MFRS source statements into upstream canonical schemas.
- **Scope:** income statement, balance sheet, cash flow, TTM computation, fiscal periods, currency, restatements, null policy.
- **Acceptance criteria:** `/financials/income-statements`, `/financials/balance-sheets`, `/financials/cash-flow-statements`, and `/financials` produce schema-complete responses with field-level source lineage.

### 10. Financial metrics and screener engine

- **Goal:** Compute historical/snapshot metrics and enable screeners/line-item search.
- **Scope:** `/financial-metrics`, `/financial-metrics/snapshot`, `/financials/search/screener`, `/financials/search/line-items`, filters endpoint, docs-only ticker lists.
- **Acceptance criteria:** Metrics are reproducible from stored prices/statements; field availability is exposed; unsupported filters return documented 400 errors.

### 11. Segment disclosures

- **Goal:** Extract and normalize MFRS 8/issuer segment data.
- **Scope:** Income, balance sheet, cash-flow segment endpoints and all-segments bundle.
- **Acceptance criteria:** Segment fixtures cover product, geography, and operating-segment disclosures with empty/null behavior for missing categories.

## P3 - Ownership, Corporate Actions, News, Macro, and KPIs

### 12. Corporate actions and entitlements model

- **Goal:** Model dividends, rights, bonus issues, share splits/subdivisions, warrants, share buybacks, treasury-share changes.
- **Scope:** Internal model plus dependencies for price adjustment, shares outstanding, dividends/share, and metrics.
- **Acceptance criteria:** Entitlement and share buyback announcements are parsed and linked to affected security codes.

### 13. Insider/director/substantial shareholder endpoint

- **Goal:** Implement `/insider-trades` and docs-only list endpoints using Bursa director/substantial-shareholder announcements.
- **Scope:** Party identity, directorate/title, direct/indirect holdings, share changes, action taxonomy, filing-date filters.
- **Acceptance criteria:** Supports name/type/date filters and returns stable holdings fields where computable.

### 14. Institutional ownership approximation

- **Goal:** Implement `/institutional-ownership` with clear Malaysia semantics.
- **Scope:** Substantial shareholder notices, annual report top shareholders, investor-name aggregation, security type mapping.
- **Acceptance criteria:** Query by ticker and investor both work with caveat metadata; no claim of 13F equivalence.

### 15. BNM macro/rates adapter

- **Goal:** Implement `bank=BNM` macro interest-rate endpoints.
- **Scope:** OPR historical/snapshot, optional MYOR/MYOR-i/MGS/FX extension codes after API design approval.
- **Acceptance criteria:** `/macro/interest-rates`, `/macro/interest-rates/snapshot`, `/macro/interest-rates/banks` return verified BNM/data.gov.my sourced rows.

### 16. News source/licensing adapter

- **Goal:** Implement `/news` with Bursa announcements plus licensed editorial feeds.
- **Scope:** Company news by ticker, market news without ticker, source labeling, URL/date normalization.
- **Acceptance criteria:** No unlicensed full-text redistribution; response shape matches upstream `NewsResponse`.

### 17. KPI, guidance, and non-GAAP extraction

- **Goal:** Extract sector KPIs, guidance, and adjusted/non-MFRS metrics from Bursa filings and presentations.
- **Scope:** Sector dictionaries, unit normalization, source text spans, confidence/QA workflow, `/kpi/*` and docs-only ticker endpoints.
- **Acceptance criteria:** At least 5 sector-specific KPI dictionaries and validated examples; all extracted values cite source text/URL.

## Cross-Cutting Requirements

- Do not expose secrets in examples or fixtures.
- Keep all source documents, parser outputs, and normalized rows tied to source URLs and parser versions.
- Add legal/commercial review gates before enabling production ingestion from Bursa or news publishers.
- Add live-data drift monitoring for Financial Datasets docs/OpenAPI changes.
