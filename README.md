# Early Failure Warning and Repair Prioritisation for Village Water Pumps

> **Hack for Social Cause 2027** · Theme: AI, GIS & Emerging Technologies (secondary: Rural Development & Livelihoods)
> **Team:** [Team Name] · **State/UT:** [State/UT] · **License:** MIT

Village motor pumps in piped water schemes often fail unnoticed. Panchayat operators learn of a
breakdown only after villagers complain, and limited repair crews have no way to decide which pump
to fix first. This project scores each pump's failure risk from routine readings (run-time, current,
voltage, flow, restarts, repair history), ranks pumps for repair, and sends alerts by SMS/WhatsApp in
the local language. A one-tap "fixed" reply from the operator closes the loop and builds labelled data.

> **Data notice:** the prototype runs on **simulated** data (`data/sample/`). No real field data is included.

## Project status

Update this table before submission so judges see exactly what works.

| Component | Status |
|---|---|
| Sample data generator + data integrity tests | Done |
| Technical design document | Done (`docs/TECHNICAL_DESIGN.md`) |
| Feature engineering and risk engine | [Update: Done / In progress / Planned] |
| FastAPI backend | [Update] |
| Streamlit dashboard | [Update] |
| Alert service (mock / SMS / WhatsApp) | [Update] |

## Repository structure

```
pump-failure-early-warning/
├── README.md
├── LICENSE
├── .env.example               # environment variable template
├── requirements.txt
├── docs/
│   ├── TECHNICAL_DESIGN.md    # architecture, data model, risk logic, diagrams
│   └── TEST_CASES.md          # functional test cases
├── data/sample/               # simulated pumps, telemetry, repairs (CSV)
├── scripts/generate_sample_data.py
└── tests/test_sample_data.py
```

Add your application folders (for example `app/` for the API and `dashboard/` for the UI) as you build them.

## Prerequisites

- Python 3.10 or newer
- `pip` and (recommended) `venv`
- Git

## Setup

```bash
# 1. Clone
git clone [YOUR-REPO-URL]
cd pump-failure-early-warning

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment variables
cp .env.example .env               # Windows: copy .env.example .env
# then edit .env (defaults work for local use with ALERT_MODE=mock)
```

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `DATABASE_URL` | `sqlite:///./pumps.db` | Database connection (SQLite locally, PostgreSQL optional) |
| `DATA_DIR` | `data/sample` | Folder containing the CSV inputs |
| `ALERT_MODE` | `mock` | `mock` logs alerts only; `sms` or `whatsapp` sends them through a gateway |
| `SMS_GATEWAY_API_KEY` | empty | API key for your SMS provider (only if `ALERT_MODE=sms`) |
| `SMS_GATEWAY_SENDER_ID` | empty | Sender ID registered with the SMS provider |
| `WHATSAPP_API_TOKEN` | empty | Token for the WhatsApp Business API (only if `ALERT_MODE=whatsapp`) |
| `WHATSAPP_PHONE_NUMBER_ID` | empty | WhatsApp sender number ID |
| `DEFAULT_LANGUAGE` | `en` | Language code for alert templates |
| `RISK_THRESHOLD_MEDIUM` | `40` | Risk score (0-100) at which a pump is flagged Medium |
| `RISK_THRESHOLD_HIGH` | `70` | Risk score at which a pump is flagged High and an alert is sent |
| `ANOMALY_CONTAMINATION` | `0.05` | Expected share of anomalous readings for the anomaly detector |
| `LOG_LEVEL` | `INFO` | Logging verbosity |

Keep real API keys only in `.env`, which is git-ignored.

## Generate sample data and run the tests

```bash
python scripts/generate_sample_data.py     # regenerates data/sample/*.csv (seed 42)
pytest -q                                  # runs the data integrity tests
```

The sample set has 20 pumps, 90 days of daily readings (1,800 rows), and 7 repair records. Five pumps
degrade over the 14 days before failing, which gives you labelled examples to develop against.

## Run the prototype

[Update this section with your final commands, for example:]

```bash
uvicorn app.main:app --reload          # API at http://localhost:8000
streamlit run dashboard/app.py         # dashboard at http://localhost:8501
```

## How it works (summary)

1. Ingest pump readings and repair logs (CSV upload or API).
2. Build features: change vs the pump's own 30-day baseline, current-per-flow ratio, restarts per day.
3. Score risk (0-100) from rule-based checks combined with an anomaly-detection score.
4. Rank pumps by risk weighted by households served.
5. Alert operators by SMS/WhatsApp and show the ranked list to block engineers.
6. Operator replies "fixed"; the repair is stored as a label for tuning.

Full details and diagrams: [`docs/TECHNICAL_DESIGN.md`](docs/TECHNICAL_DESIGN.md). Test cases: [`docs/TEST_CASES.md`](docs/TEST_CASES.md).

## AI tools disclosure

[Fill in honestly: estimated % of work AI-assisted, which tasks (for example brainstorming, documentation drafting,
code scaffolding), and that the team reviewed and tested all outputs.]

## Limitations

- Prototype data is simulated; thresholds are illustrative and must be tuned on real pump data.
- Risk scores are decision support for operators and engineers, not a guarantee of failure or safety.

## License

[MIT](LICENSE)
