# The Bullion Bulletin

A self-grading South African macro & markets ledger. Every day it snapshots six core indicators (CPI y/y, SARB repo rate, ZAR/USD, JSE ALSI, gold, platinum) plus an extended board (prime rate, PPI, R2030/R209 bond yields, ZAR/EUR, ZAR/GBP, trade-weighted rand, Brent≈), writes a short analysis with a **specific, checkable forecast**, a policy recommendation, and an event-watch diary, and — once each forecast's deadline passes — grades it **hit / miss / partial** against what actually happened. An Analytics Desk derives the measures readers expect (real repo rate, term spread, curve slope, gold/platinum ratio, ALSI drawdown, rand momentum) from the ledger itself. The graded ledger is permanent and append-only.

Charts never mix units: the default view is grouped like-for-like panels; click any ticket, chip, or extended-board row to chart one series alone; the "Rebased = 100" view compares *moves* across units and unlocks once 3+ entries exist.

**Research & portfolio practice tool only. Not investment, trading, or policy advice.**

## The one file to open

**`dashboard.html`** — double-click it. Masthead, today's tickets and briefing, indicator trend charts, scoreboard, and the stamped ledger. It's regenerated after every write, so it always matches the database.

## How the daily run works

A Cowork Scheduled Task (`bullion-bulletin-daily`, daily at 08:02) tells Claude to: read recent entries for continuity → fetch real numbers (SARB API, gold-api.com; JSE ALSI via search, marked ≈approximate) → search current SA economic news → write today's entry → grade any forecast whose deadline has passed → rebuild the dashboard. All storage goes through `bulletin.py`, which validates units and refuses to ever modify a past entry.

Two things to know about Cowork scheduling:

1. **Runs fire only while your computer is awake and the Claude app is open.** A missed run fires on next launch rather than being lost — but if your laptop is never open around 08:00, change the time in the Scheduled sidebar to one that matches your routine.
2. **Set the model by hand once:** the scheduling API can't pin a model, and the brief calls for Claude Opus 4.8. Open the **Scheduled** sidebar → *bullion-bulletin-daily* → set model to **Claude Opus 4.8**. After the first automatic run, open that run's session and confirm it actually executed on Opus (there are reports of the selection not sticking).

You can also just ask Claude in any Cowork session to "file today's Bullion Bulletin entry" — the scheduled task's prompt lives at `~/Claude/Scheduled/bullion-bulletin-daily/SKILL.md`.

## Running the pieces yourself

From this folder (any machine with Python 3, no packages needed):

    python3 bulletin.py init             # create/upgrade the database
    python3 bulletin.py fetch            # try direct HTTP fetch of all indicators
    python3 bulletin.py ingest           # append today's entry (JSON on stdin — shape in bulletin.py)
    python3 bulletin.py recent 10        # last N entries with gradings
    python3 bulletin.py pending-grades   # forecasts due for grading
    python3 bulletin.py grade            # append a verdict (JSON on stdin)
    python3 bulletin.py scoreboard       # hit rate, streak, counts
    python3 bulletin.py render           # rebuild dashboard.html
    python3 bulletin.py status           # integrity check + counts

Note: inside Cowork's sandbox, direct HTTP is blocked, so `fetch` fails there by design — the daily prompt has Claude fetch the URLs in `config.json` with its own web tools and pass values to `ingest`. On your own machine, `fetch` works directly.

## How grading works

Each entry stores a `forecast_deadline` matching the forecast's own horizon (default 21 days). Once the deadline passes, `pending-grades` surfaces it; the daily run searches for what actually happened and files a one-line verdict with evidence. Verdicts are stamped on the ledger next to the *original, untouched* forecast text. The scoreboard counts partials at half-weight.

## Data integrity, in one paragraph

The database (`data/bulletin.db`, SQLite) has triggers that make `entries`, `readings`, and `gradings` physically append-only — UPDATE and DELETE fail at the database level. One entry per day, one grading per entry, no grading before deadline, and every numeric reading must pass a sanity range before it's stored. Back it up by copying the single `data/bulletin.db` file.

## Files

    bulletin.py           all app logic (CLI)
    render_dashboard.py   dashboard generator (called by `render`)
    config.json           data source registry + grading defaults
    data/bulletin.db      the ledger (SQLite, append-only)
    dashboard.html        generated view — open this
    DECISIONS.md          build + runtime decision log
    METHODOLOGY.md        how measurement and grading are defined
