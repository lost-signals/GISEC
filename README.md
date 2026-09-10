# Adversarial Input Red-Teaming Toolkit

Plain software that attacks an AI inference endpoint and produces a robustness
report. The toolkit contains **no ML of its own** — the AI is only the *target*,
reachable at an HTTP URL. The toolkit is the attacker.

## The three ways a target fails (what we hunt)
- **crash** — endpoint errors/times out on malformed input (ordinary robustness bug)
- **flip** — a tiny, meaning-preserving edit silently inverts the prediction (ML-specific)
- **inconsistency** — meaning-identical inputs get different labels (our unique metamorphic check)

## Layout
```
main.py                 pipeline entrypoint
plugins.py              <-- YOUR payloads & operators go here (empty by design)
toolkit/
  models.py             frozen contracts: Attack, Prediction, Result  (freeze first)
  registry.py           the empty payload/operator registries
  target.py             the one HTTP door to the target
  runner.py             async suite runner (captures response/error/latency)
  fliphunter.py         feedback-guided minimal-flip search (sensitivity probe)
  metamorphic.py        invariance oracle (UNIQUE — the dual of the flip hunter)
  semantic.py           semantic gate (real flip vs. just a different sentence)
  storage.py            SQLite (the dashboard will read straight from this)
  report.py             minimal HTML report (dashboard supersedes this)
target_stub/server.py   the punching bag (infra owns this; here so it runs Day 1)
```

## Run
```bash
pip install -r requirements.txt
uvicorn target_stub.server:app --port 8000        # start the target
python main.py --url http://127.0.0.1:8000 --seeds seeds.txt
```
Until you register payloads and at least one operator in `plugins.py`, the
static suite fires nothing and the flip hunter has nothing to perturb with —
that's expected. `plugins.py` shows every signature.

## The unique feature — metamorphic invariance testing
Metamorphic testing (a formal software-QA technique) checks a *relation* that
must hold, not a known answer. Ours: *inputs that mean the same must get the same
label.* For each seed we apply meaning-preserving operators, confirm meaning was
preserved via the semantic gate, and flag any label change as an invariance
violation — the model contradicting itself on inputs a human calls identical.
It's the exact dual of the flip hunter (which *searches* for one such edit),
runs off the same operators and gate, and turns every seed you add into a whole
family of tests automatically.
