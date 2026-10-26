# Test Cases

Status: **Automated** = implemented in `tests/test_sample_data.py` (run `pytest -q`). **Planned** = to implement with the application code. Update the status column as you build.

## A. Data integrity (automated, sample data)

| ID | Scenario | Expected result | Status |
|---|---|---|---|
| D-01 | Load `pumps.csv` | 20 unique pump IDs with required columns | Automated |
| D-02 | Load `telemetry.csv` | Exact column list, no null values | Automated |
| D-03 | Check pump/day coverage | One row per pump per day; 90 rows per pump | Automated |
| D-04 | Cross-file IDs | Every telemetry and repair pump ID exists in `pumps.csv` | Automated |
| D-05 | Value ranges | No negative values; voltage 380-450 V; labels are 0 or 1 | Automated |
| D-06 | Downtime readings | Failed pumps read zero flow and run-hours from failure to repair date | Automated |
| D-07 | Label timing | Label = 1 only in the 7 days before a recorded failure | Automated |
| D-08 | Visible drift | Pre-failure current rise or flow drop above 5% vs baseline | Automated |

## B. Risk engine (planned)

| ID | Input | Expected result |
|---|---|---|
| R-01 | Healthy pump (readings near baseline) | Score below `RISK_THRESHOLD_MEDIUM`; band Low |
| R-02 | Pump with current +20% vs baseline and restarts 5/day | Band High; reasons mention current and restarts |
| R-03 | Pump with flow -25% vs baseline | Score above Medium; reason mentions flow drop |
| R-04 | Pump with fewer than 30 days of history | No crash; flagged "insufficient history" with low confidence |
| R-05 | Missing readings for one day | Day skipped in rolling windows; score still produced |
| R-06 | Zero-flow, zero-current day (pump down) | Treated as failed or offline, not scored as healthy |
| R-07 | Same input scored twice | Identical score (deterministic with fixed seed) |

## C. Ranking (planned)

| ID | Input | Expected result |
|---|---|---|
| K-01 | Two High pumps, one serving 400 households and one 100 | Larger-population pump ranks first |
| K-02 | Equal priority | Pump with older last-service date ranks first |
| K-03 | Filter by block | Only pumps from that block returned, in rank order |

## D. Alerts and feedback (planned)

| ID | Input | Expected result |
|---|---|---|
| A-01 | Pump enters High band, `ALERT_MODE=mock` | One alert logged with pump ID, reason and action |
| A-02 | Same pump still High next hour | No duplicate alert on the same day |
| A-03 | Unsupported `DEFAULT_LANGUAGE` | Falls back to English template |
| A-04 | Operator replies "FIXED" for a High pump | Repair stored; pump status reset; label saved |
| A-05 | Reply from unknown number | Rejected and logged; no data changed |
| A-06 | Gateway error | Alert retried, then marked failed; dashboard still updates |

## E. API and dashboard (planned)

| ID | Input | Expected result |
|---|---|---|
| P-01 | `POST /readings` with valid CSV | 200 OK; rows stored |
| P-02 | `POST /readings` with negative flow | 422 error with clear message |
| P-03 | `GET /ranking` | Pumps sorted by priority, highest first |
| P-04 | `GET /pumps/UNKNOWN` | 404 error |
| P-05 | Dashboard loads sample data | Ranked table and a trend chart render without errors |

## F. End-to-end demo scenario

1. Run `python scripts/generate_sample_data.py`.
2. Score all pumps on the day 7 days before the first simulated failure.
3. Confirm the failing pumps appear in the High band or near the top of the ranking.
4. Show the alert (mock mode), send a "FIXED" reply, and confirm the pump's status resets.
