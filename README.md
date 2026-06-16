# LinkedIn Job Scraper

A Chrome Extension (Manifest V3) that automates the collection of LinkedIn job postings and exports them as structured JSON data for downstream processing, analysis, or LLM-assisted job application workflows.

## Overview

This project scrapes LinkedIn Jobs search results, extracts detailed job information, and stores the results in a structured format suitable for automation pipelines.

The scraper navigates search results, captures job metadata, expands descriptions, extracts hiring-team information when available, resolves external application links, and exports the collected data as JSON.

## Features

* Scrape LinkedIn Jobs search results automatically
* Extract:

  * Job title
  * Company
  * Location
  * Salary information
  * Date posted
  * Apply type
  * Full job description
  * Company information
  * Hiring team contacts
  * External application URLs
* Automatically paginate through results
* Pause, resume, stop, and download at any time
* Configure a target number of jobs to process (1–500)
* Export as:

  * A single aggregated JSON file
  * Individual JSON files per job
* Retry interrupted downloads automatically
* Preserve partial results when extraction is incomplete
* Maintain detailed run statistics and event logs

---

## Installation

No build step is required.

1. Clone this repository.

2. Open Chrome and navigate to:

   ```
   chrome://extensions
   ```

3. Enable **Developer Mode**.

4. Click **Load Unpacked**.

5. Select the project directory.

6. Log in to LinkedIn.

7. Navigate to a LinkedIn Jobs search results page.

8. Click the extension icon.

9. Open the scraper controls using **Ready to Scrape**.

10. Configure the desired job count.

11. Click **Start**.

### Reloading During Development

Because the extension loads directly from source files:

1. Reload the extension from `chrome://extensions`.
2. Refresh any open LinkedIn tabs.

> **Important:** Reloading the extension invalidates existing content script contexts. Refreshing the LinkedIn page is required after every extension reload.

---

## Architecture

```text
manifest.json       → MV3 configuration and permissions
popup.html/js       → Extension popup UI and controls
popup_state.js      → LinkedIn page detection logic
content_script.js   → Core scraping engine and page interaction
background.js       → Service worker and download handling
scrape_session.js   → Runtime state management
in_page_controls.js → In-page modal and UI components
json_export.js      → Export formatting and serialization
```

### Message Flow

```text
popup.js
  → inject content_script.js
  → send start command

content_script.js
  → scrape jobs
  → emit progress updates
  → accumulate export buffer

background.js
  → receive export requests
  → generate downloads
  → retry interrupted exports
```

---

## How It Works

1. Open a LinkedIn Jobs search results page.
2. Start a scraping session from the extension UI.
3. The scraper iterates through job cards in the results panel.
4. Each card is opened and validated to ensure fresh content is loaded.
5. Job details are extracted from both the results list and details panel.
6. Extracted records are normalized into structured JSON objects.
7. Results are buffered locally during the run.
8. Pagination continues until:

   * The target count is reached
   * Results are exhausted
   * The user stops the session
9. Clicking **Download** exports the collected data.

---

## Export Formats

### Single JSON File

```text
~/Downloads/scraped-jobs-YYYY-MM-DD.json
```

### One File Per Job

```text
~/Downloads/scraped-jobs/YYYY-MM-DD/
  company_title_jobId.json
```

### Example Record

```json
{
  "title": "Senior Software Engineer",
  "company": "Acme Corp",
  "location": "San Francisco, CA",
  "salary": "$150K - $200K",
  "jobId": "4384082246",
  "applyUrl": "https://company.com/careers/123",
  "linkedinUrl": "https://www.linkedin.com/jobs/view/4384082246/",
  "description": "Full job description...",
  "aboutCompany": "Company overview...",
  "hiringTeam": [],
  "missingFields": [],
  "exhaustedRetries": false
}
```

---

## Technical Notes

### Chrome Extension Constraints

* `chrome.storage.session` is not available inside content scripts; `chrome.storage.local` is used instead.
* Manifest V3 service workers cannot rely on `Blob` or `URL.createObjectURL()` for downloads.
* Large strings cannot be safely encoded with a single `btoa()` call; export payloads are encoded in chunks.
* Content script reinjection is guarded to prevent duplicate initialization errors.
* Progress messages are treated as fire-and-forget events because the popup may be closed.

### Download Recovery

Interrupted final exports are automatically retried for up to five seconds.

If all retries fail, diagnostic information is stored in:

```text
chrome.storage.local.failedDownloads
```

These records are intended for debugging and do not affect scrape statistics.

---

## LinkedIn DOM Strategy

LinkedIn frequently changes CSS class names, making class-based selectors unreliable.

This scraper intentionally avoids hashed classes and instead targets stable attributes such as:

* `data-testid`
* `aria-label`
* `role`
* `componentkey`

This approach significantly improves resilience across LinkedIn UI updates.

### Current Key Selectors

| Purpose                   | Selector                                                        |
| ------------------------- | --------------------------------------------------------------- |
| Job list container        | `[data-component-type="LazyColumn"]`                            |
| Job cards                 | `div[role="button"][componentkey]`                              |
| Description expand button | `[data-testid="expandable-text-button"]`                        |
| Description content       | `[data-testid="expandable-text-box"]`                           |
| External apply link       | `a[aria-label="Apply on company website"]`                      |
| Pagination next button    | `button[data-testid="pagination-controls-next-button-visible"]` |

> Job IDs are not available from the job card itself and must be extracted from the URL after opening a posting.

---

## Limitations

* Requires an active LinkedIn session.
* Designed specifically for the LinkedIn Jobs search-results interface.
* LinkedIn UI changes may require selector updates.
* Scraping performance depends on LinkedIn page responsiveness and network conditions.

---

## Development

### Project Structure

```text
.
├── manifest.json
├── background.js
├── content_script.js
├── popup.html
├── popup.js
├── popup_state.js
├── scrape_session.js
├── in_page_controls.js
└── json_export.js
```

### Design Documentation

Additional implementation details and design decisions can be found in:

```text
docs/superpowers/specs/2026-03-17-linkedin-scraper-design.md
```

---

## License

MIT
