# Google Colab MCP 

This folder is a small sandbox to understand and test the new Google Colab MCP server, plus a playful “real” notebook project (downloading SEC 10-Q filings) to see how well an agent can drive Colab end-to-end.

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

## The “small project test” notebook

`colabMcp/Financial_Report_Analyzer.ipynb` is the main fun test project (it already produces charts/dashboards and saves output files).

### What it does

It fetches and analyzes **SEC EDGAR 10-Q quarterly financial reports** and then visualizes the results.

High-level flow:

1. Searches for a company (by name or ticker) to get the required `CIK`
2. Retrieves the company’s most recent 10-Qs
3. Extracts key financial metrics (revenue, net income, assets, etc.)
4. Builds trend charts
5. Produces a dark-themed, 4-panel dashboard (radar + scores + risk landscape + outlook)

### Outputs it saves (so you can “see results”)

Running the notebook writes files into the Colab `/content` folder, including:

```text
/content/{TICKER}_10Q_financial_data.csv
/content/{TICKER}_ai_dashboard.png
/content/comparison_dashboard.png
```

### How to run it (in Colab)

1. Open `Financial_Report_Analyzer.ipynb` in Google Colab
2. Edit the company inputs in the notebook (for example `SEARCH_QUERY` and the selected company index)
3. Make sure the SEC `User-Agent` header is set to your real name/email (there is a placeholder in the code)
4. Run cells from top to bottom

## Packages used (quick list)

- `uv` + `git` + `python` (for running the MCP server)
- Notebook deps:
  - `edgartools`
  - `requests`
  - `numpy`
  - `matplotlib`

## Why this exists 

This is intentionally for fun: it’s a practical way to verify the Colab MCP server can drive a non-trivial notebook workflow—install dependencies, execute Python, and produce real artifacts—without manual copy/paste between local tooling and Colab.

## Optional: bulk downloader

If you also want to “collect raw filings” first, see `colabMcp/SEC_EDGAR_10Q_Downloader.ipynb`, which downloads 10-Q HTML + `metadata.json` into a structured folder under `/content/data/tickers/...`.

### All Rights Reserved - TheKingHippopotamus - 2026 
