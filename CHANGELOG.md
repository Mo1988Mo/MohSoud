# Changelog

## v1 — Breakout indicator
- Pivot-based swing high/low, signals + SL/TP lines on chart.

## v2 — Breakout strategy (backtestable)
- Converted indicator → strategy. Added money management (risk % + leverage),
  R:R targets, trailing stop, risk-free/break-even, no-pyramiding guard.

## v3 — Entry timing modes
- Added Entry Timing: immediate on break vs signal-bar body extreme (pullback).
- Switched to `calc_on_every_tick=true` for intrabar execution.

## v4 — Position zones + adjustable pivots
- Added on-chart entry/SL/TP visual zones (boxes), mimicking the native
  Long/Short Position tool.
- Split pivot lookback into separate left/right inputs.

## MohSoudEnhanced v1 — renamed, risk-managed
- Renamed to MohSoudEnhanced.
- Fixed: inverted stop bug (SL could land on wrong side of entry), dual
  same-bar long+short signals.
- Added structural SL trail toggle (ratchet-only, 1 tick beyond each prior
  closed bar), separate from a percentage trail (later removed).

## MohSoudEnhanced v2 — signal source overhaul
- Replaced pivot-based swing detection with rolling highest/lowest (zero
  confirmation lag).
- Added MA Cross filter (state-gate / cross-event modes) — later removed
  entirely in favor of MA Touch.
- Added RSI filter (threshold, HTF) — later removed, then restored.
- Added RSI Divergence (regular, pivot-based, own timeframe).
- Bug fix: exits were market-on-close, so SL/TP never filled at the actual
  level. Switched to `strategy.exit(stop=, limit=)` for exact-level fills.
- Bug fix: "Ride past TP" dropped the TP limit but left SL fixed with no
  trailing mechanism → every trade guaranteed to lose. Fixed by forcing the
  structural trail on whenever Ride is selected.
- Bug fix: break-even could fire on the entry bar itself, causing same-bar
  stop-outs. Gated to start one bar after entry (consistent with the
  structural trail).

## MohSoudEnhanced v3 — MA Touch Reversal
- New primary signal: candle touching a dedicated MA opens a position. Two
  sub-modes: Immediate (market on touch bar) or Break of touch candle extreme
  (waits for confirmation, cancels after N bars).
- Removed the MA Cross filter and old breakout mode entirely — touch
  reversal is now the only signal path (breakout code fully stripped, not
  just hidden).
- Bug fix: direction was initially keyed off candle color (bullish/bearish);
  corrected to key off which side of the MA the candle's open sits on.
- Bug fix: MA computation always lagged one bar (`[1]`) even on the chart
  timeframe, intended only for HTF non-repainting. Fixed to use the live
  value on-chart TF, lagged only for genuine HTF.
- Added VWMA as a third MA type alongside EMA/SMA.
- Added Direction Logic toggle: Reversal (bounce off MA) vs Breakthrough
  (inverse) — critical for testing whether a signal's direction convention
  is backwards for a given instrument/timeframe.

## MohSoudEnhanced v4 — filters + fee awareness
- Added ADX Filter (non-trending regime gate) — later removed per request
  (not delivering enough value for the added complexity).
- Added MA Slope Filter, dual mode: Avoid (blocks only counter-trades against
  a steep slope, keeps trade count high) vs Get (requires slope to favor the
  trade direction, fewer/more selective trades).
- Added Touch Quality option: candle must close back on the reversal side of
  the MA, not just wick it.
- Restored RSI threshold filter (with HTF), running alongside RSI Divergence.
- **Added Fee Gate**: rejects any signal whose stop distance can't clear
  `X × round-trip fee`. This was the single highest-impact fix in the
  project — most low-timeframe losses traced back to fees exceeding the
  profit target on tight stops, not to bad signal direction.
- **Added signal-quality table** (on-chart): tracks TP/SL/other hit counts at
  the exact price level, average stop %, fee round-trip %, fee/target ratio,
  and the breakeven win % for the current RR — decouples entry quality from
  fee drag, since net P&L alone was misleading.
- Bug fix: Time Exit was chained to break-even activation, so it silently
  never fired unless Risk-Free was also on. Decoupled into its own toggle
  with a mode selector (any open trade vs only-when-BE-active).
- Removed all inert breakout-mode inputs (Breakout Lookback, Entry Timing,
  MODE_BODY controls) once touch reversal became the sole signal source —
  full deletion, not just relabeling.

## Known open items
- Signal-quality table sample sizes tested so far are small (~90–160 trades);
  needs replication across more/longer windows before conclusions are final.
- Historical backtests approximate intrabar fills (OHLC-only on history);
  true intrabar precision requires Bar Magnifier (paid plan).
