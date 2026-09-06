# Job Search Assistant — Automated Workflow

Runs on a fixed cadence (e.g. every 3 days). No emails or messages are ever sent — this pipeline only researches, drafts, and files documents for the candidate's own review.

## Step 0 — Load context

Read, in this order:
1. `candidate_profile.md` (product-helper folder) — candidate background, target roles, dealbreakers.
2. `must_check_companies.md` (product-helper folder) — target companies with an automatic scoring bonus.
3. `applied_roles_log.md` (product-helper folder) — roles already processed in a prior run. Any (Company + Title) or exact posting Link already logged with status `applied-docs-generated` is excluded from this run's candidate pool.

If any of these three files is missing, stop and ask the user rather than guessing at their contents.

## Step 1 — Search and filter (skill: role-search-filter)

Search the web for open roles matching the target role(s) defined in `candidate_profile.md`, in the target market ([TARGET_MARKET]). Must-check sources:
- Each company's own careers page for every company in `must_check_companies.md` — check these explicitly every run, not just via general search.
- General web search across other job boards/company sites for the same role types.

Invoke the `role-search-filter` skill to score every surviving posting (after hard filters: target-market location, matching title, no required experience unambiguously above the profile's experience ceiling) against the candidate profile, producing a shortlist of ≤30 scored roles with the full per-criterion breakdown.

Before finalizing this step's output, cross-check every candidate posting's (Company, Title, Link) against `applied_roles_log.md`. Drop postings already logged as `applied-docs-generated`; note the count dropped this way in the run summary.

## Step 2 — Shortlist top picks (skill: role-shortlisted)

Feed the scored batch from Step 1 into the `role-shortlisted` skill. This selects the top 5 (per the skill's own logic: drop SKIPs, rank by score, real target count if fewer than 5 clear MODERATE FIT, freshness mix, company-diversity check, flagged ties).

Carry forward, unmodified: the "why this made the cut" reasoning, main risk, freshness mix, concentration notes, and any flagged ties/close calls. These get reused verbatim in the final summary — do not paraphrase them away.

## Step 3 — Generate CV and cover letter per shortlisted role

For each of the (up to 5) shortlisted roles:

1. **CV** — invoke the `cv-tailoring` skill with the role's job description. This fetches the base CV, tailors summary/bullet selection/phrasing to the role, and uploads a new tailored `.docx` (never modifying the original). Record the resulting link.
2. **Cover letter** — invoke the cover-letter-builder skill with the same job description. This drafts the letter per the house style (word cap, impact-first evidence, mandatory humanize pass) and saves it. Record the resulting link.

If either skill fails to produce a link (access issue, generation error), do not fabricate a placeholder link — flag that role's documents as "not yet generated" and note the reason in the run summary.

After generating both documents for a role, append a row to `applied_roles_log.md`:

| Date run | Company | Title | Link | Score | Verdict | Status | CV link | CL link |

Status should read `applied-docs-generated` once both links exist, or `docs-failed` with a one-line reason if generation failed for that role.

## Step 4 — Produce summary (skill: jobsearch-summary)

Invoke the `jobsearch-summary` skill with the shortlisted batch (Step 2 output) plus each role's CV and CL links (Step 3). This produces one PDF covering:
- Cover section: date, postings reviewed, thin-batch warning if fewer than 10 postings were scored, any flagged ties/close calls.
- One card per role: company/title/location/freshness/link, score/verdict, "why this made the cut," main risk, and the CV + cover letter links.

Save the PDF to the product-helper folder as `jobsearch-summary-YYYY-MM-DD.pdf` (today's date), so runs never overwrite each other.

## Step 5 — Hand back to the user

Present the finished PDF (via `mcp__cowork__present_files`) with a short note: how many postings were reviewed, how many were new vs. already-processed (from the applied-roles check), and how many made the final shortlist. No further action is taken automatically — the candidate reviews the PDF and the linked CV/CL docs and decides what to actually submit.

## Guardrails specific to this automated run

- **No emails, messages, or applications are ever sent automatically.** This pipeline stops at drafts and files.
- **No fabricated postings, scores, links, or facts** anywhere in the chain — each skill's own guardrails (no invented URLs, no invented CV facts, no invented candidate qualifications) apply at every step.
- **Idempotency**: the `applied_roles_log.md` check in Step 1 is mandatory every run, so the cadence never re-does work on a role already fully processed. If a previously-`docs-failed` role reappears, it's fine to retry it.
- **If any step's required input is missing or malformed** (e.g. Step 2 gets a batch with no scores, Step 4 gets a role with no CV link), stop and follow that skill's own instruction to ask the user rather than inventing the missing piece.
