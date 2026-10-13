# Basic Strategy Coach

A small iPhone-friendly web app for memorizing blackjack basic strategy. It deals random hands, asks you for a move, and grades each decision. It shows whether the move was right, how much a wrong choice costs, and why the correct play is correct.

It is a trainer, not a blackjack game. There is no bankroll or betting. The point is learning the chart until it's automatic.

## Rules it teaches

| Rule | Setting |
|---|---|
| Decks | 6 |
| Dealer soft 17 | Hits (H17) |
| Double after split | Allowed (DAS) |
| Surrender | Late surrender |
| Dealer peek | Dealer checks for blackjack before you act |
| Blackjack payout | 3 to 2 |

This is the most common shoe game in US casinos. Other rule sets change a handful of chart cells (see *Changing the rules* below).

## Features

- **Full hands, not just the first decision.** Hands play out to the end: hitting, doubling, splitting into two hands, surrender, then the dealer draws and each hand is settled.
- **Instant feedback banner.** Every move plays immediately. A color-coded banner above the buttons shows whether you were right and what a mistake cost, so you can drill quickly. Tap it for the full breakdown.
- **Full breakdown on demand:**
  - a plain-English explanation of why the correct play is correct, using live numbers such as the dealer's bust rate for that upcard
  - the expected value of every available move, in dollars per $100 bet
  - a gauge showing how serious the mistake was: trivial, close call, mistake, or costly
- **Three-level hints.** Each tap reveals a bit more without naming the move:
  1. Difficulty: how close the best two options are
  2. Key facts: the dealer's bust rate, and your own chance of busting if you hit
  3. Nudge: a guiding question, and which options can be ruled out
- **Two deal modes.** *Tough spots* (the default) weights toward soft hands, pairs, and hard 12–16. *Real deals* deals cards randomly, as at a casino.
- **Chart tab.** The full strategy chart, color-coded, with a red badge on every cell you've missed.
- **Stats tab.** Accuracy overall and by hand type (hard, soft, pairs), best streak, average cost per decision, and your recent misses. A correct answer after a hint counts toward accuracy but doesn't extend your streak.

## How the grading works

The **chart decides what is correct.** It is the standard 6-deck, H17, DAS, late-surrender basic strategy chart.

The **dollar values show how much is at stake.** For every decision, the app calculates the expected value of each legal move:

- Dealer outcomes are computed exactly by recursion over the dealer's drawing rules (hits soft 17), conditioned on the dealer not having blackjack. This matches what you know at the table after the dealer peeks.
- Player values assume optimal play after each move. Split hands may double (DAS) but not re-split. Split aces get one card each.
- The shoe is treated as an **infinite deck**, so each card's probability stays fixed at 1/13 (4/13 for ten-valued cards). That is within a fraction of a dollar per $100 of exact 6-deck values. On razor-thin decisions, such as hit vs. stand on 16 vs. 10, the two values can show as a tie.

A mistake's cost is the expected value of the best play minus the expected value of the move you chose.

## Installing on an iPhone

The app is one self-contained HTML file. It loads Google Fonts but has no other dependencies and no build step.

1. Put `index.html` and `apple-touch-icon.png` in a GitHub repository.
2. In the repository, go to **Settings → Pages**. Set the source to deploy from the `main` branch, root folder.
3. After a minute, the app is live at `https://<your-username>.github.io/<repo-name>/`.
4. On the iPhone, open that address in Safari. Tap **Share → Add to Home Screen**.

The home-screen icon opens it full-screen like an app.

## Where your data lives

Stats and settings are saved in the browser's `localStorage` on your device. Nothing is sent anywhere. Clearing Safari's website data erases them, and stats don't sync between devices.

The app needs an internet connection to load. It does not currently work offline.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app |
| `apple-touch-icon.png` | Home Screen icon |
| `.github/copilot-instructions.md` | Guidance GitHub Copilot reads automatically when working in this repo |

## Code layout

Everything is in `index.html`, in this order:

| Section | What it does |
|---|---|
| `<style>` | Design tokens on `:root` (colors, fonts), then layout and components |
| Engine (`ENGINE START` … `ENGINE END`) | Dealer outcome recursion (`dealerFrom`, `dealerDist`), player EV functions (`evalFor`), and the strategy chart (`hardCode`, `softCode`, `pairCode`) |
| `decide()` | For the current hand, works out the legal moves, the correct move (including fallbacks when double or surrender aren't allowed), and each move's EV |
| `explain()` | The written explanation for each situation |
| `hintLines()` | The three hint levels |
| Game flow | `newHand`, `advance`, `apply`, `finishHand`: dealing, playing moves, the dealer's draw, and settlement |
| Rendering | Table, feedback banner, details sheet, chart tab, and stats tab |

Chart codes used in the engine: `H` hit, `S` stand, `Dh` double (else hit), `Ds` double (else stand), `P` split, `Rh`/`Rs`/`Rp` surrender (else hit/stand/split).

## Changing the rules

The chart lives in three small functions: `hardCode`, `softCode`, and `pairCode`. The dealer's soft-17 rule is the line in `dealerFrom` marked `// dealer hits soft 17`.

To switch to a dealer-stands-on-soft-17 game, change that line and update the chart cells that differ:
- 11 vs. A: hit instead of double
- 15 vs. A and 17 vs. A: no surrender
- soft 18 vs. 2: stand
- soft 19 vs. 6: stand
- 8,8 vs. A: split

## Testing a change

There is no test suite. After editing:

1. Open `index.html` in a desktop browser and check the browser console for errors.
2. Open the Chart tab and confirm the cells match a published chart for your rules.
3. Play a few hands of each type: hard, soft, a pair you split, a double, and a surrender.
