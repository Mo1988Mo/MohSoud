# MohSoudEnhanced

Pine Script v5 strategy — MA-touch reversal/breakthrough, with risk-based sizing,
a fee-aware entry gate, and a signal-quality tracker that measures the entry's
true hit rate independent of trading costs.

## Core mechanics

- **Signal**: a candle "touches" a dedicated MA (`low ≤ MA ≤ high`). Direction is
  set by which side the candle's open sits on relative to the MA (not candle
  color). Two logic modes:
  - **Reversal**: touch from above → long, from below → short.
  - **Breakthrough (inverse)**: touch from above → short, from below → long.
- **Entry timing**:
  - *Immediate on touch bar*: market entry on the touch candle. SL = 1 tick
    beyond that candle's own extreme.
  - *Break of touch candle extreme*: waits for a later bar to break the touch
    candle's high/low, enters on that break. SL = 1 tick beyond the touch
    candle's opposite extreme. Cancels if unfilled within N bars.
- **Position sizing**: risk % of equity ÷ stop distance, capped by
  `equity × leverage ÷ price`. Set leverage high enough that the cap never
  binds, or realized risk drops below the target %.
- **Take profit**: Fixed R:R target, or Ride (trail stop, no fixed target —
  forces the structural trail on, since Ride needs a way to capture profit).
- **Stop loss**: fixed at entry by default; optional structural trail (ratchets
  to 1 tick beyond each prior closed bar, tighten-only, starts one bar after
  entry); optional break-even.
- **Fee Gate**: skips any signal whose stop distance can't clear
  `X × round-trip fee` (as % of price). This is the single most important
  setting for low-timeframe trading — see "Why fees matter" below.
- **Filters** (all independent, all default off): RSI threshold (with HTF),
  RSI Divergence (pivot-based, with HTF), MA Slope (Avoid/Get modes), Touch
  Quality (candle must close on the reversal side of the MA).
- **Signal-quality table** (on-chart, top-right): tracks TP-hits vs SL-hits at
  the exact price level, independent of fees — this is the number that
  actually tells you if the entry has an edge, since TradingView's net P&L
  conflates entry quality with fee drag.

## Why fees matter (the core lesson from testing)

Fees are charged on **notional**, not on risk. On low timeframes, stops are
tight, so risk-based sizing requires large notional to keep risk at 1% —
which means fees can exceed the profit target even on a winning trade.

Rule of thumb: **fee share of target = round-trip fee ÷ (stop% × RR)**.
Tighter stops or lower RR make this worse. Two levers fix it:
- Raise RR (2–3 on low TF, not 1).
- Use the Fee Gate to reject signals whose stops are too tight to clear fees
  with margin.

## Test protocol

Before trusting any result:

1. All filters OFF, Immediate entry, fixed 1R, EMA-50, fixed window, 15m (or
   your target TF).
2. Run **Reversal**, note Signal win % from the table.
3. Run **Breakthrough**, identical everything else, note Signal win %.
4. Whichever direction clears its **breakeven win %** (shown in the table,
   `100/(1+RR)`) with margin over 300+ trades — that's a real signal. Re-test
   at RR 2–2.5 to confirm the fee/target ratio improves.
5. If neither clears breakeven, the touch entry has no edge on that
   instrument/timeframe — move to a different entry idea rather than adding
   more filters.

## Settings reference (as last tested)

- Fee per side: 0.04% (after cashback) → round trip 0.08%
- Money Management: Risk 1%, Leverage 100 (cap must not bind)
- Direction Logic: Breakthrough (best result so far — see CHANGELOG)
- RR: 1.9
- Fee Gate: ON, min stop = 2× round-trip fee

## Status

Not validated for live or margin trading. Best backtest to date: +3.02% PnL,
profit factor 1.049, Signal win 47.8% vs breakeven 34.5%+fees, over 90 trades
on 5m BTCUSDT.P — small sample, needs replication across more windows before
any real capital is considered.
