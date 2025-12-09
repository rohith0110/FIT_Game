# Zelda Reloaded – Project Report

## 1. Project Overview

- **Genre & Scope**: 2D top-down action RPG built in Unity 6000.2.8f1 featuring free-form exploration, weapon-based combat, and light RPG systems.
- **Core Fantasy**: Players traverse layered tilemap biomes built from the Tiny Swords pack, swapping between melee slashes and directional archery while clearing enemy waves and looting upgrades.
- **Feature Pillars**:
  - Responsive twin-stick-like movement with contextual elevation handling (`Assets/Scripts/PayerScripts/PlayerMovement.cs`, `Assets/Scripts/tilemap_scripts`).
  - Dual-weapon combat loop (sword + bow) with unlockable upgrades driven by the skill tree (`SkillTree` folder).
  - Persistent progression hooks: experience/levelling, inventory consumables, stat UI, death + retry flows.
  - Multi-scene world stitched with custom spawn points, Cinemachine-follow camera, and scripted tilemap traversal aids.
- **Target Platforms**: Windows desktop build tested via `Build/ZeldaRelloaded_Data`; other platforms possible with matching Unity build support.

![Overview Scene](/DocImages/overview.png)

## 2. Gameplay Breakdown

### Movement & Traversal

- `PlayerMovement` drives WASD/arrow input via legacy Input Manager axes (`Horizontal`, `Vertical`). Facing flips scale for sprite mirroring and handles knockback recovery.
- Elevation & bridge scripts (`tilemap_scripts/ElevationController.cs`, `Bridge.cs`, `Elevation_entry.cs`) toggle layers/sorting orders so the same tilemap supports lower ground, ramps, and mountain tops.
- Cinemachine confiner (`CameraScript/ConfinerFinder.cs`) dynamically binds to the active scene's `Confiner` collider, keeping the camera within authored bounds.

![Movement & Elevation](./DocImages/movement-elevation.png)

### Combat & Weapons

- `PlayerCombat` handles melee swings triggered by the "Slash" button, using overlap circles to damage `Enemy_Health` and apply `Enemy_Knockback`.
- `Player_Bow` provides 8-directional aiming, separate launch points, and arrow prefab instantiation (`Arrow.cs`) tied to `playerShootCooldown`. Arrows inherit knockback stats and stick to enemies/obstacles.
- Skill unlocks gate melee/bow availability; until the relevant skill activates, the weapon scripts remain disabled.

![Combat Snapshot](./DocImages/combat.png)`
![Arrow snapshot](./DocImages/arrow-shot.png)`

### Progression & Skills

- `ExperienceManager` subscribes to `Enemy_Health.OnMonsterDefeated` to gain EXP, scale `expToNextLevel`, and emit `OnLevelUp` (1–3 ability points depending on level bands).
- `SkillTreeManager` listens for level-ups and button clicks. `SkillSlot` assets encode prerequisites/max levels, while `SkillManager` interprets each skill name into concrete stat or ability buffs (unlock slash/bow, damage boosts, health increases).
- Stats and health are centralized in `StatsManager`, exposing runtime-modifiable values and UI hooks. `StatsUI` plus the stats banner toggler display real-time sword/bow damage, speed, kills, and high score.

![Stats Overlay Snapshot](./DocImages/stats-overlay.png)
![Skill Tree Snapshot](./DocImages/SkillTree.png)
![Death Banner Snapshot](./DocImages/death-banner.png)

### Inventory, Loot & Economy

- Scriptable objects (`Inventory & Shop/ItemSO.cs` and per-item assets in `ItemSOs/`) define consumables, gold, and buffs.
- `Loot.cs` handles world pickups, broadcasting via `Loot.OnLootPickedUp`. `InventoryManager` listens, stacks identical items, and spawns overflow loot at `lootDropPoint`.
- Left-click consumption applies stat deltas through `UseItem`, optionally scheduling timed reversions; right-click drops the stack using the same loot prefab to keep art consistent.
- Gold tracking (`goldText`) gives a rudimentary shop/economy hook even if UI is not yet exposed.

![Inventory UI Snapshot](./DocImages/inventory.png)

### World Structure & Scene Flow

- Two hand-authored scenes (`Assets/Scenes/Scene1.unity`, `Scene2.unity`) share persistent systems kept alive by `GameManager` (`DontDestroyOnLoad`).
- `SceneChanger` & `SceneChanger2` colliders teleport the player between scenes with optional fade animations and spawn repositioning.
- `PlayerSpawner` enforces first-entry tutorial flow and ensures consistent spawn placement per scene.
- `SpawnManager` + `SpawnArea` dynamically place enemies and loot using ground tile sampling, respecting forbidden tiles and minimum distance from the player. Newly spawned actors inherit elevation state to blend with the layered tilemaps.

![Teleport](./DocImages/Teleport.png)

### UI, Feedback & Meta Systems

- **Banners**: `TutorialBanner` pauses the game until dismissed; `DeathBanner` freezes time, shows kills/high score, and provides a full reset pipeline (stats, inventory, XP, skill tree, weapons).
- **Stats / Skill overlays**: `ToggleSillTree` and `StatsUI` pause/resume `Time.timeScale` when open, preventing combat while players plan builds.
- **Persistence**: `Stats` and `PlayerData` wrap `PlayerPrefs` for best kill count/high score tracking and first-time detection.

![Tutorial Banner Snapshot](./DocImages/Welcome.png)

## 3. Technical Architecture

### Managers & Singletons

- `GameManager`, `StatsManager`, `InventoryManager`, and `ExperienceManager` act as lightweight singletons for global data/state.
- `DeathBanner.ResetAllSystems()` demonstrates how subsystems cooperate when resetting after failure.

### Data & Scriptable Objects

- `SkillSO` and `ItemSO` live under `Assets/Scripts/SkillTree/Skills` and `Assets/Scripts/Inventory & Shop/ItemSOs`. Designers can add new assets without touching code, then wire them to `SkillSlot` or loot tables.
- Enemy prefabs reference the shared stats in `StatsManager`, allowing quick global retuning.

### AI & Combat

- Enemy state machine (`Enemy_Movement`) supports idle, chase, direction-biased attacks, and temporary knockback states.
- `Enemy_Combat` selects attack points per animation, running overlap checks for the player layer, then applying `PlayerHealth` damage + knockback.

### World & Tilemaps

- `SpawnArea` samples `Tilemap.CellToWorld` coordinates and ensures characters never pop into forbidden tiles or too close to the player.
- Elevation controllers use Unity layer switching (`PlayerTop`, `PlayerBottom`, etc.) for collision/renderer sorting, letting bridges and ramps feel physical without separate scenes.

### Camera & Presentation

- Cinemachine 2D confiner auto-wires on scene load, so each map just needs a `Confiner` GameObject with a 2D collider defining bounds.
- Animations (`Enemy_idle.anim`, `Enemy_Torch.controller`, player animator states) tie into code via `anim.SetBool` and animation events (`AnimationEvent_AttackHit`).

## 4. Content & Asset Pipeline

- **Art Packs**: Tiny Swords (Update 010) for tiles, props, characters; `simple_rpg_icons` for skill sprites; TextMesh Pro for crisp UI.
- **Prefabs**: Stored under `Assets/Prefabs` (enemies, arrows, loot, UI banners). Ensure newly created prefabs live here for easy versioning.
- **Scenes**: `Scene2` (starting zone) and `Scene1` (secondary biome) each host tilemap stacks, spawners, and scene change triggers. Additional scenes should replicate the confiner + spawner setup for consistency.

![Assets & Prefabs Snapshot](./DocImages/image.png)

## 5. Build & Execution Instructions

### Prerequisites

1. Install **Unity 6000.2.8f1** (Windows, matching `ProjectSettings/ProjectVersion.txt`).
2. Ensure the legacy Input Manager axes/buttons include: `Horizontal`, `Vertical`, `Slash`, `Shoot`, `ToggleStats`, `ToggleSkillTree`. These live in `ProjectSettings/InputManager.asset` and load automatically with the repo.
3. Install Cinemachine and TextMesh Pro packages via Unity Package Manager if the library cache was cleared.

### Opening the Project

1. Clone/copy the entire `Final2` folder.
2. Launch Unity Hub → **Open** → select `Final2/Final2.sln` or the containing folder.
3. Allow the Library to re-import (first open may take a few minutes because of the Tiny Swords textures).

### Running in Editor

1. Open `Assets/Scenes/Scene2.unity`.
2. Press Play. The tutorial banner pauses time until closed.
3. Use keyboard inputs:
   - `WASD` / arrow keys – movement & aiming direction.
   - `LMB` – melee attack once unlocked.
   - `RMB` – bow attack once unlocked.
   - `Esc` – stats banner toggle (`ToggleStats`).
   - `1` – open/close skill tree (`ToggleSkillTree`).
4. Kill enemies to earn EXP and ability points; open the skill tree to unlock weapons and upgrades.
5. Loots via the glowing chest/coin prefabs; left-click to consume, right-click to drop back into the world.

### Building a Standalone

1. File → Build Settings → ensure both `Scene2` and `Scene1` are in the Scenes In Build list.
2. Target platform: Windows (x86_64). Switch if necessary.
3. Click **Build** → output to `Build/Windows` (existing structure already contains a reference build named `ZeldaRelloaded.exe`).

### Save Data Notes

- High score (`PlayerPrefs` key `HighScore`) persists between runs. Delete via `Edit → Clear All PlayerPrefs` or manually reset in code for clean captures.
- `PlayerSpawner.hasSpawnedInThisScene` prevents re-triggering the tutorial; call `DeathBanner.Retry()` or restart the editor to test first-time flow.

## 8. Team Contributions

Team roster: Rohith Rathod (SE23UCSE034), Arasu Sujith Reddy (SE23UCSE029), Manohar Chowdary (SE23UCSE109), Shiva Karan (SE23UCSE063), Aditya Goyal (SE23UARI003), Chirag Goyal (SE23UCSE050).

| Team Member        | Responsibilities                                                                                                                        | Demo Focus                                                                        |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Rohith Rathod      | Lead gameplay engineer: movement + combat systems, EXP/skill progression, death/reset flow, and overall integration/debugging.          | Show controller/skill tree, walk through scripts, explain architecture decisions. |
| Arasu Sujith Reddy | Tilemap & elevation helper: painted the maps, placed elevation triggers/bridges, and lined up the Cinemachine confiners.                | Tour Scene1/Scene2 layouts and show how the elevation swaps keep depth readable.  |
| Manohar Chowdary   | Inventory helper: wired the inventory UI prefab to ItemSO data, configured loot pickups, and updated gold/quantity text.                | Open inventory, pick up/drop loot, explain how items show up in slots.            |
| Shiva Karan        | Enemy helper: duplicated the basic enemy prefab, attached spawn areas, and verified knockback works with default stats.                 | Spawn enemies live and show they chase/attack using the provided prefab.          |
| Aditya Goyal       | UI & spawn helper: set up StatsUI/PlayerHealth/TutorialBanner prefabs, TextMesh Pro styling, and assisted with PlayerSpawner placement. | Highlight tutorial/health banners, stats overlay toggle, and initial spawn flow.  |
| Chirag Goyal       | Flow helper: added PlayerPrefs wrappers, the simple GameManager bootstrap, and wired the SceneChanger triggers between both scenes.     | Show how high score persists, how GameManager keeps objects, and demo scene swap. |

## 9. Script Ownership Map

Only Rohith authored new gameplay code; the rest of the team primarily hooked up existing scripts in scenes/prefabs as listed below.

| Team Member        | Key Scripts / Paths                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Rohith Rathod      | `Assets/Scripts/PayerScripts/PlayerMovement.cs`, `Assets/Scripts/PayerScripts/PlayerCombat.cs`, `Assets/Scripts/PayerScripts/Player_Bow.cs`, `Assets/Scripts/PayerScripts/ExperienceManager.cs`, `Assets/Scripts/StatsManager.cs`, `Assets/Scripts/SkillTree/SkillManager.cs`, `Assets/Scripts/SkillTree/SkillTreeManager.cs`, `Assets/Scripts/SkillTree/SkillSlot.cs`, `Assets/Scripts/SkillTree/ToggleSillTree.cs`, `Assets/Scripts/DeathBanner.cs`. |
| Arasu Sujith Reddy | Hooked up scene objects that reference `Assets/Scripts/tilemap_scripts/Bridge.cs`, `Assets/Scripts/tilemap_scripts/ElevationController.cs`, `Assets/Scripts/tilemap_scripts/Elevation_entry.cs`, and `Assets/Scripts/CameraScript/ConfinerFinder.cs`.                                                                                                                                                                                                  |
| Manohar Chowdary   | Configured inventory prefabs that rely on `Assets/Scripts/Inventory & Shop/InventoryManager.cs`, `InventorySlot.cs`, `ItemSO.cs`, `UseItem.cs`, `Loot.cs`, plus the ScriptableObjects in `Inventory & Shop/ItemSOs/`.                                                                                                                                                                                                                                  |
| Shiva Karan        | Duplicated the provided enemy prefab/spawner setup using `Assets/Scripts/EnemyScripts/Enemy_Movement.cs`, `Enemy_Combat.cs`, `Enemy_Health.cs`, `Enemy_Knockback.cs`, `SpawnManager.cs`, and `SpawnArea.cs`.                                                                                                                                                                                                                                           |
| Aditya Goyal       | Wired `Assets/Scripts/StatsUI.cs`, `Assets/Scripts/PayerScripts/PlayerHealth.cs`, `Assets/Scripts/TutorialBanner.cs`, and positioned `Assets/Scripts/PlayerSpawner.cs` in both scenes.                                                                                                                                                                                                                                                                 |
| Chirag Goyal       | Added the lightweight persistence wrappers `Assets/Scripts/PlayerData.cs`, `Assets/Scripts/Stats.cs`, set up `Assets/Scripts/GameManager.cs`, and wired `Assets/Scripts/SceneChange/SceneChanger.cs` / `SceneChanger2.cs`.                                                                                                                                                                                                                             |

--- End of Report --
