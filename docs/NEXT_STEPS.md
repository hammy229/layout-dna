# Next steps / handoff

A snapshot of where LayoutDNA stands and what is intentionally left as a placeholder, so the
project can be picked up cleanly later. Everything below is grounded in the code as it exists;
no results are claimed that the repo cannot reproduce.

## Current state (snapshot: 2026-10-18)

- **Standalone project:** a React/TypeScript web client (`add-on/`) and a FastAPI solver
  (`solver/`), joined by one versioned scene contract. No host application required.
- **Green baseline:** `./scripts/check.sh` passes end to end — typecheck, lint, Prettier,
  the web test suite, the webpack build, `ruff`, `pytest`, and the evaluation harness.
- **Tests:** 31 web-client tests (pure logic + jsdom component/integration) and 96 solver
  tests (gene proxies, reflow, collisions, text-fit, CLI, API, and the eval harness).
- **Honest by construction:** gene scores are labeled geometric proxies, the solver is
  deterministic, re-layout is non-destructive, and dropped optional elements are named in the
  explanation trace (see `AGENTS.md`).

**Development is paused** as of the snapshot date; this repository is a portfolio-quality
stopping point, not an actively maintained product.

## Placeholders for the next iteration

These are real, known gaps — each is either stubbed or documented as future work, never faked.

1. **Perceptual gene validation.** The six genes are deterministic *geometric* proxies, not
   perceptually validated measures. `study/` holds a human-evaluation protocol but no collected
   data. Blinded evaluation must come before any quality (vs. structure) claim.
2. **Differentiable optimizer.** `solver/optimize/` is a deterministic role-aware reflow;
   `losses.py` are proxy measurements, not gradients. The differentiable renderer / optimal-transport
   path in the roadmap is unimplemented, and PyTorch/GeomLoss are deliberately not installed.
3. **VLM role labeling.** `solver/labeling/vlm.py` is a `Protocol` only — no hosted or local
   model is wired in. Role inference currently falls back to the keyword heuristic in `rules.py`.
4. **Inter-element clear space.** `logoClearSpace` is currently enforced as distance to the
   nearest canvas edge, not clearance from neighboring elements. True inter-element clearance is
   unmodeled.
5. **Extreme-aspect feasibility.** The eval harness honestly flags infeasible extreme-aspect
   targets (e.g. a 4:5 poster into an 8:1 strip) as best-effort rather than solving them. A
   recorded sacrifice/crop strategy for those cases is future work.
6. **Example corpus.** Only a handful of example scenes exist in `examples/`. Expanding the
   corpus (and subtype-aware scene extraction) would strengthen the evaluation matrix.

## How to resume

```bash
npm install
python3 -m venv .venv && .venv/bin/python -m pip install -e './solver[dev]'
./scripts/check.sh      # confirm the baseline is still green
./scripts/dev.sh        # run the solver + web client together
```

Entry points: solver API in `solver/app.py`, reflow in `solver/optimize/reflow.py`, the web
client in `add-on/src/ui/App.tsx`. See [architecture](architecture.md), [fingerprint](fingerprint.md),
[optimization](optimization.md), and [evaluation](evaluation.md).
