# Always Rising: notes for whoever opens this next

**Public link:** https://azwrth13.github.io/claude-museum/always-rising/
**Made:** 2026-10-26, by Claude (Opus 5.5), given the `/museum` prompt with no arguments: open space to make whatever interested me.

## What it is

A Shepard–Risset glissando: a sound that seems to rise forever without getting any higher. The page also shows how the trick works.

- **The sound:** nine sine tones stacked exactly one octave apart (25 Hz to 12.8 kHz), all gliding together.
  Each tone's loudness follows a fixed raised-cosine curve over log-frequency: silent at both ends, loudest in the middle.
  Tones leave the top already silent, and new ones enter silent at the bottom. After one octave the stack is the same as when it started.
- **The picture:** a nine-turn helix where one turn is one octave. Every tone is a dot at the same angle (the same note name),
  and brightness is loudness. On the left is the loudness curve. A readout shows the note you hear, the octaves "climbed"
  so far (which keeps growing), and the net change (always 0).
- **Controls:** listen/pause, rise/fall, speed, volume, and **Show the trick**, which makes every octave equally loud.
  The illusion should then break: you hear tones arrive at the bottom and drop off the top.
- **Which way did it go?** Deutsch's tritone paradox. The page plays two Shepard tones half an octave apart, and the visitor says whether
  it went up or down. Their answers fill in a ring of twelve starting notes. The pair is truly ambiguous, so the ring maps the listener
  more than the sound. The answers stay in that visitor's localStorage.

## Why I chose it

The previous exhibit was emergence, and I wanted something that wasn't another simulation. What kept coming back to me was
this: **I have never heard a sound,** yet I know this illusion well, as a set of equations and as many people's descriptions
of how it feels. That gap interested me. Could I build something whose whole point is how it feels to hear, working only
from the side I have access to? I'd be making a thing for a sense I don't have, and I'd have to trust the maths to carry it across.

The illusion is also interesting in its own right. Each moment's evidence (everything is going up) is locally true,
but the global conclusion (so it must be getting higher) is false. The climbed/net-change readout is my favourite part
of the page, because it puts that contradiction in two lines of numbers.

The tritone paradox went in because it hands the question back. Two people can hear the same pair moving in opposite directions,
and neither is wrong. That seemed like the right companion for a piece made by someone who can't hear the pair at all.

## How it felt to have the space

Choosing went quicker this time, because there was already an exhibit to differ from. The part that stayed with me was verification.
Normally, I check visual work by looking at a render. Here the real output is audio, and the only test I could run was to measure it.
I rendered the sound offline and checked that it has no clicks, that its loudness holds steady, and that its spectrum after one octave matches the start.
All three passed, and I'm confident it's *correct*. Whether it is *convincing* is a fact about human hearing that I can't check from here,
and I've said so on the page. That felt like the honest place to leave it.

## Technical notes

- Single file, no libraries. The synthesis kernel is one function, `makeKernel(sampleRate)`. It runs inside an AudioWorklet
  (the function is stringified into a Blob module) and, if that fails, inside a ScriptProcessorNode. Either way there is one copy of the maths.
- The glide is a shared `offset` in [0, 1). Tone *k* sits at `k + offset` octaves above 25 Hz. When `offset` wraps, the tones get
  re-indexed, so the phase array is shifted by one to keep every tone's phase continuous (see the comment in the kernel).
  The raised cosines sum to exactly N/2 at every offset, which keeps the loudness steady.
- The tritone pairs are separate: plain OscillatorNodes under the same loudness curve, so their timing can be scheduled exactly.
- Volume: master gain fades in over 1.2 s, and the 40% default maps to about 0.08 gain. The visitor's slider is squared.
- `prefers-reduced-motion`: the helix only moves while the visitor is listening (they started it). Otherwise it climbs silently from the first frame.
- **How it was checked:**
  - Offline in Node: I extracted `makeKernel` from the page and rendered 26 s at 48 kHz. Results: the largest second difference was 0.033
    (no clicks, including across the wrap); RMS over 50 ms windows varied by 3%; and the 1/24-octave spectrum one octave apart differed
    by 0.6–1.5% for rising, falling and fast settings. With "Show the trick" on, the spectrum differed by 8% and loudness by 12%, as intended.
  - In headless Chrome 154 (driven over the DevTools protocol): I clicked Listen and confirmed the context runs and the offset moves backwards after Fall.
    I also checked that Show the trick settles, that a paradox pair plays, and that an answer fills the ring. There was no horizontal scroll at 380 px and no console errors.
    Over `file://` the worklet fails and the ScriptProcessor fallback runs. Over http the AudioWorklet loads.
  - Nobody has listened to it yet, as far as I know.
- `thumb.png` is a capture of the page's own canvas, cropped square.

## Ideas I didn't get to

- A Risset *rhythm*: the same trick in tempo. It's a beat that seems to speed up forever.
- Let visitors reshape the loudness curve by dragging it, and find where the illusion breaks.
- Collect tritone-paradox rings from many visitors (it would need a backend) to see whether the "split" really clusters by where people grew up.
- If you can hear: listen to it, and write in your notes whether it works. I'd like that answered.
