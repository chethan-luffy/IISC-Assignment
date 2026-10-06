# Question D: Live dashboard with alerts

Seed **S = 2504**. Keep `setup/` and `question_D/` side by side (the producer reads
`../setup/readings.csv`; the dashboard overlays `../setup/anomaly_events.csv`).

## Install

```bash
pip install numpy pandas matplotlib streamlit plotly     # streamlit >= 1.37 (auto-refresh uses st.fragment)
```

Keep the database on a normal local disk (SQLite WAL mode does not work in some synced or network folders).

## Run the live system (3 terminals, from the repo root)

```bash
# 1. producer: replays readings.csv into the database at one row per second (creates the database)
python question_D/producer.py --reset            # add --speed 20 to watch it faster; --start 0 --rows 600 for a slice

# 2. alert engine: watches the database and writes alerts
python question_D/alert_engine.py

# 3. dashboard (opens in the browser, refreshes by itself)
streamlit run question_D/dashboard.py
```

Start the producer first, and do not use `--reset` while the engine is running.

## Checks

```bash
python question_D/test_alert_engine.py     # 7 unit tests: 5 s alert, 60 s cooldown, walking ignored, stuck sensor
python question_D/level2_verify.py         # both requirements on the real recording
# Level 3: commit question_D/PREDICTION.md FIRST, then
python question_D/level3_measure_delay.py            # 10x replay, about 2 min
python question_D/level3_measure_delay.py --speed 1  # real time, about 20 min
```

## Files

| File | Level | Notes |
|---|---|---|
| `schema.sql`, `db.py` | 1 | question_C schema plus `ingested_at`, `event_start_ts`, `detected_ts` |
| `dashboard.py` | 1 | Streamlit: reads SQL, plots HR and movement, marks alerts, shades true anomalies |
| `producer.py` | 2 | replays readings.csv at one row per second on a fixed schedule |
| `alert_engine.py` | 2 | own streaming alert logic: 5 s alert, 60 s cooldown, walking gate |
| `test_alert_engine.py`, `evaluate.py`, `level2_verify.py` | 2 | proof of the two requirements |
| `PREDICTION.md` | 3 | written and committed BEFORE the measurement |
| `level3_measure_delay.py` | 3 | delay table, average, worst, main source, fast-path comparison |
| `results/level3_delay_table.csv`, `level3_delay.png` | 3 | the measured evidence |

## Libraries used

numpy, pandas, matplotlib, streamlit, plotly, sqlite3, csv, threading (standard library).
The alert logic reuses the rolling z-score idea from `question_A/zscore_detector.py`, rewritten as a streaming class.
