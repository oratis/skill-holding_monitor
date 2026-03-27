# Holding Monitor - Claude Code Skill

A Claude Code skill that tracks your stock holdings and sends automated alerts at market open and close.

## Features

- **Multi-market support**: US (NYSE/NASDAQ), China (SSE/SZSE), Hong Kong (HKEX), Japan (TSE), UK (LSE)
- **Auto market detection**: Automatically identifies the stock exchange from ticker format
- **Scheduled alerts**: Get portfolio updates 10 minutes after market open and 10 minutes before market close
- **Simple data storage**: Holdings stored in a human-readable Markdown table

## Installation

1. Clone this repository into your Claude Code skills directory:

```bash
git clone https://github.com/your-username/holding-monitor.git ~/.claude/skills/holding-monitor
```

Or copy the `SKILL.md` and `holding.md` files to any directory and update the file paths in `SKILL.md` accordingly.

2. The skill will be automatically detected by Claude Code.

## Usage

### Add a stock

Simply tell Claude a stock ticker:

```
Add AAPL to my holdings
```

```
Add 600519 to my portfolio
```

```
Track 0700.HK
```

Claude will:
- Detect the stock market automatically
- Look up the company name
- Add it to `holding.md`
- Offer to set up open/close alerts

### Remove a stock

```
Remove AAPL from my holdings
```

### View holdings

```
Show my holdings
```

### Set up alerts

```
Set up alerts for my US stocks
```

This creates two scheduled tasks per market:
- **Open alert**: 10 minutes after market opens — shows overnight changes
- **Close alert**: 10 minutes before market closes — shows end-of-day status

## Supported Markets

| Market | Ticker Pattern | Example | Trading Hours (Local) |
|--------|---------------|---------|----------------------|
| US (NYSE/NASDAQ) | 1-5 uppercase letters | `AAPL`, `MSFT` | 09:30-16:00 ET |
| China Shanghai | 6-digit starting with 6 | `600519` | 09:30-15:00 CST |
| China Shenzhen | 6-digit starting with 0 or 3 | `000858`, `300750` | 09:30-15:00 CST |
| Hong Kong | 4-5 digits or `xxxx.HK` | `0700.HK`, `9988` | 09:30-16:00 HKT |
| Japan | 4 digits + `.T` | `9984.T`, `7203.T` | 09:00-15:00 JST |
| UK | Letters + `.L` | `VOD.L`, `SHEL.L` | 08:00-16:30 GMT |

## File Structure

```
holding-monitor/
├── SKILL.md      # Skill definition and instructions
├── holding.md    # Your holdings data (auto-created)
└── README.md     # This file
```

## How It Works

### holding.md

Your holdings are stored as a Markdown table:

```markdown
# My Holdings

| Ticker | Name | Market | Currency | Added Date |
|--------|------|--------|----------|------------|
| AAPL | Apple Inc. | US | USD | 2026-03-27 |
| 600519 | Kweichow Moutai | CN_SSE | CNY | 2026-03-27 |
```

### Scheduled Alerts

Alerts are created as Claude Code scheduled tasks. Each market gets two tasks:

- `holding-{market}-open` — fires 10 min after market open
- `holding-{market}-close` — fires 10 min before market close

Cron expressions are automatically converted to your local timezone.

**Alert content includes:**
- Current price for each holding
- Day change (absolute and percentage)
- Brief market sentiment summary

## Customization

Edit `SKILL.md` to:
- Add new markets
- Change alert timing (default: open+10min, close-10min)
- Modify the alert report format
- Add additional data columns to holdings

## Requirements

- [Claude Code](https://claude.com/claude-code) CLI
- Internet access (for stock price lookups via web search)

## License

MIT
