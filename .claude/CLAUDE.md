# Cloud sessions (Claude Code on the web and mobile)

This part applies only when `baserom.us.z64` is missing, which is the
case in a cloud session. On the Mac, ignore it and follow `AGENTS.md`.

Work on the `cheat-hunt` branch. Everything you make goes in `cheats/`:

- `cheats/sm64_hacks.md`: one entry per patch or GameShark code.
- `cheats/patches/<slug>.diff`: the C source change for a patch.

The Mac copies these back later. It builds the ROM, makes the BPS
patches and syncs Batocera and MinUI. None of that is possible here, so
don't try it, and don't write to `~/Downloads`.

## What you can't do here

- There is no ROM. `baserom.us.z64` is not in git and must never be.
  Never commit a ROM, any file made from one (extracted textures, sound,
  `build/`), or bytes copied out of one.
- Don't build. The build takes its assets out of the ROM.
- Until `cheats/sm64.us.map` exists, you have no RAM or ROM addresses.
  Don't make up addresses or copy them from the web.

## Making a patch

A patch here is a change to the C source, saved as a diff. The source
files themselves are never committed.

1. Find the logic in the C source (`src/game/`, `src/engine/`,
   `src/audio/`, `include/` and so on). Make the smallest C change that
   does the job.
2. Save it, then undo it:

       git diff -- src include levels actors > cheats/patches/<slug>.diff
       git checkout -- src include levels actors

   `<slug>` is a short kebab-case form of the title. Check with
   `git status` that no file outside `cheats/` is changed.
3. Check that it still applies: `git apply --check cheats/patches/<slug>.diff`.
4. Write the entry at the bottom of `cheats/sm64_hacks.md`.

Each diff must stand alone: made against `master`, with no other patch
applied. Don't change the Makefile, the linker script or `tools/`.

Keep changes small. Prefer changing a constant, a condition or a single
call. A change that adds code moves every function after it, which is
fine for a BPS but makes the patch hard to combine with others.

## GameShark codes

Only once `cheats/sm64.us.map` exists. Get variable addresses from that
map and struct offsets from the C headers (for example, `struct
MarioState` in `include/types.h`). Format: `8XXXXXXX 00YY` writes one
byte, `81XXXXXX YYYY` writes two bytes (even address). `D0XXXXXX 00YY` /
`D1XXXXXX YYYY` run the next line only if the value there matches.

## Entry format

```
## Short title that says what the player sees

**Source diff:** `cheats/patches/<slug>.diff`
**Changes:** not built yet
**Recorded:** YYYY-MM-DD HH:MM:SS

**Mechanism:** what the original code does (quote the C, with the
function name and source file), and what the patched code does.

**Evidence:** functions, variables and headers checked, and the
`git apply --check` result.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- Side effects and limits.
```

For a GameShark code, put `**Codes:** \`8033B21E 0008\` + ...` in place of
**Source diff:** and **Changes:**, and the map symbol, struct field and
source file in **Evidence:**.

- Newest entry last, in **Recorded:** order. Use simple technical
  English.
- No `Tags:` line unless the user asks for tags. Add or remove tags only
  when asked.
- When porting a cheat from another game, say which one in the Mechanism.
  Other games' cheat files are not in this repo; the user can paste one.

## Commits

Commit each change on `cheat-hunt` and push it. Commit only files in
`cheats/`. A git pre-commit hook (`cheats/githooks/pre-commit`, turned on
by the SessionStart hook) refuses any other file. Messages: `Add: <title>`
for a new entry, `Remove: <title>` for a deleted one, otherwise a short
plain sentence. No attribution lines.
