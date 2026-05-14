# RAHUL-41 Bursa Malaysia Company Announcements Retrieval Research

Prepared 2026-05-14. This is the scoped research and design memo for retrieving Bursa Malaysia company announcements for Malaysia/Bursa coverage in a Financial Datasets-compatible API.

## Summary

The Bursa Malaysia company announcements page is backed by a JSON endpoint that can enumerate announcement rows:

```text
GET https://www.bursamalaysia.com/api/v1/announcements/search
```

The endpoint is a real web-interface API used by the public page, but it is not documented as a stable public API. Direct CLI requests and automated browser requests in this run received Cloudflare security challenges from both `www.bursamalaysia.com` and `disclosure.bursamalaysia.com`. For production, the best programmatic path is therefore **not ad hoc scraping of the public website**. The safer path is to pursue Bursa's official or semi-official announcement access products: Bursa Malaysia's Company Announcement Gateway (CAG), a Bursa-provided URL, or an Authorized Information Vendor under the Information Services Licence Agreement (ISLA).

If legal/commercial approval permits a public-site prototype, the observed web API appears sufficient to enumerate the currently exposed web archive from **01 Apr 1999 through 14 May 2026** when explicit `dd/mm/yyyy` date filters are supplied. A naive unfiltered crawl is not complete: unfiltered pagination stopped at 03 Jan 2000 during this run, while date-filtered calls returned 28,035 records for 1999. No records before 01 Apr 1999 were observed from the interface.

Recommended ingestion shape:

1. Use official CAG/ISLA/vendor access for production; use the public endpoint only as a prototype fallback after legal review.
2. Backfill by date partitions, not by unfiltered pages.
3. Use `per_page=50`, traverse `page=1..ceil(recordsTotal / 50)`, and split dense partitions further.
4. Parse list rows for `ann_id`, stock code, issuer name, date, title, amended marker, and source URLs.
5. Fetch each detail page or disclosure iframe to recover `Reference Number`, category-specific structured fields, amendment links, and attachment metadata.
6. Download attachments through a separate rate-limited queue with content hashes, object-storage keys, retries, and parser versioning.
7. Deduplicate primarily by Bursa `ann_id` plus disclosure `Reference Number`; keep attachment ids and hashes separately.
8. Incremental sync with a rolling lookback window because amendments and late corrections are visible in the UI.

## Current Repository Baseline

The current repository source of truth contains the Financial Datasets/Bursa baseline under `docs/specs/rahul-39-*.md`. No `RAHUL-40`-named document was present in the fetched `master` checkout, so this memo builds on the merged baseline files that are available now:

- `docs/specs/rahul-39-financialdatasets-api-inventory.md`
- `docs/specs/rahul-39-bursa-equivalence-map.md`
- `docs/specs/rahul-39-bursa-data-sources-gaps.md`
- `docs/specs/rahul-39-next-implementation-tickets.md`

The most relevant existing decisions are:

- Preserve Financial Datasets paths and envelopes first.
- Use Bursa-native identifiers internally, including stock code, market, sector, announcement id, attachment id, and source URL.
- Map Bursa company announcements to `/filings`, `/filings/items`, `/filings/types`, `/filings/tickers`, earnings, financials, insider/shareholder events, corporate actions, and selected news-like disclosure views.
- Treat production rights for Bursa announcements and non-display use as a gating decision.

## Source Interface

| Item | Observation |
| --- | --- |
| Public page | <https://www.bursamalaysia.com/market_information/announcements/company_announcement> |
| Detail page pattern | `https://www.bursamalaysia.com/market_information/announcements/company_announcement/announcement_details?ann_id=<ANN_ID>` |
| Disclosure iframe pattern | `https://disclosure.bursamalaysia.com/FileAccess/viewHtml?e=<ANN_ID>#<detail-url>` |
| Attachment pattern | `https://disclosure.bursamalaysia.com/FileAccess/apbursaweb/download?id=<ATTACHMENT_ID>&name=<ATTACHMENT_TYPE>` |
| Date accessed | 2026-05-14 UTC. Bursa announcement dates should be treated as `Asia/Kuala_Lumpur`. |

The company-announcement search form is `method="GET"` and serializes these fields into the announcement API request:

| UI field | Query param | Observed values / notes |
| --- | --- | --- |
| Announcement type | `ann_type` | Hidden input with value `company`. Invalid values returned zero rows in probes. |
| Company | `company` | Select contains 3,861 options in the HTML snapshot. Values are stock/security codes, including alphanumeric suffixes such as `0823EA`, so treat as strings. |
| Keyword | `keyword` | Text input for announcement title. UI says minimum 5 characters, but `keyword=oil` returned results, indicating at least part of that validation is client-side only. |
| Start date | `dt_ht` | Must be `dd/mm/yyyy`; ISO dates were ignored/fell back to broad results. |
| End date | `dt_lt` | Must be `dd/mm/yyyy`; reversed dates returned zero rows. |
| Category | `cat` | Select contains 26 real categories plus default. Values can be comma-delimited internal codes such as `FA,FRCO`. |
| Subcategory | `sub_type` | Populated client-side from category option data attributes. |
| Market | `mkt` | `MAIN-MKT`, `ACE-MKT`, `LEAP-MKT`, `WARRANTS`, `ETF`, `BOND&LOAN`. |
| Sector | `sec` | Populated from market option `data-sector-val`. |
| Subsector | `subsec` | Populated from market/sector option metadata. |

## Observed Network/API Surface

| Endpoint | Method | Purpose | Parameters | Response/schema | Auth/session/CSRF observations |
| --- | --- | --- | --- | --- | --- |
| `/market_information/announcements/company_announcement` | GET | Public UI and embedded filter options. | Query string mirrors form values on navigation. | HTML page with form, table shell, category/company options, linked JS bundles. | Direct requests received Cloudflare challenge in this environment. Page includes CSRF meta tags, but the list API is a GET and the inspected JS does not attach a CSRF header for it. |
| `/api/v1/announcements/search` | GET | Server-side DataTables list endpoint. | `ann_type`, `page`, `per_page`, plus optional `company`, `keyword`, `dt_ht`, `dt_lt`, `cat`, `sub_type`, `mkt`, `sec`, `subsec`. | JSON object with `recordsTotal`, `recordsFiltered`, `category_message`, and `data` array. Each `data` row has four HTML-string columns. | Same-origin JS endpoint. Direct access received Cloudflare challenge. No API key was observed. No documented public contract was found. |
| `/market_information/announcements/company_announcement/announcement_details?ann_id=<ANN_ID>` | GET | Bursa landing page for one announcement. | `ann_id`. | HTML page whose main content is an iframe pointing at `disclosure.bursamalaysia.com/FileAccess/viewHtml?e=<ANN_ID>`. | Direct access received Cloudflare challenge; iframe is cross-domain. |
| `https://disclosure.bursamalaysia.com/FileAccess/viewHtml?e=<ANN_ID>` | GET | Actual disclosure HTML. | `e=<ANN_ID>`. | Category-specific HTML disclosure with `Company Name`, `Stock Name`, `Date Announced`, `Category`, `Reference Number`, optional amended reference, structured fields, and attachments. | Direct access received Cloudflare challenge. No API key was observed. |
| `https://disclosure.bursamalaysia.com/FileAccess/apbursaweb/download?id=<ATTACHMENT_ID>&name=<ATTACHMENT_TYPE>` | GET | PDF/attachment download. | `id`, `name`. | Binary attachment in normal browser use; rendered text was visible through the research fallback for PDFs. | Direct access received Cloudflare challenge. Use an attachment queue, not inline list crawling. |
| `/misc/announcement_notification.json?_=<epoch>` | GET | Browser notification polling for subscribed/followed announcements. | Cache-busting `_` timestamp. | JSON notification structure consumed from localStorage subscriptions. | Not a primary ingestion surface; only triggered when browser notification state exists. |

### List endpoint request example

```http
GET /api/v1/announcements/search?ann_type=company&page=1&per_page=20&dt_ht=01%2F01%2F2026&dt_lt=14%2F05%2F2026 HTTP/1.1
Host: www.bursamalaysia.com
Accept: application/json
```

Abridged response shape observed from the list endpoint:

```json
{
  "recordsTotal": 34081,
  "recordsFiltered": 34081,
  "category_message": "",
  "data": [
    [
      1,
      "<div class='d-lg-none'>14 May<br/>2026</div><div class='d-lg-inline-block d-none'>14 May 2026</div>",
      "<a href='/trade/trading_resources/listing_directory/company-profile?stock_code=0823EA' target=_blank>PRINCIPAL FTSE CHINA 50 ETF</a>",
      "<a href='/market_information/announcements/company_announcement/announcement_details?ann_id=3666025' target=_blank>NET ASSET VALUE / INDICATIVE OPTIMUM PORTFOLIO VALUE</a><p><span class='text-danger'>(Amended Announcement)</span></p>"
    ]
  ]
}
```

The `data` rows are not normalized JSON objects. They are four-column DataTables rows containing HTML fragments:

1. Row number.
2. Announcement date HTML.
3. Issuer/company link HTML; stock code is in `company-profile?stock_code=<CODE>`.
4. Title/detail link HTML; announcement id is in `announcement_details?ann_id=<ANN_ID>`. Additional paragraph text and `(Amended Announcement)` may appear in this column.

### Client-side DataTables behavior

The inspected JavaScript initializes `.ann-table` with `serverSide: true`, `pageLength: 20`, and `lengthMenu` values `10`, `20`, and `50`. Its AJAX URL is:

```javascript
window.location.origin + "/api/v1/announcements/search"
```

The AJAX data function serializes `.announcement-filter`, then appends:

```text
per_page=<selected length>
page=<DataTables current page + 1>
```

No request method is specified in the DataTables block, so jQuery uses GET.

## Filters and Pagination Model

### Page size and paging

Observed page-size behavior:

| Probe | Observed rows returned | Notes |
| --- | ---: | --- |
| `per_page=20` | 20 | Default. |
| `per_page=50` | 50 | Maximum exposed by the UI and highest safe value observed. |
| `per_page=100` | 20 | Server fell back to 20 rows. |
| `per_page=1000` | 20 | Server fell back to 20 rows. |

Recommendation: use `per_page=50` for backfills and treat any response with fewer rows before the computed final page as a retryable anomaly unless `recordsTotal` says the partition is exhausted.

### Date filters and historical coverage

Observed date-filter behavior:

| Query | Observed count / result | Inference |
| --- | ---: | --- |
| No date filters, first page | ~2.064M records; latest rows dated 14 May 2026 | Current archive summary count. Counts drifted by a few records during the run as new announcements arrived. |
| No date filters, expected last page at 20/page | 3 rows dated 03 Jan 2000 | Naive unfiltered crawl does not reach all 1999 records. |
| `dt_ht=01/04/1999&dt_lt=14/05/2026` | 2,091,918 records | Date-filtered broad query includes more records than unfiltered query. |
| `dt_ht=01/01/1990&dt_lt=31/12/1999` | 28,035 records; first page 31 Dec 1999 | 1999 records are reachable with date filters. |
| `dt_ht=01/01/1998&dt_lt=31/03/1999` | 0 records in the observed probe | No pre-01 Apr 1999 records were found. |
| `dt_ht=01/01/2000&dt_lt=31/12/2000` | 60,791 records | Year-sized partitions can be very large. |
| `dt_ht=01/01/2026&dt_lt=14/05/2026` | 34,081 records | Recent data volume is high enough to justify daily partitioning for current years. |
| `dt_ht=2026-01-01&dt_lt=2026-05-14` | Fell back to broad/unfiltered result count | Date parser expects `dd/mm/yyyy`; ISO input should not be used. |
| Reversed dates | 0 records | Client should validate before requesting. |

Completeness conclusion: **the observed public interface can enumerate the public web archive back to 01 Apr 1999 only when date filters are used**. It did not expose earlier records. Any requirement for older records or a contractual guarantee of completeness requires licensed Bursa/vendor access.

### Category and subcategory filters

The category select contains 26 real categories. The most ingestion-relevant categories are:

| Category | `cat` value | Subcategory examples |
| --- | --- | --- |
| Annual Audited Account | `AA,AACO` | none embedded |
| Annual Report | `AR,ARCO` | none embedded |
| Changes in Shareholdings | `SH,CHSH` | director interest, substantial shareholder interest, notice of interest, notice of ceasing |
| Dealings in Listed Securities | `DRCO` | dealings during/outside closed period, intention to deal |
| Entitlements | `EA,ENCO` | cash dividend `DVCA`, rights issue, bonus issue, subdivision, share consolidation, DRIP, redemption |
| Financial Results | `FA,FRCO` | `FA1,QRFR`, `FA2,CFYE` |
| General Announcement | `GA,GACO` | others, NAV/indicative portfolio value, transactions, litigation, PN17/GN3, public shareholding spread, winding up, etc. |
| General Meetings | `GM,MECO` | notice and outcome of meeting |
| IPO Announcement/Admission to LEAP Market Announcement | `IO,IPOA` | IPO/admission, prospectus document, information memorandum, timetable |
| Shares Buy Back | `SB,SBBA` | immediate buyback, treasury share changes, resale/cancellation |
| Take-over Offer | `TECO` | mandatory and voluntary take-over offer |
| Unusual Market Activity | `UMA,UMCO` | UMA query and reply |

Other observed categories: Additional Listing Announcement/Subdivision of Shares, Change of Corporate Information, Circular/Notice to Shareholders, Delisting of Securities, Disclosure On Quarterly Production, Expiry/Maturity/Termination of Securities, Important Relevant Dates for Renounceable Rights, Investor Alert, Listing Circulars, Listing Information and Profile, Prospectus, Reply to Query, Special Announcements, and Transfer of Listing.

Observed filter probe examples:

| Query | Observed count | Notes |
| --- | ---: | --- |
| `cat=FA,FRCO` | 106,012 | All Financial Results category rows in the exposed archive. |
| `cat=EA,ENCO&sub_type=DVCA` | 5,778 | Cash dividend/income distribution rows in observed archive. |
| `company=1818` | 3,366 | All Bursa Malaysia Berhad rows in observed archive. |
| `company=1818&dt_ht=01/01/2026&dt_lt=14/05/2026` | 80 | Useful for issuer-focused verification. |
| `keyword=quarterly` | 104,447 | Keyword search against titles/descriptions. |
| `keyword=oil` | 2,590 | Endpoint accepted a 3-character keyword despite UI guidance. |
| `mkt=MAIN-MKT&sec=TECHNOLOGY` | 102,043 | Market/sector partitioning works, but should not be needed for normal date backfills unless daily partitions are too dense. |

### Sorting

The DataTables configuration has `ordering: false`. A probe adding `sort_by=annDt&sort_dir=asc` still returned newest rows first. Treat the endpoint as descending by Bursa's default ordering and do not rely on server-side sorting parameters.

## Detail Pages, Attachments, and Identifiers

List rows expose enough to start normalization, but detail pages are required for stable reference numbers, structured fields, attachment metadata, and amendment lineage.

Observed detail examples:

| `ann_id` | List row | Detail/disclosure observations |
| --- | --- | --- |
| `3666010` | ALAM MARITIM RESOURCES BERHAD, Financial Results, 14 May 2026 | Disclosure page had `Company Name`, `Stock Name=ALAM`, `Date Announced=14 May 2026`, `Category=Financial Results`, `Reference Number=FRA-14052026-00008`, structured financial-result fields, and attachment `id=237913&name=EA_FR_ATTACHMENTS` with filename `AMRB ANNOUNCEMENT Q3-2026.pdf` and displayed size `235.5 kB`. |
| `3665980` | BURSA MALAYSIA BERHAD, substantial-shareholder change, 14 May 2026 | Disclosure page had `Reference Number=CS2-14052026-00036`, category `Change in the Interest of Substantial Shareholder Pursuant to Section 138 of CA 2016`, no attachment observed, and structured shareholder fields. |
| `3666025` | PRINCIPAL FTSE CHINA 50 ETF, amended NAV announcement, 14 May 2026 | List row had `(Amended Announcement)`. Disclosure page had `Reference Number=GA1-14052026-00073`, amendment link to earlier reference `GA1-14052026-00013`, and attachment `id=169293&name=EA_GA_ATTACHMENTS` with filename `MY0030 - Reformatted NAV Report.PDF` and displayed size `15.3 kB`. |

Attachment links are relative in disclosure HTML:

```html
<a href="/FileAccess/apbursaweb/download?id=237913&name=EA_FR_ATTACHMENTS">
  AMRB ANNOUNCEMENT Q3-2026.pdf
</a>
```

Production attachment storage should persist both the source URL and immutable content metadata. Do not rely on filename alone; use attachment id, announcement id, and content hash.

## Official or Semi-Official API Assessment

### Public web API

`/api/v1/announcements/search` is a semi-official internal API used by the Bursa public web interface. It is useful for discovery because it exposes server-side pagination and counts, but it is not documented as a public, stable, licensed machine interface. Observed risks:

- Direct access was Cloudflare-protected in this run.
- Response rows are HTML fragments, not a formal normalized schema.
- Broad unfiltered pagination missed 1999 rows that date-filtered calls returned.
- Counts can drift while current-day announcements arrive.
- Detail and attachment access is on a separate Cloudflare-protected disclosure host.
- No rate-limit headers, quotas, changelog, schema contract, or service-level terms were found.

### Bursa licensed access

Bursa Information Services Guidelines describe company announcement subscription options. The document states that company announcements are published on the Bursa website and that options for all PLC announcements include Bursa Malaysia's **Company Announcement Gateway (CAG)**, a **URL provided by Bursa**, or **Authorised Information Vendors** under ISLA. It also requires attribution, prohibits editing/modifying/translating beyond layout/headlines, and prohibits using announcements to create a real-time database or non-display use without prior written permission.

Recommendation: treat CAG/Bursa-provided URL/authorized vendor access as the production source and treat the public web endpoint as a validation/prototype reference only.

## Historical Backfill Strategy

### Access decision gate

Before implementation, decide and document one of:

1. **Licensed CAG/ISLA/vendor ingestion** - recommended for production.
2. **Public website prototype** - only after legal approval; conservative traffic; no technical-control bypassing.
3. **Hybrid** - use licensed feed as source of truth and public website as spot-check/reference.

### Public-site prototype algorithm

If approved, implement this robust crawl rather than unfiltered page traversal:

1. **Discover bounds.** Use `dt_ht=01/04/1999&dt_lt=<today>` as the current observed lower bound. Confirm regularly whether earlier records become available through a licensed source.
2. **Partition by date.** Start with daily windows for current/high-volume years. Weekly or monthly windows are acceptable only when a preflight `recordsTotal` is well below a safe threshold.
3. **Preflight each partition.** Call page 1 with `per_page=50` and record `recordsTotal`, `recordsFiltered`, request URL, retrieved timestamp, and raw response hash.
4. **Traverse pages.** For page `1..ceil(recordsTotal / 50)`, fetch the list endpoint and parse all rows. Stop only after the expected count is reached or the first empty page after the expected final page.
5. **Split dense or unstable partitions.** If a partition has too many pages, count changes materially during traversal, or returns early empty pages, split the date range. If same-day volume is still too high, split by category before splitting by company.
6. **Persist raw and normalized data.** Keep raw list response JSON and normalized rows. Store parsing errors separately.
7. **Fetch details.** For each unique `ann_id`, fetch the Bursa landing page and/or directly the disclosure iframe URL. Persist raw detail HTML and parse reference number, category-specific fields, attachment links, and amendment metadata.
8. **Queue attachments.** Download attachments separately with lower concurrency and content hashing. Attachment failures must not block list/detail progress.
9. **Checkpoint.** Store partition status, page status, count seen, retry count, and source hashes. A partition is complete only when expected rows, unique `ann_id`s, detail fetch status, and attachment queue status are accounted for.
10. **Audit completeness.** Reconcile per-day counts against stored row counts and rerun suspicious partitions.

### Deduplication keys

Preferred keys:

- Announcement: `source_exchange='BURSA'` + `source_announcement_id=<ann_id>`.
- Secondary announcement identity: disclosure `Reference Number` such as `FRA-14052026-00008` or `GA1-14052026-00073`.
- Attachment: `source_attachment_id=<download id>` + `announcement_id` + `source_name_param` + normalized filename.
- Raw payload version: SHA-256 over canonicalized raw list row/detail HTML/attachment bytes.

Fallback key if an old row lacks `ann_id`:

```text
sha256(source_detail_url | announcement_date | issuer_stock_code | normalized_title)
```

### Update/version handling

- Store `first_seen_at`, `last_seen_at`, `retrieved_at`, and raw hashes.
- Mark list-row amended indicators (`(Amended Announcement)`) separately from detail-level amendment links.
- Link amended rows via `amends_source_reference_number` or `supersedes_source_announcement_id` when detail text exposes an earlier reference.
- Do not overwrite prior raw payloads when hashes change; create a version record.

## Incremental Sync Strategy

Recommended schedule depends on source license and allowed latency. For a licensed feed, poll more frequently during Bursa announcement hours. For a public-site prototype, use slow, conservative polling and backoff.

Incremental algorithm:

1. Poll `dt_ht=<today - lookback>&dt_lt=<today>` with `per_page=50`.
2. Use a rolling lookback of at least 7 days; 30 days is safer while amendment behavior is being learned.
3. Parse rows and upsert by `ann_id`.
4. Re-fetch details when any list-row hash changes, an amended marker appears, or the last detail fetch failed.
5. Re-download attachments only when new attachment ids appear or metadata/content hashes changed.
6. Maintain a high-water mark by `announcement_date` and `ann_id`, but never rely on it alone because the endpoint is date-only and descending.
7. Run a daily reconciliation for the last 30 days and a weekly reconciliation for the last year.

## Proposed Data Model

### `announcements`

| Field | Purpose |
| --- | --- |
| `id` | Internal UUID or database id. |
| `source` | `bursa_malaysia`. |
| `source_market` | `MY`. |
| `source_exchange` | `BURSA`. |
| `source_announcement_id` | Bursa `ann_id` from list/detail URLs. |
| `source_reference_number` | Disclosure reference number such as `FRA-14052026-00008`. |
| `source_detail_url` | Bursa detail URL on `www.bursamalaysia.com`. |
| `source_disclosure_url` | Disclosure iframe URL on `disclosure.bursamalaysia.com`. |
| `source_list_url` | List API URL/partition that discovered the row. |
| `issuer_id` | Internal issuer id. |
| `issuer_name` | Issuer/company display name from list/detail. |
| `issuer_stock_code` | Bursa stock/security code from list link. |
| `stock_name` | Disclosure `Stock Name`, if available. |
| `announcement_title` | Normalized title from list/detail. |
| `announcement_description` | Additional paragraph/body summary where available. |
| `announcement_category` | Category label from detail. |
| `announcement_category_code` | `cat` code when known. |
| `announcement_subcategory` | Subcategory label when available. |
| `announcement_subcategory_code` | `sub_type` code when known. |
| `announcement_date` | Bursa announcement date. |
| `announcement_datetime` | If time becomes available from licensed feed/detail; otherwise null with date retained. |
| `timezone` | `Asia/Kuala_Lumpur`. |
| `published_at` | Normalized publication timestamp when available. |
| `updated_at` | Source update timestamp when available. |
| `retrieved_at` | Ingestion timestamp. |
| `language` | Likely `en` for observed pages; keep nullable. |
| `status` | Active, amended, withdrawn, parser_failed, etc. |
| `is_amended` | Boolean from list/detail markers. |
| `amends_source_reference_number` | Earlier reference number for amended announcements. |
| `supersedes_source_announcement_id` | Link when resolvable. |
| `raw_list_payload_hash` | Canonical hash of list row/raw JSON. |
| `raw_detail_payload_hash` | Canonical hash of detail HTML. |
| `content_hash` | Hash over normalized content fields. |
| `parser_version` | Parser version used for normalized fields. |

### `issuers` / `listed_companies`

| Field | Purpose |
| --- | --- |
| `source_issuer_id` | Bursa stock/security code or licensed issuer id. |
| `stock_code` | Bursa code as string. |
| `stock_name` | Bursa short stock name. |
| `short_name` | Short display name. |
| `legal_name` | Full legal name from listing/profile/detail. |
| `market` | Main, ACE, LEAP, ETF, warrants, bond/loan. |
| `board` | Listing board where applicable. |
| `sector` | Bursa sector. |
| `subsector` | Bursa subsector/classification. |
| `security_type` | Share, ETF, warrant, bond, loan stock, etc. |
| `isin` | If available from Bursa/security master. |
| `source_profile_url` | Bursa company profile URL. |
| `active_status` | Active, suspended, delisted, historical-only. |
| `first_seen_at` | First ingestion timestamp. |
| `last_seen_at` | Most recent observation timestamp. |

### `announcement_attachments`

| Field | Purpose |
| --- | --- |
| `source_attachment_id` | `id` query parameter from `download?id=...`. |
| `announcement_id` | Internal announcement FK. |
| `source_name_param` | `name` query parameter such as `EA_FR_ATTACHMENTS`. |
| `title` | Attachment display title if separate from filename. |
| `filename` | Displayed/downloaded filename. |
| `mime_type` | Detected MIME type. |
| `file_extension` | Normalized extension. |
| `source_url` | Absolute disclosure download URL. |
| `storage_key` | Object-storage path. |
| `content_length` | HTTP content length, if available. |
| `source_reported_size` | Displayed size such as `235.5 kB`. |
| `sha256` | Hash of downloaded bytes. |
| `downloaded_at` | Download completion timestamp. |
| `download_status` | Pending, success, retrying, failed, blocked. |
| `failure_reason` | Error or HTTP status summary. |
| `text_extraction_status` | Pending, success, failed, not_supported. |
| `parser_version` | Attachment/text parser version. |

Recommended storage key:

```text
bursa/company_announcements/{ann_id}/attachments/{attachment_id}/{sha256_prefix}-{normalized_filename}
```

### `announcement_categories`

| Field | Purpose |
| --- | --- |
| `source_category_code` | `cat` value or code component. |
| `name` | Category label. |
| `parent_category` | Optional grouping. |
| `source_subcategory_code` | `sub_type` value. |
| `subcategory_name` | Subcategory label. |
| `description` | Optional notes/source text. |
| `first_seen_at` | First observation timestamp. |
| `last_seen_at` | Most recent observation timestamp. |

### `announcement_versions`

Use a separate version table or append-only history with:

- `announcement_id`
- `source_announcement_id`
- `source_reference_number`
- `observed_at`
- `raw_list_payload_hash`
- `raw_detail_payload_hash`
- `attachment_manifest_hash`
- `is_current`
- `change_reason` (`new`, `amended`, `detail_changed`, `attachment_changed`, `parser_changed`)

## Mapping to Financial Datasets-Compatible API Coverage

| Financial Datasets-compatible area | Bursa announcement source | Mapping recommendation |
| --- | --- | --- |
| `/filings` | All company announcement rows. | Use Bursa stock code as `ticker`; use `ann_id` or `Reference Number` as `accession_number`; preserve Bursa category as `filing_type`; include source URLs and attachment URLs. |
| `/filings/items` | Disclosure HTML sections and attachment text extraction. | Bursa has no SEC item taxonomy. Use category/subcategory and parsed section labels as item types while preserving original labels. |
| `/filings/types` and `/filings/items/types` | Category and subcategory options embedded in the page. | Return Bursa-native filing types, optionally with compatibility aliases such as Financial Results -> 10-Q-like, Annual Report -> 10-K-like, but never hide the source category. |
| `/filings/tickers` | Issuers discovered from list rows and listing directory. | Return Bursa stock/security codes and maintain alias support separately. |
| `/earnings` | Financial Results category (`FA,FRCO`) and annual financial-result attachments. | Use detail structured fields for report period, quarter, audit status, currency, revenue/profit/EPS/dividend where present. Estimates/surprises require separate vendor data. |
| `/financials/*` and `/financials/*/as-reported` | Financial Results, Annual Audited Accounts, Annual Reports, and PDF/XHTML attachments. | Build parsers with source spans and MFRS-to-canonical mapping; keep raw/as-reported trees. |
| `/insider-trades` | Director interest, substantial shareholder, and dealings categories/subcategories. | Map Bursa actions and Companies Act sections to internal party/holding events; do not claim SEC Form 4 equivalence. |
| `/institutional-ownership` | Substantial shareholder notices plus annual report shareholder tables. | Best-effort Malaysia ownership view; store party aliases and confidence. |
| Corporate-action dependencies | Entitlements, shares buyback, additional listings, rights, subdivisions, warrants, takeovers. | Internal corporate-action model for price adjustment, shares outstanding, dividends, and security lifecycle. |
| `/news` | General announcements and Bursa media/issuer disclosures. | Treat as official disclosures, not editorial news. Label source and licensing constraints. |

## Failure Modes and Protections

| Failure mode | Observed or expected behavior | Handling |
| --- | --- | --- |
| Cloudflare challenge / bot protection | Direct CLI and automated browser requests received 403 challenge pages on Bursa, disclosure detail, and attachment URLs. | Do not bypass. Prefer licensed access. If public prototype is approved, use a normal browser session only within terms, low rate, and clear user agent/contact. Back off on 403/429. |
| Date parser fallback | ISO dates were ignored/fell back to broad results. | Validate `dd/mm/yyyy` before request and verify response date range. |
| Naive unfiltered incompleteness | Unfiltered last page stopped at 03 Jan 2000; date-filtered 1999 records were reachable. | Always date-partition. Audit earliest/latest date per partition. |
| Count drift | Total counts changed by a few rows during live probing. | Snapshot partition counts; rerun current-day partitions; use rolling lookback. |
| Page-size fallback | `per_page=100` and `per_page=1000` returned only 20 rows. | Use 50 maximum and verify returned row count. |
| HTML-fragment schema drift | List rows are arrays with HTML strings. | Centralize parser, fixture test all selectors, monitor parse-failure rate. |
| Detail/attachment host changes | Details and attachments live on `disclosure.bursamalaysia.com`. | Store landing URL and disclosure URL; test both; monitor iframe selector changes. |
| Attachment failures | PDF download can be independently blocked or fail. | Queue retries separately; keep announcement records even if attachment status is pending/failed. |
| Amended announcements | Amended marker appears in list; detail can point to earlier reference. | Store append-only versions and amendment links. |
| Legal/licensing block | Terms restrict copying/storing/downloading/redistribution without consent beyond personal/non-commercial uses. | Make source-access approval a P0 implementation task. |

## Legal, Terms, Robots, and Safer Defaults

Observed `robots.txt` did not disallow `/market_information/announcements/company_announcement`, but it did disallow `/locomotive/*` and `/market/listed-companies/company-announcements/subscribe/*`. Robots is not sufficient permission for production ingestion.

Terms and disclaimer observations from Bursa pages:

- Website contents include information/data/materials and are protected by IP laws.
- Users may view and print one copy solely for personal/non-commercial use, research, or information.
- Copying, storing, downloading, transmitting, publishing, reproducing, distributing, selling, or licensing contents to third parties without prior written consent is prohibited outside the stated allowance.
- Announcements are provided as-is; Bursa acts as a passive conduit and does not verify or endorse issuer-submitted content.
- ISLA guidance identifies CAG, Bursa-provided URLs, and authorized vendors as subscription options for all PLC company announcements, and requires attribution and prior written permission for real-time database or non-display uses.

Safer operating defaults if a prototype is approved:

- Use licensed access where possible; do not attempt to defeat Cloudflare or other technical controls.
- Identify the client with a descriptive User-Agent and contact email.
- Limit list/detail requests to roughly 1 request/second or slower; lower concurrency for attachments.
- Use exponential backoff with jitter for 403, 429, 5xx, timeouts, and malformed responses.
- Cache immutable details and attachments; do not repeatedly download the same PDF.
- Keep legal/source-access review as a deployment gate before commercial or customer-facing use.

## Recommended Follow-Up Implementation Tickets

### 1. Source-access and legal decision

- **Goal:** Decide whether production will use Bursa CAG, a Bursa-provided URL, an Authorized Information Vendor, or a public-site prototype.
- **Deliverables:** Written access matrix covering completeness, latency, allowed uses, redistribution, non-display rights, rate limits, cost, and contacts.
- **Acceptance criteria:** Engineering has an approved source and traffic policy before any production crawler is scheduled.

### 2. Discovery client

- **Goal:** Build a small typed client that reproduces the Bursa announcement search and detail requests for approved sources.
- **Scope:** Request parameter builder, `dd/mm/yyyy` validation, page-size enforcement, response schema checks, Cloudflare/403 detection, raw payload persistence.
- **Acceptance criteria:** Fixture tests parse list rows for `ann_id`, date, issuer name, stock code, title, amended marker, and source URLs.

### 3. Historical backfill worker

- **Goal:** Retrieve all reachable announcement rows from 01 Apr 1999 forward or from the licensed source's true lower bound.
- **Scope:** Date partitioning, page traversal, adaptive partition splitting, checkpointing, retries, count reconciliation, raw response storage.
- **Acceptance criteria:** Backfill can resume from checkpoints and prove partition completeness by expected counts and unique ids.

### 4. Detail/disclosure parser

- **Goal:** Parse Bursa detail landing pages and disclosure iframe HTML.
- **Scope:** `Reference Number`, company/stock name, date, category, structured category-specific fields, amended-reference links, attachment manifests.
- **Acceptance criteria:** Fixtures cover Financial Results, Shareholdings, General Announcement, Entitlements, Annual Report, and Shares Buy Back.

### 5. Attachment downloader and storage

- **Goal:** Download and store PDFs/attachments idempotently.
- **Scope:** Queueing, low concurrency, retries, MIME detection, content length, SHA-256, object-storage key design, text extraction status.
- **Acceptance criteria:** Duplicate attachment ids are not re-downloaded; failed attachments are retryable without blocking announcement ingestion.

### 6. Storage and schema

- **Goal:** Add normalized tables/models for announcements, issuers, categories, attachments, and versions.
- **Scope:** Source ids, hashes, timestamps, status fields, amendment/version history, parser versions, raw payload pointers.
- **Acceptance criteria:** Upserts are idempotent; schema supports changed details and new attachments without data loss.

### 7. Incremental sync worker

- **Goal:** Keep recent announcements current after backfill.
- **Scope:** Recent date polling, rolling lookback, changed-hash detection, amended announcement reconciliation, daily count audits.
- **Acceptance criteria:** New and amended rows are detected in repeated fixture/live-like runs without duplicates.

### 8. Financial Datasets-compatible API endpoints

- **Goal:** Expose Bursa announcements through planned Financial Datasets-compatible paths.
- **Scope:** `/filings`, `/filings/items`, `/filings/tickers`, `/filings/types`, earnings feed for Financial Results, internal corporate-action and ownership feeds.
- **Acceptance criteria:** Response envelopes follow `rahul-39-financialdatasets-api-inventory.md`; Bursa-specific identifiers remain available as metadata.

### 9. Validation tests

- **Goal:** Prevent parser and API regressions.
- **Scope:** Golden fixtures from captured list/detail responses, date partition tests, dedupe/version tests, attachment manifest tests, response-envelope compatibility tests.
- **Acceptance criteria:** CI fails on schema drift, missing ids, invalid date formats, duplicate primary keys, or broken Financial Datasets envelopes.

### 10. Monitoring and operations

- **Goal:** Detect source drift, completeness gaps, and legal/source-access failures.
- **Scope:** Endpoint status, records/day baselines, partition completeness, parse-failure rate, attachment failures, 403/429/5xx rates, count drift, source hash changes.
- **Acceptance criteria:** Alerts exist for sustained failures, zero-result current-day anomalies, high parse-failure rates, and license/access misconfiguration.

## Open Questions / Blockers

1. **Production source rights are unresolved.** Bursa terms and ISLA guidance indicate production storage/redistribution/non-display use requires prior permission or licensed access. This is the main blocker for production ingestion.
2. **Complete history before 01 Apr 1999 was not available from the observed public interface.** If older announcements are required, obtain CAG/vendor/historical access or written confirmation of source bounds.
3. **The public web API has no documented stability contract.** It may change URL, parameters, row HTML, Cloudflare policy, or limits without notice.
4. **Announcement times were not exposed in list rows.** Detail pages observed a date, not a precise timestamp. A licensed feed may expose more precise publication times.
5. **Update semantics need more sampling.** Amended announcements expose earlier reference numbers in some details, but the full relationship between amendments, replacements, withdrawals, and duplicate `ann_id`s needs broader fixtures.
6. **Attachment binary access was Cloudflare-protected from direct automation.** Production download design should be validated against the licensed access path rather than assuming public direct downloads will be reliable.
7. **RAHUL-40-specific repo artifact was not present in the current checkout.** This memo cross-references the available merged Financial Datasets/Bursa specs in `docs/specs/rahul-39-*.md`.

## Appendix: Evidence Captured During This Run

Evidence files were captured under `.sigma/tmp/rahul-41` during the run and are not committed. Key observed facts are summarized here so the committed research doc remains self-contained.

### Direct access and bot-protection evidence

Direct requests to the Bursa UI, API, detail pages, terms/disclaimer pages, JavaScript assets, and attachment URLs returned Cloudflare challenge HTML with `Just a moment...` and `cf-mitigated: challenge`-style behavior. Playwright navigation to the company-announcement page also reached a 403 challenge page that stated the website uses a security service to protect against malicious bots. This is why the design recommends licensed access and conservative public-site fallback only with approval.

A live HTTP rendering fallback was used for research visibility when direct access was challenged. That fallback is not recommended as an ingestion dependency.

### Endpoint response examples

Unfiltered first page:

```text
GET /api/v1/announcements/search?ann_type=company&page=1&per_page=20
recordsTotal ~= 2,063,883
first row: 14 May 2026, PRINCIPAL FTSE CHINA 50 ETF, ann_id=3666025, amended NAV announcement
```

Unfiltered last page at 20/page:

```text
recordsTotal = 2,063,883
page = 103195
rows = 3
last observed date = 03 Jan 2000
```

Date-filtered historical query:

```text
GET /api/v1/announcements/search?ann_type=company&dt_ht=01%2F01%2F1990&dt_lt=31%2F12%2F1999&page=1&per_page=20
recordsTotal = 28,035
first page date = 31 Dec 1999
```

Broad date-filtered query:

```text
GET /api/v1/announcements/search?ann_type=company&dt_ht=01%2F04%2F1999&dt_lt=14%2F05%2F2026&page=1&per_page=20
recordsTotal = 2,091,918
```

Filter examples:

```text
cat=FA,FRCO                         -> 106,012 records
cat=EA,ENCO&sub_type=DVCA            -> 5,778 records
company=1818                         -> 3,366 records
company=1818&dt_ht=01/01/2026&dt_lt=14/05/2026 -> 80 records
keyword=quarterly                    -> 104,447 records
keyword=oil                          -> 2,590 records
mkt=MAIN-MKT&sec=TECHNOLOGY          -> 102,043 records
```

### Source links

- Bursa company announcements: <https://www.bursamalaysia.com/market_information/announcements/company_announcement>
- Bursa detail example: <https://www.bursamalaysia.com/market_information/announcements/company_announcement/announcement_details?ann_id=3666010>
- Bursa disclosure iframe example: <https://disclosure.bursamalaysia.com/FileAccess/viewHtml?e=3666010>
- Bursa announcement disclaimer: <https://www.bursamalaysia.com/disclaimer_company_announcement>
- Bursa terms/disclaimer/linking policy: <https://www.bursamalaysia.com/term_conditions_of_use_disclaimer_and_linking_policy>
- Bursa robots.txt: <https://www.bursamalaysia.com/robots.txt>
- Bursa Information Services Guidelines / ISLA reference path from existing RAHUL-39 work: <https://www.bursamalaysia.com/sites/5d809dcf39fba22790cad230/assets/5dc0e9fd39fba246b7c87516/ISLA_Guidelines_V1.1_29102019.pdf>
