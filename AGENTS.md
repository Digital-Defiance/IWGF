# IWGF — Agent Guide

**IWGF** (Interstellar Warp Gaming Federation) is the parent brand / **meta-repo** for two sibling game products plus **shared federation surfaces** (leaderboard + ops). This GitHub repo tracks orchestration docs, `bin/iwgf`, and **git submodules** — not one merged monorepo. Each product keeps its own remote and history.

| Path | Remote | Role |
|------|--------|------|
| `Warp12/` | [Digital-Defiance/Warp12](https://github.com/Digital-Defiance/Warp12) | Warp Dominoes + Warp functions + **Firestore rules deploy** + hosting staging |
| `subspace-lattice/` | [Digital-Defiance/subspace-lattice](https://github.com/Digital-Defiance/subspace-lattice) | Subspace Lattice + lattice functions |
| `leaderboard/` | [Digital-Defiance/Warp12-leaderboard](https://github.com/Digital-Defiance/Warp12-leaderboard) | Federation standings → **iwgf.org** |
| `ops/` | [Digital-Defiance/iwgf-ops](https://github.com/Digital-Defiance/iwgf-ops) | Federation ops → **ops.iwgf.org** |

Clone with `git clone --recurse-submodules` (or `git submodule update --init --recursive`).

**Canonical guides:** `Warp12/AGENTS.md`, `subspace-lattice/AGENTS.md`, root `AGENTS.md` (this file). Prefer `./bin/iwgf <product> <action>`.

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
| `leaderboard` | `warp-12-leaderboard` | **iwgf.org**, profile.iwgf.org | `leaderboard/` | **Shared** |
| `ops` | `warp-12-ops` | **ops.iwgf.org** | `ops/` | **Shared** |
| `lattice` | `subspacelattice` | lattice.iwgf.org | `subspace-lattice/apps/web` | Lattice only |

**Deploy mechanics:** Firebase `public` dirs must live under `Warp12/` (where `firebase.json` lives). `Warp12/scripts/sync-federation-hosting.sh` copies `leaderboard/dist` or `ops/dist` → `Warp12/federation-hosting/{leaderboard,ops}/` before hosting deploy.

**Shared identity:** Auth + `playerProfiles` / federation call signs (never Google `displayName`). Captain avatar (`captainGender`), narration pronouns (`captainPronouns`), and TTS spoken-as (`speakAs`) live on `playerProfiles` and dual-write to Warp `playerStats` so every title can narrate consistently.

**Separate TEI pools:**

| Product | Collection | Leaderboard route |
|---------|------------|-------------------|
| Warp | `playerStats` | iwgf.org/leaderboard/warp |
| Lattice | `latticeTei` (`localAi` \| `online`) | iwgf.org/leaderboard/lattice |

**Lattice-namespaced Firestore:** `latticeRooms`, `latticeRoomCodes`, `latticeTei`, `latticeRatingEvents`.

### Critical deploy rule

- **Firestore rules/indexes are owned by Warp12** (`Warp12/firestore.rules` includes the Lattice fragment).
- Lattice deploy scripts **refuse** Firestore deploy on purpose.
- When Lattice rules change: update **both** `subspace-lattice/firestore.rules` and `Warp12/firestore.rules`, then deploy rules from **Warp12**.

---

## 3. Where to work

| Task | Directory | First read |
|------|-----------|------------|
| Warp gameplay / AI / TEI / Bridge UI | `Warp12/` | `Warp12/AGENTS.md` |
| Lattice gameplay / AI / rooms | `subspace-lattice/` | `subspace-lattice/AGENTS.md` |
| Federation standings / profiles | `leaderboard/` | `leaderboard/README.md` |
| Federation ops / moderation | `ops/` | `ops/README.md` |
| Shared Firestore rules | `Warp12/firestore.rules` | Lattice comment block |
| Store / Tauri Warp | `Warp12/` | `.env.example` |
| Store / Tauri Lattice | `subspace-lattice/` | `docs/desktop-build.md` |

Run installs **per sibling** (`yarn install` in each). Or use `./bin/iwgf`.

---

## 4. `bin/iwgf` cheat sheet

```bash
./bin/iwgf auth                    # firebase + gcloud + npm login
./bin/iwgf warp serve              # Bridge :4200
./bin/iwgf lattice serve           # web :4200
./bin/iwgf leaderboard serve       # :4210
./bin/iwgf ops serve               # :4220
./bin/iwgf warp tauri:dev
./bin/iwgf lattice build:mac
./bin/iwgf federation deploy:leaderboard
./bin/iwgf federation deploy:ops
./bin/iwgf federation deploy:firestore
./bin/iwgf lattice -- yarn evolve  # escape hatch
./bin/iwgf help
```

---

## 5. Stack commonalities

- **Nx 23** + **Yarn 4** for game monorepos; leaderboard/ops are plain Vite packages
- **TypeScript strict**, React 19, Vite 8, Vitest, Playwright
- Package source conditions: `@warp12/source` / `@subspace-lattice/source`
- Prefer Warp12 direct `yarn build:*` (avoid hanging `nx run`); Lattice uses `yarn nx` normally

---

## 6. Agent guardrails

1. Four siblings, four remotes — do not assume shared `node_modules`.
2. **Never deploy Firestore from `subspace-lattice`.**
3. Leaderboard + Ops changes are **federation-wide**.
4. Secrets stay local (`.env`, keystores, `client_secret_*`).
5. Emulators opt-in via local `.env.local` only.
6. Call signs from Federation Profile, not Google display names.
7. After changing submodule HEADs, update the IWGF meta-repo gitlinks.

---

## 7. Doc map

| Doc | Location |
|-----|----------|
| This guide | `AGENTS.md` |
| Dispatcher | `bin/iwgf` |
| Warp | `Warp12/AGENTS.md` |
| Lattice | `subspace-lattice/AGENTS.md` |
| Federation move notes | `Warp12/FEDERATION.md` |
| Lattice desktop/store | `subspace-lattice/docs/desktop-build.md` |
