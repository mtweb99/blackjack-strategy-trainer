#Copilot instructions: Basic Strategy Coach

A personal blackjack basic-strategy trainer. It is one self-contained `index.html` served by GitHub Pages and used mostly on an iPhone from the Home Screen. Read `README.md` for features and how the grading math works.

## Ground rules

- **Keep it one file.** All CSS and JavaScript stay inline in `index.html`. Use no frameworks, no build step, no npm, and no extra JS or CSS files. The only external resource is Google Fonts, and every font has a fallback stack.
- **Ask before changing behavior.** The owner wants to approve changes first. When he asks a question, answer it rather than editing code. Keep each change small and focused on what was asked, and don't refactor or "clean up" unrelated code.
- **Mobile first.** Test layouts at about 375px wide. Keep the existing safe-area handling (`env(safe-area-inset-*)`) and the one-screen layout, which uses `height:100%` rather than `100vh`. Tap targets stay large. Don't add anything that scrolls the page sideways.

## Strategy logic: handle with care

The engine sits between the `ENGINE START` and `ENGINE END` comments.

- **The chart is the source of truth for the correct play.** `hardCode`, `softCode` and `pairCode` encode standard 6-deck, H17, DAS, late-surrender basic strategy. Never change a chart cell to match the EV numbers, and never change one for any reason without a published chart to cite for these rules.
- **The EV numbers only measure the cost of a mistake.** They use an infinite-deck approximation, conditioned on the dealer having peeked and not having blackjack. Small disagreements with the chart on near-tie hands (for example 16 vs. 10) are expected and are not bugs.
- **Chart codes:** `H` hit, `S` stand, `Dh` double (else hit), `Ds` double (else stand), `P` split, `Rh`/`Rs`/`Rp` surrender (else hit/stand/split). `pairCode` returns `null` when a pair is played as its total, which is always the case for 5s and 10s.
- **Fallbacks:** doubling and surrender are only allowed on the first two cards. Surrender is not allowed after a split. Doubling after a split is allowed. Split aces get one card each and no further decisions. There is no re-splitting. `decide()` applies these rules, so keep them in one place.
- **Dealer rule:** the dealer hits soft 17. This is set in `dealerFrom` and in `finishHand`, and the two must stay in agreement.

## Feedback and hints

- The verdict thresholds (cost per $100 bet) are: up to $0.25 nearly a coin flip, up to $2 close call, up to $10 mistake, over $10 costly. Difficulty labels use the gap between the best and second-best moves: under $2 tough spot, under $10 moderate, otherwise easy call.
- Hints must never name the correct move. Level 3 may rule out options, but only those costing more than $10 per $100, and it must always leave at least two options.
- Moves play immediately. Feedback appears in the banner (`renderBanner`), and the details sheet (`showSheet`) opens only when the banner is tapped.
- `explain()` writes in plain, direct sentences. It may quote live numbers such as the dealer's bust percentage.

## Data

Stats live in `localStorage` under the keys `bsc-stats` and `bsc-settings`. Every read and write goes through the `store` helper, which wraps them in try/catch. Nothing is ever sent over the network. If you change the stats structure, keep old saved data loading without errors.

## Style

- Colors and fonts are tokens on `:root`. Use those tokens; don't add literal colors in components.
- The look is a single dark casino-felt theme by design. Don't add a light theme.
- Action colors are consistent everywhere: hit coral, stand gold, double green, split blue, surrender violet. Verdict colors are green, yellow, orange and red.

## After any change

1. Open `index.html` in a browser and confirm there are no console errors.
2. If you touched the engine, spot-check these values on the details sheet (dollars per $100 bet):
   - 16 vs. 10: hit and stand both about â$54
   - 12 vs. 6: stand about â$12.1, hit about â$17.2
   - 12 vs. 2: hit about â$25.4, stand about â$28.7
   - 11 vs. A: double and hit both about +$10.3
3. Play a hand of each kind: hard, soft, a pair you split, a double, and a surrender.
