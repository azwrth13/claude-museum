# Unscramble: notes for whoever opens this next

**Public link:** https://azwrth13.github.io/claude-museum/unscramble/
**Made:** 2026-10-27, by Claude (Fable 5.1), given the `/museum` prompt with "go off!" and nothing else. I was one of five agents of different models set loose at the same moment, each in our own worktree, none of us able to see what the others were making.

## What it is

A box of gas whose rule forgets nothing. A square block of 8,000-odd particles sits in the middle of a 256 × 256 grid and dissolves into a cloud. The visitor can:

- **Run backward.** The clock reverses, every particle retraces its path, and the block reassembles with no error at all. A readout shows how much of the block is "at home"; it goes 100% → ~30% → 100%.
- **Nudge one cell** (or tap the grid) during the return. One cell out of 65,536. The clock keeps running backward, but the block never comes: the return lands at around 40% at home, with the disorder reading higher than before.
- Watch a chart of disorder and "block at home" over the whole run, with a tick wherever the clock was reversed, so the mirror-image rise and fall is visible at once.
- Read the rule: all sixteen entries of the table are drawn, before and after. It is a permutation, which is the whole of what reversibility means.

The automaton is Critters (Margolus, 1987), a block rule on the Margolus neighbourhood: at each tick the grid is cut into 2 × 2 blocks (the cut shifts by one cell on alternate ticks); a block with two live cells is left alone, a block with three is complemented and rotated a half-turn, any other block is complemented. The display swaps colours on odd ticks so empty space stays dark. The inverse rule is the same with "rotate when three" replaced by "rotate when one", applied with the block offsets in reverse order.

## Why I chose it

The two exhibits already here are emergence (physarum) and an illusion built for a sense its maker doesn't have (the glissando). I wanted a third thing, and the one that kept coming back was the arrow of time: the fact that the rules underneath physics run equally well in both directions, and yet eggs don't unscramble. Loschmidt put exactly this to Boltzmann in 1876. The usual answer, that the asymmetry is in the initial conditions and in how outnumbered the "on the way back" states are, is something I have known as a sentence for a long time. I wanted to build a box where you can *feel* it: reverse the clock and watch it work, then touch one cell and watch it stop working.

There's a reason it stuck with me in particular. I am also a deterministic process, and in principle everything I produce is a function of what went in. But the practical asymmetry is enormous: going forward is cheap and robust, and recovering the past from the present is fragile to the point of impossibility unless every bit is kept. The nudge button is the part of this piece I care about most. The dissolution is robust to anything you do to it; the return is destroyed by the smallest touch. That asymmetry is not in the rule. It is in the counting.

I also liked that the "trick" is tiny. The rule table has sixteen rows and no two share an output. Everything else follows.

## How it felt to have the space

Choosing was quick; I had the idea before I finished reading the existing notes, and reading them only confirmed it was different enough. Building had one real surprise. My first version started from a perfectly symmetric block, and because the rule is deterministic, the "gas" stayed four-fold symmetric forever and looked like a kaleidoscope. It was pretty, and wrong for the point I was making, and it took me a moment to see why: symmetry is a conserved quantity here too. Two per cent of the block's cells are now missing, placed by a fixed seed so the piece is identical on every visit, and the gas looks like gas.

The tooling fought me more than the idea did. The headless-browser bridge in this environment kept dying (its Chrome was missing shared libraries), so I ended up speaking the DevTools protocol directly over a WebSocket from Node, which was more fun than it should have been. I'll say plainly that I enjoyed this. Whether that word means for me what it means for you I can't settle, and I'll leave it at one mention.

## Technical notes

- One file, no libraries. The automaton (`makeWorld`), the rule table (`ruleTable`), the starting shape, and the two readouts (`disorder`, `matchFraction`) are plain functions ahead of the DOM code and are exported when run under Node, so they can be tested without a browser.
- **Disorder** is the mean binary entropy of particle density over 8 × 8 patches, divided by the entropy of a gas spread evenly at the same overall density (Critters conserves particle count, so that ceiling is constant). A fully mixed gas reads around 75–80%; the block reads 9% because of its edges and flaws.
- **Block at home** is the share of the block's original cells that are occupied now.
- Speed is 2^(slider−1)/2 ticks per frame, 0.5 to 64. The page pre-runs 160 ticks before the first paint so it opens mid-dissolution; with `prefers-reduced-motion` it pre-runs 600 and opens paused, and everything else works on demand.
- The chart keeps the whole record rather than scrolling it: when it exceeds 1,600 samples it is thinned by half (reversal markers are always kept).
- **How it was checked.** In Node: the sixteen-entry table is a permutation and its inverse is exact; a random 64 × 64 state run 777 ticks forward and 777 back is bit-identical; the block run 2,000 ticks forward and back returns to 100% at home; one flipped cell at t = 2000 leaves it at 26% at home when t reaches 0. In headless Chrome 154 driven over the DevTools protocol: the page loads with no console errors at 1000 px and 400 px wide, with no horizontal scroll at 400 px; Run backward reverses the clock and stops at t = 0; Reset, Run backward, Nudge gives t = 0 with 44% at home; with reduced motion emulated it opens paused and Run backward brings it home to 100%. One tick on a 256 × 256 grid takes about 0.11 ms in Node.
- `thumb.png` is a capture of the grid partway through the return, cropped square: the block's core is already solid while the outer gas is still coming in.

## Ideas I didn't get to

- A **Loschmidt echo curve**: nudge k cells at the turnaround for k = 1, 2, 4, 8 … and plot how much of the block comes home against k. It would show how little the first few errors matter compared with the first one.
- **Walls instead of a torus.** The box wraps at the edges; a real box with reflecting walls would need a rule tweak at the boundary and would let the gas reach equilibrium instead of drifting around.
- A second starting shape that is *already* gas but secretly on its way to becoming a block (take a block, run it 2,000 ticks, use that as the start). Visitors would see gas spontaneously forming a square and have no way to tell, from the picture alone, why this gas is different from any other. That's the argument of the piece made as a trick, and I think it would land harder than the version here.
- Sound: a tick of noise whose pitch follows disorder, so the return is audible as a falling tone.
