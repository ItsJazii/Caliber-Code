# MEMORY.md — Caliber Studio Windows work (jazii)

Portable handoff/context file. Everything done so far, how to run it, and what's next —
so this work can continue from any machine (clone the fork, read this, go).

**Last updated:** 2026-08-14

---

## What this repo is

**Caliber Studio** — duola's (Ziwen's) AI game-making tool, built on a fork of
[opencode](https://opencode.ai). Prompt on the left, agent builds a game, and the game
runs live in a neon "Arcade" panel (CRT cartridge UI) on the right, with telemetry
(FPS / events / errors) flowing back to the agent so it can verify its work by playing.

- Upstream repo: `duolahypercho/Caliber-Code` (single squashed initial commit)
- My fork: **`ItsJazii/Caliber-Code`** ← work lives here
- ~95% is upstream opencode monorepo. The Caliber-original parts:
  - `opencode/caliber-core/` — Rust (axum) service, port **4870**: game scaffold /
    discover / register / static play-serving / playtest frame recordings / activity feed
  - `opencode/packages/app/src/caliber/` — the Arcade panel UI (SolidJS + CSS, ~1.8k LOC)
  - `opencode/CALIBER.md`, `CALIBER-GAMES.md` — identity + agent-facing game rules
  - Root planning docs: `CALIBER_STUDIO_PLATFORM_PLAN.md` (master plan v4: multi-engine,
    Web3D → Godot → Unity → Unreal), PRD/master-plan .docx, `CALIBER_LOOP_STATE.md`
    (duola's build-loop iteration log — read this to see how far HE is)

## What we did (2026-07-24 → 08-14)

1. **Cloned + audited** the repo (security-clean: no secrets; permissive CORS +
   localhost file-writes are dev-tool-acceptable).
2. **Windows port — DONE and verified end-to-end:**
   - Fixed `caliber-core/src/bin/app.rs`:
     - hardcoded `/Users/ziwenxu/...` opencode dir → walk-up discovery of
       `packages/opencode` from exe + cwd (`CALIBER_OPENCODE_DIR` still overrides)
     - bundled-studio lookup now checks `<exe dir>/studio` (Win/Linux) in addition
       to `../Resources/studio` (macOS .app)
   - `cargo build` clean (stable Rust 1.96, MSVC). caliber-core serves on :4870 —
     health/scan/scaffold/discover/play all verified (scaffolded "Windows Test" game
     with Windows paths and served it back).
   - Studio web UI boots (`bun run dev:web`, :3000); `opencode serve` listens (:4096).
   - **Native `caliber-app` window opens via WebView2** — first non-macOS run ever.
3. **Shipped as [PR #1](https://github.com/duolahypercho/Caliber-Code/pull/1)**
   (branch `windows-support`, commits `57c9970` + `5c6d0f7`).
   - `5c6d0f7` fixes a real bug Codex auto-review caught (walk-up skipped the starting
     directory itself). Status when last checked: MERGEABLE, GitGuardian green,
     "ready to merge" comment posted. **Waiting on duola's review.**

## Machine setup (Windows box: jazii's laptop)

- Working copy: **`D:\Caliber-Code`** (branch `windows-support`)
  - ⚠ MUST be on **NTFS**. bun's cache/linking hard-fails on exFAT (E: is exFAT —
    a stale abandoned first clone sits at `E:\Game Developement\caliber\Caliber-Code`).
- bun 1.3.14 at `E:\Tools\bun` (add `E:\Tools\bun\bin` to PATH); Node 24; Rust/cargo 1.96
- bun cache: `BUN_INSTALL_CACHE_DIR=D:\BunCache`
- Install gotchas (all non-fatal):
  - `tree-sitter-powershell` postinstall wants node-gyp → safe to skip
  - husky warns `.git can't be found` (opencode/ isn't the git root) → harmless
  - If node_modules half-installs (e.g. was on exFAT), RENAME it aside and
    `bun install` fresh — don't try to repair in place.

## How to run the full stack (3 processes)

```bash
# 1. Studio web UI  → http://localhost:3000
cd opencode && bun run dev:web

# 2. Agent backend  → 127.0.0.1:4096
cd opencode/packages/opencode && bun run --conditions=browser src/index.ts serve --port 4096

# 3. Rust core      → 127.0.0.1:4870
cd opencode/caliber-core && cargo run
```

Native shell instead of a browser tab:
```bash
CALIBER_STUDIO_URL=http://localhost:3000 ./opencode/caliber-core/target/debug/caliber-app.exe
```

Smoke test: `curl http://localhost:4870/health` → `{"ok":true,...}`; POST
`/games/scaffold` with `{"directory":"<abs dir>","name":"My Game"}` → returns play_url.

## TODO / next steps

- [ ] **duola merges PR #1** (nudge him)
- [ ] **Windows packaging** — mirror Caliber.app: build studio dist, place at
      `<exe>/studio`, ship caliber-app.exe + core as one bundle (installer or zip).
      caliber-app already spawns `opencode serve` as managed child.
- [ ] Laptop/perf pass (16GB RAM target)
- [ ] Linux untested
- [ ] Cosmetic: scaffold response mixes `/` and `\` in returned paths on Windows
- [ ] Longer term (from duola's master plan): agent-plays-the-game MCP tool, Godot
      adapter spike, engine-neutral protocol — the **Unreal adapter** (phase 4) is
      basically our GT-Caliber remote-exec/PIE workflow productized; that's my lane.

## Related context (not in this repo)

- The game itself (GT-Caliber, UE 5.8, Perforce //GTCaliber/main @ CL47) is **paused**
  as of 2026-08-14. Car work (BMW M4 import + chassis fix, Porsche drift-tune restore)
  is saved locally + backed up at `D:\GTC-pause-backup`; NOT yet submitted to Perforce
  (server = duola's Mac over Tailscale, was offline). Chassis-box drive-verdict never run.
- Fuller machine-side memory lives in Claude Code's memory dir on the laptop:
  `C:\Users\jazii\.claude\projects\E--Game-Developement\memory\`

---

## Session 2026-08-14: repo became CaliCode; fixed duola's red main (PR #40)

- duola **merged PR #1**, then **rewrote the repo → "CaliCode"** (force-pushed main,
  75 commits): native game-dev coding agent, Rust core + TS/React client, no more
  opencode fork. Old windows-support work is historical (merged pre-rewrite).
- His `e4a416a` left main's CI red. Shipped **PR #40** (branch `fix/visual-baselines`,
  ALL CHECKS GREEN): ① 8 regenerated Linux visual baselines (via his visual-baselines
  workflow on the fork — workflow_dispatch, artifact download); ② deflaked
  `loop-gate.spec.ts` (project click double-remounts `<AgentPanel key={slug:revision}>`
  — wait for `[data-empty-game-hint]` before typing); ③ **real bug fix** in
  AgentPanel.tsx `runLoop`: only the blocked path persisted the transcript after the
  loop — completed/capped/stopped loops lost their tail lines on reload (one
  `persistLoopTranscript()` after the exit branches); ④ hardened visual-baselines.yml
  (continue-on-error + `if: always()` upload — a flaky spec had discarded all 8 PNGs).
- Fork main synced to the rewritten upstream. GT-Caliber game remains paused.
