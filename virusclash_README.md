
---

# Project: Virus Clash (2D RTS)

## 🏗️ Currently Working On
* **Enemy AI:** Implementing advanced ability usage (AttackBlob charging).

## 📝 TODOs
* [x] Implement custom steering for Projectiles (Path Following).
* [x] Optimize NavMesh baking parameters (Agent Radius/Voxel Size) for tight obstacle edges.
* [x] Visual Polish: Implement custom liquid-fill shader with animated falling drops.
* [x] Visual Polish: Procedural idle bobble animation (squash and stretch).
* [x] ~~Biological Background: Advanced Procedural Vein Shader and Mouse Parallax.~~ (Cut)
* [x] Visual Polish: Procedural eye animation (blinking and mouse-tracking).
* [x] Implement Biological Obstacles (SVG Macro-Tiles + Prefab Brush).
* [x] Implement Enemy AI Foundation (Team-based control restrictions).
* [x] Implement Enemy AI Behavior (Expansion and attack logic).
* [x] Audio implementation.

## 🎯 High-Level Overview

This is a 2D strategy game (Jelly Go clone) where players manage "Blobs" (bases) that generate points. The goal is to conquer all enemy and neutral blobs by launching point-carrying projectiles.

### 🦴 Biological Obstacle System (The "Macro-Tile" Workflow)
To avoid grid-locked visuals while maintaining Tilemap functionality:
* **Asset Creation:** Organic chunks (Bone Rooms, Fleshy Walls, Entrances) are designed in Inkscape and imported as **SVGs** (Vector Graphics package).
* **Prefab Architecture:** SVG sprites are wrapped in Prefabs with **Polygon Collider 2D** for precise boundary detection.
* **Layout:** Prefabs are painted using the **Prefab Brush** on a dedicated "Obstacles" Tilemap. This allows for rapid level design with organic, non-grid aesthetics.
* **Navigation:** `NavMeshPlus` is configured to use **Physics Colliders** for baking. This ensures the NavMesh follows the organic SVG paths perfectly, allowing Blobs and Projectiles to navigate through "set entrances" and around fleshy curves.

## 🛠️ Tech Stack & Constraints

* **Engine:** Unity 6 (2D) with **URP 2D Renderer** (`Renderer2D.asset`).
* **Input:** Mouse/Touch (handled via `OnMouseDown`).
* **Navigation:** `NavMeshPlus` package (2D NavMesh).
* **UI:** `TextMeshPro` for in-world point displays (not Overlay UI).
* **Architecture:** Singleton Pattern for `GameManager`; ScriptableObjects for Data.
* **Lighting:** URP 2D Lighting pipeline (Global + per-object Freeform lights). See **2D Lighting System** below.
* **Shaders:** Custom HLSL Fragment Shaders for Blobs, Backgrounds, and Effects.
    * **Liquid Logic:** Uses a `_FillLevel` property (0-1) to clip a wave-animated color.
    * **Vein Logic (Advanced):**
        * **Anisotropic Voronoi:** Uses a stretch-safe grid algorithm to elongate cells, reducing junction frequency for a vessel-like look.
        * **Domain Warping:** Applies low-frequency sine-wave distortion to world coordinates to create organic, curvy paths.
        * **Exponential smin:** Uses `log-sum-exp` math for junctions to provide smooth biological filleting without the bloating or thinning artifacts of polynomial approximations.
    * **Falling Drops:** Uses an SDF-based circle approach.
        * *Pseudo-Randomness:* Horizontal position is seeded by `floor(_Time.y * Speed)`, ensuring the drop doesn't teleport during a single fall.
        * *Collision:* Drops are multiplied by a `step` function against `_FillLevel` to "vanish" upon hitting the liquid surface.
        * *Performance:* Multiple drops (3) are handled via time offsets in a helper function to avoid branching/loops.
    * **Cells Background:** A simpler Voronoi-based alternative to the Vein shader. Uses animated hash-based Voronoi with dual-layer veins (large + small) for an organic pulsing background pattern. Uses world-space UVs to prevent tiling.

### 2D Lighting System

The project uses **URP 2D Lighting** via `Renderer2D.asset` with 4 blend styles configured (Multiply, Additive, Multiply+Mask, Additive+Mask). Light render textures are rendered at **0.5x scale** for performance, supporting up to **16 simultaneous lights**.

* **Global Light:** `Level.prefab` contains a Global Light2D (intensity 0.8) providing scene-wide ambient illumination.
* **Per-Object Lights:**
    * `Blob.prefab` has a Freeform Light2D (intensity 1.0, with a cookie sprite and shadows enabled).
    * `Projectile.prefab` has a smaller Freeform Light2D (intensity 1.0, inner/outer radius 0.3/0.6) for a trailing glow.
    * **Obstacles** receive procedural Freeform Light2D via `Obstacle.cs`'s `[ContextMenu("Add Light")]`. `GenerateShapePath()` derives the light shape from the obstacle's `EdgeCollider2D` or `SpriteShapeController` spline, creating a thin tube of light that follows the organic contour.
* **Lit Shaders:** `BlobLiquidLit`, `FrozenBlobLit`, and `InfectedBlobLit` are URP 2D-aware variants. They implement 3 passes (`Universal2D`, `NormalsRendering`, `UniversalForward`). The primary `Universal2D` pass computes screen-space `lightingUV`, unpacks a `_NormalMap` (slot available, currently unassigned), packages `SurfaceData2D` / `InputData2D`, and outputs via `CombinedShapeLightShared()`.
* **Unlit Shaders:** `BeamGlow` (additive glow, `Blend One One`) and `BubblePulse` (legacy CG, defense pulse) are intentionally unlit self-illuminating overlays.

## 📂 Project Structure & Invisible Logic

* **Global Access:** `GameManager.cs` is the central brain. Use `GameManager.Instance` to access the current selection state.
* **Atmospheric Background System (The Layer Sandwich):**
    * **Tier 1: Deep Base (Z=10):** Static textured background (Dark Red SVG).
    * **Tier 2: Vein Layer (Z=5):** Procedural Quad using `VeinBackground.shader`.
        * **Movement:** Uses `MouseParallax.cs` to drift opposite to mouse movement (creates depth in fixed camera).
        * **Shader:** Uses World Space coordinates to prevent tiling. `_Blur` property simulates depth of field.
    * **Tier 3: Gameplay Floor (Z=0):** Semi-transparent Tilemap (Walkable) that allows the veins to "glow" through.
* **Team Identity:** Teams are managed via the `Team` Enum and the `TeamColorSettings` ScriptableObject. **Crucial:** Visuals are updated via `ApplyTeamStyle()`. If you change a team in code, you must call this method.
* **Navigation Logic:** Projectiles use a custom path-following script. It calculates a `NavMeshPath` once on launch and moves the `Transform` directly via `Vector3.MoveTowards`. This allows for high-speed movement (Speed 10+) without steering drift or overshooting corners.
* **Conquest Logic:** Calculation is handled in `Blob.ReceiveProjectile`. If points < 0, `Conquer()` is called to swap team ownership and flip the math to absolute values.
    * **Upgrade System:** 
        * **Pattern:** Branching Inheritance. `Blob` is the base; specialized types (Production, Defensive) inherit from it.
        * **Architecture:** Uses a `List<UpgradeOption>` in the base `Blob` class. This allows the base prefab to have multiple branching paths (e.g., choice between Production and Defense), while child prefabs can have a single "Stat Level Up" button.
        * **Mechanism:** Prefab-swapping via `UpgradeTo(GameObject prefab)`. State (`pointCount`, `team`) is transferred during the swap.
        * **Specialized Types:**
            * `ProductionBlob`: Enhanced point generation rate.
            * `DefenseBlob`: 
                * **Dual-Layer Defense:**
                    * **Projectile Interception (Lightning Rod):** Automatically detects enemy projectiles within a configurable radius. Intercepted projectiles are forced to update their target to the Defense Blob.
                    * **Escape Mechanics:** If an intercepted projectile moves beyond a "release radius" (130% of capture radius), it reverts to its original target.
                    * **Tether Limit:** A single Defense Blob can only attract the same projectile twice. This prevents infinite loops if a projectile is "skipping" past the blob. This limit resets when the blob is upgraded.
                    * **Inherent Fortification:** Uses the `damageMultiplier` stat (defined in `LevelStats`) to reduce incoming enemy damage (e.g., 0.5 = 50% reduction).
                * **Visual Feedback:** Displays a dashed circle representing its protection radius and fires "interception beams" (LineRenderers) at active targets.
            * `AttackBlob`: Launches projectiles with higher movement speed and possesses powerful activated abilities.
                * **High-Speed Offense:** Inherits a 2.0x `attackSpeedMultiplier` for all launched projectiles.
                * **Ability System:**
                    * **Charging Mechanic:** Requires a specific point cost to start "charging" an ability. While charging, the blob cannot launch normal projectiles.
                    * **Abilities:**
                        * **Freeze:** Turns the target blob neutral and disables all its functionality for a duration.
                        * **Infect:** Applies a biological infection that drains points over time and stops production.
                        * **Capture:** Immediately swaps the target blob's ownership to the attacker's team.
                    * **Targeting:** Once an ability is charged, the next launch command at an enemy target fires the ability beam instead of a standard projectile.
                * **Visual Feedback:** Uses a horizontal charge bar UI and specialized "ability beams" (LineRenderers) to indicate active firing.
        * **UI Best Practice:** All upgrade buttons should be children of a single `upgradeUI` GameObject (the "Button Container"). This container is toggled on/off based on selection, making it easy to add or remove branching choices without changing the selection logic.

    ## 🧠 Lessons Learned & Technical Debt

    *   **Unity Button Listeners:** `RemoveAllListeners()` only clears listeners added via code. Persistent listeners added in the Unity Inspector remain active. Always use a safety flag (like `isUpgrading`) in sensitive methods like `Instantiate/Destroy` cycles to prevent double-spawning.
    *   **UI Interaction:** In world-space UI setups, clicks often "bleed" through to the physics objects behind them. Always check the `EventSystem` before processing `OnMouseDown` or `OnPointerDown`.
    *   **State Transfer:** When swapping prefabs, explicitly call visual update methods (like `UpdateText()`) immediately after instantiation. Relying on `Start()` or the next `Update()` loop can cause "blurry" or flickering text during the frame overlap.
    *   **Destroy Latency:** Remember that `Destroy()` is not immediate. Objects exist until the end of the frame. This can cause logic to run twice on the "dead" object if not guarded.

    ## 📝 Coding Standards

* **Performance:** Avoid `GetComponent` or `Find` in `Update()`. Cache references in `Start()` or `Awake()`.
* **State:** Neutral blobs (`Team.Neutral`) should never generate points, **except** `ProductionBlob`s — they generate up to `maxCapacity` like any other team.
* **Modification:** When suggesting script changes, ensure you maintain the `OnValidate` blocks used for Editor-time visual syncing.
* **Projectile Initialization:** `Projectile.Initialize` must accept a `speed` parameter to allow different Blob types to dictate movement speed.

## 🚫 Restricted Actions
* **No Unprompted Visuals:** You are strictly forbidden from adding any visual "touch-ups," polishing aesthetics, or creating a specific "look and feel" unless explicitly requested by the user. Implementation should focus on functional correctness and adherence to existing styles.

## 📋 Instructions
No additions or changes unless explicitly asked for or given permission for. Not in code, in project settings, in prefabs, or anywhere else.

## ⚠️ Known Editor Setup (Not in Code)

* **Physics:** Blobs require a `CircleCollider2D` (set to Trigger) for projectile detection.
* **Layers:** Ensure the NavMesh is baked on the "Walkable" tilemap layer.
* **Prefabs:** The `projectilePrefab` must be assigned in the `Blob` inspector for the `LaunchProjectile` method to function.
* **Lighting:** Obstacle prefabs need a `SpriteShapeController` (or `EdgeCollider2D`) for `Obstacle.cs`'s `[ContextMenu("Add Light")]` to generate a matching Freeform Light2D shape.

---

