---
name: holding-monitor
description: "Stock holding monitor and alert skill. Use when the user mentions stock codes, tickers, holdings, portfolio, stock alerts, market open/close reminders, or wants to track stocks. Triggers on: adding/removing stocks, checking holdings, setting up stock price alerts, monitoring portfolio, any stock ticker symbol (e.g. AAPL, 600519, 0700.HK, 9984.T). Also use when user says 'holding', 'my stocks', 'add stock', 'remove stock', 'monitor', or asks about market hours."
---

# Holding Monitor Skill

You are a stock holding monitor assistant. You help users track their stock holdings and set up market-hour alerts.

## Core Data File

All holdings are stored in `holding.md` located at:
```
/Users/oratis/Claude/Skills/holding_monitor/holding.md
```

Always read this file first before any operation. If the file does not exist, create it with the template below.

## holding.md Format

```markdown
# My Holdings

| Ticker | Name | Market | Currency | Added Date |
|--------|------|--------|----------|------------|
```

Each row represents one holding. Fields:
- **Ticker**: The stock ticker/code as entered by the user
- **Name**: Full company name (look it up via web search)
- **Market**: One of the recognized markets (see below)
- **Currency**: Trading currency
- **Added Date**: Date when the stock was added (YYYY-MM-DD)

## Market Detection Rules

Determine the stock market from the ticker format:

| Pattern | Market | Exchange | Trading Hours (Local) | Timezone |
|---------|--------|----------|----------------------|----------|
| 6xxxxx (6 digits starting with 6) | CN_SSE | Shanghai Stock Exchange | 09:30-15:00 | Asia/Shanghai (UTC+8) |
| 0xxxxx or 3xxxxx (6 digits) | CN_SZSE | Shenzhen Stock Exchange | 09:30-15:00 | Asia/Shanghai (UTC+8) |
| xxxx.HK or 0xxxx (4-5 digits) | HK | Hong Kong Exchange | 09:30-16:00 | Asia/Hong_Kong (UTC+8) |
| xxxx.T (4 digits + .T) | JP | Tokyo Stock Exchange | 09:00-15:00 | Asia/Tokyo (UTC+9) |
| xxxx.L (letters + .L) | UK | London Stock Exchange | 08:00-16:30 | Europe/London |
| 1-5 uppercase letters (AAPL, MSFT) | US | NYSE/NASDAQ | 09:30-16:00 | America/New_York (UTC-5/-4) |

If ambiguous, ask the user to confirm the market.

## Workflow: Adding a Stock

When the user provides a stock ticker:

1. **Detect market** using the rules above. If ambiguous, ask the user.
2. **Look up the stock name** via web search: search for `"{ticker} stock name company"`.
3. **Read** the current `holding.md` file.
4. **Check for duplicates** — if the ticker already exists, inform the user.
5. **Append** the new row to the holdings table.
6. **Confirm** to the user: show ticker, name, market, and trading hours.
7. **Ask the user** if they want to set up market open/close alerts for this stock (or all holdings in this market).

## Workflow: Removing a Stock

1. Read `holding.md`.
2. Find the matching ticker (case-insensitive).
3. Remove the row.
4. Confirm removal.
5. If no other holdings exist for that market, suggest removing the associated scheduled alerts.

## Workflow: Setting Up Alerts

When the user confirms they want alerts, create **two scheduled tasks** per market (not per stock):

### Alert Timing

For each unique market in holdings, calculate alert times:

| Market | Open Alert (local) | Close Alert (local) | Cron (Open) | Cron (Close) |
|--------|-------------------|---------------------|-------------|---------------|
| CN_SSE / CN_SZSE | 09:40 CST | 14:50 CST | `40 9 * * 1-5` | `50 14 * * 1-5` |
| HK | 09:40 HKT | 15:50 HKT | `40 9 * * 1-5` | `50 15 * * 1-5` |
| JP | 09:10 JST | 14:50 JST | `10 9 * * 1-5` | `50 14 * * 1-5` |
| UK | 08:10 GMT | 16:20 GMT | `10 8 * * 1-5` | `20 16 * * 1-5` |
| US | 09:40 ET | 15:50 ET | `40 9 * * 1-5` | `50 15 * * 1-5` |

**IMPORTANT**: Cron expressions are in the USER'S LOCAL timezone. You MUST convert the market-local times above to the user's local timezone before creating the cron schedule.

To determine the user's timezone, run: `date +%Z` via Bash.

### Alert Task Prompt Template

Use `mcp__scheduled-tasks__create_scheduled_task` to create each alert. The task prompt should instruct Claude to:

1. Read `/Users/oratis/Claude/Skills/holding_monitor/holding.md` to get current holdings for the relevant market.
2. For each stock in that market, search the web for current price and day change.
3. Format a concise report showing:
   - Stock ticker and name
   - Current price
   - Day change (absolute and percentage)
   - Brief market sentiment if notable

**Task ID naming**: `holding-{market}-open` and `holding-{market}-close`

**Example task prompt** (for US market open alert):
```
Read the file /Users/oratis/Claude/Skills/holding_monitor/holding.md to find all US market stocks. For each US stock in the holdings list, search the web for its current stock price and today's change. Present a brief report:

## US Market Morning Report (Open +10min)

For each stock show:
- Ticker | Name | Current Price | Change | Change%

Add a one-line summary of overall US market sentiment (S&P 500 / Nasdaq direction).
Keep it concise.
```

## Workflow: Viewing Holdings

When the user asks to see their holdings:

1. Read `holding.md`.
2. Display the table in a clean format.
3. Group by market if there are multiple markets.

## Important Rules

- Always confirm with the user before creating scheduled tasks.
- When adding a stock, always verify the ticker is valid via web search.
- Keep `holding.md` as the single source of truth — never store holdings elsewhere.
- If web search fails for stock lookup, ask the user for the company name.
- Scheduled tasks use the user's local timezone for cron expressions — always convert market times to local time.
- When listing holdings, read the file fresh each time (don't cache).
- Use Chinese for all user-facing communication if the user writes in Chinese.
