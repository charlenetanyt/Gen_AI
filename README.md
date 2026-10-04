# Trustworthy Self-Serve Analytics Agent

IS469 Group Project (G1, August Term 2026)

A natural-language analytics agent that lets non-technical marketplace staff ask business questions in plain English. The agent connects to a PostgreSQL database through an MCP server, retrieves only the schema it needs (Schema RAG), writes and self-corrects multi-table SQL, and runs every query through a guardrail harness that enforces read-only, least-privilege access.

> **Status:** Project setup. Sections marked *(planned)* describe components that are not built yet.

## How it works

1. The user asks a question in plain English.
2. **Schema RAG** retrieves the relevant tables, columns, join paths and example queries.
3. The **agent** plans and writes SQL, then calls the MCP server's `run_read_query` tool.
4. The **guardrail harness** checks the SQL (SELECT-only, allow-lists, row limit, timeout, cost check) and blocks it or runs it on a read-only connection.
5. On an error, the agent reads the message and repairs its SQL (up to 3 attempts).
6. The agent answers in plain language, showing the SQL and row count.

## Research questions

| # | Question | Baseline | Improved variant |
|---|---|---|---|
| RQ1 | Does Schema RAG beat putting the full schema in the prompt, and how does that change as the schema grows? | Full schema in prompt | Schema RAG |
| RQ2 | How much does an agent loop with self-correction improve over single-shot text-to-SQL? | Single-shot SQL | Agent loop with repair |
| RQ3 | How well do layered guardrails block unsafe behaviour, and what do they cost in false blocks? | No guardrails | Full guardrail harness |

## Repository structure

```
.
├── config.yaml          # all experiment settings in one place
├── docker-compose.yml   # PostgreSQL 16
├── requirements.txt
├── .env.example         # copy to .env and fill in
├── data/                # raw data (downloaded, not committed)
├── db/                  # schema, Olist loader, synthetic + distractor tables
├── schema_rag/          # schema docs, index builder, retriever
├── mcp_server/          # MCP server exposing database tools
├── agent/               # agent loop and single-shot baseline
├── guardrails/          # SQL validation and safety checks
├── eval/                # benchmark, attacks, runner, scoring
├── notebooks/           # analysis and charts
├── results/             # experiment outputs
└── CONTRIBUTIONS.md     # running log of who did what
```

## Team

| Role | Owner | Folders |
|---|---|---|
| Data and Schema RAG | Charlene | `db/`, `schema_rag/` |
| Agent and MCP | TBC | `mcp_server/`, `agent/` |
| Guardrails and Security | TBC | `guardrails/`, `eval/attacks/` |
| Evaluation and Integration | TBC | `eval/`, `notebooks/` |

## Setup

### Prerequisites

- Python 3.11+
- Docker Desktop
- A Google AI Studio API key for Gemini
- A Kaggle account (to download the dataset)

### 1. Clone and create a virtual environment

```bash
git clone <repo-url>
cd is469-nl-db-agent
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

Then fill in the values in `.env`. **Never commit `.env`**; it is already in `.gitignore`.

### 3. Start the database

```bash
docker compose up -d
```

PostgreSQL runs on `localhost:5432` (or the port set in `.env`). SQL files in `db/init/` run automatically the first time the container starts. To rebuild from scratch:

```bash
docker compose down -v
docker compose up -d
```

### 4. Download the data

Download the [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and unzip the CSVs into `data/raw/`. See `data/README.md`.

### 5. Load the data *(planned)*

```bash
python db/load_olist.py
python db/generate_synthetic.py
```

### 6. Run the system and experiments *(planned)*

```bash
python -m agent.cli "Which product categories had the most late deliveries in 2018?"
python -m eval.runner --config config.yaml
```

## Workflow

- Work on your own branch (e.g. `charlene/db-loader`), never directly on `main`.
- Open a pull request and get one teammate's review before merging.
- Run the end-to-end pipeline on the development split each week.
- Log your work in `CONTRIBUTIONS.md` as you go.

## Reproducibility

- All experiment conditions are set in `config.yaml`.
- Random seeds are fixed for synthetic data, question splits and sampling.
- The LLM model version and library versions are pinned (run `pip freeze > requirements.lock.txt` once dependencies are stable).
- Results are written to `results/` with the config used to produce them.

## Data and ethics

- The Olist dataset is public and anonymised; see its Kaggle page for licence terms.
- All personal-looking data in the synthetic tables (emails, phone numbers, bank details, salaries) is fake, generated with Faker.
- Attack tests without guardrails run only against a disposable database copy.

## AI use

We follow the IS469 AI-use policy. AI assistance with code is disclosed in `CONTRIBUTIONS.md`.
