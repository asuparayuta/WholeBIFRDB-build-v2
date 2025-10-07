# WholeBIF‑RDB – Build & App Starter

> Production‑ready boilerplate for the **Whole Brain Information Flow – Relational Database (WholeBIF‑RDB)** and the companion **Gradio web app** to browse circuits, connections, evidence, references, and scores.

This README explains dependencies, setup, commands, project layout, and common workflows (local & Docker). It also describes how to import data from spreadsheets, run migrations, and launch the public query UI.

---

## Key Features

- **PostgreSQL 16** schema for **Circuits / Connections / Evidence / References / Scores / Changelog** (DHBA alignment ready)
- **Gradio** app for interactive querying and browsing of circuits and their subcircuits
- **Bulk data import** helpers (CSV/Google Sheets → DB) with idempotent upserts
- **Scoring hooks** (PDER / DSI / methodscore / citationscore placeholders)
- **Reproducible** (Poetry/conda) and **containerized** (Docker Compose) runs
- **CI‑friendly** scripts and clear environment configuration

---

## Repository Structure

A condensed tree is below. See `FILES.md` for the full tree.

```text
WholeBIFRDB-build/
└── WholeBIFRDB-build
    ├── src
    │   ├── visualize
    │   │   ├── gradio_wholebif_query_app_flexpair_public.py
    │   │   └── gradio_wholebif_query_app_flexpair_public_v2.py
    │   ├── build_wholebifrdb.ipynb
    │   ├── import_bdbra_into_wholebif_v2.py
    │   └── import_bdbra_into_wholebif_v3.py
    └── README.md
... (see FILES.md for the full tree)
```

---

## 1. Requirements

Choose one of the following environments.

### Option A: Local (Python 3.11+)

- Python 3.11 (3.10 also works in most cases)
- PostgreSQL 16 (or compatible managed instance)
- (Recommended) **Poetry** or **conda**
- OS packages: `libpq`, `psql`, and build tools (macOS: `brew install postgresql@16`)

### Option B: Docker

- Docker 24+
- Docker Compose v2+

---

## 2. Quick Start (Docker)

```bash
# 1) Copy your env template
cp .env.example .env

# 2) Update credentials
#   - POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB
#   - APP_HOST, APP_PORT
#   - (optional) GOOGLE_SHEETS_* / OPENAI_* if you import from cloud services

# 3) Start database + app
docker compose up -d --build

# 4) Apply migrations and seed (if provided)
docker compose exec app python scripts/db_migrate.py
docker compose exec app python scripts/seed_demo.py

# 5) Open the app
# Gradio UI → http://localhost:${os.getenv('APP_PORT', '7860')}
```

Stop everything:

```bash
docker compose down -v
```

---

## 3. Quick Start (Local)

```bash
# 1) Create and activate env
# Using conda
conda create -n wholebif python=3.11 -y
conda activate wholebif

# Or Poetry
# poetry env use 3.11

# 2) Install deps
pip install -U pip wheel setuptools
pip install -r requirements.txt  # or: poetry install

# 3) Configure env
cp .env.example .env
# edit .env to point to your Postgres

# 4) Initialize DB
python scripts/db_migrate.py

# 5) (Optional) import CSV/Sheets
python scripts/import_from_csv.py data/incoming/*.csv

# 6) Run the Gradio app
python gradio_wholebif_query_app.py --host 0.0.0.0 --port 7860
```

---

## 4. Environment Variables

Create `.env` from `.env.example`. Common keys:

- `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`
- `APP_HOST` (default `0.0.0.0`), `APP_PORT` (default `7860`)
- **Optional ingestion**: `GOOGLE_SHEETS_CREDENTIALS_JSON`, `SHEETS_DOC_ID`
- **Optional scoring**: `OPENAI_API_KEY` (or other providers)
- **Locale & logging**: `TZ=Asia/Tokyo`, `LOG_LEVEL=INFO`

---

## 5. Database

### 5.1 Schema Overview

- **circuits**: `circuit_id (PK)`, `names`, `uniform`, `subcircuit[]`, `status`
- **connections**: `circuit_id (FK)`, `receiver_id`, `connection_flag`, `status`
- **evidence**: quotes + figure pointers per connection, `reference_id`, `status`
- **references_tbl**: `reference_id`, `title`, `doc_link (DOI URL)`, `bibtex_link`, `doi`, `journal_names`, `contributor`
- **scores**: `pder`, `dsi`, `methodscore`, `citationscore`, etc., linked per connection
- **changelog**: audits and provenance

> Field names reflect the normalization discussed in our docs.

### 5.2 Migrations

```bash
python scripts/db_migrate.py  # applies SQL in migrations/
```

### 5.3 Seeding (Optional)

```bash
python scripts/seed_demo.py   # inserts a small curated dataset
```

---

## 6. Data Import

### CSV

```bash
python scripts/import_from_csv.py data/incoming/your_file.csv   --table connections   --if-exists upsert
```

### Google Sheets (optional)

Set `GOOGLE_SHEETS_CREDENTIALS_JSON` and `SHEETS_DOC_ID`, then:

```bash
python scripts/import_from_sheets.py --sheet "Connections"
```

---

## 7. Gradio App

```bash
python gradio_wholebif_query_app.py --host 0.0.0.0 --port 7860   --concurrency 4 --max-queue 64
```

**Features**

1. Search circuits by name/abbrev (DHBA aligned)
2. Click a **Receiver ID** to pivot and re‑query as **Circuit ID**
3. Expand **Subcircuits** for full details (connections, evidence, references, scores)

---

## 8. Development

- Code style: **ruff** + **black**
- Type checks: **mypy**
- Tests: **pytest**

```bash
pip install -r requirements-dev.txt
ruff check .
black .
mypy .
pytest
```

---

## 9. Internationalization (i18n) & Comment Policy

All user‑facing text, docstrings, and comments should be **English‑only**.  
A report of lines that still contain Japanese is generated at `jp_comment_report.csv`.  
Please translate remaining items before merging PRs.

---

## 10. Troubleshooting

- `psycopg` errors → verify Postgres is reachable and credentials are correct
- Gradio error `Blocks.queue() got an unexpected keyword argument 'concurrency_count'` → update Gradio to 4.0+ and use `--concurrency`
- CSV import "value too long for type character varying(255)" → update column width or truncate; see `migrations/2025_00_alter_columns.sql`

---

## License

MIT (unless otherwise noted in subdirectories)
