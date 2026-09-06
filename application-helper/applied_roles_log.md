# Applied Roles Log

Tracks every role that has already gone through the job-search pipeline (searched, shortlisted, and had a CV/cover letter generated), so repeat runs of the automation don't re-process the same posting or spend effort re-generating documents for a company/role already handled.

**How this is used:** before scoring a posting in `role-search-filter`, check its (Company, Title, Link) against the table below. If the same Company+Title combination (or same URL) already appears with status `applied-docs-generated`, skip it from this run's shortlist candidates — note it as "already processed" rather than re-scoring it. If a posting's status is `expired`/`not found` on a later check, leave the row as historical record; don't delete rows.

> This is a template. The row below shows the expected schema — replace it with your own
> runs, or clear the table and start empty. Do not commit real application data to a public repo.

| Date run | Company | Title | Link | Score | Verdict | Status | CV link | CL link |
|---|---|---|---|---|---|---|---|---|
| [YYYY-MM-DD] | [COMPANY_A] | [Role title] | https://example.com/jobs/role-id | — | Shortlisted (top 5) | applied-docs-generated | [CV_DRIVE_LINK] | [CL_DRIVE_LINK] |
