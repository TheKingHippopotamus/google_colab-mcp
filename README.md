# Google Colab MCP 

This folder is a small sandbox to understand and test the new Google Colab MCP server, plus a playful “real” notebook project to see how well an agent can drive Colab end-to-end.

## What is the “Colab MCP” (Google)?

Google released an open-source **Colab MCP (Model Context Protocol) Server**. The core idea is: an MCP-compatible AI agent can **programmatically control Google Colab** (not just run code somewhere else).

Instead of copying snippets from a terminal into Colab cells, the agent can:

- Manage notebook dependencies (for example, installing libraries at runtime)
- Create and edit notebook content (including writing new `.ipynb` files and markdown cells)
- Write and execute Python cells in the notebook
- Reorganize content so the final notebook reads like a finished workflow

In short: Colab becomes a fast, reproducible, cloud sandbox that the agent can actively build inside.

## Repo setup (your local MCP server config)

This repo uses a standard MCP-style config file: `colabMcp/.mcp.json`.

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

## notebook
[Financial_Report_Analyzer.ipynb](/Financial_Report_Analyzer.ipynb)


### What it does

It fetches and analyzes **SEC EDGAR 10-Q financial data** end-to-end by:

1. Searching for a company (by name or ticker) to get its CIK
2. Fetching the most recent 10-Q filing history
3. Pulling structured financial metrics from SEC `companyfacts` (XBRL)
4. Building tables + multiple trend/risk charts
5. Optionally running a narrative analysis using a local LLM via **Ollama**

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

This is intentionally for fun: it’s a practical way to verify the Colab MCP server can drive a non-trivial notebook workflow—install dependencies, execute Python, and produce real artifacts—without manual copy/paste between local tooling and Colab. 

### All Rights Reserved - TheKingHippopotamus - 2026 
