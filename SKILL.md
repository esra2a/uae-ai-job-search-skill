---
name: uae-ai-job-search
description: Search Indeed (or a similar job board) for a target role category, filter out low-quality/injected results, and compile a clean, shareable spreadsheet of matching listings. Use when the user asks to search for jobs, scrape job postings, or build a job-search tracker for a specific role and region.
---

# UAE AI Job Search Skill

## When to Use This Skill

Use this skill when the user wants to:
- Search a job board (e.g., Indeed) for a specific role category in a specific country/region
- Build an organized, shareable list of current job postings (title, company, location, link, salary if available)
- Repeat a job search periodically and want a consistent, reusable method

## Prerequisites

- A web-scraping tool capable of fetching and reading live pages (this skill was built around Firecrawl's MCP tools: `firecrawl_scrape`, `firecrawl_search`).
- Any required API key stored in `.env` (never hardcoded in scripts or committed to version control — see this project's `tools/README.md` for the pattern).
- A spreadsheet library for the output step (e.g., `openpyxl` for `.xlsx`).

## Method

### 1. Construct the search URL

Indeed-style search URLs generally follow this pattern:

```
https://<country-domain>/jobs?q=<query>&l=<location>&fromage=<days>&sort=date
```

- `q=` — search keywords (role title, e.g. "AI Automation Engineer"). Try a few closely related keyword variants (e.g. "AI Engineer", "GenAI Engineer", "Automation Specialist") across separate searches — different phrasing surfaces different postings even for the same role category.
- `l=` — location. Start narrow (a single city) and widen to the whole country/region if the initial result count is too low to be useful.
- `fromage=N` — restricts results to postings from the last N days (e.g. `7` for the last week).
- `sort=date` — surfaces the newest postings first.

### 2. Scrape efficiently

- Default to a plain-text/markdown scrape format for reading listings — it's cheaper (in API credits) than structured JSON extraction.
- Only request structured JSON output (schema-based extraction) when you need strict per-field data and are prepared to spend more credits on it.
- **Pagination limitation:** many job boards block deep pagination (e.g. a `start=N` offset) behind a login wall for non-authenticated requests. Workaround: instead of paginating, run additional searches with varied-but-related keywords to surface a different first page of results.

### 3. Filter for data quality

Scraped job-board pages can contain low-quality or anomalous entries — watch for:
- **Suspicious duplicate listings:** the same job appearing twice, where one copy links through a listing-ID pattern that looks non-random (e.g. sequential or patterned hex digits) instead of the board's normal randomized ID format. Treat these as unreliable and exclude them rather than including them in the final results.
- **Off-topic or mis-scoped results:** postings that matched the keyword but are the wrong seniority, wrong location, or a different role family — tag these as a lower-confidence match rather than silently dropping or silently including them.
- **True duplicates across searches:** the same real posting found again in a later keyword search — deduplicate by title + company + location.

### 4. Triage relevance

For each retained listing, classify it (e.g., "Core match" vs. "Adjacent match") against the target role rather than treating every result as equally relevant. This keeps the final list honest about fit instead of padding it with loosely-related postings.

### 5. Compile the output spreadsheet

Build a spreadsheet with columns such as:

| # | Job Title | Company | Location | Work Mode | Salary | Match | Key Details | Job Link | Source | Date Posted | Date Collected |

Formatting suggestions (via `openpyxl` or equivalent):
- Bold, colored header row with wrapped text
- Frozen header row (`freeze_panes`) and an autofilter across the full data range
- Hyperlink the Job Link column directly to each posting
- Conditional fill color on the Match column (e.g., green for Core, yellow for Adjacent) for quick visual scanning
- Reasonable column widths per field (short numeric/category columns narrow, detail/description columns wide)

## Notes

- Be transparent with the user about result volume: if a narrow search returns far fewer results than hoped for, say so plainly and offer to widen scope (broader location, more keyword variants) rather than padding the list with irrelevant postings to hit a target count.
- Keep any output spreadsheet free of scraped anomalies — it's better to under-include with a note than to silently ship suspicious data.
