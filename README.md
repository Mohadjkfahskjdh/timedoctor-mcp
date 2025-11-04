# Time Doctor Scraper

**Production Ready MCP Server** for extracting Time Doctor time tracking data via Claude AI.

## What It Does

Fetches time tracking reports from Time Doctor and returns them as CSV data. Integrates directly with Claude Code as an MCP server.

**Features:**
- Single-session scraping (login once, get multiple dates)
- Works with any date range (1 day to 365+ days)
- Returns CSV data as text (no file writing issues)
- Parses Time Doctor's "3h 50m" format
- Aggregates duplicate tasks
- Includes TOTAL row

## Quick Setup

### 1. Install Dependencies

```bash
# Install UV (fastest)
brew install uv

# Setup
cd /Users/apple/tdoctorscraper
./setup-with-uv.sh
```

### 2. Configure Credentials

Create `.env`:
```bash
nano .env
```

Add:
```env
TD_EMAIL=your-email@example.com
TD_PASSWORD=your-password
TD_BASE_URL=https://2.timedoctor.com
HEADLESS=true
```

### 3. Add to Claude Code

**Option A: MCP Config in JSON**

Edit config:
```bash
# macOS
open ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

Add:
```json
{
  "mcpServers": {
    "timedoctor": {
      "command": "/Users/apple/tdoctorscraper/.venv/bin/python",
      "args": ["/Users/apple/tdoctorscraper/src/mcp_server.py"],
      "env": {
        "TD_EMAIL": "your-email@example.com",
        "TD_PASSWORD": "your-password",
        "TD_BASE_URL": "https://2.timedoctor.com",
        "HEADLESS": "true"
      }
    }
  }
}
```

**Option B: Let It Load from .env**

```json
{
  "mcpServers": {
    "timedoctor": {
      "command": "/Users/apple/tdoctorscraper/.venv/bin/python",
      "args": ["/Users/apple/tdoctorscraper/src/mcp_server.py"]
    }
  }
}
```

### 4. Restart Claude Code

Completely quit and reopen.

## Usage in Claude

**Get data for date range:**
```
Get my Time Doctor data from August 25 to September 5
```

**Today's data:**
```
Show me my Time Doctor hours for today
```

**Last week:**
```
Get my Time Doctor data for the last 7 days
```

**Custom analysis:**
```
Get my Time Doctor data from last month and tell me which project took the most time
```

Claude will receive CSV data like:
```csv
Date,Project,Task,Description,WORK HOUR
08/25/2025,Jira: AYR BMS,ABMS-606,Code Review,5.00
08/25/2025,Jira: AYR BMS,ABMS-700,Bug Fix,3.50
TOTAL,,,,8.50
```

You can then ask Claude to save it wherever you want!

## MCP Tools Available

### 1. `export_weekly_csv`
Get time tracking data for any date range in CSV format.

**Parameters:**
- `start_date` (required): YYYY-MM-DD format
- `end_date` (required): YYYY-MM-DD format

**Returns:** CSV data as text with summary statistics

### 2. `get_daily_report`
Get detailed report for a specific date.

**Parameters:**
- `date` (required): YYYY-MM-DD format or "today"

**Returns:** Formatted report with project breakdown

### 3. `get_hours_summary`
Quick hours breakdown by project.

**Parameters:**
- `date` (required): YYYY-MM-DD format or "today"

**Returns:** Summary by project

### 4. `export_today_csv`
Quick access to today's data.

**Parameters:** None

**Returns:** Today's CSV data with summary

## How It Works

1. **Login:** Playwright automation logs into Time Doctor web interface
2. **Navigate:** Goes to Projects & Tasks report
3. **Date Navigation:** Uses arrow buttons to change dates
4. **Extract:** Parses Angular Material tree structure for project/task data
5. **Parse:** Converts "3h 50m" format to decimal hours
6. **Aggregate:** Combines duplicate tasks
7. **Return:** CSV data as text in MCP response

## Architecture

**Single-Session Efficiency:**
- Login once
- Navigate through all dates in one session
- Extract data for each date
- Close browser once
- **Result:** 2-3x faster than logging in for each date

**CSV Data Format:**
```csv
Date,Project,Task,Description,WORK HOUR
11/04/2025,Jira: AYR BMS,ABMS-606,Code Review - ABMS-606,0.25
11/04/2025,Jira: AYR BMS,ABMS-4979,Deal template update,2.47
TOTAL,,,,2.72
```

- **Date:** MM/DD/YYYY format
- **WORK HOUR:** Decimal hours (5.00, 1.50, 0.25)
- **TOTAL:** Sum of all hours

## Project Structure

```
tdoctorscraper/
├── .env                    # Your credentials (git-ignored)
├── requirements.txt        # Python dependencies
├── setup-with-uv.sh       # Setup script
└── src/
    ├── scraper.py         # Browser automation & login
    ├── parser.py          # HTML parsing (Angular Material tree)
    ├── transformer.py     # CSV formatting
    └── mcp_server.py      # MCP server with 4 tools
```

## Troubleshooting

### MCP Server Won't Start

Check the log:
```bash
tail -f /Users/apple/tdoctorscraper/timedoctor_mcp.log
```

Common issues:
- ❌ **Missing credentials:** Add to `.env` or MCP config
- ❌ **Wrong Python path:** Use `.venv/bin/python` not system Python
- ❌ **Dependencies not installed:** Run `./setup-with-uv.sh`

### Login Fails

**Check log for:**
```
Login failed - still on login page
```

**Solutions:**
- Verify credentials in `.env` are correct
- Check if Time Doctor requires 2FA (not supported)
- Try logging in manually at https://2.timedoctor.com/login
- Look for error messages in the log

**The scraper now logs which email it's using:**
```
TimeDocorScraper initialized with email: your-email@example.com
```

### No Data Returned

**Check log for:**
```
Parsed 0 time entries
```

**Solutions:**
- Verify you have time tracking data for those dates
- Check that "Expand All" button appears on the report page
- Try with today's date first to confirm it works
- Check log for HTML parsing errors

### Claude Code Not Seeing Tools

**Solutions:**
1. Verify config path: `~/Library/Application Support/Claude/claude_desktop_config.json`
2. Check JSON syntax is valid
3. Completely quit and restart Claude Code
4. Check Claude Code settings → MCP Servers

## Requirements

- **Python:** 3.12 or 3.13 (3.14 not supported yet)
- **Dependencies:** Playwright, BeautifulSoup4, MCP SDK, python-dotenv
- **System:** macOS (tested), Linux, Windows (should work)
- **Time Doctor:** Active account with login credentials

## Performance

**Single-session architecture:**
- **7 days:** ~45 seconds (vs ~90 seconds with multiple logins)
- **30 days:** ~3 minutes
- **Login:** 5 seconds (once)
- **Per date:** 3-4 seconds (navigation + extraction)

## Security

- ✅ Credentials stored in `.env` (git-ignored)
- ✅ Headless browser (no GUI)
- ✅ Local execution only
- ✅ No data sent to third parties
- ⚠️ Store `.env` securely (chmod 600 recommended)

## Development

**Test login:**
```bash
source .venv/bin/activate
python debug_login.py
```

**Test parsing:**
```bash
python test_parser.py
```

**Test complete flow:**
```bash
python test_complete_flow.py
```

**View logs:**
```bash
tail -f timedoctor_mcp.log
```

## Why Web Scraping?

Time Doctor has an API, but this scraper uses web automation because:
- ✅ No API token setup needed
- ✅ Same access as web interface
- ✅ Works with all Time Doctor plans
- ✅ More reliable for "Projects & Tasks" report

If you prefer API access, Time Doctor's API is available at: `https://api2.timedoctor.com/api/1.0/`

## License

MIT License - Use freely for personal automation.

## Disclaimer

For personal use only. Ensure compliance with Time Doctor's Terms of Service and your organization's policies.

---

**Questions?** Check the log file: `/Users/apple/tdoctorscraper/timedoctor_mcp.log`

**Working?** Ask Claude: "Get my Time Doctor data for today"
