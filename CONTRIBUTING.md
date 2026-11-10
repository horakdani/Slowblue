# Contributing

Thank you for looking. This is a finished game rather than a framework, so the most
useful contributions are usually small and specific: a bug you can reproduce, a device
the layout breaks on, a section that is unfair in a way you can describe.

## Getting set up

```bash
npm install && npm run dev
```

Node 20 or newer (`.nvmrc` says 22). [docs/DEVELOPING.md](docs/DEVELOPING.md) has the
commands, the layout of the code and the URL parameters that open the admin and debug
surfaces.

## Before you open a pull request

Run the gate:

```bash
npm run verify
```

That is lint, typecheck, the unit suite, the production build and the e2e suite, in that
order. **There is no CI here**, so it is the only gate there is: nothing runs on your
pull request, and a change that was not verified locally was not verified. The e2e half
takes a quarter of an hour, which is the price of that.

**The whole gate is green on a clean checkout**, so a red run is something to look at
rather than something to recognise. If a test fails that you do not think you touched,
say so in the pull request with the output; a test that is wrong is worth fixing, and a
test that is quietly ignored stops guarding anything.

If you touched gameplay numbers, also run `npm run sweep`. The balance sweeps are the
only thing in this repo that may justify a tuning number, and "it felt better" is not
an argument this project accepts on its own.

## House rules the code already follows

Read [CLAUDE.md](CLAUDE.md) first — it is written for whoever is working in the repo next,
human or otherwise, and it names the decisions that are settled. Three of them matter
most to an outside change:

* **There is no network code in this project, and there never will be.** Nothing is
  uploaded, nothing is fetched. `package.json` has no runtime dependencies, and adding
  one is a decision, not a convenience.
* **Comments explain why**, and often name the wrong solution that was already tried.
  Write that kind. A comment that restates the line below it will be asked about.
* **Where a measurement can decide instead of taste, measure.** Several files exist only
  to make a measurement possible; use them rather than arguing around them.

## Reporting a bug

Open an issue with what you did, what happened, what you expected, and the device and
browser. A run is reproducible from its seed: the seed is printed on the game-over screen
and accepted back by `window.__slowblue.restart(seed)`, so a seed plus the act is often
the whole report.

If a report cannot be reproduced by any probe in this repo, the probe is the suspect —
that has happened before and it is written down in CLAUDE.md. So a report that no test
can see is still worth filing.

## Code of conduct

By taking part you agree to the [Code of Conduct](CODE_OF_CONDUCT.md).
