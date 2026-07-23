# IWGF

Meta-repo for the **Interstellar Warp Gaming Federation** workspace. Product code lives in git submodules; this repo holds federation docs, Cursor rules, and (soon) shared entrypoints.

## Layout

| Path | Remote | Role |
|------|--------|------|
| [`Warp12/`](https://github.com/Digital-Defiance/Warp12) | submodule | Warp Dominoes + Warp functions + Firestore deploy |
| [`subspace-lattice/`](https://github.com/Digital-Defiance/subspace-lattice) | submodule | Subspace Lattice + lattice functions |
| `leaderboard/` | *(planned extract)* | Federation standings → iwgf.org |
| `ops/` | *(planned extract)* | Federation ops → ops.iwgf.org |

Shared Firebase project: **`warp-12`**.

## Clone

```bash
git clone --recurse-submodules https://github.com/Digital-Defiance/IWGF.git
cd IWGF
# If you cloned without submodules:
git submodule update --init --recursive
```

Each product still has its own `yarn install` / scripts — run from that subdirectory (or via `bin/iwgf` once added).

## Agents

Start at [`AGENTS.md`](AGENTS.md), then the product guide under that submodule.
