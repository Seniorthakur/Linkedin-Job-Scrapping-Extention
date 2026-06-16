# Changelog

## 1.0.2

### Fixed
- **Run no longer hijacked to a `/feed/update/...` post and silently killed.**
  LinkedIn interleaves non-job interactive cards (promoted feed posts, suggestions)
  into the results column. They match the same `div[role="button"][componentkey]`
  selector and carry a `Dismiss …` button, so the previous "any dismissible card is a
  job" filter clicked them. Because real job cards select via same-page history
  navigation while a feed card is a real link, clicking one navigated the tab to
  `https://www.linkedin.com/feed/update/urn:li:activity:…`. The content script only
  runs on `https://www.linkedin.com/jobs/*`, so it unloaded there and the run stopped.

### Changed
- Job cards are now identified strictly: a card qualifies only when its dismiss label
  denotes a job (`"Dismiss … job"`, `(Verified job)` tolerated) **or** it resolves to a
  numeric `/jobs/view/<id>` job id (`job_dom_adapters.js`: new `isJobCard`, used by
  `getJobCards` and `findJobListContainer`).
- Added a defense-in-depth navigation guard (`content_script.js`): while a run is
  active, a capture-phase listener blocks any click that would navigate the tab off the
  `/jobs/` area, and a card click that still lands off `/jobs/` stops the run cleanly
  instead of extracting against an unrelated page.

### Tests
- Added regression coverage for excluding dismissible non-job cards and for `isJobCard`.
