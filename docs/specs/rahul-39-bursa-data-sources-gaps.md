# RAHUL-39 Bursa Data Sources, Conventions, Gaps, and Risks

Prepared 2026-05-14. This is the implementation research memo for sourcing a Bursa Malaysia clone of the Financial Datasets API.

## Source Review Summary

### Financial Datasets sources crawled

| Source | What was reviewed | Implementation relevance |
| --- | --- | --- |
| <https://docs.financialdatasets.ai/llms.txt> | Official documentation index. Listed company facts, earnings, SEC filings, financial metrics, financials, segmented/as-reported financials, insider trades, institutional ownership, KPI, macro, news, prices, guides, market coverage, MCP, OpenAPI, quickstart, support. | Drove the crawl; all listed API pages were downloaded. |
| <https://financialdatasets.ai/openapi.json> and <https://docs.financialdatasets.ai/api/openapi.json> | OpenAPI 3.0.1 spec with 45 operations, security scheme, parameters, responses, and 64 schemas. | Canonical schema inventory and response field semantics. |
| Endpoint markdown pages under `docs.financialdatasets.ai/api/**` | Narrative docs, examples, availability endpoints, edge notes, defaults, and doc/OpenAPI discrepancies. | Identified extra docs-only endpoints absent from OpenAPI and endpoint-specific behavior. |
| Guides and support pages | Python examples, quickstart flows, market coverage claims, and SDK/MCP usage. | Confirmed examples use `X-API-KEY`; found quickstart's screener `results` response discrepancy. |

### Bursa/Malaysia sources reviewed

| Source | Observed relevant facts | Use in clone |
| --- | --- | --- |
| Bursa Company Announcements: <https://www.bursamalaysia.com/market_information/announcements/company_announcement> | Search filters include Company, keyword, date range, Category, Sub Category, Market, Sector, Subsector. Categories observed include Annual Audited Account, Annual Report, Changes in Shareholdings, Entitlements, Financial Results, General Announcement, General Meetings, IPO, Listing Information/Profile, Shares Buy Back, Take-over Offer, Transfer of Listing, and Unusual Market Activity. | Primary filing/announcement inventory; categories seed `/filings/types` and parsers. |
| Bursa announcement disclaimer: <https://www.bursamalaysia.com/disclaimer_company_announcement> | Bursa states company announcements are submitted electronically by/on behalf of issuers and are provided as-is; Bursa acts as a passive conduit and does not verify/endorse contents. | Provenance/risk note for filings, news, and financials. |
| Bursa Announcements Message Structure v1.6 PDF | Announcement types include Financial Results, Entitlement, Change in Boardroom, substantial shareholder changes, director interest changes, share buy-back notices, treasury-share changes, audit committee and CEO changes. Financial Results fields include financial date, report status, financial quarter, revenue, profit lines, EPS, NTAB/share, gross dividend/share, period dates. Entitlement fields include ex-date, lodgement date/time, payment date, entitlement numerator/denominator/percentage/sen. | Best structured source model for financial results, corporate actions, insider/shareholder changes, and Bursa-specific filing types. |
| Bursa Listing Directory: Main/ACE/LEAP pages | Market selector includes Main Market, ACE Market, LEAP Market, LFX Market, PN17 & GN3 Companies, Change of Name. Pages list company names and websites; observed counts included Main Market 813 entries, ACE 253 entries, LEAP 55 entries during this run. | Reference universe and market classification seed. Counts are point-in-time, not hard-coded. |
| Bursa company profile example (`stock_code=1818`) | Exposes company name, Market, Sector, company website, Shariah Compliant flag, price snapshot, Stock Code, recent Financial Result announcements, General Meetings, Entitlements, same-sector/classification links. | Company facts, prices snapshot, announcement pointers, sector mapping, corporate actions. |
| Bursa ISIN page: <https://www.bursamalaysia.com/trade/trading_resources/equities/isin> | Bursa Malaysia is Malaysia's official National Numbering Agency for ISINs. ISIN categories include securities listed on Bursa Malaysia, debt instruments, unlisted Malaysian securities, and Labuan-incorporated unlisted securities. | ISIN/reference data; confirms source for Malaysia security identifiers. |
| Bursa Listing Criteria page | Bursa offers Main Market, ACE Market, and LEAP Market. Main is for established companies; ACE is sponsor-driven for growth companies; LEAP is adviser-driven and for sophisticated investors. | Market taxonomy and company facts category semantics. |
| Bursa Securities Market page | Securities trading sessions observed: Monday-Friday, first session 09:00-12:30 and second session 14:30-17:00. Website prices are delayed by 15 minutes; real-time data requires stockbroker or Information Services Vendors. | Market calendar/session model and delayed vs real-time risk. |
| Bursa Information Vendors page | Lists authorized vendors subscribing to Bursa Malaysia real-time market information, including Bloomberg, FactSet, ICE Data Services, Morningstar Real-Time, Refinitiv, SIX, and local vendors. Table updated 17 Dec 2025 during run. | Licensed data-provider options for prices/snapshots. |
| Bursa Information Services Guidelines / ISLA PDF | Describes Information Services Licence Agreement, real-time/delayed/EOD/historical qualities, public website display restrictions, non-display usage, original created works, and monthly usage reporting. | Legal/commercial constraints for production redistribution and analytics. |
| Bursa BTS2 FIX Specification - Market Data PDF | Documents BTS2 FIX 5.0 SP1 market-data messages, Market Data Request, Snapshot/Full Refresh, Incremental Refresh, Trading Session Status, Security Status, and session establishment. | Real-time market data feed adapter design. |
| Bursa Who We Are page | Bursa says it helps over 1,100 companies raise capital across Main, ACE, and LEAP markets and offers equities, derivatives, offshore/Islamic assets, ETFs, REITs, ETBS. | Coverage expectations and product universe. |
| data.gov.my Interest Rates dashboard: <https://data.gov.my/dashboard/interest-rates> | Uses BNM Monthly Highlights and Statistics; OPR is the primary policy tool of Bank Negara Malaysia; dashboard is government open data. | Macro/rates source for historical interest-rate endpoint. |
| BNM Financial Markets Portal: <https://financialmarkets.bnm.gov.my/about-myor-myor-i> | Shows OPR, MGS yield, MYOR, KL USD/MYR reference rate, FX turnover; defines MYOR/MYOR-i as transaction-based unsecured overnight MYR interbank benchmarks and gives publication timing. | Macro snapshot source and extended rates/FX references. |
| Bursa Listing Requirements pages/PDF search results | Public listing-requirements pages exist for Main/ACE/LEAP. Search results and secondary references indicate quarterly reports are due within two months after quarter end and annual reports within four months after financial year-end for Main Market listed issuers. | Reporting-period expectations; should be confirmed from the latest official Listing Requirements before encoding validation. |

## Bursa-Specific Conventions to Encode

### Markets and product universe

- Main Market, ACE Market, and LEAP Market are distinct listing markets with different issuer profiles and investor access.
- Bursa product universe includes ordinary shares, REITs, ETFs, ETBS, structured warrants, company warrants, rights, loan stocks, and other securities. The API must distinguish issuer-level company records from security-level tradable instruments.
- Stock codes are strings. Do not parse as integers; codes may contain letters/suffixes.
- Bursa sectors differ from US GICS/SIC. Use Bursa sector and subsector/classification as source-of-truth unless a licensed crosswalk is added.
- Shariah-compliant status is relevant company/security metadata and should be stored even if it is not exposed by the upstream Financial Datasets schema.

### Trading sessions, timezone, and prices

- Exchange-local timezone is `Asia/Kuala_Lumpur`.
- Securities market sessions are Monday-Friday 09:00-12:30 and 14:30-17:00, subject to holidays/special sessions.
- Public website prices are delayed by 15 minutes. Real-time snapshots and market snapshots require licensed feed arrangements.
- Volume should be normalized to shares/units in API responses, while raw source units/lots should be retained. Bursa website labels volume as `Vol ('00)` in observed pages.
- Decide raw vs adjusted OHLCV policy before implementing `/prices`; Financial Datasets docs do not describe split/dividend adjustment behavior.

### Financial reporting

- Bursa Financial Results announcements provide structured core fields but do not guarantee every line item needed for the upstream normalized statement schemas.
- Annual reports and annual audited accounts contain complete statements but are likely PDF/HTML attachments requiring table extraction, OCR fallbacks, and statement hierarchy reconstruction.
- Malaysia reporting follows MFRS/IFRS-style statements. Build a mapping layer from source labels to Financial Datasets canonical fields rather than assuming US GAAP names.
- Banks, insurers, REITs, plantations, and utilities require sector-specific statement and KPI mapping.
- Period labels must support non-calendar fiscal years.

### Announcements, filings, and corporate actions

- Bursa announcement category/subcategory is the closest equivalent to SEC filing type.
- Corporate actions are not separate Financial Datasets endpoints, but they are required dependencies for price adjustment, shares outstanding, dividends per share, entitlement dates, and financial metrics.
- Entitlement records need ex-date, entitlement date/lodgement/book-closure fields, payment date, entitlement type, numerator/denominator, percentage/sen, description, and affected security code.
- Rights/warrants/bonus issues create temporary or new securities; maintain a security master and lifecycle table.

### Insider and ownership data

- Bursa director and substantial-shareholder disclosures are the closest equivalent to insider trading data.
- SEC Form 4 concepts such as officer role and transaction code do not map perfectly; transaction type and action should use Bursa terms with optional compatibility aliases.
- Institutional ownership in the US is standardized by 13F; Malaysia does not have an equivalent all-institution quarterly portfolio filing. Treat `/institutional-ownership` as a best-effort aggregation of substantial-shareholder changes, annual report top shareholders, and any licensed shareholder datasets.

### Macro and currencies

- Default currency is `MYR`; preserve issuer-reported currencies where different.
- `BNM` should be the primary `bank` code for Malaysia interest-rate endpoints.
- OPR is the policy-rate analogue. MYOR/MYOR-i, KLIBOR-transition rates, MGS yields, and KL USD/MYR reference rate are adjacent macro/rates datasets but may require code extensions or separate internal measures.

## Unknowns, Blockers, and Data-Source Gaps

| Gap/blocker | Impacted endpoints | Severity | Recommended resolution |
| --- | --- | --- | --- |
| Production rights for Bursa real-time, delayed, EOD, historical, and non-display market data are not established. | `/prices`, `/prices/snapshot`, `/prices/snapshot/market`, metrics, screeners. | Blocker for production. | Contact Bursa Information Services or an authorized vendor; decide display/non-display/original-created-work licensing and monthly reporting obligations. |
| Official machine-readable Bursa announcement API/feed access is not confirmed. Public pages are JS-rendered and Cloudflare-protected. | `/filings`, `/filings/items`, earnings, financials, insider trades, corporate actions, news. | Blocker for reliable ingestion. | Pursue Bursa LINK/announcement feed/licensed data; use public pages only for prototyping with legal review. |
| Stable Bursa announcement identifiers are not yet modeled. | `accession_number`, dedupe, updates, provenance. | High. | Inspect official feed/HTML/PDF metadata; define deterministic fallback id from URL/date/category/ticker/title hash. |
| Financial statement parser is not designed. | All `/financials*`, `/financial-metrics*`, screener, line-items. | High. | Build PDF/table extraction pipeline with source fixtures across sectors; create canonical line-item map and QA checks. |
| `search_results` vs `results` screener envelope discrepancy unresolved. | `POST /financials/search/screener`. | Medium. | Live-test upstream with a harmless request if API key/test account is available; otherwise support both keys temporarily. |
| `institutional_ownership` vs `institutional-ownership` envelope discrepancy unresolved. | `/institutional-ownership`. | Medium. | Live-test; choose one canonical and compatibility alias. |
| Earnings feed mode absent from OpenAPI. | `/earnings`. | Medium. | Implement both company and feed modes; add API compatibility tests for omitted ticker. |
| Docs-only availability endpoints absent from OpenAPI. | Financials/metrics/insider/institutional/KPI list endpoints. | Medium. | Add routes behind feature flags; verify live response envelopes before public release. |
| Bursa sector/subsector source and history need validation. | `/company/facts`, screeners, KPIs, metrics. | Medium. | Capture current Bursa listing directory/classification and add reclassification history ingestion. |
| Historical delisted issuer/security coverage is unknown. | All historical endpoints. | Medium. | Determine whether clone scope is active-only or full historical universe; obtain delisted/change-of-name records. |
| Analyst estimates and surprises are not available from Bursa primary data. | `/earnings` estimate/surprise fields and signals. | Medium. | License consensus estimates or return optional fields only when available. |
| News licensing is not resolved. | `/news`, KPI/source text if using publisher reports. | Medium. | Use Bursa announcements/media first; add licensed publishers for editorial news. |
| Calendar/holiday feed not selected. | `/prices`, filing windows, market snapshot freshness. | Medium. | Ingest Bursa securities trading calendar and special-closure announcements; do not reuse derivatives calendar for equities. |
| Corporate-action adjustment policy not defined. | `/prices`, financial metrics, per-share fields. | Medium. | Define raw vs split/dividend-adjusted OHLCV and shares outstanding methodology. |
| Identity resolution for shareholders/investors is hard. | `/institutional-ownership`, `/insider-trades/names`. | Medium. | Build normalized party table with aliases, nominee-account handling, and confidence scores. |
| MFRS-to-upstream canonical field map incomplete. | Financial statements, line item search, metrics. | High. | Create source-label dictionary and sector-specific mapping tests. |
| Rate limits and quota headers are undocumented upstream. | All endpoints. | Low for docs, medium for implementation. | Choose internal rate-limit policy and expose standard error body; update after upstream/live testing. |

## Recommended Data Model Additions

Even if public responses mimic Financial Datasets, the Bursa clone should maintain these internal tables:

- `securities`: stock_code, short_name, long_name, ISIN, market, sector, subsector, product_type, shariah_flag, listing_status, listing_date, delisting_date.
- `issuer_aliases`: stock_code, vendor_symbol, short_name aliases, old names/codes, source, valid_from/to.
- `announcements`: announcement_id, stock_code, category, subcategory, title, announcement_datetime_myt, source_url, attachment_urls, source_hash, ingestion_version.
- `announcement_items`: announcement_id, item_type, heading, text, source_page/span, parser_confidence.
- `financial_periods`: stock_code, report_period, fiscal_period, period_type, announcement_id, filing_date, audited_flag, currency, scale.
- `normalized_financials`: one table per statement or wide canonical table with source field lineage.
- `as_reported_nodes`: announcement_id/period/statement, parent_id, label, full_label, value, order_index, source bbox/span.
- `segment_disclosures`: stock_code, period, statement, segment_type, label, metric, value, currency, source.
- `corporate_actions`: stock_code/security_code, action_type, ex_date, entitlement_date, payment_date, ratio, cash_amount, currency, source announcement.
- `insider_shareholder_events`: stock_code, party_id, role/directorate, action, shares, price, direct_indirect, holdings_after, event_date, filing_date, source.
- `ownership_positions`: stock_code, party_id, report_period, shares, percent, market_value, source, confidence.
- `price_bars` and `price_snapshots`: stock_code, interval, open/high/low/close/volume, local_time, utc_time, adjustment flags, source/license.
- `macro_rates`: bank_code, measure_code, date, rate, source_url.

## Source Access Notes From This Run

- Direct `curl`/`urllib` requests to several Bursa web pages returned HTTP 403; Playwright browser fetches succeeded for the pages used in this memo. This reinforces that production ingestion should not depend on ad hoc scraping.
- Bursa PDFs could be downloaded through browser automation and parsed with `pypdf` for research. Production PDF extraction needs deterministic tooling, fixtures, parser versioning, and legal review.
- No secrets or API keys were used during this research. All Financial Datasets examples in repo docs use placeholders.

## Reference Links

- Financial Datasets docs index: <https://docs.financialdatasets.ai/llms.txt>
- Financial Datasets OpenAPI JSON: <https://financialdatasets.ai/openapi.json>
- Bursa Company Announcements: <https://www.bursamalaysia.com/market_information/announcements/company_announcement>
- Bursa Company Announcements disclaimer: <https://www.bursamalaysia.com/disclaimer_company_announcement>
- Bursa ISIN/NNA page: <https://www.bursamalaysia.com/trade/trading_resources/equities/isin>
- Bursa Listing Criteria: <https://www.bursamalaysia.com/listing/get_listed/listing_criteria>
- Bursa Listing Directory Main Market: <https://www.bursamalaysia.com/trade/trading_resources/listing_directory/main_market>
- Bursa Listing Directory ACE Market: <https://www.bursamalaysia.com/trade/trading_resources/listing_directory/ace_market>
- Bursa Listing Directory LEAP Market: <https://www.bursamalaysia.com/trade/trading_resources/listing_directory/leap_market>
- Bursa Securities Market: <https://www.bursamalaysia.com/trade/market/securities_market>
- Bursa Information Vendors: <https://www.bursamalaysia.com/market_information/market_data/information_vendors>
- Bursa Announcements Message Structure v1.6 PDF: <https://www.bursamalaysia.com/sites/5bb54be15f36ca0af339077a/assets/5bc04a415f36ca5cab8bf756/Announcements_Message_Structure_v1.6.pdf>
- Bursa BTS2 FIX Specification Market Data PDF: <https://www.bursamalaysia.com/sites/5d809dcf39fba22790cad230/assets/671b56a6cd34aa31b546b75a/BTS2_FIX_Specification_-_Market_Data_v1-19.pdf>
- Bursa Information Services Guidelines PDF: <https://www.bursamalaysia.com/sites/5d809dcf39fba22790cad230/assets/5dc0e9fd39fba246b7c87516/ISLA_Guidelines_V1.1_29102019.pdf>
- data.gov.my Interest Rates dashboard: <https://data.gov.my/dashboard/interest-rates>
- BNM Financial Markets Portal, MYOR/MYOR-i: <https://financialmarkets.bnm.gov.my/about-myor-myor-i>
