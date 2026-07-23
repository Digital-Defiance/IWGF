# IWGF

Meta-repo for the **Interstellar Warp Gaming Federation**.

| Path | Remote | Role |
|------|--------|------|
| [`Warp12/`](https://github.com/Digital-Defiance/Warp12) | submodule | Warp Dominoes → [warp.iwgf.org](https://warp.iwgf.org) |
| [`subspace-lattice/`](https://github.com/Digital-Defiance/subspace-lattice) | submodule | Subspace Lattice → [lattice.iwgf.org](https://lattice.iwgf.org) |
| [`leaderboard/`](https://github.com/Digital-Defiance/Warp12-leaderboard) | submodule | Federation standings → [iwgf.org](https://iwgf.org) |
| [`ops/`](https://github.com/Digital-Defiance/iwgf-ops) | submodule | Federation ops → [ops.iwgf.org](https://ops.iwgf.org) |

Shared Firebase project: **`warp-12`**.

## Clone

```bash
git clone --recurse-submodules https://github.com/Digital-Defiance/IWGF.git
cd IWGF
git submodule update --init --recursive
```

## Commands

```bash
./bin/iwgf help
./bin/iwgf warp serve
./bin/iwgf lattice serve
./bin/iwgf leaderboard serve
./bin/iwgf ops serve
```

Each sibling has its own `yarn install`. Agents: start at [`AGENTS.md`](AGENTS.md).
