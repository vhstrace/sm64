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

## Mario never loses a life

**Source diff:** `cheats/patches/never-lose-a-life.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:22:42

**Mechanism:** The four death-exit actions in `src/game/mario_actions_cutscene.c`
(`act_death_exit`, `act_unused_death_exit`, `act_falling_death_exit` and
`act_special_death_exit`) each take a life when Mario lands outside the
painting:

```c
m->numLives--;
// restore 7.75 units of health
m->healCounter = 31;
```

The patch deletes the four `m->numLives--;` lines. Mario still dies and
is thrown out of the course, but the life count never goes down. Ported
from the Zelda cheat "Link cannot die".

**Evidence:** `grep -rn "numLives--" src/` finds only these four lines. The game-over
checks in `level_update_warp_out()`-area code (`src/game/level_update.c`,
`if (m->numLives == 0)`) can then never be true unless you start at 0. `git apply --check cheats/patches/never-lose-a-life.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- Mario still dies and loses his place in the course. Only the life
  count is kept.
- Game over can no longer happen.

## Enemies do no damage

**Source diff:** `cheats/patches/enemies-do-no-damage.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:23:12

**Mechanism:** `take_damage_from_interact_object()` in `src/game/interaction.c` adds
the touching object's damage to the hurt counter:

```c
m->hurtCounter += 4 * damage;
```

The patch makes it `0 * damage`, so nothing is added. Every object
interaction that hurts Mario (`interact_damage`, `interact_flame`,
`interact_shock`, `interact_bully`, `interact_mr_blizzard` and the other
`take_damage_and_knock_back` callers) goes through this function. Ported
from the Zelda cheat "Blue Lynels barely hurt you".

**Evidence:** Other `hurtCounter +=` lines found with grep: lava
(`check_lava_boost`, `interaction.c`), fall damage
(`mario_actions_airborne.c`), burning (`mario_actions_automatic.c`,
`mario_actions_cutscene.c`). The patch does not touch these. `git apply --check cheats/patches/enemies-do-no-damage.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- Fall damage, lava, burning, drowning and falling out of the level
  still hurt or kill Mario.
- Mario is still knocked back and still flashes after a hit.

## The Wing Cap lasts four times as long

**Source diff:** `cheats/patches/wing-cap-lasts-4x-longer.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:23:42

**Mechanism:** `interact_cap()` in `src/game/interaction.c` sets the timer for a cap
taken from a block:

```c
case MARIO_WING_CAP:
    capTime = 1800;
```

The patch changes `1800` (60 seconds) to `7200` (4 minutes). Ported from
the Zelda cheat "Bait lasts about three times as long".

**Evidence:** `capTime` is a `u16`, so 7200 fits. The timer counts down in
`update_and_return_cap_flags()` (`src/game/mario.c`). `git apply --check cheats/patches/wing-cap-lasts-4x-longer.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- The Wing Cap that Mario starts with in Tower of the Wing Cap is set
  in `set_mario_initial_cap_powerup()` (`level_update.c`, 1200) and is
  not changed.
- Collecting a star that exits the course still ends the cap.

## The Metal and Vanish Caps last four times as long

**Source diff:** `cheats/patches/metal-and-vanish-caps-last-4x-longer.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:24:12

**Mechanism:** `interact_cap()` in `src/game/interaction.c` sets the timer for both caps
to 600 frames (20 seconds):

```c
case MARIO_VANISH_CAP:
    capTime = 600;
...
case MARIO_METAL_CAP:
    capTime = 600;
```

The patch changes both to `2400` (80 seconds).

**Evidence:** Exactly two `capTime = 600;` lines in `interaction.c`, both in
`interact_cap()`. `git apply --check cheats/patches/metal-and-vanish-caps-last-4x-longer.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- The caps Mario starts with in the Cavern of the Metal Cap and the
  Vanish Cap level (`set_mario_initial_cap_powerup()`, `level_update.c`)
  are not changed.

## Caps never run out

**Source diff:** `cheats/patches/caps-never-run-out.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:24:42

**Mechanism:** `update_and_return_cap_flags()` in `src/game/mario.c` counts the cap
timer down every frame unless Mario is reading or in a cannon:

```c
if ((m->capTimer <= 60)
    || ((action != ACT_READING_AUTOMATIC_DIALOG) && (action != ACT_READING_NPC_DIALOG)
        && (action != ACT_READING_SIGN) && (action != ACT_IN_CANNON))) {
    m->capTimer -= 1;
}
```

The patch changes `||` to `&&`. Now the timer only counts down once it is
at 60 or less. A normal cap timer starts at 600 or more, so it never
counts down.

**Evidence:** Places that still end a cap, because they set the timer to 60 or less:
collecting a star that exits the course (`interact_star_or_key()`,
`capTimer = 1`) and the ending cutscene (`mario_actions_cutscene.c`,
`capTimer = 60`). `init_mario()` sets `capTimer = 0` on every level or
area load. `git apply --check cheats/patches/caps-never-run-out.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- The cap lasts until you collect a star that exits the course, die,
  or leave the level.
- The cap music keeps playing for as long as the cap lasts.
- Stacks with the two "lasts four times as long" patches, which then do
  nothing.

## The 100-coin star appears at 50 coins

**Source diff:** `cheats/patches/100-coin-star-at-50-coins.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:25:12

**Mechanism:** `interact_coin()` in `src/game/interaction.c` spawns the 100-coin star
when a coin takes the count from below 100 to 100 or more:

```c
&& m->numCoins - o->oDamageOrCoinValue < 100 && m->numCoins >= 100) {
    bhv_spawn_star_no_level_exit(STAR_INDEX_100_COINS);
```

The patch changes both `100`s to `50`. Ported from the Zelda cheat
"Guaranteed drops kick in five times sooner".

**Evidence:** `COURSE_IS_MAIN_COURSE` still limits it to the 15 main courses. `git apply --check cheats/patches/100-coin-star-at-50-coins.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- Reaching 100 coins no longer spawns a second star.
- The coin score saved with the star is whatever you have when you
  touch it.

## Mario runs twice as fast

**Source diff:** `cheats/patches/mario-runs-twice-as-fast.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:25:42

**Mechanism:** `update_walking_speed()` in `src/game/mario_actions_moving.c` sets a
target speed from the stick (at most 32), accelerates toward it with
`1.1f - forwardVel / 43.0f` (which stops near 43), and caps the speed at
48:

```c
targetSpeed = m->intendedMag < maxTargetSpeed ? m->intendedMag : maxTargetSpeed;
...
m->forwardVel += 1.1f - m->forwardVel / 43.0f;
...
if (m->forwardVel > 48.0f) {
    m->forwardVel = 48.0f;
```

The patch doubles the target speed (up to 64), changes `43.0f` to
`86.0f` so acceleration does not stop early, and raises the cap from 48
to 64. Ported from the Zelda cheat "Link walks much faster".

**Evidence:** Only walking and running use `update_walking_speed()` (called from
`act_walking()`). `perform_ground_step()` (`mario_step.c`) moves in four
quarter-steps, so 64 is 16 units per step. `git apply --check cheats/patches/mario-runs-twice-as-fast.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- Long jumps still cap at 48 (`set_mario_action_airborne()`,
  `mario.c`), so a long jump from a full run is slower than the run.
- Slow (`SURFACE_SLOW`) floors still have a lower target, now 48.
- At this speed Mario may run off ledges and into walls more easily.
  Wall collisions at speed 64 are not checked.

## Star doors and the endless stairs need no stars

**Source diff:** `cheats/patches/star-doors-need-no-stars.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:26:12

**Mechanism:** `interact_door()` in `src/game/interaction.c` opens a star door only if
the save file has enough stars:

```c
if (numStars >= requiredNumStars) {
```

The patch makes it `numStars >= 0`, which is always true.

The endless stairs to Bowser in the Sky are not a door. `check_instant_warp()`
in `src/game/level_update.c` skips the stairs' warp-back floor only with
70 stars:

```c
if (gCurrLevelNum == LEVEL_CASTLE
    && save_file_get_total_star_count(...) >= 70) {
    return;
```

The patch makes it `>= 0`. `play_infinite_stairs_music()` in
`src/game/sound_init.c` plays the stairs music while
`gMarioState->numStars < 70`; the patch makes it `< 0` so the music stops
too. Ported from the Zelda cheat "Level 9's entrance opens with no
Triforce pieces".

**Evidence:** The star-door behaviour files do not check stars themselves
(`grep numStars src/game/behaviors/star_door.inc.c` finds nothing). `git apply --check cheats/patches/star-doors-need-no-stars.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- The key doors (basement and upstairs) still need the Bowser keys;
  see `interact_warp_door()`.
- The star-door dialog ("you need X stars") can no longer appear.
