# Job-Search Automation Framework

A complete, self-contained set of skills and reference files for automating a job search for
**any role** — you set the target role(s) in your profile. It searches and scores postings,
shortlists, tailors CVs and cover letters, and produces a run summary. Everything here is a
**template** — all personal data has been removed and replaced with placeholders. Fill in your
own details before running.

## The pipeline

`job_search_workflow.md` orchestrates the whole run. Every skill it calls is included:

1. **Search & filter** → `role-search-filter` — scores every posting against your profile, ≤30-role shortlist.
2. **Shortlist** → `role-shortlisted` — picks the top 5 (freshness mix, company-diversity check).
3. **Generate docs** → `cv-tailoring` + `cover-letter-builder` — one tailored CV and cover letter per role.
4. **Summarize** → `jobsearch-summary` — one PDF covering the batch.

## What's in here

### Skill bundles (`.skill`)
Installable skill archives (zip). Each contains a `SKILL.md` plus its reference files.

| Bundle | What it does |
|---|---|
| `role-search-filter.skill` | Search and score postings for your target role(s) against your profile; output a ranked shortlist. |
| `role-shortlisted.skill` | Take a scored batch and pick the top 5 to actually apply to. |
| `cv-tailoring.skill` | Tailor your base CV to a specific job description (hiring-manager lens, one page). |
| `cover-letter-builder.skill` | Draft a cover letter from a job description + your CV (impact-first, length-disciplined; saves the finished letter to Google Drive). |
| `hummanized-writting.skill` | Strip AI tells from prose (mandatory second pass for the cover-letter skill). |
| `jobsearch-summary.skill` | Turn a scored, shortlisted batch into one PDF summary. |

### Framework / reference files (`.md`)

| File | Purpose |
|---|---|
| `job_search_workflow.md` | The end-to-end pipeline (the 4 steps above). |
| `candidate_profile.md` | **Template.** Your background, target roles, location, dealbreakers. Read at the start of every run. |
| `must_check_companies.md` | **Template.** Your target companies (automatic scoring bonus). |
| `applied_roles_log.md` | **Template.** Log of already-processed roles, for idempotent repeat runs. |
| `cover_letter_guidelines.md` | General cover-letter structure and tone guidance. |

## Placeholder convention

Every slot you need to fill is a bracketed `UPPER_SNAKE` token, so you can grep for them:

```
grep -rn "\[.*\]" .
```

Common tokens:

| Token | Replace with |
|---|---|
| `[FULL_NAME]` / `[FIRST_NAME]` | Your name |
| `[EMAIL]` | Your email |
| `[LINKEDIN_URL]` | Your LinkedIn URL |
| `[LOCATION]` | Your city/region |
| `[TARGET_MARKET]` | The market you're searching in (e.g. a country) |
| `[COMPANY_1]` … `[COMPANY_7]` | Your target companies |
| `[EMPLOYER_1]` … `[EMPLOYER_3]` | Your past employers |
| `[UNIVERSITY_1]` / `[UNIVERSITY_2]` | Your schools |
| `[BASE_CV_GDOC_URL]` / `[BASE_CV_GDOC_FILEID]` | Your base-CV Google Doc (see CV setup below) |
| `[FILL IN]` | Any freeform field to complete |

## Getting started

1. Fill in `candidate_profile.md`, `must_check_companies.md`, and set `[TARGET_MARKET]` /
   `[COMPANY_*]` where they appear.
2. **Set up your base CV** — two options for `cv-tailoring`:
   - *Google Drive (default):* store your CV as a Google Doc and set `[BASE_CV_GDOC_URL]` /
     `[BASE_CV_GDOC_FILEID]` in `cv-tailoring`'s `SKILL.md`. Requires a Google Drive connector.
     The skill fetches it fresh, tailors it, and uploads the tailored `.docx` back to Drive.
   - *Local fallback (no Google account):* leave those placeholders unset and put your CV in a
     local `references/cv.md`. The skill reads from there and delivers the tailored `.docx`
     locally. The cover-letter skills already read the CV from `references/cv.md`.
3. Start `applied_roles_log.md` empty (or with your own history — but keep real application
   data out of a public repo; see `.gitignore`).
4. Follow `job_search_workflow.md` to run the pipeline.

## Note on privacy

These files were genericized specifically so they could be shared publicly. If you fork this
and fill it in with real data, keep your filled-in copies local — the `.gitignore` excludes
`*.local.md` and generated summary PDFs for that reason. The `cv-tailoring` Google Doc link,
once you set it, points at your own private document; don't commit a real link to a public repo.
