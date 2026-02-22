# Open Data Portal

A forkable, Jekyll-based open data portal template for small government agencies.
Staff upload CSVs via the GitHub web UI; a GitHub Action auto-generates HTML preview
tables; Jekyll rebuilds the site on GitHub Pages. No npm, no complex tooling.

---

## Quick Start (Fork & Deploy)

1. **Fork** this repository to your GitHub organization.
2. In your fork → **Settings → Pages**: set *Source* to **GitHub Actions**.
3. In your fork → **Settings → Actions → General**: ensure *Workflow permissions*
   is set to **Read and write permissions**.
4. Push any change to `main` (or trigger the workflows manually) — GitHub Actions
   will build and deploy the site.

---

## Adding a Dataset (for non-technical staff)

### Step 1 — Upload the CSV

1. Navigate to the `datasets/` folder in the GitHub web UI.
2. Click **Add file → Upload files**.
3. Drop your CSV file (e.g. `my-new-dataset.csv`) and commit to `main`.

The `generate-previews` Action will run automatically and create an HTML table
preview. You do **not** need to run anything locally.

### Step 2 — Create the dataset metadata page

1. Navigate to `_datasets/` in the GitHub web UI.
2. Click **Add file → Create new file**.
3. Name it `my-new-dataset.md` (must match the CSV filename without extension).
4. Paste and fill in the following template:

```yaml
---
title: My New Dataset
description: A short description of what this dataset contains.
organization: City Finance Office
category: Budget & Finance
license: CC BY 4.0
maintainer: Your Name
maintainer_email: you@example.gov
updated: 2024-07-01
created: 2024-07-01
preview: my-new-dataset
resources:
  - name: My New Dataset CSV
    url: https://raw.githubusercontent.com/YOUR_ORG/YOUR_REPO/main/datasets/my-new-dataset.csv
    format: CSV
---

Optional longer description or notes about this dataset go here.
```

5. Commit the file to `main`. The `pages` Action will rebuild the site and your
   dataset page will be live within a few minutes.

> **Important:** The `preview:` value must exactly match the CSV filename
> (without `.csv`). This links the metadata page to the auto-generated preview table.

---

## Available Categories

| Category | Use in front matter |
|---|---|
| Budget & Finance | `Budget & Finance` |
| Planning & Zoning | `Planning & Zoning` |
| Arts Culture & History | `Arts Culture & History` |
| Economy | `Economy` |
| Education | `Education` |
| Elections & Politics | `Elections & Politics` |
| Environment | `Environment` |
| Food | `Food` |
| Health & Human Services | `Health & Human Services` |
| Parks & Recreation | `Parks & Recreation` |
| Public Safety | `Public Safety` |
| Real Estate & Land Records | `Real Estate & Land Records` |
| Transportation | `Transportation` |
| Uncategorized | `Uncategorized` |

---

## Local Development

### Prerequisites

- Ruby >= 3.1 + Bundler
- Python >= 3.10 + pip

### Setup

```bash
# Install Ruby dependencies
bundle install

# Install Python dependencies
pip install -r requirements.txt
```

### Generate previews locally

```bash
python scripts/generate_previews.py
```

### Serve the site

```bash
bundle exec jekyll serve
# Visit http://localhost:4000
```

---

## File Structure

```
.
├── .github/
│   └── workflows/
│       ├── generate-previews.yml   # Runs on CSV push, commits preview HTML
│       └── pages.yml               # Builds & deploys Jekyll site
├── _dataset_categories/            # 14 category collection files
├── _datasets/                      # One .md per dataset (metadata + preview link)
├── _includes/
│   ├── datasets.html               # Reusable dataset card component
│   ├── footer.html
│   ├── head.html
│   ├── header.html
│   └── previews/                   # Auto-generated HTML table snippets
├── _layouts/
│   ├── category.html
│   ├── dataset.html                # Metadata sidebar + resource table + dynamic preview
│   ├── default.html
│   └── organization.html
├── _organizations/                 # Organization collection files
├── _data/
│   └── schemas/
│       └── default.yml             # Fields shown in dataset metadata sidebar
├── datasets/                       # Raw CSV files (excluded from Jekyll build)
├── scripts/
│   └── generate_previews.py        # Bridges CSVs to _includes/previews/*.html
├── _config.yml
├── categories.html
├── datasets.html
├── index.html
├── organizations.html
├── Gemfile
└── requirements.txt
```

---

## How It Works

```
Staff uploads CSV to datasets/
        |
        v
GitHub Action: generate-previews
  - pandas reads CSV
  - Jinja2 renders Bootstrap table HTML
  - Commits _includes/previews/<name>.html [skip ci]
        |
        v
GitHub Action: pages (next push or manual trigger)
  - bundle exec jekyll build
  - dataset layout includes previews/<name>.html dynamically
  - Deployed to GitHub Pages
```

---

## Customization

- **Site title & description** — edit `_config.yml`
- **Navigation links** — edit `navigation` list in `_config.yml`
- **Metadata sidebar fields** — edit `_data/schemas/default.yml`
- **Styling** — edit `_includes/head.html` (Bootstrap CDN, inline CSS)
- **Organizations** — add/edit files in `_organizations/`
- **Categories** — add/edit files in `_dataset_categories/`
