# Dynamic S/R Zones

A Pine Script v6 support/resistance project for TradingView that builds and manages **zones rather than fixed price lines**, tracks how price interacts with them, and provides a companion strategy for backtesting zone-based entries.

The project separates three responsibilities across timeframes:

- **CTF (chart timeframe)** — touch episodes, reactions, breaks, role flips, and strategy execution.
- **ZTF (zone timeframe)** — zone construction, merging, ATR scale, and staleness aging.
- **STF (structure timeframe)** — directional market-structure context.

The required relationship is **STF > ZTF >= CTF**.

## What it does

Zones are born from confirmed swing pivots that pass a relative-volume gate. Their initial width is ATR-based and extends inward from the pivot. Nearby active zones with the same role can merge.

Each zone keeps lifetime interaction evidence:

- **T — Touches:** distinct CTF interaction episodes.
- **R — Reactions:** favorable moves after a touch, measured with ZTF ATR.
- **B — Breaks:** confirmed closes beyond the zone plus an ATR tolerance.

Zones are displayed as **UNTESTED**, **TESTED**, or **RESPONSIVE** from their observed touch/reaction history. A break flips support to resistance or resistance to support until the configured break limit is reached. Old, distant zones can also be frozen by the staleness rules instead of disappearing from the chart.

## Market structure

Directional context is derived from confirmed STF swing structure rather than moving-average trend filters:

- higher high + higher low → **BULLISH**
- lower high + lower low → **BEARISH**
- mixed structure → **NEUTRAL**

STF pivots use their own sensitivity settings independently of zone pivots.

## Strategy

`src/sr_zones_strategy.pine` adds a backtestable execution layer on top of the same zone model.

The default entry mode waits for a genuine new CTF touch and then for price to close back through the favorable side of the zone. An experimental immediate-interaction mode is also available. A break/role flip cannot immediately arm the newly flipped role: price must first leave the zone and later create a fresh touch.

Risk controls include:

- stop placement beyond the source zone with a ZTF-ATR buffer;
- configurable minimum reward in R;
- optional maximum-reward cap;
- optional breakeven management;
- Long / Short / Both direction selection;
- optional neutral-structure entries.

When a qualifying opposite-role zone lies beyond the minimum target, the strategy can use that zone as the structural target.

## Non-repainting MTF design

Higher-timeframe requests deliberately use the **previous completed ZTF/STF bar** with `lookahead_on`. This makes confirmed higher-timeframe information available from the beginning of the next requested-timeframe period without using the unfinished higher-timeframe candle.

Pivot markers are drawn at the historical pivot location for readability, but the pivot is not actionable until its configured right-side confirmation bars have completed. CTF lifecycle logic is evaluated on confirmed chart bars.

## Files

| File | Purpose |
| --- | --- |
| `src/sr_zones.pine` | Indicator: dynamic zones, lifecycle, evidence classification, and STF context |
| `src/sr_zones_strategy.pine` | Strategy: the same zone engine plus entries, exits, risk controls, backtesting, and optional Pine Logs diagnostics |

## Important inputs

The main controls are grouped by purpose in TradingView: timeframes, pivot detection, market structure, volume birth gate, zone geometry, lifecycle, merging, reaction detection, strength classification, strategy behavior, and display.

The source defaults are intended as a starting configuration, not optimized parameters for a particular market.

## Backtesting notes

The strategy uses TradingView's broker emulator with `process_orders_on_close = true` and `pyramiding = 0`. Backtest results depend materially on symbol, timeframe, date range, commission, slippage, order size, and input configuration.

The strategy was built as an engineering/backtesting example. Historical results from a particular configuration are **not** a claim of expected profitability and should not be interpreted as financial advice.

## Implementation highlights

This project intentionally handles several state-management cases that simple S/R scripts often omit: stable zone IDs, one-touch episodes rather than counting every overlapping candle, fresh-touch arming, persistent zone evidence across role reversals, setup invalidation when source state changes, staleness based on the ZTF clock, and diagnostic run/trade IDs for inspecting strategy behavior in Pine Logs.

## Screenshots

The final portfolio set uses three views:

1. **Indicator overview** — dynamic support/resistance zones with lifetime T/R/B evidence, strength classification, frozen history, and timeframe/structure context.
2. **Strategy execution** — zone-driven entries and exits on the chart together with TradingView's trade list.
3. **Backtest overview** — Strategy Tester performance for one historical configuration; shown as implementation evidence, not an expected-return claim.

## License

See [LICENSE](LICENSE).
