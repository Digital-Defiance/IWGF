# IWGF — Agent Guide

**IWGF** (Interstellar Warp Gaming Federation) is the parent brand / **meta-repo** for two sibling game products plus **shared federation surfaces** (leaderboard + ops). This GitHub repo tracks orchestration docs and **git submodules** for each product — not one merged monorepo. Each product keeps its own remote and history.

| Path | Remote | Role |
|------|--------|------|
| `Warp12/` | [Digital-Defiance/Warp12](https://github.com/Digital-Defiance/Warp12) (submodule) | Warp Dominoes (The Bridge) + Warp functions + **shared Firestore rules deploy** |
| `subspace-lattice/` | [Digital-Defiance/subspace-lattice](https://github.com/Digital-Defiance/subspace-lattice) (submodule) | Subspace Lattice (Chess+Go fleet game) |
| `leaderboard/` | *(extract in progress — today: `Warp12/Warp12-leaderboard/`)* | Federation standings SPA → **iwgf.org** (Warp + Lattice TEI) |
| `ops/` | *(extract in progress — today: `Warp12/apps/WarpOps/`)* | Federation ops console → **ops.iwgf.org** (Warp + Lattice) |

Clone with `git clone --recurse-submodules` (or `git submodule update --init --recursive`).

**Canonical product guides** (read these when working inside a product):

- Warp: `Warp12/AGENTS.md`
- Lattice: `subspace-lattice/AGENTS.md`

---

## 1. Products at a glance

| | **Warp 12** | **Subspace Lattice** |
|--|-------------|----------------------|
| Genre | Multi-trail Interstellar Dominoes (Warp 9/12/15/18) | 11×11 Chess+Go fleet tactics + Sensor Net |
| Web | warp.iwgf.org | lattice.iwgf.org |
| Tauri app | `Warp12/apps/Warp12` | `subspace-lattice/apps/desktop` |
| Bundle ID | `org.digitaldefiance.app.warp12` | `org.digitaldefiance.app.subspacelattice` |
| Platforms | macOS, iOS, Android, Windows | macOS, iOS, Android, Windows |
| Shipping rules | Multi-trail + modules; TEI on **Warp 12 only** | Soft-ship **`hybrid-fleet`** |
| Engine package | `warp12-engine` (`Warp12/libs/engine`) | `@subspace-lattice/core` |
| Functions codebase | Firebase `default` (`Warp12/functions`) | Firebase `lattice` (`apps/functions`) |

Both are **Tauri 2** shells over React/Vite SPAs. Game logic lives in TypeScript, not Rust.

---

## 2. Shared Firebase / federation

**One Firebase project:** `warp-12`

| Hosting target | Site | Public URL | Built from | Scope |
|----------------|------|------------|------------|-------|
| `bridge` | `warp-12` | warp.iwgf.org | `Warp12/apps/Warp12` | Warp only |
| `leaderboard` | `warp-12-leaderboard` | **iwgf.org**, profile.iwgf.org | `Warp12/Warp12-leaderboard` | **Shared** (Warp + Lattice) |
| `ops` | `warp-12-ops` | **ops.iwgf.org** | `Warp12/apps/WarpOps` | **Shared** (Warp + Lattice) |
| `lattice` | `subspacelattice` | lattice.iwgf.org | `subspace-lattice/apps/web` | Lattice only |

**Shared federation surfaces** (live in the Warp12 repo, serve both titles):

- **Leaderboard** — TEI standings, profiles, crews/charters
- **WarpOps** — moderation / fleet administration at ops.iwgf.org (Auth custom claims `ops` / `moderator` / `admin`)

**Shared identity:** Auth + `playerProfiles` / federation call signs (never Google `displayName` for public identity).

**Separate TEI pools (same presentation family):**

| Product | Collection | Leaderboard route |
|---------|------------|-------------------|
| Warp | `playerStats` | iwgf.org/leaderboard/warp |
| Lattice | `latticeTei` (`localAi` \| `online`) | iwgf.org/leaderboard/lattice |

Hub: https://iwgf.org/leaderboard — “one federation, separate TEI pools.”

**Lattice-namespaced Firestore:** `latticeRooms`, `latticeRoomCodes`, `latticeTei`, `latticeRatingEvents` (see `subspace-lattice/packages/subspace-lattice/src/lib/firebase/collections.ts`).

### Critical deploy rule

- **Firestore rules/indexes are owned by Warp12** (`Warp12/firestore.rules` includes the Lattice fragment).
- Lattice deploy scripts **refuse** Firestore deploy on purpose — deploying Lattice’s copy alone would wipe Warp rules.
- When Lattice rules change: update **both** `subspace-lattice/firestore.rules` (source fragment) **and** `Warp12/firestore.rules`, then deploy rules from **Warp12**.

---

## 3. Where to work

| Task | Directory | First read |
|------|-----------|------------|
| Warp gameplay / AI / TEI / Bridge UI | `Warp12/` | `Warp12/AGENTS.md` |
| Lattice gameplay / AI / rooms | `subspace-lattice/` | `subspace-lattice/AGENTS.md` |
| Federation standings, profiles, both TEI boards | `Warp12/Warp12-leaderboard/` | that SPA’s README + pages under `src/app/pages/` |
| Federation ops / moderation (both titles) | `Warp12/apps/WarpOps/` | `apps/WarpOps/src/app/` |
| Shared Firestore rules | `Warp12/firestore.rules` (+ Lattice fragment sync) | comment block “Subspace Lattice” |
| Store / Tauri packaging Warp | `Warp12/` scripts `build:mac`, `build:ios-appstore`, `build:android`, `build:windows*` | `Warp12/docs/`, `.env.example` |
| Store / Tauri packaging Lattice | `subspace-lattice/` | `docs/desktop-build.md` |

Run commands **from the product repo root**, not from this IWGF folder (each has its own Yarn install / Nx).

---

## 4. Stack commonalities

- **Nx 23** + **Yarn 4** (`yarn@4.17.0`) monorepos
- **TypeScript strict**, React 19, Vite 8, Vitest, Playwright e2e
- **Firebase** Auth / Firestore / Functions / Hosting
- Package source condition: `@warp12/source` vs `@subspace-lattice/source` (no tsconfig path maps)
- SCSS / CSS modules; kebab-case filenames
- Co-located `*.spec.ts(x)`

**Differences that matter:**

| | Warp12 | Subspace Lattice |
|--|--------|------------------|
| Prefer | Direct `yarn build:*` / `test:*` / `serve:bridge` (avoid hanging `nx run`) | `yarn nx …` / `yarn serve:web` is normal |
| Nx hang recovery | `yarn nx:unlock` + `pkill -f "Warp12/node_modules/nx"` | less common; still `nx daemon --stop` if stuck |
| Lib layout | `libs/*` + `vendor/double-eighteen` submodule | `packages/*` |
| Online authority | Client+Functions mix; Warp match sync | **Functions write** rooms/moves/TEI |

---

## 5. Quick command cheat sheet

### Warp12 (`cd Warp12`)

```bash
yarn install
yarn serve:bridge          # Vite :4200
yarn tauri:dev
yarn build:all             # libs + bridge + functions
yarn build:leaderboard     # federation SPA
yarn test:engine | test:react | test:all
yarn deploy:firebase       # needs FIREBASE_PROJECT
```

### Subspace Lattice (`cd subspace-lattice`)

```bash
yarn install
yarn emulators             # Terminal A
yarn serve:web             # Terminal B :4200
yarn serve:desktop         # or yarn tauri:dev
yarn nx run-many -t lint test build typecheck
yarn deploy:firebase       # hosting:lattice + functions:lattice ONLY
```

### Leaderboard

```bash
cd Warp12 && yarn build:leaderboard
# or: cd Warp12/Warp12-leaderboard && yarn build
```

---

## 6. Agent guardrails (federation-wide)

1. **Treat the two products as sibling remotes** — do not assume a parent git repo or shared `node_modules`.
2. **Never deploy Firestore from `subspace-lattice`.** Merge rules into Warp12 and deploy from there.
3. **Never deploy the wrong hosting target** — Lattice scripts must not touch `bridge` / `leaderboard` / `ops`; Warp must not overwrite Lattice hosting unless intentional. Leaderboard and Ops are **shared** — treat changes as federation-wide.
4. **Secrets stay local** — `.env`, keystores, `.p12`, `client_secret_*`, Google plists. Never commit them.
5. **Emulators are opt-in** — `VITE_USE_FIREBASE_EMULATORS=true` only in local `.env.local`, never in shippable env.
6. **Call signs** come from Federation Profile / IWGF identity, not Google display names.
7. When a change spans both products (rules, TEI display, shared profile, **ops**, leaderboard), update the relevant repos/surfaces and say so in the summary.

---

## 7. Doc map

| Doc | Location |
|-----|----------|
| This guide | `AGENTS.md` (here) |
| Warp agent bible | `Warp12/AGENTS.md` |
| Lattice agent bible | `subspace-lattice/AGENTS.md` |
| Warp rules (normative) | `Warp12/RULES.md` |
| Lattice rules (normative) | `subspace-lattice/docs/rules.tex` / `rules.pdf` |
| Lattice player overview | `subspace-lattice/docs/player-overview.md` |
| Lattice desktop/store | `subspace-lattice/docs/desktop-build.md` |
| Lattice ADRs | `subspace-lattice/docs/adr/` |
| Warp platform / submission checklists | `Warp12/docs/` |
