# Polymarket Historical Data

**9.5M+ price snapshots across 9,550+ prediction markets.** 30 days of 15-minute resolution data.

## What's Inside

| Table | Rows | Description |
|-------|------|-------------|
| `markets` | 9,550+ | Market metadata (question, category, volume, liquidity, resolution date) |
| `prices` | 9,300,000+ | 15-min OHLC price snapshots for YES/NO outcomes |
| `orderbooks` | 792,000+ | Bid/ask depth snapshots |

## Coverage

- **Date range:** Rolling 30 days (auto-refreshing)
- **Granularity:** 15-minute intervals
- **Markets:** All active Polymarket markets (politics, sports, crypto, economics, geopolitics)
- **Source:** Polymarket Gamma API + CLOB API

## Free Samples

- [HuggingFace — polymarket-historical-prices](https://huggingface.co/datasets/manja316/polymarket-historical-prices) (27 downloads)
- [HuggingFace — crypto-prediction-market-signals](https://huggingface.co/datasets/manja316/crypto-prediction-market-signals) (BTC/ETH/SOL + Gold + PM probabilities)

## Full Dataset

| Product | Price | Link |
|---------|-------|------|
| Sample (1 day) | $1 | [Gumroad](https://manja8.gumroad.com/l/polymarket-data) |
| Full dataset (30 days) | $9 | [Gumroad](https://manja8.gumroad.com/l/agyjd) |
| Live subscription (auto-refresh) | $29/mo | [Gumroad](https://manja8.gumroad.com/l/luneql) |

## Also Available

**Crash Recovery Strategy Kit** — 75% win rate backtest across 6,225 trades. Jupyter notebook + data.
- [$19 on Gumroad](https://manja8.gumroad.com/l/PolyStrategy)

## Data Schema

```sql
-- Markets
SELECT id, question, category, volume_24h, liquidity, end_date
FROM markets WHERE active = 1;

-- Prices (15-min snapshots)
SELECT market_id, outcome, price, ts
FROM prices WHERE market_id = '...' ORDER BY ts;

-- Orderbooks
SELECT market_id, bids, asks, spread, ts
FROM orderbooks WHERE market_id = '...' ORDER BY ts;
```

## Collection Infrastructure

Data collected via automated pipeline running 24/7:
- Market Universe collector (launchd, 15-min intervals)
- Cross-signal collector (BTC/ETH/SOL + Gold + PM crypto probabilities)
- Powered by [ForgeOS](https://github.com/LuciferForge/forgeos)

## Contact

LuciferForge@proton.me | [@gmanjuu](https://x.com/gmanjuu)
