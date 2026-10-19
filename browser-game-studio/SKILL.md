---
name: "browser-game-studio"
description: "Use when the user wants to design, build, extend or playtest a game in the Games project (Emberveil, Hashfire, Year of Ice or a new one), so work follows the studio playbook, the Director-led studio and its cost rules."
---

# Game studio

Studio motto: be creative, be innovative, use agents. Spend usage wisely.

## Before doing anything
1. Read `claude/STUDIO-PLAYBOOK.md` in the Games project with the Projects tool. It holds the lessons learned, the cost rules, the Director chain and the 16 studio rules (section 6).
2. For an existing game, read only that game's short docs: canon (`<game>/00-canon.md`), DEVGUIDE, ROADMAP, PLAYTEST (feedback outranks the roadmap) and CHANGELOG. For Emberveil these are under `emberveil/web/`. Do not reread the whole codebase. Open only the files the task touches.
3. Get the current source from the live Artifact. Each game lives in its own Artifact: list its files, then read them with `paths`. The starter kit is the `starter-kit/` docs in the project.
4. If the game is a Minecraft mod (Year of Ice), also load the `minecraft-mod-studio` skill. Its build, test and ship rules replace the browser ones below.

## Who does what (the Director chain)
- The chain is the user → Director → Studio lead → departments → sub-agents. The main chat is the Director.
  - The Director is the only one that talks to the user.
  - It writes briefs, settles conflicts, holds the gates and keeps the 16 studio rules.
  - It does not write the content itself when a department can.
- Escalate only the user's decisions, each with options and one recommendation. Keep an "Open decisions" list and don't re-ask settled ones.
- Report each finished piece in a few plain lines: what it adds, the conflicts you ruled on, and what is still running. No recaps of process.

## Size the work to the task (cost control)
- **Small fix or tweak:** do it directly in the main chat. Use no agents.
- **Feature round:** 2-3 specialists with strict file ownership (they use the hook bus or events instead of editing core files), plus one lead/QA agent last.
- **Big fan-outs (5+ agents):** only for major design or pre-production pushes, and only when the user asks.
  - Each long writing agent costs about 400-500k tokens.
  - Say the size before launching. Never launch a duplicate of a running agent.
- **Scope:** one focused goal per round. Batch playtest notes into a single round.
- **Check-ins:** agents notify when they finish. Use at most one fallback check-in every 60-120 minutes. Keep scheduled nightly builds paused unless the user wants them.

## Design-first projects (the blueprint workflow, from Year of Ice)
Use this when the user wants the design "fleshed out before development".
1. **One spec doc with tabs.** The spec is a Claude Docs doc.
   - A tab holds at most 20,000 blocks, so give each big topic its own tab from the start. A full tab still takes text replacements, but no new blocks.
   - Record every tab's id, body node and order key in the session notes and the canon.
2. **Audit the source first.** When the game is based on a video, use the `youtube-watch` skill. Write a numbered concept table: concept, chapter, and where it lives in the blueprint. Then read the comments and sort viewer ideas into Add, Adapt or Decline, with a reason for each.
   - the user wants every concept from the source in. Re-check the table whenever he names one ("did you get X").
   - If a concept exists only as a mention, give it a full system or a tab.
3. **Shared briefs in the scratchpad.** Agents read briefs, not the whole doc.
   - `TAB-BRIEF.md` holds the style rules, the fixed facts (time rates, body formulas, names, sizes, engine) and how to create a tab.
   - `DEPT-BRIEF.md` is for detail passes under existing entries; `DEV-BRIEF.md` is for engineering passes.
   - Quote the user's request verbatim in every brief. When the user changes a fixed fact (for example the engine), add it to the brief at once and SendMessage every running agent.
4. **Every agent:** writes its full text to its own draft file first, uses its own chunk folder (a shared `chunks/` folder caused collisions), inserts about 12 KB per call, checks every ack, never resends what landed, and touches only its own tab or blocks. Its reply gives counts, new ids and numbers, and conflicts with a proposed resolution.
5. **One reconcile list.** Log every cross-tab conflict as a numbered line in `RECONCILE.md`, then rule on it as Director (the engine and lead-architect conventions win). Agents never edit other tabs; one consistency pass applies the list later.
6. **Engineering order.** Run the lead architect first, so the shared layers and names exist. Start the other developers after that, or tell them to align to the lead's section when it lands. In Year of Ice the client developer had to rewrite after the lead finished.
7. **Sanity-check the numbers.** After each tab, check that time, distance, calorie and price numbers still play well, and that names don't clash. Earlier problems were a 4.2 h-per-km travel time, two characters with the same surname, and three different syrup ratios.
8. **Review before development.** Once content is in, run a whole-team review for missing foundations, vague parts, cohesion and survival gaps, and fix what it finds rather than noting it. Then run focused audits of each core loop, and finish with a "Ready for development" checklist.
9. **Keep the project current.** Update the canon and the playbook in the Games project after each round. Export the tabs to markdown (decode `data.bytes_b64` from the export file) for the repo's `docs/`.

## Build rules (browser games)
- JS `terrainHeight` and GLSL `terrainH` must stay bit-identical. Never call `uint()` on a negative int in GLSL; offset coordinates by 1048576.
- Keep saves compatible: put new state under `S.ext.<area>`, with defaults in the `newGame` and `load` hooks. Never rename save keys.
- Taste learned from playtests:
  - The player must always be clearly visible.
  - Keep skies clean, with no screen-wide particle clutter (the ember-fall was removed). Soft-clip bright effects.
  - Give new players a guided first five minutes and an objective beacon.
  - Draw controls as keycaps.
  - Include free-play activities.

## Verify, then ship (browser games)
1. Run `node tools/smoke.js <dir>` (copy it from `emberveil/web/tools-smoke.js` or `starter-kit/smoke.js`). It must print SMOKE PASSED. Use a 960x540 viewport and 120 s screenshot timeouts. Checks wait on game time, never fixed milliseconds. Never run two browser tests at once.
2. Look at a few screenshots with Read: day, night, combat and phone width.
3. Publish to the same Artifact URL with a `files` map of every `js/*.js` file. Bump the build line on the title screen.
4. Update CHANGELOG, ROADMAP, DEVGUIDE (when the architecture changed) and PLAYTEST in the project. Add any new hard-won lesson to `claude/STUDIO-PLAYBOOK.md`.
5. Tell the user in two or three plain sentences what changed and how to try it.

## Starting a new game
Follow section 4 of the playbook:
1. Write a one-page canon using the template.
2. Choose the engine:
   - A browser Artifact is the default for small and medium games.
   - A Minecraft mod (see `minecraft-mod-studio`) suits big open-world sims that need a real voxel world, multiplayer and modding.
3. Browser: copy the starter kit and rename its keys. Build v0.1 in the main chat: the world, a visible player, one great mechanic, one short quest, and the tutorial with its beacon.
4. Smoke-test, publish, and get it to a human fast. Then run small agent rounds.
5. Add the new game to the playbook's games table.