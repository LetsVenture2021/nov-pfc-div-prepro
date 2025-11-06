.
├── README.md # You are here
├── LICENSE # Project license (fill in before sharing)
├── .gitignore # Python + macOS + data exclusions
├── pyproject.toml # Build + dependency management (or use requirements.txt)
├── Makefile # Common automation commands
├── .env.example # Pipeline environment variables (copy to .env)
│
├── config/
│ ├── pipeline.default.yaml # Master config (weights, thresholds, channel rules)
│ ├── pipeline.dev.yaml # Development overrides
│ └── pipeline.prod.yaml # Production overrides
│
├── src/
│ ├── pipeline/
│ │ ├── init.py
│ │ ├── cli.py # Entry point to run the pipeline end-to-end
│ │ ├── ingest.py # Reads source CSVs, harmonises columns, drops per-slot fields
│ │ ├── outputs.py # Writes CSV/JSON artifacts + automation docs
│ │ └── transform/
│ │ ├── init.py
│ │ ├── phone_rollup.py # Weighted contact rollups (cell / landline / email / DNC)
│ │ ├── scoring.py # OPS/DI/UI/OLS/CI calculations + category blends
│ │ ├── channel_plan.py # Maps DI/UI/CI/DNC to activation channels
│ │ └── offer_strategy.py# Equity + condition driven offer routing
│ └── utils/
│ ├── io.py # Path helpers, CSV/JSON readers
│ ├── logging.py # Structured logging
│ └── validators.py # Config + schema guards
│
├── data/ # Local data (kept out of version control)
│ ├── raw/ # Ingested source exports (gitignored)
│ ├── interim/
│ ├── processed/
│ └── reports/
│
├── docs/
│ ├── architecture.md # System overview, data flow, scoring maths
│ ├── playbooks/
│ │ ├── automation_routing.md
│ │ └── playbook_matrix.md
│ └── CHANGELOG.md
│
├── tests/
│ ├── init.py
│ ├── conftest.py
│ ├── data/ # Fixtures mirroring source CSVs
│ ├── test_ingest.py
│ ├── test_transform.py
│ └── test_outputs.py
│
├── scripts/
│ ├── sync_dialer.sh # Push combined_dialer_ready.csv to dialer
│ ├── sync_mail_house.sh # Send combined_mail_only.csv to mail vendor
│ └── update_crm_tags.sh # Update CRM tags from automation docs
│
└── .github/
└── workflows/
└── ci.yml # Lint, type-check, test pipeline
