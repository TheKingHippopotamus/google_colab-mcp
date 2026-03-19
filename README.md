# Google Colab MCP 

This folder is a small sandbox to understand and test the new Google Colab MCP server, plus a playful  notebook project to see how well an agent can drive Colab end-to-end.

## What is the "Colab MCP" (Google)?

Google released an open-source **Colab MCP (Model Context Protocol) Server**. The core idea is: an MCP-compatible AI agent can **programmatically control Google Colab** (not just run code somewhere else).

Instead of copying snippets from a terminal into Colab cells, the agent can:

- Manage notebook dependencies (for example, installing libraries at runtime)
- Create and edit notebook content (including writing new `.ipynb` files and markdown cells)
- Write and execute Python cells in the notebook
- Reorganize content so the final notebook reads like a finished workflow

In short: Colab becomes a fast, reproducible, cloud sandbox that the agent can actively build inside.

## Repo setup (your local MCP server config)

This repo uses a standard MCP-style config file: `colabMcp/.mcp.json`

Current config:

```json
{
  "mcpServers": {
    "colab-mcp": {
      "command": "uvx",
      "args": ["git+https://github.com/googlecolab/colab-mcp"],
      "timeout": 30000
    }
  }
}
```

So, when your MCP-capable client starts the server, it uses `uvx` to run the `googlecolab/colab-mcp` package.

Prerequisites (as described by the upstream project/blog):
- `uv`
- `git`
- `python`

---

## Notebook — SEC EDGAR 10-Q Financial Report Analyzer

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/TheKingHippopotamus/google_colab-mcp/blob/main/Financial_Report_Analyzer.ipynb)
[![nbviewer](https://img.shields.io/badge/render-nbviewer-orange?logo=jupyter)](https://nbviewer.org/github/TheKingHippopotamus/google_colab-mcp/blob/main/Financial_Report_Analyzer.ipynb)

### What it does

It fetches and analyzes **SEC EDGAR 10-Q financial data** end-to-end by:

1. Searching for a company (by name or ticker) to get its CIK
2. Fetching the most recent 10-Q filing history
3. Pulling structured financial metrics from SEC `companyfacts` (XBRL)
4. Building tables + multiple trend/risk charts
5. Optionally running a narrative analysis using a local LLM via **Ollama**

### Notebook sections at a glance

> You can view the fully rendered notebook (with outputs & charts) directly on nbviewer using the badge above, or expand the preview below.

| # | Section | Description |
|---|---------|-------------|
| 1 | **Company Search** | Look up any public company by name or ticker |
| 2 | **Select Company & Fetch 10-Q Filings** | Choose from results and pull filing history |
| 3 | **Extract Financial Data from Company Facts** | Parse XBRL data for key metrics (Revenue, Net Income, Assets, EPS, etc.) |
| 4 | **Financial Data Table** | Tabular summary of quarterly financials |
| 5 | **Revenue & Profitability Trend Charts** | Matplotlib visualizations of revenue, income, and profitability over time |
| 6 | **Quarter-over-Quarter Growth Analysis** | Calculate and chart QoQ growth percentages |
| 7 | **Key Financial Ratios** | Compute ratios like profit margin, asset-to-liability, etc. |
| 8 | **YoY Growth Comparison Chart** | Year-over-year comparison visualizations |
| 9 | **Filing Links & Raw Data Access** | Direct links to SEC filings and raw data |
| 10 | **Ollama AI Gateway — Setup** | Install and configure a local LLM (Ollama) on Colab |
| 11 | **Ollama AI Gateway — Client** | Python client to interact with the Ollama API |
| 12 | **AI Financial Analysis** | LLM-generated narrative analysis of the financial data |
| 13 | **Parallel Analysis Demo** | Batch-mode analysis of multiple companies simultaneously |
| 14 | **AI-to-Chart Algorithm** | Automated radar chart, scorecard, and risk heatmap from AI output |
| 15 | **Parallel Multi-Company Analysis** | Side-by-side comparison dashboard across companies |

<details>
<summary><strong>Preview: Setup & Imports (click to expand)</strong></summary>

```python
# Install dependencies
!pip install requests pandas matplotlib plotly -q

import requests
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import json
import time
from datetime import datetime

# SEC EDGAR requires a User-Agent header with contact info
HEADERS = {
    "User-Agent": "SECAnalyzer/1.0 (research@example.com)",
    "Accept": "application/json"
}

BASE_URL = "https://efts.sec.gov/LATEST"
DATA_URL = "https://data.sec.gov"
```

</details>

<details>
<summary><strong>Preview: Company Search function (click to expand)</strong></summary>

```python
def search_company(query, max_results=10):
    """Search for a company by name or ticker using SEC EDGAR."""
    url = f"{BASE_URL}/search-index?q={query}&dateRange=custom&startdt=2020-01-01&forms=10-Q"
    resp = requests.get(url, headers=HEADERS)
    tickers_url = "https://www.sec.gov/files/company_tickers.json"
    tickers_resp = requests.get(tickers_url, headers=HEADERS)
    tickers_data = tickers_resp.json()

    results = []
    query_lower = query.lower()
    for key, entry in tickers_data.items():
        name = entry.get("title", "").lower()
        ticker = entry.get("ticker", "").lower()
        if query_lower in name or query_lower == ticker:
            results.append({
                "cik": str(entry["cik_str"]).zfill(10),
                "ticker": entry.get("ticker", "N/A"),
                "name": entry.get("title", "Unknown")
            })
        if len(results) >= max_results:
            break
    return results
```

</details>

<details>
<summary><strong>Preview: Financial Metrics Extraction (click to expand)</strong></summary>

```python
# Key financial metrics to extract from 10-Q filings
METRICS = {
    "Revenues": ["Revenues", "RevenueFromContractWithCustomerExcludingAssessedTax",
                  "SalesRevenueNet", "RevenueFromContractWithCustomerIncludingAssessedTax"],
    "NetIncome": ["NetIncomeLoss", "ProfitLoss"],
    "TotalAssets": ["Assets"],
    "TotalLiabilities": ["Liabilities"],
    "StockholdersEquity": ["StockholdersEquity",
                           "StockholdersEquityIncludingPortionAttributableToNoncontrollingInterest"],
    "CashAndEquivalents": ["CashAndCashEquivalentsAtCarryingValue",
                           "CashCashEquivalentsAndShortTermInvestments"],
    "OperatingIncome": ["OperatingIncomeLoss"],
    "EarningsPerShare": ["EarningsPerShareBasic"],
    "GrossProfit": ["GrossProfit"],
}
```

</details>

### Results it saves (Colab)

The notebook writes its outputs into:

- `/content/{TICKER}_10Q_financial_data.csv`
- `/content/{ticker}_ai_dashboard.png` (per-company AI dashboard)
- `/content/comparison_dashboard.png` (multi-company comparison dashboard)

### How to run it (in Colab)

1. Open `Financial_Report_Analyzer.ipynb` in Google Colab
2. Ensure the SEC `User-Agent` header includes your contact info (the notebook includes a placeholder you should replace if needed)
3. Run cells from top to bottom

Because the SEC requires a descriptive `User-Agent`, leaving any placeholder can lead to blocks or rate limiting.

## Packages used (quick list)

- `uv` + `git` + `python` (for running the MCP server)
- Notebook deps:
  - `requests`
  - `pandas`
  - `matplotlib`
  - `plotly`
  - plus Ollama install/setup steps (inside the notebook)

## Why this exists

This is intentionally for fun: it's a practical way to verify the Colab MCP server can drive a non-trivial notebook workflow—install dependencies, execute Python, and produce real artifacts—without manual copy/paste between local tooling and Colab.

### All Rights Reserved — TheKingHippopotamus — 2026
