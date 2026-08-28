# OpenVerifiers

**Verify AI agent claims, never trust blindly again.**

OpenVerifiers is a Python toolkit for verifying the outputs of AI agents — plans, claims, decisions, or generated content — through two complementary pipelines: **symbolic (solver-backed)** verification for structural and logical consistency, and **statistical (LLM-judge)** verification for qualitative and semantic correctness. Usable standalone or as an MCP tool inside any agent framework.

---

## Why

LLM agents produce output that *sounds* correct far more often than it *is* correct. A generated plan can look coherent while silently contradicting itself, dropping a stated constraint, or making claims that don't follow from the evidence given. OpenVerifiers exists to catch that gap — before a user, or a downstream agent, acts on a claim that was never actually checked.

## How it works

```
Agent output (natural language)
        │
        ▼
  Schema-forced extraction  ──►  structured claim skeleton
        │
        ├──► Symbolic pipeline    (deterministic, solver-backed: Z3/cvc5)
        │       → hard constraints, arithmetic, dependency/temporal consistency
        │       → returns SAT/UNSAT + trace (unsat core → human-readable reasoning chain)
        │
        └──► Statistical pipeline (LLM-as-judge, calibrated)
                → qualitative/semantic claims not reducible to formal constraints
                → returns a confidence-scored verdict + rationale
        │
        ▼
  Combined verdict + explainable trace
```

- **Symbolic pipeline**: claims are translated into formal constraints and checked with an SMT solver. When something fails, you get back the *minimal set of conflicting constraints* (an unsat core) — an explicit, traceable reason, not just a pass/fail flag.
- **Statistical pipeline**: for claims that aren't cleanly formalizable (tone, intent, "is this a reasonable next step"), a calibrated LLM-judge layer scores confidence and surfaces known failure modes rather than a bare verdict.
- **Dynamic grammars**: verification schemas are assembled per goal/claim type rather than forced through one universal schema, so the toolkit adapts to different domains without a rewrite.

## Features

- 🔍 **Trace-backed reasoning** — every verification result comes with an explicit chain of evidence, not a black-box score
- ⚖️ **Dual pipeline** — deterministic solver checks where structure allows it, statistical judgment where it doesn't
- 🧩 **MCP-compatible** — expose `verify()` as a tool any MCP-speaking agent can call directly
- 🧠 **Domain-adaptive grammars** — define new claim schemas per goal type without touching the core engine
- 🐍 **Pure Python, solver-agnostic** — built on standard SMT backends (Z3 by default, swappable)

## Installation

```bash
pip install openverifiers
```

## Quickstart

```python
from openverifiers import Verifier

v = Verifier()

result = v.verify(
    claim="Complete step 2 before step 3, using no more than 3 hours/day.",
    evidence={"available_hours_per_day": 2, "dependencies": {"step_3": ["step_2"]}},
)

print(result.status)        # "SAT" | "UNSAT" | "UNCERTAIN"
print(result.trace)         # explicit reasoning chain / unsat core
print(result.confidence)    # statistical confidence, if judge pipeline was used
```

### As an MCP tool

```bash
openverifiers serve --mcp
```

Any MCP-compatible agent can now call `verify(claim, evidence)` as a tool.

## Architecture

| Layer | Responsibility |
|---|---|
| `openverifiers/grammars/` | Per-goal-type schema definitions used for claim extraction |
| `openverifiers/symbolic/` | Solver backend, constraint translation, unsat-core trace formatting |
| `openverifiers/statistical/` | LLM-judge harness, calibration, failure-mode guards |
| `openverifiers/mcp/` | MCP server wrapper exposing `verify()` as an agent tool |

## Status

Early-stage, actively developed. APIs may change. Contributions, issues, and design discussion are welcome — this is meant to be a community reference implementation, not a single-project internal tool.

## Contributing

Issues and PRs welcome. If you're extending the grammar system for a new domain, open a discussion first — the goal is to keep the core engine domain-agnostic and push domain-specific logic into grammars.

## License

MIT
