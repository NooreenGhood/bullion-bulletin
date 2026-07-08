# The Bullion Bulletin

A self-grading South African macro and markets ledger. Every day it snapshots six core indicators (CPI y/y, SARB repo rate, ZAR/USD, JSE ALSI, gold, platinum) plus an extended board (prime rate, PPI, R2030/R209 bond yields, ZAR/EUR, ZAR/GBP, trade-weighted rand, Brent≈), records a short analysis with a **specific, checkable forecast**, a policy recommendation, and an event-watch diary, and once each forecast's deadline passes it grades the forecast **hit / miss / partial** against what actually happened. An Analytics Desk derives the measures readers expect (real repo rate, term spread, curve slope, gold/platinum ratio, ALSI drawdown, rand momentum) from the ledger itself. The graded ledger is permanent and append-only.

Charts never mix units: the default view is grouped like-for-like panels; click any ticket, chip, or extended-board row to chart one series alone; the "Rebased = 100" view compares *moves* across units and unlocks once 3+ entries exist.

**Research and portfolio practice tool only. Not investment, trading, or policy advice.**

## The one file to open

**`dashboard.html`**, double-click it. Masthead, today's tickets and briefing, indicator trend charts, scoreboard, and the stamped ledger. It is regenerated after every write, so it always matches the database.

## How the daily run works

An automated daily run (08:02) does the following: reads recent entries for continuity, gathers real numbers (SARB API, gold-api.com; JSE ALSI via search, marked ≈approximate), reviews current SA economic news, records the day's entry, grades any forecast whose deadline has passed, and rebuilds the dashboard. All storage goes through `bulletin.py`, which validates units and refuses to ever modify a past entry.

## Running the pieces yourself

From this folder (any machine with Python 3, no packages needed):

    python3 bulletin.py init             # create/upgrade the database
    python3 bulletin.py fetch            # try direct HTTP fetch of all indicators
    python3 bulletin.py ingest           # append today's entry (JSON on stdin, shape in bulletin.py)
    python3 bulletin.py recent 10        # last N entries with gradings
    python3 bulletin.py pending-grades   # forecasts due for grading
    python3 bulletin.py grade            # append a verdict (JSON on stdin)
    python3 bulletin.py scoreboard       # hit rate, streak, counts
    python3 bulletin.py render           # rebuild dashboard.html
    python3 bulletin.py status           # integrity check + counts

Note: in some sandboxed environments direct HTTP is blocked, so `fetch` fails there by design; in that case the day's values are gathered separately and passed to `ingest`. On your own machine, `fetch` works directly.

## How grading works

Each entry stores a `forecast_deadline` matching the forecast's own horizon (default 21 days). Once the deadline passes, `pending-grades` surfaces it; the daily run finds what actually happened and files a one-line verdict with evidence. Verdicts are stamped on the ledger next to the *original, untouched* forecast text. The scoreboard counts partials at half-weight.

## Data integrity, in one paragraph

The database (`data/bulletin.db`, SQLite) has triggers that make `entries`, `readings`, and `gradings` physically append-only: UPDATE and DELETE fail at the database level. One entry per day, one grading per entry, no grading before deadline, and every numeric reading must pass a sanity range before it is stored. Back it up by copying the single `data/bulletin.db` file.

## Files

    bulletin.py           all app logic (CLI)
    render_dashboard.py   dashboard generator (called by `render`)
    config.json           data source registry + grading defaults
    data/bulletin.db      the ledger (SQLite, append-only)
    dashboard.html        generated view, open this
    DECISIONS.md          build and runtime decision log
    METHODOLOGY.md        how measurement and grading are defined
