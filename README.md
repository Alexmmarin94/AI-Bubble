# Bogle & AI Bubble — Deterministic Portfolio Insights Dashboard

This repository contains the codebase for **Bogle & AI Bubble**, a Streamlit dashboard that consolidates a broad set of market indicators and transforms them into **transparent, rules-based portfolio insights** for a classic **Bogle-style allocation** (e.g., VTI / VXUS / BND) under elevated uncertainty—where a potential **AI-cycle bubble / fragility regime** is one of the main risks considered.

The emphasis is **not** on “smart algorithms” or machine learning. It is a concept-first system: define the decision problem clearly, collect the relevant signals, encode the logic deterministically, and present outputs in a way that is usable and inspectable.

> **Disclaimer (not investment advice):** This is a personal research project and provides informational insights only. Nothing here is a recommendation to buy/sell securities or to time the market.

---

## 🔗 Live Demo

**👉 Streamlit App:** https://bogleai-bubble-3gnctcftcpciv3izgvssyt.streamlit.app/

> ⚠️ Streamlit Community Cloud apps may sometimes look inactive due to sleep/idle behavior.  
> This project includes **daily automated refresh** (GitHub Actions) to keep datasets up to date, so it is likely to be active most days.  
> If you ever see it sleeping and want it woken up for testing, contact: **alexbmeist@gmail.com**.

---

## What This Project Is (and Is Not)

- ✅ A **deterministic, reproducible** pipeline that turns multiple public data inputs into **banded allocation insights**.
- ✅ A dashboard that makes the **“why”** inspectable: drivers, persistence rules, and known limitations.
- ✅ A **backtest-style simulator** that translates the same recommendation logic into actions for a hypothetical portfolio under DCA.

- ❌ Not a promise of outperformance.  
- ❌ Not a system that claims it can reliably “predict crashes” or do market timing.  
- ❌ Not a machine learning project (by design).  

---

## Dashboard Overview

The app is organized into four tabs:

### 1) Daily Snapshot
A quick read of the current regime and **persisted signals**. The signals shown here are the system’s **allocation recommendations** (subject to persistence rules and banding), with supporting drivers.

### 2) AI Bubble Diagnostics
Leadership, concentration, and fragility diagnostics combined into an **AI Bubble Score** and an amplified **Crash Risk**. These are **additional insights** intended to reinforce (or challenge) the guardrails implied by the Daily Snapshot.

### 3) Portfolio Actions & Backtest
Materializes the reallocation recommendations using a hypothetical **VTI/VXUS/BND** portfolio and a fixed **DCA schedule** (contributions on day 1 & 15; invested on the next trading day close). It also compares how those actions would have performed vs a benchmark under the same cashflows.

### 4) Resources
Supporting reading and definitions that help answer “why” questions, such as:
- why certain actionables are framed as **Bogle-compatible** (vs. market timing),
- why some market conditions can justify **higher monitoring** even under a Bogle philosophy,
- and deeper notes on the **technical framework** and known limitations for anyone who wants to inspect details.

---

## 🖼️ Visual Preview (In Case the Demo Is Unavailable)

Screenshots are included in `Screenshots/` (as `Screenshot_1.png` … `Screenshot_8.png`) in the same order as below:

1) Home navigation (4 cards + “Open” buttons)  
![Screenshot 1](Screenshots/Screenshot_1.png)

2) Daily Snapshot (stress + drivers + allocation summaries)  
![Screenshot 2](Screenshots/Screenshot_2.png)

3) AI Bubble Diagnostics (scores + interpretability)  
![Screenshot 3](Screenshots/Screenshot_3.png)

4) Band guidance and drivers  
![Screenshot 4](Screenshots/Screenshot_4.png)

5) Stress amplifiers time series  
![Screenshot 5](Screenshots/Screenshot_5.png)

6) Portfolio Actions & Backtest (allocation translation)  
![Screenshot 6](Screenshots/Screenshot_6.png)

7) Backtest overview (full period)  
![Screenshot 7](Screenshots/Screenshot_7.png)

8) Backtest zoom (COVID shock example window)  
![Screenshot 8](Screenshots/Screenshot_8.png)

---

## Data Inputs (High Level)

The pipeline pulls and maintains a curated dataset from multiple public sources, such as:

- **Market prices** (ETFs used as benchmarks / sleeves)
- **Macro and rates** (yields, spreads, volatility proxies)
- **Public SDMX sources** (ECB / IMF endpoints)
- **Context-only fundamentals** (SEC feeds used as contextual inputs, where applicable)

Exact series and transformations are documented in the **Resources** tab and in the technical markdowns under `content/learn/`.

---

## Processing Pipeline (Daily Updates)

A GitHub Actions workflow runs a full refresh pipeline. At a high level:

1. Backfill / update raw sources (incrementally).
2. Build derived datasets used by the app (Parquet/JSON “state” files).
3. Commit updated outputs back to the repository so Streamlit Cloud can render the latest state.

The orchestration entrypoint is:

- `scripts/99_update_all.py`

Typical outputs include:

- `data/state/daily_state/*.json`
- `data/state/daily_state_history/*.parquet`
- `data/state/portfolio_targets_history/*.parquet`
- `data/state/backtests/*.parquet`

> Note: daily state is constrained by **trading days** and by upstream data publication cadence.  
> If markets are closed or an upstream source hasn’t published new values yet, a “today” file may not be produced.

---

## Folder and File Structure (Simplified)

```text
root/
│
├── app/
│   ├── Home.py                       # Streamlit entrypoint (homepage)
│   └── pages/
│       ├── 01_Daily_Snapshot.py
│       ├── 02_AI_Bubble_Diagnostics.py
│       ├── 03_Portfolio_Actions_Backtest.py
│       └── 04_Learn_Glossary.py      # “Resources” tab (markdown library)
│
├── content/
│   └── learn/                        # Markdown resources rendered in the app
│
├── data/
│   ├── raw/                          # Cached raw inputs (some are large / externalized)
│   └── state/                        # Derived state consumed by the Streamlit UI
│
├── scripts/                          # Data refresh + state builders
├── .github/workflows/                # Daily refresh workflow
├── requirements.txt
└── README.md
```

---

## Secrets and Configuration

This project supports three execution environments:

### 1) Local development
Create `.streamlit/secrets.toml` (not committed) with keys and config, for example:

```toml
# Core APIs
FRED_API_KEY = "..."
TIINGO_API_KEY = "..."
SEC_USER_AGENT = "Bogle_AI-Bubble/0.1 (contact: you@example.com)"

# SDMX endpoints
ECB_SDMX_BASE_URL = "https://data-api.ecb.europa.eu/service"
IMF_SDMX_BASE_URL = "https://sdmxcentral.imf.org/ws/public/sdmxapi/rest"

# App config
APP_ENV = "local"
CACHE_TTL_SECONDS = 3600
HTTP_TIMEOUT_SECONDS = 30
HTTP_MAX_RETRIES = 3
HTTP_BACKOFF_SECONDS = 1.0
```

### 2) GitHub Actions (daily refresh)
Secrets are stored as a **single TOML secret** and written at runtime into `.streamlit/secrets.toml` before running the refresh scripts.

### 3) Streamlit Community Cloud
Define the same variables in the Streamlit app settings (`Advanced settings → Secrets`) using TOML format.

---

## Large Files (IMF Structures XML)

Some IMF “structures” XML files can exceed GitHub’s per-file limit. The approach used here is:

- Store compressed XML assets in **GitHub Release assets**
- Download them during the GitHub Actions refresh job
- Cache/refresh them as needed as part of the pipeline

This keeps the repo lightweight while preserving reproducibility.

---

## Local Setup

```bash
# Create and activate a virtualenv (example)
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Add secrets
mkdir -p .streamlit
# Create .streamlit/secrets.toml with your keys (see section above)

# Run the app
streamlit run app/Home.py
```

---

## Deployment (Streamlit Cloud)

- Connect the GitHub repo in Streamlit Cloud
- Set:
  - **Main file path**: `app/Home.py`
  - **Python version**: as supported by Streamlit Cloud (typically 3.11)
- Add your TOML secrets in Streamlit Cloud settings
- Deploy

---

## Why This Is a Useful Portfolio Piece (Fintech / Product Analytics Lens)

This project intentionally prioritizes **product thinking** over “complexity for its own sake”:

- The system is **inspectable**: deterministic rules, explicit drivers, clear persistence logic, and documented limitations.
- It converts a broad and messy set of signals into **practical outputs**: banded recommendations, a consistent narrative, and a backtest-based sanity check.
- It demonstrates an end-to-end workflow: data ingestion → state building → automated refresh → UI delivery.

In many real fintech settings, the hard part is not “adding ML,” but **defining the problem correctly**, building the right guardrails, and communicating outputs responsibly.

---

## Contact

**Alex (Alexmmarin94)** — alexbmeist@gmail.com
