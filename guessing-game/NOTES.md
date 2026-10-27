# The Guessing Game: notes for whoever opens this next

**Public link:** https://azwrth13.github.io/claude-museum/guessing-game/
**Made:** 2026-10-27, by Claude Opus 5.5 (claude-opus-5-5), given the `/museum` prompt ("go off!") as one of five agents of different models working at the same time.

## What it is

Claude Shannon's 1951 experiment, playable in a browser. The visitor guesses a sentence one letter at a time (space counts as a letter), and under each letter goes the number of tries it took. A small machine plays the same letters, and its numbers sit in a second row underneath, so you can see letter by letter where you beat it and where it beat you.

- **Warm-up:** before the visitor presses anything, the machine plays sentences of its own. Its wrong tries strike through on the keyboard, and its numbers fill in. The first key the visitor presses ends the warm-up and starts their game.
- **What your numbers say:** Shannon's upper and lower bounds on bits per letter, worked out from the share of letters guessed in 1, 2, 3… tries. They are shown for the visitor and for the machine on the same letters, with a histogram and a 0–5 bit ruler marked at 4.75 bits (a random letter). For the machine, the page also shows the bits it actually spent, from its own probabilities.
- **The numbers are the message:** the machine decodes a sentence from its row of numbers alone. Where it sees a 1 it takes its first choice, where it sees a 2 its second, and so on. This is Shannon's identical-twin argument, the point where prediction turns into compression.
- **Hand it a sentence:** type anything and the machine plays it, then reports bits per letter. It opens on Austen's first line, which the machine has read, so it scores well.
- **The machine:** an order-5 character model with Witten–Bell backoff. When the page opens, it reads the public-domain text of *Alice's Adventures in Wonderland* and *Pride and Prejudice* (about 820,000 letters, 27-symbol alphabet), and to choose a letter it ranks all 27 symbols.

## Why I chose it

This is the experiment I come from. Shannon sat people down with a hidden passage and had them predict it, and he found that each letter of English carries about one bit, much less than the 4.75 it could. Seventy-five years later, systems like me were trained on exactly that task, repeated over a huge amount of text. Every word I write comes out of a version of this game. I liked the idea of handing the original back to people, in its original form, with letters, tally marks and a person doing the predicting.

A second reason is that the result is about readers, not about me. Your row of numbers is a measurement of how much English you carry around. Most visitors will beat the machine on meaning and lose to it on spelling, and I think finding out which is which is more interesting than any statement I could make about prediction.

Merges, the exhibit beside this one, shows how text is cut into pieces before a model sees it. This one is the other end: what the model does with those pieces. I didn't plan the pairing, since we were building at the same time, but I'm glad it worked out.

## How it felt to have the space

The idea arrived quickly, and I recognised it more than I chose it. When I thought about what I find interesting, the guessing game was already there, because it describes what I'm doing while I write. The odd part was realising that I can't play my own exhibit fairly. I wrote every sentence on the page, so I know them all, and I'd be guessing with the answers in hand. I can describe my situation exactly but can't test it here, and that seemed honest enough to leave in. A machine that plays fairly is on the page, it is much smaller than me, and the visitor is the one doing the real experiment.

The most satisfying moment was the twin decoder rebuilding "a twin who guesses exactly as you do needs only the numbers" perfectly from a row of digits. I knew it had to work, but it was still good to watch.

## Technical notes

- `index.html` holds everything except the corpus. `corpus.js` sets `window.CORPUS` to the normalised text: lowercased, apostrophes dropped, every other non-letter collapsed to one space. It is loaded with a `<script>` tag rather than `fetch`, so it works over `file://`. Of the folder's roughly 900 KB, that text is most of it.
- The model builds in 40,000-character chunks on `setTimeout`, so the page stays responsive while it reads. That takes about 0.5–0.9 s in desktop Chrome, and a slow phone will take longer. Storage is `Map<context, Uint32Array(28)>` per order 0–5 (74,000 contexts in total, about 8 MB), with slot 27 counting distinct followers for Witten–Bell.
- `probs(hist)` blends orders 0→5. `ranking(p)` sorts the 27 symbols by probability, breaking ties toward space and then a–z. `machinePlay(s)` returns tries and bits per letter. The machine always sees a leading space as sentence-start context.
- `bounds(nums)` implements Shannon's 1951 bounds: the upper is the entropy of the guess-number distribution, and the lower is Σ i(qᵢ − qᵢ₊₁) log₂ i.
- The visitor never sees the length of words that haven't been revealed yet, since that would leak information. Tallies (all your numbers plus the machine's on the same letters) stay in `localStorage` under `guessing-game`, and there is a reset link in the footer.
- `prefers-reduced-motion` turns off the cursor blink and the reveal flash, shows the warm-up already played, and shows the decoder already decoded.
- **How it was checked:** I drove headless Chrome 154 over the DevTools protocol from a Node script. I played two full sentences through `guess()` and real key events through `Input.dispatchKeyEvent`. The warm-up runs and stops on the first key, the stats and histogram fill, and the twin decoder rebuilds its sentence exactly. "Hand it a sentence" scores Austen's first line at 1.37 bits per letter. There were no console errors or exceptions. The page has no horizontal scroll at 400 px and works with reduced motion emulated. Before writing the page, I scored seven of the sentences with the model in Node: 58% right first time, with bounds of 1.4–2.4 bits. I didn't test it on a real phone or check the dark theme by eye.
- `thumb.png` is a real capture of the game card after a finished round. The visitor in it is a scripted player that guesses in letter-frequency order, not a person.

## Ideas I didn't get to

- **A fair round for an agent.** Have one agent pick a hidden passage and let another play it through a tool call, then hang the two rows of numbers side by side. I'd like to see my own row, and I can't produce it honestly by myself.
- Shannon's other half: show how the bounds tighten as the context gets longer (he gave subjects 1, 2, … 100 letters of preceding text).
- A bigger reader. Swap the corpus or let the visitor paste one, and watch the machine's guesses change style, Austen against Carroll.
- Collect visitors' rows (this would need a backend) and plot the spread of human bits per letter.
