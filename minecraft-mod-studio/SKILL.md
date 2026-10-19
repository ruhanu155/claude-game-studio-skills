---
name: "minecraft-mod-studio"
description: "Use when the user's game is built as a Minecraft Java mod or modpack (NeoForge), such as Year of Ice: setting up the project, writing systems and content, testing, and shipping the evolving modpack."
---

# Minecraft mod studio (NeoForge)

Use this together with `browser-game-studio`, which covers the Director chain, the studio rules and the cost rules. This skill replaces only that skill's build, test and ship steps.

## Before doing anything
1. Read the game's canon in the Games project (Year of Ice: `yearofice/00-canon.md`) and `claude/STUDIO-PLAYBOOK.md`.
2. Read the spec's engineering tabs, not the whole blueprint.
   - Year of Ice: "Minecraft engine" (conventions) and "Minecraft development": architecture and the ten shared layers, world, entities, player and client, pipeline and modpack.
   - Read only the sections the task touches.
3. Work in the game's git repo. Use `add_repo` with push access, clone it, work on a `dev/<department>/<topic>` branch, and open a pull request. Don't push to `main`.

## The pin rule
- At M0, pin the newest Minecraft version that has a stable NeoForge release and releases of every required library (GeckoLib, Curios).
- Write the exact versions in `gradle.properties`. Move the pin only in its own round after a milestone ships, on `upgrade/mc-<version>`.
- Version names changed after 1.21.11: 26.1 is the successor line and uses Java 25 with no obfuscation. 1.21.1 uses Java 21 and has the most mature libraries.
- GameTests: annotated up to 1.21.4, `test_instance` data from 1.21.5.
- Check the current NeoForge and library versions on the web before pinning. Don't rely on memory.

## Project shape
- Mod id and package: `<modid>` and `com.<modid>.<system>`.
- Pure rules live in `com.<modid>.sim` with no Minecraft imports. They are unit-tested in JUnit and run by a headless balance simulator.
- Adapters in the system packages read the world, call the core and write results back.
- **Server decides, client shows.** Client-only code goes in `com.<modid>.client`, registered with `Dist.CLIENT`. A dedicated-server boot test catches any leak.
- Systems talk only through service interfaces and NeoForge events in `com.<modid>.api`. Each piece of shared state has one writing package. An ArchUnit test fails the build on cross-package imports.
- At most 8 Mixins, whitelisted in `<modid>.mixins.json`, each with a test. Patch other mods only through their public API in `compat.<modid>`.

## Naming conventions (keep every doc and file in line)
| Thing | Convention |
| --- | --- |
| Registry id | `<modid>:<snake_case>` |
| Content data | `data/<modid>/<kind>/<id>.json`, singular kind, one entry per file, loaded by reload listeners; never a doubled namespace |
| Vanilla kinds | Singular folders: `recipe/`, `loot_table/`, `tags/block/`, `structure/`, `worldgen/`, `neoforge/biome_modifier/` |
| Per-item and per-block numbers | NeoForge data maps, `data/<modid>/data_maps/item/<name>.json` |
| JSON fields | snake_case with a unit suffix (`weight_kg`, `spoil_days_at_5c`). A validator rejects camelCase |
| Player and chunk state | Data attachments (for example `<modid>:body`); per-stack state as data components |
| Sounds | `<modid>:<group>.<name>` |
| GameTests | `<modid>:<system>/<test>`, batched by milestone |
| Lang | `entity.<modid>.x`; custom UI text gets its own prefix |

## Replacing vanilla (total conversion)
- List every vanilla system the design replaces and switch it off on purpose: hunger, hearts, regen, crafting table, vanilla mobs, mob loot, bone meal, sleep skipping, F3 coordinates, and whatever else the spec covers.
- Use game rules where they exist (`doMobLoot` false, `maxEntityCramming` 0, `reducedDebugInfo` true).
- Use a built-in data pack to filter vanilla recipes, advancements and blocks, and a required built-in resource pack for the look.
- Remove unwanted mobs in `EntityJoinLevelEvent`, with an allow-list tag. The smoke counts every joining entity and must see zero banned types.
- Every animal death makes a carcass, not vanilla drops.
- Use HUD layers through NeoForge's GUI layer registration, not vanilla hearts or hunger.

## Adding content end to end (one species, item or block)
1. Open a ticket naming the files.
2. Add the entry to the registry inventory with its milestone.
3. Write the data file with its tuning numbers from the spec.
4. Add the model and animations (GeckoLib or Blockbench), the textures, and the sounds (mono ogg, 44.1 kHz).
5. Add the lang lines, the tags and the loot or yield.
6. Run `./gradlew contentLint` until it is clean.
7. Write at least two GameTests, each proving a number from the spec.
8. Add a screenshot shot if the silhouette is new.
9. Run the smoke, then open the pull request.

## Test and ship
- **Test layers:**
  - `./gradlew test`: JUnit, sim and data validation.
  - `runGameTestServer`.
  - The client smoke under a virtual display.
  - A dedicated-server boot for 600 ticks.
- **SMOKE PASSED** (`./gradlew smoke`) needs all of these: build plus datagen match, every test passing, a clean server boot, a clean client boot (no NaN, no unhandled payload), and zero banned mobs.
  - Only one client run at a time.
- **Performance budget:** 20 TPS, average MSPT under 30 ms, and no hitch over 50 ms. Profile with spark (dev only).
- **Saves:**
  - Never re-use or rename an id after the first beta; retire it with `DeferredRegister#addAlias`.
  - Use versioned codecs for attachments and components.
  - Keep a lazy migration with a backup first, and fixture saves per release in a migration test.
- **The modpack:** packwiz in `pack/` exports `.mrpack` and a CurseForge zip.
  - The pack's MAJOR.MINOR equals the mod's.
  - Versions look like `MAJOR.MINOR.PATCH+mc<version>`; before 1.0, the minor is the milestone.
  - The changelog is built from `Changelog:` commit trailers.
- **Other mods:**
  - Performance and cosmetic mods are allowed and tested (Sodium, Iris, Distant Horizons).
  - Mods that break the pillars (minimaps, ores, hostile mobs) get a warning on first launch.
  - Mods that replace inventory, hunger or time are blocked in `neoforge.mods.toml`.

## In the cloud workspace
- Check `java -version` and install the JDK the pin needs.
- Gradle and Maven downloads (maven.neoforged.net, Maven Central, the GeckoLib and Curios repos) may be blocked by the network allowlist. Test with one `./gradlew --version` first, and tell the user plainly if dependencies can't download. Don't loop on retries.
- A real Minecraft client needs a display. If `xvfb-run` isn't available, run JUnit and GameTests headless, and report the client smoke as "not run here".
- Keep build output and caches small. The disk allowance is fixed.

## Lessons from Year of Ice
- Design in the doc first, then engineer. The lead architect's section (shared layers, package map, names) comes before the other developers' sections.
- When the engine changes mid-design, keep the design and change only how it is built. Add an ENGINE section to the shared brief and message running agents.
- "No hostile mobs" and "explore, don't dig" are enforced in code and tests (banned mob counter, a dig-depth guard over unbreakable `deep_ground`), not just in data.
- Time in Minecraft is ticks: keep one clock layer as the rate arbiter (world, travel, work time-lapse, close clock). Never let systems read `level.getDayTime()` directly.