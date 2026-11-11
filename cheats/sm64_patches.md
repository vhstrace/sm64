# Super Mario 64 (US) patches and GameShark codes

Made in cloud sessions on the `cheat-hunt` branch. See `.claude/CLAUDE.md`
for the rules. Each patch's C change is in `patches/<slug>.diff`. The Mac
builds them into BPS patches. Newest entry last.

## Start a new game with 99 lives

**Source diff:** `cheats/patches/start-with-99-lives.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:22:12

**Mechanism:** `init_mario_from_save_file()` in `src/game/mario.c` sets the life count
each time a save file is loaded:

```c
gMarioState->numLives = 4;
```

The patch sets it to `99` instead. Ported from the Zelda cheat "Start a
new game with the maximum number of keys".

**Evidence:** `numLives` is an `s8` in `struct MarioState` (`include/types.h`), so 99
fits. `update_hud_values()` in `src/game/level_update.c` caps lives at
100, so the HUD shows 99 without trouble. `git apply --check cheats/patches/start-with-99-lives.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- It runs on every file load, not only for a new file: every time you
  pick a file from the file select, you start with 99 lives.
- 1-ups still add lives, up to the usual cap of 100.
