<div align="center">

# TENFOLD

**The SPX ↔ SPY strike mirror for 0DTE.**

SPY is not SPX ÷ 10 — and on 0DTE that difference is a strike.

Try it: **https://ratterau.github.io/tenfold/**

*One HTML file. No install, no signup, no key.*

</div>

---

<div align="center">
  <img src="docs/chain.png" width="440" alt="TENFOLD strike chain in light mode">
  <img src="docs/dark.png" width="440" alt="TENFOLD strike chain in dark mode">
</div>

<div align="center"><sub>Light and dark, toggled with the button next to the wordmark. The ladder sizes itself to your screen — as many strikes each side as fit, never a scrollbar.</sub></div>

---

## The problem

SPY tracks the S&P 500 minus accrued dividends, so it doesn't sit at exactly one tenth of the index. The real ratio floats around **10.02–10.06** and drifts through every quarter.

Which means the reflex — *SPX 7430, so SPY 743* — is wrong by half a strike. SPY 743 is really SPX **7450**. On a 30-day position, shrug. On 0DTE, where the whole trade lives inside two strikes of gamma, that's the difference between the contract you meant and the one you clicked.

TENFOLD does the division for you, against the live ratio, continuously.

## Reading the screen

**Strikes ascend** down the page, the way a broker chain reads. **Green above spot, red below** — the side you're on is legible from across the room. The ladder measures your viewport and fills it with as many strikes as fit, so nothing important is ever one scroll below the fold.

Each quote carries **how old the exchange says the print is** — `2s ago` under a live tick, amber once it drifts past 30 seconds, and hours when the market's shut. A number that stops updating should say so rather than sit there looking current.

The **spot line** rides between the two bracketing strikes and drifts inside the gap as price moves, so you can see spot creeping toward a strike rather than watching a divider snap five points at a time. Both spots sit in the pill.

Then there's the **blue dot**, which is the whole reason this exists:

```
  SPY      SPX      EXACT
  740     7,420    739.98   ●
  740     7,425    740.48
─────── 7,428.78  740.86 ───────
  741     7,430    740.98   ●
  741     7,435    741.48
```

Two SPX strikes round onto the same SPY strike, and at a glance they look equally valid. They aren't. `7430 → 740.98` is a clean SPY 741. `7435 → 741.48` is half a strike off and is nothing like the same contract. The dot marks the real pair; its twin is dimmed so it can't be misread.

## Pricing a strike

Tap any row and it opens in place — no separate calculator, no losing your spot on the ladder.

<div align="center">
  <img src="docs/pricing.png" width="560" alt="Inline pricing panel">
</div>

Type what you paid on SPX, get the SPY equivalent. Premium scales by the live ratio at the **exact** equivalent strike, and the gap to the nearest listed SPY strike is shown rather than buried — because that gap is real money, and rounding it away silently is how a converter lies to you.

Treat the output as an anchor for sizing and comparison. It is not a quote.

## Why there are no bid/ask columns

Because nothing free is fast enough to deserve them.

No public feed quotes options faster than once per second. Yahoo's chain is delayed and only refreshes on request; everything genuinely real-time — Polygon, Tradier, CBOE, broker APIs — is keyed and paid. Painting delayed numbers in a layout that implies they're live is worse than showing nothing, so TENFOLD shows strikes.

Spot for SPX and SPY refreshes every 3 seconds and is near-real-time.

Want real option prices? Replace `quote()` in `index.html` with a keyed feed. It's the only function that touches the network.

## Running it yourself

```bash
git clone https://github.com/RatterAU/tenfold.git
cd tenfold
open index.html
```

That's the whole setup. No build, no dependencies, no package.json. Serve it with `python3 -m http.server 8000` if you'd rather, or drop `index.html` on any static host.

Sanity-check the strike math anytime: open the console, run `tenfoldCheck()`.

## Under the hood

| | |
|---|---|
| **Size** | one file, ~14 KB, zero dependencies |
| **Data** | Yahoo Finance chart endpoint (`^GSPC`, `SPY`), 3s poll |
| **Ladder** | auto-fits the viewport, 5-point SPX grid |
| **SPY grid** | $1 strikes |
| **Clock** | America/New_York, DST handled |
| **Theme** | light / dark, remembered in localStorage |
| **Mobile** | responsive to 360px |

Change `WIDTH` at the top of the script for a different SPX strike spacing. Row count is automatic — it estimates from the viewport height, then shrinks until the page genuinely fits rather than trusting the arithmetic.

**On the CORS relay:** Yahoo sends no CORS headers, so the browser reaches it through a public relay — `cors.lol`, then `cors.sh`, then `allorigins`, failing over automatically. Those are free, rate-limited, and run by strangers. Fine for personal use. If this ever picks up real traffic they'll throttle it and the app will sit on RECONNECTING. The fix is a small Cloudflare Worker proxying Yahoo, then point `RELAYS` at it — the free tier covers far more than this needs.

## What this isn't

No chains, no IV, no greeks, no P/L, no order entry. Strike and price geometry, nothing else.

And SPX and SPY options are **comparable, not interchangeable**. SPX is cash-settled, European-exercise, 1256-taxed, $100 multiplier. SPY is American-exercise with early-assignment and ex-dividend risk. TENFOLD maps the strikes; it doesn't claim the two positions are the same trade.

## Fine print

Little tool I built for myself. It's not financial advice, I'm not liable for anything you do with it, and the numbers come from a free feed that can be wrong or stale — check every strike on your broker before you send an order.

MIT licensed. Take it, fork it, do whatever.
