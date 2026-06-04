# OBHRM Literature Monitor

This repository packages a reusable Codex skill for monitoring new articles from an approved OBHRM/HCI/preprint target-source whitelist.

The current whitelist contains approved OBHRM/HCI/preprint target sources selected from FT50, UTD24, AJG/ABS 2024 `4`/`4*`, ABDC 2025 `A`/`A*`, 9 SJR Q1 HCI journals from `uni.ubicomp.net/hci/`, plus SSRN and NBER. `Operations Research` is excluded from all selectable source lists.

## Safety Boundary

The project does not automate institutional login, bypass paywalls, solve CAPTCHAs, bypass anti-bot checks, or download PDFs. Reports keep per-article access information to DOI URLs. Readers use their own authorized university access manually if they want full text.

## What The Skill Produces

- Markdown weekly literature report.
- CSV record table with stable field order.
- Standalone HTML report suitable for static hosting.
- Keyword trajectory charts in the HTML report, backed by `obhrm_keyword_trends.json`.
  The combined chart supports `% of keyword peak` and `Raw counts` views, plus year-level hover details.
- Optional short Lark webhook summary containing only concepts, time window, and journal/platform counts.

Per-article report fields are:

```text
title
journal
publication date
doi url
authors
affiliations
abstract status
abstract
keywords
matched concepts
matched fields
```

## Setup

Install Python dependencies:

```powershell
pip install -r requirements.txt
```

Use `.env.example` as a template for local environment variables. Never commit `.env`, `config/monitor.yaml`, outputs, logs, or Lark webhook secrets.

## Build Or Refresh The Whitelist

The repository includes the generated whitelist files:

```text
data/whitelist/journals.csv
data/whitelist/journals_review.md
```

The original ABDC workbook is not committed to this repository. To rebuild the whitelist from a local copy of the ABDC JQL workbook, pass its path explicitly:

```powershell
python skills/obhrm-literature-monitor/scripts/build_whitelist.py --abdc-file "C:\path\to\ABDC-JQL-2025-v1-260326.xlsx"
```

Review `data/whitelist/journals_review.md` before using a newly rebuilt whitelist for production monitoring.

## Run A Weekly Scan

Copy and edit the example monitor config if needed:

```powershell
Copy-Item config/monitor.example.yaml config/monitor.yaml
```

Run the previous full week in the configured timezone:

```powershell
python skills/obhrm-literature-monitor/scripts/run_daily_scan.py --previous-week
```

The default production scan uses the `openalex-source` strategy. It resolves each whitelist source to an OpenAlex source id, searches each source/concept/window combination, and writes `obhrm_scan_trace.csv` so the traversal can be audited.

Run a specific window with separate keyword concepts:

```powershell
python skills/obhrm-literature-monitor/scripts/run_daily_scan.py --timezone Asia/Tokyo --start "2026/05/18 00:00" --end "2026/05/25 00:00" --keyword Presenteeism --match-mode any
```

Use quotes when a multi-word concept should be interpreted as one ordered phrase. The outer quotes are stripped before matching, so `"Business History"` is treated as the exact phrase `Business History`:

```powershell
python skills/obhrm-literature-monitor/scripts/run_daily_scan.py --journal-list abs-4-star --journal-list ft50 --timezone Asia/Tokyo --start "1970/01/01 00:00" --end "2026/06/01 00:00" --keyword '"Business History"' --keyword Asia --keyword Engagement --match-mode all
```

The production search retrieves OpenAlex candidates by source/concept/window, then locally checks title, abstract, and keyword metadata. The CSV/Markdown `matched_fields` column shows which fields matched.

When multiple keyword concepts are supplied, the report's `At a Glance`, `Articles With Abstracts`, and `Missing Abstract` sections are split by keyword concept. Articles that match multiple concepts can appear in multiple concept sections so each reader can browse from their own concept of interest.

Choose one or more source lists when broad keywords would produce too many articles. Repeating `--journal-list` scans the union of those lists:

```text
all-whitelist       all approved OBHRM/HCI/preprint whitelist sources
abs-4-and-4-star    ABS/AJG 2024 4 and 4* sources within the whitelist
abs-4-star          ABS/AJG 2024 4* sources within the whitelist
ft50                FT50 sources within the whitelist
utd24               UTD24 sources within the whitelist
```

Example:

```powershell
python skills/obhrm-literature-monitor/scripts/run_daily_scan.py --journal-list abs-4-star --timezone Asia/Tokyo --start "2000/01/01 00:00" --end "2026/06/01 00:00" --keyword Asia --keyword Asian --match-mode any
```

```powershell
python skills/obhrm-literature-monitor/scripts/run_daily_scan.py --journal-list abs-4-star --journal-list ft50 --timezone Asia/Tokyo --start "2026/05/18 00:00" --end "2026/05/25 00:00" --keyword Presenteeism
```

Render the Markdown report as standalone HTML:

```powershell
python skills/obhrm-literature-monitor/scripts/render_report_html.py --input outputs/<run-folder>/obhrm_daily_report.md
```

Publish a rendered HTML report into the static Netlify site directory:

```powershell
python skills/obhrm-literature-monitor/scripts/publish_report_site.py --input outputs/<run-folder>/obhrm_daily_report.html
```

The published report will be available under:

```text
site/reports/<run-folder>/
```

The public site copy removes email addresses found in article metadata. Local Markdown/CSV/HTML outputs remain unchanged.

## Run From GitHub Web Page

Collaborators do not need Codex, Python, or this repository on their own computers if they use the GitHub Actions workflow.

For a step-by-step beginner guide, see `docs/OBHRM_Literature_Monitor_GitHub_Actions_User_Guide.md`.

Recommended group setup: keep one central copy of this repository under the project maintainer's personal GitHub account, then invite trusted teachers/students as collaborators. Give collaborators enough access to run manual Actions workflows; `Write` is the practical default for this project because the workflow commits published `site/` updates back to the repository.

The project maintainer should configure the central repository once:

1. Open the central repository.
2. Click `Settings` -> `Collaborators and teams`.
3. Use `Add people` to invite teachers/students as collaborators.
4. Grant the collaborators who need to run reports `Write` access.
5. Click `Settings` -> `Pages`.
6. Under `Build and deployment`, set `Source` to `GitHub Actions`.
7. Add Lark repository secrets if needed:
   - `OBHRM_LARK_WEBHOOK_URL`
   - `OBHRM_LARK_WEBHOOK_SECRET`

After that, members run reports directly in the central repository:

1. Click `Actions`.
2. Choose `Generate OBHRM Literature Report`.
3. Click `Run workflow`.
4. Fill in:
   - `keyword_1` to `keyword_5`: enter up to five concepts, one per field. Leave unused fields blank.
   - `timezone`: choose `Asia/Tokyo`, `America/Chicago`, or `Asia/Shanghai`.
   - `start_date` and `start_clock`: inclusive start date and time, such as `2026/05/18` and `00:00`.
   - `end_date` and `end_clock`: exclusive end date and time, such as `2026/05/25` and `00:00`. The workflow will fail clearly if the end is not later than the start.
   - `match_mode`: choose `any` for OR logic, or `all` for AND logic.
   - Journal list checkboxes: select one or more of `all-whitelist`, `abs-4-and-4-star`, `abs-4-star`, `ft50`, and `utd24`. The workflow scans the union of all selected lists. `abs-4-star` is selected by default as the most selective option.
   - `public_site_url`: leave blank unless you maintain a custom Netlify or GitHub Pages domain.
5. Start the workflow and wait for it to finish.

The workflow runs on GitHub-hosted servers. It generates Markdown, CSV, and HTML artifacts, publishes the public HTML copy into `site/reports/<run-folder>/`, commits the updated `site/` directory, and deploys the `site/` directory to GitHub Pages. The public index page aggregates all central-repository reports and shows the GitHub account and run timestamp for each new report.
It also uploads `obhrm_scan_trace.csv`, which shows source-by-source traversal details: journal/platform name, OpenAlex source id, concept, API total count, fetched count, pages fetched, status, and query URL.
It also uploads `obhrm_keyword_trends.json`, which stores the per-keyword yearly counts and year-level top-cited candidate metadata used by the HTML trajectory charts.
Technical OpenAlex controls are intentionally hidden from the normal `Run workflow` form; production web runs use the source-first OpenAlex strategy with exhaustive cursor paging by default.

When `public_site_url` is blank, report links are generated from the running repository's GitHub Pages URL:

```text
https://<owner>.github.io/<repo-name>/
https://<owner>.github.io/<repo-name>/reports/<run-folder>/
```

Forks are still supported as a fallback when a user cannot be added to the organization repository. In that case, the user must configure `Settings` -> `Pages` -> `Source` -> `GitHub Actions` in their own fork, and their reports will publish to that fork's Pages site rather than the central index.

If Lark secrets are configured, the workflow also sends the short Lark summary. Add these repository secrets under GitHub `Settings` -> `Secrets and variables` -> `Actions`:

```text
OBHRM_LARK_WEBHOOK_URL
OBHRM_LARK_WEBHOOK_SECRET
```

The Lark summary includes only concepts, the selected timezone window, journal/platform counts, and public report links.

## Optional Netlify Hosting

This repository includes `netlify.toml`. Netlify is optional and mainly useful for a central project site. In Netlify, connect a GitHub repository and use:

```text
Build command: leave empty
Publish directory: site
```

After each weekly report, `publish_report_site.py` updates `site/`, and the workflow commits those site files. Netlify will redeploy if it is connected to that repository. Fork users can ignore Netlify and use the GitHub Pages links produced by the workflow.

## Lark Push

Check local configuration:

```powershell
python skills/obhrm-literature-monitor/scripts/check_config.py
```

Send a test message:

```powershell
python skills/obhrm-literature-monitor/scripts/check_config.py --test-lark
```

Send the short summary after a scan:

```powershell
python skills/obhrm-literature-monitor/scripts/run_daily_scan.py --previous-week --push-lark
```

Send the short summary for an already generated CSV and hosted report:

```powershell
python skills/obhrm-literature-monitor/scripts/push_lark_report_summary.py --csv outputs/<run-folder>/obhrm_daily_records.csv --start "2026/05/18 00:00" --end "2026/05/25 00:00" --concepts "Presenteeism" --public-report-url https://example.netlify.app/reports/<run-folder>/ --public-index-url https://example.netlify.app/
```

The Lark message is intentionally brief. It includes concepts, the selected timezone window, and matched article counts by journal/platform. It does not include article titles, DOI lists, local file paths, or the full report text.
