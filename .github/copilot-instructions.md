<!-- Auto-generated guidance for AI coding agents working on the Freecam repo. -->
# Copilot / AI Agent Instructions — Freecam

Short, actionable notes to help an AI agent be productive in this repo.

1) Big picture (architecture)
- Core logic: `common/` — contains the platform-agnostic Java code (mod logic, config, utilities). Example: `common/src/main/java/net/xolt/freecam/Freecam.java` is the central runtime entry used by all platforms.
- Platform adaptors: `fabric/` and `neoforge/` — small platform-specific glue code and resources.
- Build variants: `variant/` (contains `api`, `normal`, `modrinth`) — used to produce alternate builds (e.g., Modrinth edition). Use `BuildVariant.getInstance()` in code to query variant behaviour.
- Resources & runtime hooks: mixins and access wideners live under `common/src/main/resources` (e.g. `freecam-common.mixins.json`, `freecam.accesswidener`). The Gradle Loom configuration wires these into the runtime.

2) Key build and runtime workflow (what to run)
- Local dev run (client): from repo root in PowerShell run:
```powershell
.\gradlew :fabric:runClient
```
or run the `runClient` task for whichever platform subproject you work on (e.g. `:neoforge:runClient` if present).
- Build artifacts: produce distributable jars with:
```powershell
.\gradlew build
```
- Publish (CI/local): `publishMod` is configured in subprojects when the platform is enabled. It requires API keys (GH_TOKEN, MODRINTH_TOKEN, CURSEFORGE_TOKEN) set in environment or `gradle.properties`.
```powershell
.\gradlew publishMod
```
- Notes: Gradle wrapper uses Gradle 8.11 and Java toolchain targets Java 21 — ensure the environment supports it.

3) Project-specific conventions & patterns
- Single-source logic: Keep gameplay/logic in `common/`. Platform folders should contain minimal binding code (registering events, resources). When adding behavior, prefer adding to `common` and expose hooks for platform code.
- Mixins & access wideners: Changes to Minecraft internals must be added via Mixin classes and referenced in the mixin JSON files in `common/src/main/resources` and the platform resource mixin JSON files. Access widener is at `common/src/main/resources/freecam.accesswidener` and is referenced in `build.gradle` via Loom's `accessWidenerPath`.
- Config & bindings: Mod configuration lives under `common/src/main/java/net/xolt/freecam/config` (e.g., `ModConfig`, `ModBindings`). Keybinds are defined and ticked through `ModBindings` and referenced by `Freecam`.
- Variant behavior: Respect `variant` API types (see `variant/api`) and calls to `BuildVariant.getInstance()` in code. Feature gating (cheats permitted, mod distribution differences) is handled via the build variant.
- Localization: All language files live under `common/src/main/resources/assets/freecam/lang/*.json`. Use existing keys when adding text; translations live in Crowdin (see `crowdin.yaml`).

4) Integration points & external dependencies
- Architectury + Loom: repo uses Architectury and Loom; mappings include official Mojang mappings layered with Parchment. See root `build.gradle` for mapping configuration.
- Third-party libs: `fabric-api`, `cloth-config` (cloth-config is embedded for publish), `modmenu` (optional). Dependency versions live in `gradle.properties`.
- Publishing: `com.hypherionmc.modutils.modpublisher` is used to publish artifacts to CurseForge/Modrinth/GitHub — tokens are read from environment or `gradle.properties`.

5) Code patterns and quick examples (where to look)
- Main runtime toggles & camera code: `common/src/main/java/net/xolt/freecam/Freecam.java` — use this file to understand lifecycle (enable/disable, tripod support, player control switching).
- Free camera implementation: `common/src/main/java/net/xolt/freecam/util/FreeCamera.java` and position handling in `FreecamPosition`.
- Tripod registry: `common/src/main/java/net/xolt/freecam/tripod/TripodRegistry.java` and `TripodSlot` enum — shows how multiple camera slots are stored/used.
- Mixins list: `common/src/main/resources/freecam-common.mixins.json` and platform mixin JSONs under `fabric/src/main/resources` / `neoforge/src/main/resources`.

6) What to avoid / watchouts
- Do not put gameplay logic exclusively in platform subprojects — maintain it in `common` unless platform APIs are strictly required.
- Publishing tasks are gated by `enabled_platforms` in `gradle.properties`. Adding a platform requires updating that property or the project settings.
- Some behavior is gated by whether cheats are permitted (see `BuildVariant.getInstance().cheatsPermitted()`); tests or debug runs may use different flags depending on the variant.

7) Useful files to open when coding or reviewing
- `common/src/main/java/net/xolt/freecam/Freecam.java` — runtime flow
- `common/src/main/resources/freecam-common.mixins.json` — mixin hooks
- `gradle.properties` — versioning, enabled platforms, mappings
- `build.gradle` (root) — architectury/loom + publisher config
- `variant/` — how different build outputs are produced

If anything here is incomplete or you'd like deeper examples (e.g., how to add a mixin, or how `runClient` is configured), tell me which area and I'll expand with code snippets or step-by-step edits.

-- End of instructions
