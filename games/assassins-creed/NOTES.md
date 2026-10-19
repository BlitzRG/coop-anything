# Assassin's Creed 1 (2007) — session notes

Agent session started 2026-10-19. Owner wants campaign co-op with friends.

## Phase 0: Triage answers

1. **Which exact game?** *Assassin's Creed* (2007), **Director's Cut Edition**, PC. Steam
   copy (owner's answer), installed at `<game dir>` on drive D. Two 32-bit exes:
   `AssassinsCreed_Dx9.exe` and `AssassinsCreed_Dx10.exe`, version **1.0.2.1**.
   Identity hashes (see below) match EaglePatch's "Confirmed supported exes (GOG/Steam)".
2. **Is it already done?** No co-op mod for AC1 found. Details under Phase 1.
3. **Which engine?** Ubisoft **Scimitar** (the original Anvil; renamed AnvilNext from AC2
   on). Not in `playbook/engine-families.md` → no same-family co-op framework to reuse.
4. **How much source?** Nothing scriptable and no decompilation. But the exe ships full
   MSVC RTTI with **4084 `scimitar::` classes** (2575 top-level), so class names/vtables
   are recoverable. Native hooks proven by EaglePatch.
5. **Does the engine have multiplayer?** No. Confirmed by string scan and by product
   history (series MP arrived in Brotherhood, 2010). A sync layer must be built.
6. **Shipping check.** No anti-cheat, no live online service in gameplay (only Ubisoft
   telemetry, removable). Ship as an ASI/DLL package; no game files redistributed.

## Phase 1: Research — what exists

No public co-op work for AC1, and no netcode to reuse. Observations:

| Find | What it gives | Source |
|---|---|---|
| `Sergeanur/EaglePatch` (C++, ~368★, last release v1.1 2025-12-02) | **ASI plugin** for AC1 & AC2: `DllMain` + `InjectHook`/`PatchByte`, hardcoded addresses per DX9/DX10 build, wraps the engine's own allocator (`ac_allocate`, `ac_getNewDescriptor`, `ac_delete`), reads engine classes (`scimitar::Pad`, `PadProxyPC`, `PadXenon` with vtables/sizes). Proves native in-process patching works on AC1. | https://github.com/Sergeanur/EaglePatch |
| `ThirteenAG/Ultimate-ASI-Loader` | The loader EaglePatch uses (`dinput8.dll` proxy + `scripts/` folder). Reusable foundation for a co-op ASI mod. | (linked from EaglePatch readme) |
| `gentlegiantJGC/ACExplorer` (Python, archived) | `.forge` archive explorer/exporter (Unity first, designed to extend). The local install is `DataPC_*.forge` archives (~6 GB). | https://github.com/gentlegiantJGC/ACExplorer |
| Turfster ".forge extractor/replacer" | Extract/replace AC1 + AC2 (Scimitar) `DataPC_*.forge` incl. textures. | moddb.com/downloads/forge-extractorreplacer-by-turfster |
| `Sev7nOfNine/Universal-Modder` knowledge note (2026-10-05) | Another agent's family-wide survey. Verdict on AC1: "no MP, no framework, tiny community — expect to build tooling from scratch"; "AC1 is a trap for feature work". Also: no public co-op mod for **any** AC game. | github.com/Sev7nOfNine/Universal-Modder (knowledge/games/assassin-s-creed/…) |
| `bloxtbc/SubtitleSynchAC1` | Only other AC1-specific tool named in the survey (subtitles). | (survey) |
| Reddit r/assassinscreed 14ksmrc | Community consensus: AC titles are "notoriously difficult to be modded", mostly retexture/re-shade. | reddit.com/r/assassinscreed/comments/14ksmrc/ |
| Nexus "Origins Together" (AC Origins, in development) | Not AC1; shows demand for AC co-op. | nexusmods.com/assassinscreedorigins/images/377 |

## Phase 2: Local workspace state (owner's PC — private details in `local/`)

- Install: `<game dir>` (Steam library on D:).
- **The install is already patched** by a third-party `ac1-fix` package that was placed in
  the game folder (files `RUN-AC1-FIX.cmd`, `ac1-fix.sh`, created/copied 2026-08 — run
  ~2026-10-19 22:24). Provenance unknown; not audited by us and **not run by us** (Ground
  Rules 8–9). What it did, read from its own script:
  - Zeroed the 16-byte string `gconnect.ubi.com` inside both exes (kills the logo-freeze /
    telemetry), keeping backups in `.ac1-fix-backup/`.
  - Added outbound Windows Firewall blocks for the three exes.
  - Wrote `Launch_AssassinsCreed_{Dx9,Dx10}_31_threads.cmd` (CPU affinity for 32+ thread CPUs).
  - Menu option to open the EaglePatchAC1 PCGamingWiki link (EaglePatch **not** installed).
- Consequence: the exes **on disk are modified**; the hashes below are of the clean
  originals preserved in `.ac1-fix-backup/*.original`.
- No `scripts/` folder and no `dinput8.dll` in the game dir → no ASI loader/EaglePatch yet.
- Config: `%APPDATA%\Ubisoft\Assassin's Creed\Assassin.ini` (present, currently **0 bytes**)
  and `DARE.INI` (DARE audio middleware: DS3D options, `NB_AUDIBLE_VOICES=48`).
- Save files: **none found** under `%APPDATA%`, `%LOCALAPPDATA%` or `Saved Games` → the
  story has not been played here yet, and the save path is **unknown** (Phase 3 task).

### Game identity (recorded, Ground Rule 5)

| File | Version | SHA-256 (clean original) | MD5 | SHA-1 |
|---|---|---|---|---|
| `AssassinsCreed_Dx9.exe` | 1.0.2.1 | `2FA8D50C60398CC249C6D908DE46B15F48E445DD0C605B5956E94FA2F445A713` | `8E72C3333743780E43BC2C34BBF625F9` | `A9EA476E70F076709CCA59A802E26EC894432120` |
| `AssassinsCreed_Dx10.exe` | 1.0.2.1 | `576A04841C37BE5A819974ABD59EA3ECD818F0BC36ACAB71C62EE0313DCA0129` | `CA87753255E2D14B1F18BB737C643792` | `601475FD5EFA02BBAE80D1C06DFA25A0A1E7FD2F` |

MD5/SHA-1 match the EaglePatch readme's "Confirmed supported exes (GOG/Steam)" exactly →
this is the standard Director's Cut build the community tools target.

## Phase 3 (light): Engine facts mined this session (read-only)

From RTTI and a raw string scan of the clean DX9 exe:

- 4084 classes in namespace `scimitar`, 2575 top-level. Gameplay-relevant names seen:
  `Human`, `HumanDecision`, `HumanNPCDecision`, `HumanData`/`HumanDeadData`/`HumanPoleData`/
  `HumanRopeData`, `PDeAssassin`, `AssassinFirstPersonCamera`, `AIGlobalManager`,
  `AISerializedManager`, `AIWorldSpecificManager`, `FuNPCNav`, `Scene`, `SceneHandler`,
  `WorldTransitionManager`, `GamePlayGroupHandler`, `AssassinationConditionClip`,
  `MissionItemSceneSequencer`, `AltairRanker`, `AnimTrack*` (animation track types).
  → A scene/event-driven single-player engine with heavy `EventCallback`/`EventAccumulator` use.
- Networking: `WSAStartup` ×1, `socket` ×1, `gconnect` ×1 (the removable telemetry);
  `DirectPlay`/`NetDriver`/`Replication`/`Udp` = 0. **No gameplay netcode.**
- Input stack already carries 8 pad sets (`Keyboard1..4`, `Joy1..4`) and console pad classes
  (`PadXenon`, `PS2Pad`) — a console-derived port. Not split-screen, but note the breadth.

## Phase 4: Chosen path

**Path 2 — build from zero** (no mod to revive). Recommendation and plan are in
`DESIGN.md`: an ASI plugin + host-authoritative state sync, starting with a puppet player.
This is a multi-week reverse-engineering project, not a session task. Owner chose
"research + design doc first".

## Sources

- https://github.com/cranebeaker/coop-anything (AGENTS.md, DIRECTORY.md, playbook/)
- https://github.com/Sergeanur/EaglePatch (+ its `readme.md`, `src/ac1.cpp`, `EaglePatchAC1.ini`)
- https://github.com/ThirteenAG/Ultimate-ASI-Loader
- https://github.com/gentlegiantJGC/ACExplorer
- https://github.com/Sev7nOfNine/Universal-Modder (AC family survey, 2026-10-05)
- https://www.reddit.com/r/assassinscreed/comments/14ksmrc/any_coop_mod_to_any_assassins_creed_campaign/
- https://www.nexusmods.com/assassinscreedorigins/images/377

## Unknowns

- Where AC1 stores save games and what the save format is.
- Whether a second `Human`/player entity can be spawned and driven (the gate for any co-op).
- Provenance/author of the local `ac1-fix` package.
- Whether `PDeAssassin` is the local player class (name suggests it; unverified).
- Nexus/ModDB site search + Reddit search not run with site tools (web search API returned 403).
