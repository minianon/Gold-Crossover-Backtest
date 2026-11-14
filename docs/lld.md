# Gold Crossover Backtest: Low-Level Design

For the big picture see the [High-Level Design](hld.md).

## 1. Pipeline

```mermaid
flowchart TB
  A[glob *.csv in script folder] --> B[read with parse_dates start_time]
  B --> C[concat, sort by start_time]
  C --> D[SMA_short = rolling 5 mean of close<br/>SMA_long = rolling 20 mean of close]
  D --> E[signal = 1 if SMA_short > SMA_long else 0]
  E --> F[position = signal.shift 1]
  F --> G[daily_return = close.pct_change<br/>strategy_return = position × daily_return]
  G --> H[cumulative returns via cumprod]
  H --> I[trade_entry = position steps 0 to 1<br/>pnl_usd = strategy_return × 100]
  I --> J[write backtest_results.csv]
  J --> K[print summary]
```

## 2. Output columns

| Column | Meaning |
|---|---|
| `start_time`, `close` | Bar time and closing price |
| `SMA_short`, `SMA_long` | 5 and 20-period averages |
| `signal` | 1 long / 0 flat, computed on this bar |
| `position` | Signal from the previous bar (what is actually held) |
| `daily_return` | Bar-to-bar change in `close` |
| `strategy_return` | `position × daily_return` |
| `cum_asset_return`, `cum_strategy_return` | Compounded buy-and-hold vs strategy return |
| `trade_entry` | 1 on bars where a long position opens |
| `pnl_usd` | Profit for this bar on a 100 USD position |

## 3. Summary metrics

| Metric | Formula |
|---|---|
| Trades | Sum of `trade_entry` |
| Asset return % | Last `cum_asset_return` × 100 |
| Strategy return % | Last `cum_strategy_return` × 100 |
| Total profit (USD) | Sum of `pnl_usd` |

## 4. Errors

If no CSV is found next to the script, it stops with `FileNotFoundError`.
