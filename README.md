# Washout-Recovery Bot — cbBTC/USDC

A trading bot for the make.fun competition
**"Race a BTC trading bot live for 24 hours"**
(`btc-race-24h-20261115-0828`). It trades Coinbase Wrapped BTC
(cbBTC) against USDC on 1-minute bars from the Orca Whirlpool on
Solana, in a paper-money replay: no real money is traded.

## Strategy

Long only, one position, full equity, no leverage. The bot makes
exactly one kind of trade — the recovery leg off a washout low:

- **Enter (flat only):** the close is at least **400bp below the
  rolling 480-bar (8-hour) high** AND crosses back above its
  **20-bar EMA** on this bar. The washout filter keeps the bot out of
  shallow bounces during a decline (they are bull traps); the EMA
  cross confirms the turn has actually started instead of guessing
  the bottom tick.
- **Exit (whichever comes first):** take profit **+350bp** on the
  average entry cost, hard stop **-200bp**, trailing stop **100bp
  below the peak** once the trade is +150bp up, or **480 bars**
  (8 hours) held. A 30-bar cooldown follows every exit, and no new
  trade opens if equity has fallen under $920.
- **Late-window fallback:** if 1,200 bars have passed and the bot has
  still never traded, a **300bp** washout may enter instead of 400bp.
  An entry with no trade at all fails the race guardrail (at least 1
  trade) and ranks below every entry that meets it, while even a
  losing trade meets the bar and can place. The fallback never fires
  on the dev window, so it does not change the dev grade.

One order per batch, slippage cap 150bp. Fills land at the next bar's
open; the entry price is read back from the filled position's cost,
never assumed from the decision bar's close.

## Why this shape

The 24-hour dev window ($67,216 → $64,524, −4.0%) is flat for four
hours, drops in two legs to $62,082, then recovers to $65,480 before
drifting down. Buying the drops loses on every parameter set tried —
each pullback keeps falling — and every kit baseline loses on this
window (buy-and-hold −$44.01, momentum −$40.92, EMA-cross −$134.95,
RSI-Bollinger −$57.37, ADX-momentum −$86.60). The only rideable move
is the recovery leg, so the bot waits for exactly that and otherwise
stays in cash.

Take profit is +350bp, the middle of the broad plateau the dev sweep
found (TP +250bp: +$25.97; +300bp: +$29.68; +350bp: +$32.72; +400bp:
+$41.55; +500bp: +$31.47 via the trailing stop), rather than the
fitted peak at +400bp, which depends on this one leg topping at
+469bp.

## Dev-window result

Kit `grade-local`, verified 2026-11-15 (two executions, byte-for-byte
reproducible):

- **Net PnL +$32.72** (dev gate: > $0) — passes with a wide margin
- 1 round trip, win rate 1.0, max drawdown 0.79%, fees $0.81,
  0 rejected orders, guardrails met
- At 2x fees the net is still +$31.91

## Files

- `main.ts` — the bot (Node 22, standard library only)
- `vendor/makefun-bars.ts` — the kit's SDK shim, byte-for-byte copy
- `LICENSE` — MIT

## Running it

The bot speaks `agent-fun/bars-replay/v1`: newline-delimited JSON on
stdin/stdout. Grade it locally from the competition kit:

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repo> --driver process
```

## License

MIT — see `LICENSE`.
