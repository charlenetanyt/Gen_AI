# db/  (Owner: Charlene)

Database schema, data loading and synthetic data.

- `init/` — SQL run automatically when the Postgres container is first created (tables, keys, read-only role)
- `load_olist.py` — loads Olist CSVs into PostgreSQL *(planned)*
- `generate_synthetic.py` — creates the 4 synthetic tables with Faker, seeded *(planned)*
- `generate_distractors.py` — adds distractor tables for the RQ1 scaling study *(planned)*
