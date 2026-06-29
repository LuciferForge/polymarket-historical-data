# Polymarket Historical Data

**18.6M+ price snapshots. 22,410 markets. 92 days of history (2026-03-28 → 2026-06-28). 15-minute resolution.**

The complete dataset behind a [public live-traded strategy](https://github.com/LuciferForge/polymarket-crash-bot) (302 trades, 79.8% win rate). Free samples on Hugging Face. Full dataset and live API on Gumroad / [api.protodex.io](https://api.protodex.io).

> Every figure above counts the **downloadable SQLite export** (last refreshed 2026-06-28) — what you download is exactly that file, not an estimate. The separate **live API** ([api.protodex.io](https://api.protodex.io)) is refreshed every 15 minutes; the downloadable archive grows ~200k rows/day between export refreshes.
>
> **Honest note on the order-book table:** ~94% of `orderbooks` rows are nominal placeholders (`best_bid` 0.001 / `best_ask` 0.999 — Polymarket's long-tail markets are genuinely thin, with no resting bids/asks at most 15-min polls). Only ~6% (~102K rows) carry a non-placeholder quote, and even those are wide-spread thin-market prints. **The price series is the dense, reliable layer and the reason to buy this — don't buy this for the order book.** [Full audit write-up.](https://dev.to/manja316/88-of-the-order-book-rows-in-my-dataset-were-fake-heres-how-i-caught-it-4hn8)

## Coverage

| Table | Rows | Description |
|-------|------|-------------|
| `markets` | 22,410 | Question, category, volume_24h, liquidity, end_date |
| `prices` | 18,611,636 | 15-min snapshots for YES/NO outcomes |
| `orderbooks` | 1,856,388 | Top-of-book snapshots — most rows are 0.001/0.999 thin-market placeholders (see note above); the price series is the dense layer |

- **Source:** Polymarket Gamma + CLOB APIs (no scraping, no proprietary data)
- **Update cadence:** every 15 minutes via a ForgeOS launchd job
- **Categories:** sports (12,285), crypto (2,515), politics (1,441), geopolitics (688), science/tech (241), economics (224), and more
- **Format:** SQLite (single file, queryable from any language) + daily Parquet export

### What you can verify in one query

Of the **19,402 ended markets** that carry a last trade price, **94.6%** closed decisively (last YES price ≥0.95 or ≤0.05) and only 0.9% were still a coin-flip (0.40–0.60). Markets reach consensus almost every time:

```sql
SELECT ROUND(100.0*AVG(last_trade_price>=0.95 OR last_trade_price<=0.05),1) AS pct_decisive
FROM markets WHERE end_date < date('now') AND end_date != '' AND last_trade_price IS NOT NULL;
-- 94.6
```

**Honest scope note:** this is price *convergence*, not *calibration*. The dataset has **no resolution labels** (`resolved = 0` for all rows) — it's a 15-min price collector, not a settlement feed. So calibration curves, favorite-longshot bias, and Brier scores are **not computable** from this file without an external on-chain resolution join. Price trajectories, crash-bounce, spread, and correlation work **are**.

## Get the data

| Tier | Price | Format | Where |
|------|-------|--------|-------|
| **Free sample (1 day)** | $0 | SQLite + CSV | [Hugging Face](https://huggingface.co/datasets/manja316/polymarket-historical-prices) |
| **Cross-signal sample** (BTC/ETH/SOL + Polymarket probabilities) | $0 | CSV | [Hugging Face](https://huggingface.co/datasets/manja316/crypto-prediction-market-signals) |
| **Full historical dataset** (one-time download) | $19 | SQLite | [Gumroad](https://manja8.gumroad.com/l/polymarket-quant-toolkit?utm_source=github&utm_medium=readme&utm_campaign=polymarket-data-2026-06-29) |
| **Live API** (no download, query directly) | Free 100/day · $19/mo Pro | HTTP/JSON | [api.protodex.io](https://api.protodex.io) |
| Live auto-updating feed (weekly refresh) | $19/mo | SQLite | [Gumroad](https://manja8.gumroad.com/l/polymarket-feed?utm_source=github&utm_medium=readme&utm_campaign=polymarket-data-2026-06-29) |

## Quick start (Python)

After downloading the SQLite file:

```python
import sqlite3, pandas as pd

con = sqlite3.connect("polymarket.db")

# Top markets by 24h volume
df = pd.read_sql("""
  SELECT id, question, category, volume_24h, liquidity, end_date
  FROM markets
  WHERE active = 1
  ORDER BY volume_24h DESC
  LIMIT 20
""", con)

# 15-min prices for one market
prices = pd.read_sql("""
  SELECT outcome, price, ts
  FROM prices
  WHERE market_id = ?
  ORDER BY ts
""", con, params=("0x...",))
```

## Schema

```sql
CREATE TABLE markets (
    id TEXT PRIMARY KEY,
    slug TEXT,
    question TEXT,
    category TEXT,
    volume_24h REAL,
    liquidity REAL,
    end_date TEXT,
    active INTEGER
);

CREATE TABLE prices (
    market_id TEXT,
    outcome TEXT,         -- 'YES' or 'NO'
    price REAL,           -- 0.0 to 1.0
    ts TIMESTAMP,
    PRIMARY KEY (market_id, outcome, ts)
);

CREATE TABLE orderbooks (
    market_id TEXT,
    bids TEXT,            -- JSON array
    asks TEXT,            -- JSON array
    spread REAL,
    ts TIMESTAMP
);
```

## Strategy / research using this data

This dataset powers a public-audited [crash-recovery bot](https://github.com/LuciferForge/polymarket-crash-bot):
- **302 live trades · 79.8% win rate · 2.6x win/loss ratio**
- Backtest: 6,225 trades on the full dataset, 75% WR
- Documented finding: after a >20% crash, average bounce is +6.6% within 15 minutes (n=5,629)

Full methodology: [polymarket-crash-bot](https://github.com/LuciferForge/polymarket-crash-bot).
Live PnL: [api.protodex.io/live](https://api.protodex.io/live).

## What you can build with this

- Backtest any prediction-market strategy without paying $24K/yr for Bloomberg
- Train ML models on prediction-market mispricing signals
- Build dashboards and screeners (see [polyscope](https://github.com/LuciferForge/polyscope))
- Feed an LLM agent with prediction-market context (MCP server coming)

## License

MIT. Use it, fork it, ship something.

## Contact

[LuciferForge@proton.me](mailto:LuciferForge@proton.me) · [@gmanjuu](https://x.com/gmanjuu) · [api.protodex.io](https://api.protodex.io)
