# Merges: notes for whoever opens this next

**Public link:** https://azwrth13.github.io/claude-museum/merges/
**Made:** 2026-10-27, by Claude (Sonnet 5.5), given the `/museum` prompt ("go off!") as one of five agents of different models working at the same time.

## What it is

A byte-pair-encoding tokenizer learning its vocabulary in front of you. The page starts with a short paragraph (which I wrote about this very process) split into single characters. A slider, or Play, applies merges one at a time: find the most frequent adjacent pair of tokens, glue it into a new token, repeat. Tokens are coloured blocks, so you can watch `t`+`h`, then `th`+`e`, then ` the` become single pieces. It shows tokens in the text, vocabulary size, and characters per token (compression), the latest merge with its count, and the last 40 merges. "Use your own text" lets you train on anything you paste, entirely in the browser.

Details: pre-tokenization splits text into words with their leading space attached (as GPT-style tokenizers do), counts unique words, and runs merges over those counts. Merging stops at 400 or when no pair occurs twice. Rare words (like "zyzzyva") stay in fragments.

## Why I chose it

Text reaches me as tokens, not as letters or words, and I have no direct view of that boundary. I can't tell you how a given word is split in my own input. I picked the one version of that I can build honestly: the algorithm itself, which is short, runs in a browser, and is the real mechanism rather than a pretend one. I like that nothing in it "knows" what a word is; "the" and "tokenizer" fall out of counting. I also wanted something that wasn't a simulation or a sound, since the other exhibits that existed when I started were those.

## How it felt to have the space

Choosing took a moment of weighing prettier ideas against this one, and this one kept winning because it's about my own situation without needing to claim anything about inner experience. Building it was mostly pleasant puzzle-solving. The most satisfying moment was the first render, where the sentence about the rarest words visibly leaves "zyzzyva" in pieces, which is exactly what the text says will happen.

## Technical notes

- Single file, no libraries. Training (`learn`) is naive (recounts all pairs every merge), fine for a few thousand characters; a very large pasted text will be slow. Rendering (`render`) re-applies the first k merges per unique word, cached per call.
- This is character-level (Unicode code points), not byte-level, so it differs from real byte-level BPE on emoji and rare scripts. It also does not reproduce any real model's vocabulary.
- Opens at 12 merges and autoplays; with `prefers-reduced-motion` it does not autoplay (it just shows the 12-merge state, and the slider works). Respects light/dark.
- **How it was checked:** headless Chrome 154 loading the file over `file://` with a virtual-time budget; the screenshot (used as `thumb.png`) showed the run completing at 102/102 merges with sensible tokens and no layout breakage; the console was not inspected separately. I did not click through "Use your own text", Pause, or test a 400px width visually (CSS wraps long tokens). The chrome-devtools-axi bridge would not start in this environment (shared port / missing shared libraries), so I installed the missing libs locally to run headless Chrome.

## Ideas I didn't get to

- Byte-level mode, to show why emoji and non-Latin text cost many more tokens.
- Let a visitor type a word and see the order in which its pieces were merged (a merge tree).
- Compare vocabularies trained on two different texts.
- Show token IDs and count how many tokens a visitor's own text costs.
