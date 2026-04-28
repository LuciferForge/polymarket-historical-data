# Polymarket Historical Data

**10M+ price snapshots. 9,550+ markets. 30 days rolling. 15-minute resolution.**

The complete dataset behind a [public live-traded strategy](https://github.com/LuciferForge/polymarket-crash-bot) (302 trades, 79.8% win rate). Free samples on Hugging Face. Full dataset and live API on Gumroad / [api.protodex.io](https://api.protodex.io).

## Coverage

| Table | Rows | Description |
|-------|------|-------------|
| `markets` | 9,550+ | Question, category, volume_24h, liquidity, end_date |
| `prices` | 10,000,000+ | 15-min OHLC snapshots for YES/NO outcomes |
| `orderbooks` | 800,000+ | Bid/ask depth snapshots |

- **Source:** Polymarket Gamma + CLOB APIs (no scraping, no proprietary data)
- **Update cadence:** every 15 minutes via [ForgeOS](https://github.com/LuciferForge/forgeos) launchd job
- **Categories:** politics, sports, crypto, economics, geopolitics, weather, science
- **Format:** SQLite (single file, queryable from any language)

## Get the data

| Tier | Price | Format | Where |
|------|-------|--------|-------|
| **Free sample (1 day)** | $0 | SQLite + CSV | [Hugging Face](https://huggingface.co/datasets/manja316/polymarket-historical-prices) |
| **Cross-signal sample** (BTC/ETH/SOL + Polymarket probabilities) | $0 | CSV | [Hugging Face](https://huggingface.co/datasets/manja316/crypto-prediction-market-signals) |
| Sample paid (1 day full SQLite) | $1 | SQLite | [Gumroad](https://manja8.gumroad.com/l/polymarket-data) |
| Full dataset (30 days) | $9 | SQLite | [Gumroad](https://manja8.gumroad.com/l/agyjd) |
| **Live API** (no download, query directly) | Free 100/day · $19/mo Pro | HTTP/JSON | [api.protodex.io](https://api.protodex.io) |
| Live subscription (auto-refreshing dataset) | $29/mo | SQLite | [Gumroad](https://manja8.gumroad.com/l/luneql) |

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
- **302 live trades · 79.8% win rate · 2.6× win/loss ratio**
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
