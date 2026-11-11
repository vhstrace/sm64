# Super Mario 64 (US) patches and GameShark codes

Made on the Mac and in cloud sessions. The main copy is the gist; a copy
is in `cheats/sm64_hacks.md` on the `cheat-hunt` branch of `vhstrace/sm64`,
with each patch's C change in `cheats/patches/<slug>.diff`. The Mac builds
them into BPS patches. Newest entry last.

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
checks in `level_trigger_warp()` (`src/game/level_update.c`,
`if (m->numLives == 0)`) can then never be true. `git apply --check cheats/patches/never-lose-a-life.diff`
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

The patch makes it `0 * damage`, so nothing is added. Enemy hits go
through this function: `interact_shock` and `interact_snufit_bullet`
call it directly, and `take_damage_and_knock_back()` (used by
`interact_damage`, `interact_mr_blizzard`, `interact_hit_from_below`,
`interact_bounce_top` and `interact_clam_or_bubba`) calls it too. Ported
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
- Fall damage, lava, drowning and falling out of the level still hurt
  or kill Mario.
- Flames still hurt: `interact_flame()` sets Mario on fire, and the
  burning actions do the damage.
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
**Changes:** the C change moves code (data moves by 16 bytes), so no word list. `mario-runs-twice-as-fast.bps` is 145,071 bytes.
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

## Yoshi is on the castle roof without 120 stars

**Source diff:** `cheats/patches/yoshi-on-the-roof-without-120-stars.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:26:42

**Mechanism:** `bhv_yoshi_init()` in `src/game/behaviors/yoshi.inc.c` removes Yoshi
unless the file has 120 stars:

```c
if (save_file_get_total_star_count(...) < 120
    || sYoshiDead == TRUE) {
    o->activeFlags = ACTIVE_FLAG_DEACTIVATED;
```

The patch changes `< 120` to `< 0`, which is never true.

**Evidence:** `sYoshiDead` is set when Yoshi jumps off the roof after you talk to
him, and cleared by `set_yoshi_as_not_dead()` (`src/game/obj_behaviors.c`)
on file select. So he still leaves after one talk, until you pick a file
again. `git apply --check cheats/patches/yoshi-on-the-roof-without-120-stars.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- The roof is normally reached by the cannon behind the grate, which
  also needs 120 stars. Use "The castle grounds cannon grate is always
  open", or fly up with the Wing Cap.
- Talking to Yoshi gives 100 lives and the triple-jump flag as usual.

## The castle grounds cannon grate is always open

**Source diff:** `cheats/patches/cannon-grate-always-open.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:27:12

**Mechanism:** `bhv_castle_cannon_grate_init()` in
`src/game/behaviors/castle_cannon_grate.inc.c` removes the grate over the
castle-grounds cannon once the file has 120 stars:

```c
if (save_file_get_total_star_count(...) >= 120) {
    o->activeFlags = ACTIVE_FLAG_DEACTIVATED;
```

The patch changes `>= 120` to `>= 0`, so the grate is always removed.

**Evidence:** This is the whole function. `git apply --check cheats/patches/cannon-grate-always-open.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- Combine with "Yoshi is on the castle roof without 120 stars" to meet
  Yoshi early.

## MIPS appears with no stars

**Source diff:** `cheats/patches/mips-appears-with-no-stars.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:27:42

**Mechanism:** `bhv_mips_init()` in `src/game/behaviors/mips.inc.c` shows MIPS for his
first star at 15 stars and his second at 50:

```c
if (save_file_get_total_star_count(...) >= 15
    && !(starFlags & ...MIPS_STAR_1)) {
...
else if (save_file_get_total_star_count(...) >= 50
         && !(starFlags & ...MIPS_STAR_2)) {
```

The patch changes both to `>= 0`. MIPS gives his first star, then his
second, without any stars needed.

**Evidence:** MIPS is still removed once both of his stars are collected. `git apply --check cheats/patches/mips-appears-with-no-stars.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- MIPS is in the castle basement, so you still need the first Bowser
  key (or the basement door unlocked) to reach him.

## Dialog pages turn instantly

**Source diff:** `cheats/patches/dialog-pages-turn-instantly.diff`
**Changes:** not built yet
**Recorded:** 2026-11-11 02:28:12

**Mechanism:** SM64 prints a whole page of text at once, so the slow part is the page
scroll. `render_dialog_entries()` in `src/game/ingame_menu.c` scrolls the
text up by `linesPerBox * 2` each frame until it reaches
`linesPerBox * DIAG_VAL1`:

```c
case MENU_STATE_DIALOG_SCROLLING:
    gDialogScrollOffsetY += dialog->linesPerBox * 2;
```

`DIAG_VAL1` is 16 on US, so a page takes 8 frames. The patch adds
`dialog->linesPerBox * DIAG_VAL1` instead, so the next page shows on the
next frame. Ported from the Zelda cheat "Fastest possible NPC text".

**Evidence:** `DIAG_VAL1` is defined just above the function (16 for US and EU, 20
for JP and SH). The test that ends the scroll is
`gDialogScrollOffsetY >= dialog->linesPerBox * DIAG_VAL1`, so it is met
after one frame on every version. `git apply --check cheats/patches/dialog-pages-turn-instantly.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Not tested.
- Not built. The Mac fills in **Changes:** (the ROM bytes) after it
  builds the patch.
- The box still spins open and closed at the normal speed.
- You still press A or B for each page.

## Superhot mode: time moves only while the controller is active

**Source diff:** `cheats/patches/superhot-mode.diff`
**Changes:** the C change moves code, so no word list. `superhot-mode.bps` is 157,542 bytes.
**Recorded:** 2026-11-11 02:43:53

Tags: `demo-ready` `play-start` `video-ready`

**Mechanism:** `play_mode_normal()` in `src/game/level_update.c` runs once
per frame during play. It counts up the course timer and then updates
every object:

```c
if (sTimerRunning && gHudDisplay.timer < 17999) {
    gHudDisplay.timer++;
}

area_update_objects();
```

The patch wraps both in a test of controller 1:

```c
if (gPlayer1Controller->buttonDown != 0 || gPlayer1Controller->stickMag > 0.0f) {
```

So the world moves only on frames when any button is held (A, B, Z,
Start, L, R, C buttons or the D-pad) or the analog stick is pushed. With
no input, everything stands still. Ported from the Zelda cheat "Time
moves only while a button is held", with the analog stick added.

**Evidence:** Mario is an object too: `bhvMario` (`data/behavior_data.c`)
calls `bhv_mario_update()` (`object_list_processor.c`), which runs
`execute_mario_action()`. So `area_update_objects()` moves Mario and
every other object. `adjust_analog_stick()` (`src/game/game_init.c`) sets
`stickX` and `stickY` to 0 when the raw stick is within 7 of center, so
`stickMag` is 0 inside the dead zone and above 0 outside it. The stick
counts as soon as it leaves the dead zone. `buttonDown` is the
held-buttons mask in `struct Controller` (`include/types.h`).
`git apply --check cheats/patches/superhot-mode.diff`
passes against `master`.

**Revisions:** US (rev 0) only. Others not checked.

**Caveats:**

- Tested by the user in RetroArch (Mupen64Plus-Next) on 2026-11-10: works.
- This patch adds code, so every function after `play_mode_normal()` in
  `level_update.c` moves. That is fine for a BPS, but it may not combine
  cleanly with other patches made as ROM bytes.
- Still running with no input: the camera (`update_camera()`), the HUD,
  warps and fades already started, the pause check, music and sound, and
  anything driven by `gGlobalTimer` (some animated textures and effects).
- Only controller 1 counts.
- A stick that drifts past the dead zone keeps time running.
- Demos on the title screen feed their own input to controller 1, so
  they run as usual.
