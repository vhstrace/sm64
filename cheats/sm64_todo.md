# Super Mario 64 (US) cheat to-do list

Checked items have an entry in `sm64_patches.md`. Numbers match the
first proposal, which was based on the Zelda 1 Game Genie cheats.

## Proposed cheats

### Starting state

- [x] 1. Start a new game with 99 lives
- [ ] 2. Start with both castle keys / both key doors unlocked
- [ ] 3. Start with all three cap switches pressed
- [ ] 4. Skip Peach's letter and the Lakitu intro
- [ ] 5. Turn on the built-in level select (see also "Disabled code" below)

### Health, lives and damage

- [x] 6. Mario never loses a life
- [x] 7. Enemies do no damage
- [ ] 8. Coins heal much more
- [ ] 9. Much longer immunity after a hit
- [ ] 10. Breath never runs out underwater
- [ ] 11. A hit is always fatal

### Items and power-ups

- [x] 12. The Wing Cap lasts four times as long
- [x] 13. The Metal and Vanish Caps last four times as long
- [x] 14. Caps never run out
- [ ] 15. Every coin is worth 5
- [x] 16. The 100-coin star appears at 50 coins
- [ ] 17. Red coin star after fewer red coins

### Movement

- [x] 18. Mario runs twice as fast
- [ ] 19. Much higher jumps
- [ ] 20. Long jump goes much further
- [ ] 21. Mario walks through walls
- [ ] 22. Mario never enters water and walks on the water surface

### Doors and progress

- [x] 23. Star doors and the endless stairs need no stars
- [ ] 24. Key doors open without keys
- [x] 25. Yoshi is on the castle roof without 120 stars
- [x] 26. The castle grounds cannon grate is always open
- [x] 27. MIPS appears with no stars
- [ ] 28. Every cannon is already open

### Enemies and bosses

- [ ] 29. King Bob-omb, Big Bob-omb and Whomp King go down in one hit
- [ ] 30. Bowser goes down after one bomb
- [ ] 31. Goombas run twice as fast
- [ ] 32. Chain Chomp breaks free right away
- [ ] 33. Thwomps and Whomps freeze in place
- [ ] 34. Bob-omb fuses are almost instant
- [ ] 35. Lakitu throws Bob-ombs instead of Spinies
- [ ] 36. Bullet Bill fires much faster or slower
- [ ] 37. Boos are always visible and can be hit from any side
- [ ] 38. Enemies always drop a blue coin
- [ ] 39. Koopa the Quick runs much slower

### L button

- [ ] 40. L refills health
- [ ] 41. L makes Mario invincible while held
- [x] 42. Superhot mode: time moves only while the controller is active
- [ ] 43. L gives the current cap
- [ ] 44. L exits the course without pausing
- [ ] 45. L toggles the debug text (see also "Disabled code" below)

### Presentation and sound

- [x] 46. Dialog pages turn instantly
- [ ] 47. Disable music
- [ ] 48. The castle grounds play the title-screen music
- [ ] 49. The game permanently runs at half speed
- [ ] 50. The title-screen demo starts much sooner
- [ ] 51. Mario's death animation lasts much longer
- [ ] 52. The camera turns much faster

## Disabled code

Code that is in the US ROM but that nothing in the shipped game turns
on. See the "unreachable or disabled code" notes for file and line
references.

### Debug switches

- [ ] Turn on the debug level select: `gDebugLevelSelect = TRUE`
  (`src/game/main.c`). Also gives no poison-gas or deep-water damage and
  sends "Exit Course" to the level select.
- [ ] Turn on the profiler bars: `gShowProfiler = TRUE` (`main.c`;
  drawn by `draw_profiler()`, `game_init.c`)
- [ ] Turn on the debug text (`BUF`, `MEM`, Mario's floor, speed and
  action): `gShowDebugText = TRUE` (`main.c`)
- [ ] Bring back the controller-3 key codes: call
  `handle_debug_key_sequences()` (`main.c`) from the game loop
  (↑↑↓↓←→←→ profiler, ↓↓↑↑←→←→ debug text)
- [ ] Map the key codes to controller 1 instead of controller 3

### Object and stage debug pages (`src/game/debug.c`)

- [ ] Turn on the debug print system: set `gDebugInfoFlags` to
  `DEBUG_INFO_FLAG_DPRINT` (`debug_unknown_level_select_check()`)
- [ ] Show the Mario and level info page: call
  `try_print_debug_mario_level_info()`, which also turns on the live
  value editor `try_modify_debug_controls()`
- [ ] Turn on the D-pad object spawner: call
  `try_do_mario_debug_object_spawn()`
- [ ] Find out what `debug_enemy_unknown()` does (no callers), then call it

### Mario actions

- [ ] Debug free-move (fly and pass through walls): make a button set
  `ACT_DEBUG_FREE_MOVE` (`act_debug_free_move()`,
  `mario_actions_cutscene.c`)
- [ ] Debug cap buttons: call `debug_update_mario_cap()` (`mario.c`).
  It is `UNUSED static`, so check first that it is in the ROM.
- [ ] Check whether `act_unused_death_exit` is ever reached (the castle
  places a `bhvAirborneDeathWarp`), and force it if not

### Beta and unused objects

Each needs an `OBJECT(...)` line in a level script, or a swap for an
object that is already placed.

- [ ] Beta treasure chest: `bhvBetaChestBottom` / `bhvBetaChestLid`
- [ ] Beta Boo key: `bhvBetaBooKey`
- [ ] Beta trampoline: `bhvBetaTrampolineTop` / `bhvBetaTrampolineSpring`
- [ ] Beta Bowser anchor: `bhvBetaBowserAnchor`
- [ ] Beta holdable object: `bhvBetaHoldableObject`
- [ ] Beta moving flames: `bhvBetaMovingFlamesSpawn` / `bhvBetaMovingFlames`
- [ ] Unused poundable platform: `bhvUnusedPoundablePlatform`
- [ ] Unused particle spawner: `bhvUnusedParticleSpawn`
- [ ] Unused fake star: `bhvUnusedFakeStar`
- [ ] Find out what `bhvUnused05A8`, `0DFC`, `1820`, `1F30`, `20E0`,
  `2A10` and `2A54` do, then place them
- [ ] Find out what the one placed `bhvBetaFishSplashSpawner` in Whomp's
  Fortress does (`levels/wf/script.c`)

### Leftovers (probably nothing to see)

- [ ] Check the empty level slots (`LEVEL_UNKNOWN_*`,
  `levels/level_defines.h`). They have no level data, so warping there
  will most likely crash.
