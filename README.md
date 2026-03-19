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

`colabMcp/SEC_EDGAR_10Q_Downloader.ipynb` is the fun test project.

### What it does

It downloads all **SEC EDGAR 10-Q** filings for a configured date range by:

1. Using the **SEC EDGAR Full-Text Search API (EFTS)** to enumerate matching filings
2. For each match, downloading the primary filing HTML document
3. Saving the HTML + a `metadata.json` sidecar into a consistent folder structure

Later cells render some additional dashboards/visuals (dark-themed plots) based on the collected/processed data.

### Files/folders it outputs

The notebook is designed to save into:

```
/content/data/tickers/<TICKER>/<YYYY-MM-DD>/reports/10Q-<ticker>-<accession>.html
/content/data/tickers/<TICKER>/<YYYY-MM-DD>/reports/metadata.json
```

It also supports idempotent re-runs by skipping downloads when `metadata.json` already exists.

### How to run it (in Colab)

1. Open `SEC_EDGAR_10Q_Downloader.ipynb` in Google Colab
2. In **Cell 2**, set `SEC_USER_AGENT` to something like:
   - `"CompanyName admin@yourdomain.com"`
3. (Optional) Change `START_DATE` / `END_DATE`
4. Run cells from top to bottom

Because the SEC requires a descriptive `User-Agent`, leaving the placeholder value can lead to blocks or rate limiting.

## Packages used (quick list)

- `uv` + `git` + `python` (for running the MCP server)
- Notebook deps:
  - `edgartools`
  - `requests`
  - `numpy`
  - `matplotlib`

## Why this exists 

This is intentionally for fun: it’s a practical way to verify the Colab MCP server can drive a non-trivial notebook workflow—install dependencies, execute Python, and produce real artifacts—without manual copy/paste between local tooling and Colab. 

### All Rights Reserved - TheKingHippopotamus - 2026 
