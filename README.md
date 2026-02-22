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
  - Updates updated: date in matching _datasets/<name>.md
  - Commits _includes/previews/<name>.html + _datasets/<name>.md [skip ci]
        |
        v
GitHub Action: pages (next push or manual trigger)
  - bundle exec jekyll build
  - dataset layout includes previews/<name>.html dynamically
  - Deployed to GitHub Pages
```

---

## How It All Fits Together

This portal is built on three standard tools — **GitHub**, **Jekyll**, and **GitHub Actions** — wired together so that uploading a spreadsheet is all it takes to publish a dataset page. There is no server to maintain, no database, and no custom application to update.

### The three building blocks

**GitHub as a content store**

Everything lives in a single GitHub repository. The repository holds two kinds of content side by side:

- *Data files* — the raw CSV spreadsheets in the `datasets/` folder
- *Metadata files* — one Markdown file per dataset in the `_datasets/` folder, containing the title, description, organization, category, license, maintainer, and a link back to the CSV

Staff interact with both through the GitHub web interface, the same way you would edit a document in Google Drive. No command line is required.

**Jekyll as a static site generator**

Jekyll reads the metadata files and the layout templates and produces a complete set of HTML pages — one for each dataset, one for each organization, one for each category, plus the homepage and listing pages. The output is a folder of plain `.html` files that can be served from anywhere with no server-side processing.

Jekyll's *collections* feature is what makes the dataset/organization/category structure possible. Each folder starting with an underscore (`_datasets/`, `_organizations/`, `_dataset_categories/`) is a collection: Jekyll loops over the files in it and applies the appropriate layout template to each one automatically.

**GitHub Actions as the automation layer**

Two automated workflows run in the background:

1. `generate-previews` — triggered whenever a file changes inside `datasets/`. It reads every CSV, generates an HTML table snippet showing the first 25 rows, updates the `updated:` date in the matching metadata file, and commits everything back to the repository.

2. `pages` — triggered on every push to `main`. It runs Jekyll to rebuild the full site and deploys the result to GitHub Pages.

Because the first workflow commits with the message suffix `[skip ci]`, GitHub Actions knows not to trigger itself again, preventing an infinite loop.

---

### The dataset page in detail

Each dataset page is assembled from three separate pieces at build time:

```
_datasets/annual-budget.md        ← metadata (title, org, license, etc.)
         +
_includes/previews/annual-budget.html   ← auto-generated table from the CSV
         +
_layouts/dataset.html             ← the HTML template that holds it all together
         =
_site/datasets/annual-budget/index.html  ← the finished page
```

The connection between a metadata file and its preview table is a single field in the metadata file's front matter:

```yaml
preview: annual-budget
```

Jekyll's layout uses this value to include the matching snippet:

```liquid
{% assign _preview_path = "previews/" | append: page.preview | append: ".html" %}
{% include {{ _preview_path }} %}
```

If no preview file exists yet (for example, immediately after creating a new metadata file but before the Action has run), the `{% if page.preview %}` guard prevents a build error — the page simply renders without the table.

---

### The preview generation pipeline

`scripts/generate_previews.py` is a short Python script that runs inside GitHub Actions. For each CSV it finds in `datasets/`:

1. Checks the file is not empty
2. Reads it with pandas, preserving leading zeros and literal "N/A" strings
3. Truncates cells longer than 80 characters so wide text does not break the layout
4. Renders a Bootstrap-styled HTML table showing up to 25 rows
5. Writes the result to `_includes/previews/<name>.html`
6. Updates the `updated:` field in the matching `_datasets/<name>.md` to today's date

If the file is empty, or cannot be parsed, the script writes a visible warning or error message in place of the table rather than silently failing. If any file produces a fatal error, the script exits with a non-zero code so the GitHub Actions step is marked as failed.

---

### Updating a dataset

To replace an existing dataset with newer data, staff simply upload a new CSV with the same filename. The Action will:

- Regenerate the preview table from the new data
- Set `updated:` in the metadata file to today's date automatically

No editing of the metadata file is needed.

---

### The naming convention

The link between a CSV and its dataset page is the filename stem — the part without the extension. `annual-budget.csv`, `_datasets/annual-budget.md`, `_includes/previews/annual-budget.html`, and `preview: annual-budget` in the front matter all refer to the same dataset. Keeping these in sync is the one rule staff need to follow when creating a new dataset.

---

## Adding, Removing, or Renaming Metadata Fields

Every dataset page has a sidebar showing fields like Organization, Category, License, and so on. The list of fields shown, and their display labels, is controlled by a single file: `_data/schemas/default.yml`.

### How the sidebar works

`_data/schemas/default.yml` contains an ordered list of field definitions:

```yaml
fields:
  - key: organization
    label: Organization
  - key: license
    label: License
```

- `key` is the front matter field name in the dataset's `.md` file
- `label` is the human-readable heading shown in the sidebar

The dataset layout loops over this list. For each entry, it looks up `page[key]` — if the dataset's front matter has a value for that key, the row is shown; if not, the row is silently skipped. This means fields are always **opt-in per dataset**: adding a field to the schema does not force every dataset to show it.

---

### Adding a new field

**Step 1** — Add it to the schema file.

Open `_data/schemas/default.yml` and add a new entry:

```yaml
  - key: frequency
    label: Update Frequency
```

The sidebar will now show "Update Frequency" on any dataset page that has `frequency:` in its front matter. Datasets without it are unaffected.

**Step 2** — Add the value to individual dataset files.

Open the relevant `_datasets/<name>.md` file and add the field:

```yaml
frequency: Monthly
```

That is all. No template changes, no code changes — Jekyll picks it up automatically.

---

### Removing a field

To stop showing a field in the sidebar, delete its entry from `_data/schemas/default.yml`. The field can remain in individual dataset `.md` files without causing any harm — it just will not be displayed.

To remove the value from a dataset file too, delete the corresponding line from that file's front matter.

---

### Renaming a field label

To change how a label reads in the sidebar (for example, "Organisation" instead of "Organization"), edit the `label` value in `_data/schemas/default.yml`:

```yaml
  - key: organization
    label: Organisation
```

The `key` must stay the same because it refers to the front matter field name in every dataset file. Only the `label` changes.

---

### Adding a new organization

1. Create a new file in `_organizations/`, e.g. `_organizations/revenue-commissioners.md`:

```yaml
---
title: Revenue Commissioners
description: Ireland's tax and customs authority.
website: https://www.revenue.ie
---
```

2. The organization will appear on the Organizations page and its title can now be used as the `organization:` value in any dataset's front matter.

---

### Adding a new category

1. Create a new file in `_dataset_categories/`, e.g. `_dataset_categories/climate.md`:

```yaml
---
title: Climate
description: Climate data, emissions, and environmental indicators.
---
```

2. The category will appear on the Categories page. Use its `title` exactly (including capitalisation) as the `category:` value in dataset front matter.

---

### Removing an organization or category

Delete the relevant file from `_organizations/` or `_dataset_categories/`. Any datasets that still reference it via `organization:` or `category:` will continue to display the text value in their sidebar — they just will not link to a dedicated org/category page, since that page no longer exists. To clean up fully, update those dataset files to reference a different organization or category.

---

## Customization

- **Site title & description** — edit `_config.yml`
- **Navigation links** — edit `navigation` list in `_config.yml`
- **Metadata sidebar fields** — edit `_data/schemas/default.yml`
- **Styling** — edit `_includes/head.html` (Bootstrap CDN, inline CSS)
- **Organizations** — add/edit files in `_organizations/`
- **Categories** — add/edit files in `_dataset_categories/`
