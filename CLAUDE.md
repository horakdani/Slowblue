# Slowblue — working notes for Claude

A finished mobile-first Canvas 2D game: seven acts, forty-two authored stations, one
sentence holding all of them — *you do not control the fish, you control the ocean.*
[README.md](README.md) is the pitch, [docs/GAME.md](docs/GAME.md) the design and
[docs/DEVELOPING.md](docs/DEVELOPING.md) the way in; this file is how to work on it without
breaking things that were already decided or re-learning things that were already measured.

**Talk to Daniel in Hungarian. Write code, comments and commit messages in English.**

## Shape of the thing

TypeScript + Vite, no framework. Canvas 2D (`src/render/`, no WebGL), procedural Web Audio
(`src/audio/`), Pointer Events, fixed 1/60 s simulation step (`src/core/GameLoop.ts`),
53k lines in `src/`, nearly 800 unit tests (`npm test` counts them; a number written
here goes stale with the next test), PWA with an offline service worker.

**The campaign has been rebuilt: 432 sections and 144 authored minutes, from 182 and 72.**
Six acts of about twenty-two minutes and act I frozen at ten, because it is the metronome
the other six are heard against. Read [docs/DESIGN-LENGTH.md](docs/DESIGN-LENGTH.md) before
touching any level file: it is the measurement, the three passes act V went through as the
pilot, what the phone said about each of them, and the three instruments that keep this
honest —
`tools/stations.sweep.ts` (what a station is made of, arithmetic, instant),
`tools/length.sweep.ts` (how hard an act is, by flying it, half an hour) and
`tools/rush.sweep.ts` (how much faster than the world a hand can go, and what that costs;
`SLOWBLUE_ACT=reentry` for one act in two minutes). Do not hand-maintain a table of act
lengths again; all three print one.

**The rebuild was verified by tests, not by a playthrough.** Six of the seven acts were
rebuilt in one afternoon, and tests answer *can this be flown* and *does it obey the rules* - not
whether act III still feels like patience or whether act II is still the act this repo is
proudest of. `docs/DESIGN-LENGTH.md` §12 is the watch list, in the order a playthrough
meets it.

**The rules a station obeys are `tests/stations.test.ts` now**, campaign-wide. They were
measured on act V and lived in `tests/reentry.test.ts`, where they guarded one act; six acts
are built to them, so a rule that only holds act V is a rule five acts can break silently.
Act I is exempt from two of them **on purpose** - it is 1.4-to-2.0-minute stations where a
rebuilt one is 2.5 to 5.5, and a rule learned at one scale is not automatically true at the
other.

**Every bot in this repo was a *careful* hand until 2026-08-14, and it made the whole suite
blind.** Each one gated its forward stroke on `behindFraction > 0.5..0.62` — it stopped
pushing at the camera's anchor — and `World.step`'s camera follow pins the swarm at exactly
that anchor for anyone who pushes, so the gate was unconditionally true and had never once
fired. A player reported descending far faster than the world and never being able to die;
nothing in 800 tests could see it. `makeAimBot({ rush: true })` and `{ brake: true }` exist
now. **When a report cannot be reproduced by any probe here, suspect the probe.**

**There is no network code in this project, and there never will be.** Nothing is uploaded,
nothing is fetched. `package.json` has **zero runtime dependencies**, and that is a rule
rather than an accident: adding one is a decision to argue for, not a convenience.

## Commands

```bash
npm run dev        # dev server on all interfaces — reachable from a phone on the LAN
npm run verify     # lint + typecheck + unit + build + e2e — the project's gate
npm test           # unit only, fast
npm run screens    # the eleven documentation frames
npm run sweep      # balance sweeps; the only thing that may justify a tuning number
```

## Traps — measured, not guessed

**The e2e port.** 4173 is occupied on Daniel's machine. Always
`SLOWBLUE_E2E_PORT=4271 npm run e2e`, and build first — without it the suite silently runs
against whatever was already on the port and dies in five-minute timeouts.

**`npm run verify` is green on `main`, and a red run is yours now.** From 2026-08-11 to
2026-11-10 three e2e tests failed on every clean checkout, and this file spent two
paragraphs teaching people to ignore them by name. They were fixed on 2026-11-10 (79
pass, 14.0 min), and none of the three was a flaky test - each was a correct test whose
premise the shell had moved out from under, and two of them were hiding real drift:
`every screen` counted a ninth ink tier, a literal `0.66` beside `--ink-3`'s `0.62`; `the
pinned primary` measured a primary that was no longer pinned, because the map stopped
overflowing a phone once the sound card left, and the map's padding still spelled
`env(safe-area-inset-*)` so no test inset ever reached it; `first launch` marked the
title ready at 45 ms while the ident still covered it. **The lesson is the one at the top
of this file: a test that fails for months is a probe that stopped measuring the thing,
and the fix is to find out which - not to learn its name.**

Pass extra Playwright flags to `npm run e2e`, never to `npm run verify`: npm appends them
to the *last* command of the chain and eats `--trace` itself, so `npm run verify --
--trace off` runs no e2e at all and says `No tests found`.

**The screenshots are not deterministic.** `npm run screens` rewrites all eleven PNGs on
every run, so a dirty `git status` proves nothing. Commit them only when you meant to.

**Another Claude may be on this tree.** Sessions run in parallel here. Check a file's mtime
before writing it if you were not the last one to touch it.

**This is a public repository.** Nothing that reaches a commit here is private, and git
history is hard to scrub — so no keys, no personal identifiers, no paths out of someone's
home directory. There is nothing to sign and no secret the build needs: the deploy is a
static `dist/` pushed to a branch, and it asks for nothing but a push.

That includes the commit metadata. The first commits went out under a personal address,
and rewriting published history to remove it was judged worse than leaving it; from
2026-11-10 this clone's `.git/config` commits as the GitHub noreply address
(`24735930+horakdani@users.noreply.github.com`). `.git/config` does not travel with a
clone, so a fresh checkout has to set it again: `git config user.email` before the first
commit.

**`main` is the only branch** that holds work, and it is the default branch. `gh-pages`
is build output and nothing else: `npm run deploy` replaces it with one orphan commit, so
never commit to it and never merge it anywhere.

## Decided — do not reopen these

* **No post-processing.** Bloom, glow and grain were built, hated and reverted. Fix a
  picture with composition, not a filter.
* **The visual language:** hard flat vector, discrete depth planes (not a continuum, not
  haze), outline only on large silhouettes, teal plus **one** warm second colour per
  station, and no fast foreground plane — that was built and rejected.
* **The failure system stays** as it is: the hunter, the 40 % swarm floor, the Game Over
  screen. The cozy criterion is read on the mood, not on the mechanics.
* **`blue-rise` is never regenerated.** It is the one signed picture; the other forty-one
  stations are measured against it.
* **`touch-action: none` is deliberately absent from `<body>`** — it intersects on the
  ancestor chain and killed scrolling in the shell. The canvas declares it itself.
* **The `tide.*` storage prefix stays**, even though the game is called Slowblue. Renaming
  it silently discards every existing player profile.
* **The web is the only target, decided 2026-10-26.** The Capacitor/iOS shell that was
  built in August is gone from this repo, along with the three runtime dependencies it
  needed and the store copy written for it. What replaces it is the thing that was
  underneath all along: an installable PWA, open source under MIT, served by GitHub
  Pages from the `gh-pages` branch. There is no deploy workflow and no CI, because
  pushing anything under `.github/workflows/` needs an OAuth scope this repo was created
  without; `npm run deploy` (`tools/deploy.sh`) is the deploy, run by hand, so a change
  to `main` is not live until someone runs it. Two consequences worth keeping in
  mind - the build must survive being served under a path (see `vite.config.ts`), and
  the haptics driver seam in `platform/Haptics.ts` now has exactly one driver behind it.
  The seam stays: it is what keeps the cue table the only way the game buzzes.

## House style

Comments here explain **why**, and often name the wrong solution that was already tried —
that is what makes them worth reading twice. Write that kind, not the kind that restates
the line below it.

Where a measurement can decide instead of taste, **measure**. That is the habit of this
repo, and `e2e/measure.spec.ts` exists to hold the numbers.

The hot path allocates nothing: no `new`, no `.map()`, no `.filter()` inside `tick` or
`render`. Pool instead.

Commit subjects are English sentences that name one concrete thing, in the repo's own
voice — read `git log` before writing one. Commit only when asked, and never push.
