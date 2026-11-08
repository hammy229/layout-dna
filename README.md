# LayoutDNA

**Adapts a layered design to a new aspect ratio, and measures how well six structural "design genes" (visual mass, hierarchy, reading path, alignment, negative space, grouping) survive the move. A React web client and a Python solver service joined by one versioned scene contract.**

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Active development is paused at a portfolio-quality stopping point. What works today and what is left as a placeholder is in [docs/NEXT_STEPS.md](docs/NEXT_STEPS.md).

## Results

| Claim | Measured | How it is checked |
|---|---|---|
| The solver honors its hard constraints | 0 violations across every geometrically feasible case in a 40-case matrix (8 example scenes by 5 target formats). The 8 violations are all in cases flagged infeasible, where it returns a best effort | `python -m evaluation` |
| It is deterministic | Identical output on repeated runs of every case | Same harness |
| It is fast | 0.10 ms p95 per solve, 0.18 ms worst case, on an Apple M4 | Same harness, runtime envelope |
| Both halves are tested | 31 web-client tests and 103 solver tests pass | `npm test`, `pytest solver/tests`, or `./scripts/check.sh` |

Gene drift is a geometric proxy for how much a layout's structure moved. It is not a perceptual or aesthetic score, and no quality claim is made until a blinded human study runs ([study/](study)).

## Why it matters

Ordinary resize tools preserve pixels or apply generic reflow rules; they do not describe what
made the source composition feel like itself. LayoutDNA represents a page as a versioned scene,
measures a deterministic first-pass fingerprint, and asks a solver service for a constrained
target layout plus an explanation of the tradeoffs. The current solver is intentionally a
lightweight, deterministic geometric engine, not the planned differentiable optimizer.

The fingerprint has six genes:

1. Visual mass distribution
2. Hierarchy ratios
3. Reading path
4. Alignment axes
5. Negative-space topology
6. Gestalt grouping

## How it works

1. **Scene.** A design is a versioned JSON scene ([`schemas/scene.schema.json`](schemas/scene.schema.json)), validated with Pydantic.
2. **Fingerprint.** Six deterministic proxy scores describe what makes the composition feel like itself.
3. **Reflow.** A role-aware geometric reflow places each layer on the target canvas, resolves collisions, and keeps safe margins.
4. **Explain.** The client renders before and after as SVG, next to per-gene drift and a trace of the tradeoffs.

A React/TypeScript web client loads example scenes locally, lets you pick a target format, calls
the solver over HTTP, and renders the result entirely in the browser as a before/after SVG , 
alongside the gene fingerprint, per-gene drift, and the explanation trace. If the solver is
unreachable it shows a clear offline state rather than inventing a result.

The FastAPI service validates the shared scene model with Pydantic, computes deterministic proxy
gene scores, and performs a role-aware reflow with collision resolution and safe-margin
enforcement. See [the architecture](docs/architecture.md) and [optimization plan](docs/optimization.md).

| Choice | Why |
|---|---|
| One language-neutral scene schema | The TypeScript client and Python solver cannot drift apart on the data |
| A deterministic geometric solver first | Measurable and testable now, behind the same API a differentiable optimizer will use later |
| An honest offline state | If the solver is unreachable, the client says so instead of inventing a result |
| No quality claims without human evaluation | Proxy scores are labeled as proxies |

### API

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/health` | Solver status and implementation label |
| `POST` | `/api/v1/fingerprint` | Six deterministic approximate gene scores |
| `POST` | `/api/v1/optimize` | Margin-safe target boxes, drift metrics, and explanation trace |

Interactive API docs are available at `http://127.0.0.1:8000/docs` while the solver runs.

Example request:

```bash
curl -sS http://127.0.0.1:8000/api/v1/optimize \
  -H 'Content-Type: application/json' \
  --data-binary @<(jq '{scene: ., targetWidth: 1200, targetHeight: 628, margin: 0.04}' examples/editorial-poster.json)
```

## Tech stack

React, TypeScript, Webpack, Vitest, Python, FastAPI, Pydantic, Ruff, pytest.

## Getting started

- Node.js 20+ and npm
- Python 3.11+

From the repository root:

```bash
npm install
python3 -m venv .venv
.venv/bin/python -m pip install -e './solver[dev]'
cp .env.example .env
```

No secrets are required. PyTorch and GeomLoss are deferred until real differentiable losses exist.

Start the solver:

```bash
.venv/bin/python -m uvicorn app:app --app-dir solver --reload --host 127.0.0.1 --port 8000
```

In another terminal, start the web client:

```bash
npm run dev:add-on
```

Open `http://127.0.0.1:5241`, pick an example scene and a target format, and the client renders
the source and solved layouts side by side against the local solver. `./scripts/dev.sh` starts
both processes together.

### Command line

The solver runs standalone from a terminal, no browser needed, via the `layoutdna` CLI:

```bash
layoutdna fingerprint examples/retail-promo.json          # six gene proxy scores
layoutdna optimize    examples/retail-promo.json --target leaderboard
layoutdna optimize    examples/editorial-poster.json --width 1200 --height 628 --json
```

Named target presets mirror the evaluation matrix (`square`, `story`, `landscape`, `wide`,
`leaderboard`); `--width`/`--height` override them, and `--json` emits the raw API response.

## Project structure

- `add-on/`: React + TypeScript web client (scene gallery, target picker, SVG before/after, tests)
- `solver/`: FastAPI API, Pydantic contracts, gene modules, deterministic reflow, CLI, and tests
- `schemas/scene.schema.json`: language-neutral scene contract
- `examples/`: valid source-scene fixtures
- `docs/`: architecture, fingerprint, optimization, evaluation, and demo notes
- `study/`: future human-evaluation protocol
- `scripts/`: all-checks and two-process development helpers

## Testing

```bash
npm run typecheck
npm run lint
npm test
.venv/bin/ruff check solver
.venv/bin/pytest solver/tests
.venv/bin/python -m evaluation   # solver eval harness over examples × target matrix
npm run build
./scripts/check.sh
```

The evaluation harness reports per-gene drift, hard-constraint violation counts, determinism,
and a runtime envelope across every example scene and a fixed target matrix; it exits non-zero
if a geometrically feasible case violates a hard constraint. See [evaluation](docs/evaluation.md).

## Limitations and next steps

- Gene scores are transparent geometric proxies, not perceptually validated measures.
- Optimization is a deterministic, role-aware geometric reflow with collision resolution and
  margin clamping; it does not use gradients, optimal transport, or a differentiable path yet.
- Salience and some text metrics use fixture metadata rather than rendered measurement.
- No database, authentication, hosted VLM, GPU deployment, or heavy ML runtime is included.

Next:

1. Complete loss implementations and a differentiable renderer behind the unchanged API.
2. Expand the example corpus and subtype-aware scene extraction.
3. Run fixture benchmarks and blinded human evaluation before making quality claims.

## License

MIT, see [LICENSE](LICENSE). Built by [Arham Doshi](https://arhamdoshi.dev).
