# Building a Unity Game with Claude Code and the Unity CLI: Phases 1–4

## What you need

- Unity 6.0 LTS or newer.
- The Unity CLI (installed in Phase 1).
- Git.
- Claude Code desktop app.
- A rough idea of the game you want to make.

**What remains your job:** deciding what the game is, playtesting for feel and fun, adding real art and audio, and approving each milestone before it is committed.

**Caution:** The CLI and the Pipeline package are new (released mid-2026), and the Pipeline package is labelled experimental. Command names and flags may change. When something here disagrees with `unity --help` or the [official documentation](https://docs.unity.com/en-us/unity-cli), trust those.

## Phase 1: Setup

All of setup can happen from a terminal. You do it once per project, and it takes about 15 minutes plus download time.

1. **Install the Unity CLI.** On macOS or Linux run `curl -fsSL https://unity.com/install.sh | bash`, or use `brew install --cask unity-cli`. On Windows use `winget install Unity.CLI`. Reopen the terminal and check with `unity --version`.
2. **Sign in.** Run `unity auth login`. It opens a browser sign-in.
   - [ ] If it fails with an error, sometimes it helps to close the console window, re-open it, and try again.
3. **Install an editor.** In this course (year 5787, winter) we will use `6000.3.25f1`. It's easiest to install from command line: `unity install 6000.3.25f1 -m webgl android --cm --accept-eula --yes`. It installs the required Unity 6 version, as well as the WebGL and Android modules. It may take several hours, so keep the terminal open and go do something else.
4. **Set the editor as default.** `unity editors default 6000.3.25f1`
5. **Create the project.** Open the Unity Hub. Find the newly-installed editor so that you see it in the Hub too. Then go to `Projects -> New Project`, and select the right editor version. Select a template: either `Universal 2D` or `3D URP`. Choose a name and a folder for the project. Check "Use Unity CLI". No need to check "Use AI Assistant", as we will use Claude Code. Finally, click `Create project`.
   - [ ] After creating the project, from within Unity, go to folder "Assets" and remove the subfolder "TutorialInfo" - it is not needed for your new project.
   - [ ] Note you can also create a new project directly from the CLI, using `unity projects create`. See `unity projects create --help` for more details.
6. **Verify**:
   - [ ] Inside the project folder, run `unity pipeline list`. You should see **Pipeline: Installed**. If it is not, then ensure you checked "Use Unity CLI".
   - [ ] To open the project in the editor, you can go into the project folder and write `unity open .`.
   - [ ] Check that Serialization Mode is "ForceText": `unity command eval "return UnityEditor.EditorSettings.serializationMode.ToString();"`
   - [ ] Check that Version Control Mode is "Visible Meta Files": `unity command eval "return UnityEditor.VersionControlSettings.mode;"`
   - [ ] Check that the project recompiles: `unity recompile`.
   - [ ] To close the editor, you can write `unity close .`.
7. **Set up version control.** Create a new repository in the GitHub account of your organization. Choose the Unity `.gitignore` file. Clone it to a temporary folder. Then, copy the `.git` folder and the `.gitignore` file into your Unity project.
8. **Check before committing.** Run:

   ```
   git status
   ```

   The list should include only three folders: `Assets/`, `Packages/` and `ProjectSettings/` (plus maybe some files such as `README.md` and `.gitignore`). In particular, it should **not** include `Library/`, `Temp/`, `Logs/` or `UserSettings/`. If those show up, the `.gitignore` isn't working. Don't commit until it's fixed: `Library` alone can add gigabytes to your GitHub account.

9. **If the status is OK, commit and push:**

   ```
   git add .
   git commit -m "Initial Unity project"
   git push -u origin main
   ```

10. **Run a health check:**

    ```
    unity vcs doctor
    ```

    It checks the ignore rules and Git LFS configuration a Unity project needs. You can also automatically fix problems with `unity vcs doctor --fix`.
11. **Install the Unity skill for Claude Code.** Run `unity skill install <agent>` (see `unity skill install --help` for the exact agent name). This teaches Claude how to use the CLI, so you don't have to explain every command yourself. You don't need the MCP server: Unity recommends direct `unity command` calls for agents that can run shell commands, because they are faster and use fewer tokens.

## Phase 2: Project instructions (CLAUDE.md)

`CLAUDE.md` is where you tell Claude, once, how to work in this project. Claude Code reads it at the start of every session. With the CLI available, its most important job is to make verification mandatory: no task is done until Claude has recompiled, checked errors and, where it makes sense, checked behavior in the editor.

Open Claude Code in the project folder, run `/init` to generate a starting file, then add rules like the ones below. Edit the bracketed parts.

```markdown
# Project: [Game name]

## Stack
- Unity 6000.3.25f1, URP ([2D / 3D]), new Input System
- Player input uses the actions in Assets/InputSystem_Actions.inputactions. Add new actions there rather than reading keys directly.
- C# scripts live in Assets/Scripts/, one folder per system
- Game spec: 2-design-detailed.md (1-design-raw.md is my original draft). Milestones: 3-plan.md

## Working with the editor
- The editor is usually open with the Pipeline package connected. Check with `unity status`.
- Use the Unity CLI for anything that touches the editor. Never hand-edit .unity, .prefab or .asset YAML.
- Create and change scenes, GameObjects, components and prefabs with `unity command eval` or `eval_file`.
- For setup you will repeat, write a static method with [CliCommand] in Assets/Editor/ and call it with `unity command <name>`.
- Save scenes after changing them (EditorSceneManager.SaveOpenScenes).

## Definition of done for every change
1. `unity recompile --strict` passes with no errors or warnings.
2. `unity test` passes (exit code 0). Add or update tests for new game logic.
3. For gameplay changes: enter Play mode via eval, check the state you expect, report it, and exit Play mode.
4. Summarize what changed and what I should playtest by hand.

## Rules
- Work on one milestone from 3-plan.md at a time. Don't start the next one unasked.
- Don't commit. I commit after playtesting.
- If the editor isn't connected, say so and stop. Don't fall back to editing scene files.
- Use placeholder primitives for art (cubes, capsules, solid colours).
```

Two of these rules matter most. "Never hand-edit scene YAML" removes the biggest source of broken Unity projects. "Definition of done" turns the CLI from something Claude could use into something it always uses.

If Claude Code asks permission each time it runs a `unity` command, allow `unity` commands for this project so the loop runs without interruptions.

## Phase 3: Design doc and milestone plan

You write what the game is; Claude turns it into a plan; you approve the plan before any code is written. The CLI adds one new thing to plan for: each milestone gets a check Claude can run itself.

**1-design-raw.md (you write this).** Describe the game, not the code. Cover:

- The main player experience.
- Technical details: genre, camera.
- Formal elements: players, objectives, procedures, rules, resources, conflicts, boundaries, outcomes.
- Objects in the world: enemies, pickups, hazards, and what each does.
- List of levels.
- What's out of scope for now (multiplayer, saves, menus beyond the basics).

**2-design-detailed.md (Claude drafts, you edit).** Ask Claude to read your design document, check and extend it. Example prompt: "*Read 1-design-raw.md design document. Check it for consistency and missing details. Fix inconsistency and add details. Call the new document 2-design-detailed.md.*" Read the outcome carefully, and see that it fits your planned player experience. If not, then fix it (possibly in conversation with Claude).

**3-plan.md (Claude drafts, you edit).** Ask Claude to prepare a plan made of milestones. Example prompt: "Read 2-design-detailed.md and propose a milestone plan in 3-plan.md. Don't write code yet. For each milestone, include how you will verify it with the Unity CLI and what I should playtest." Then review it. Order should run from a playable core outward: movement first, then the world, then rules, then polish.

A good milestone entry looks like this:

```markdown
## M2: Enemies patrol and damage the player  [ ]
Build: Enemy prefab (capsule placeholder), patrol between two points,
contact deals 1 damage, player has 3 HP and respawns at 0.
CLI checks:
- recompile --strict and unity test pass
- edit-mode tests for the health logic (damage, death, respawn)
- eval in Play mode: after placing the player on an enemy, HP drops by 1
Playtest: does patrol speed feel fair? is the hit readable?
```

Keep the next one or two milestones this detailed and the rest as one line each. Playtesting will change your mind about later milestones, so detail written now is mostly rewritten later.

Commit & push the new markdown files before implementing the first milestone.

## Phase 4: The milestone loop

Each milestone follows the same loop: Claude builds and verifies until its own checks pass, then you playtest, then you commit. The difference from a CLI-less workflow is the inner loop. Claude fixes compile errors, failing tests and wrong runtime state on its own before you ever see the result.

1. **Start clean.** Open a fresh Claude Code session (or `/clear`) with the Unity editor open. This is to remove clutter from previous milestones.
2. **Say** "Read 2-design-detailed.md and 3-plan.md. Implement milestone Mnnn." (where nnn is the milestone number).
3. **Claude writes the code.** Gameplay scripts go in `Assets/Scripts/`, editor-only setup tools in `Assets/Editor/`.
4. **Claude builds the scene through the editor.** Instead of editing scene files, it creates GameObjects, components and prefabs with `eval`, or writes a reusable setup command.
5. **Claude verifies.** `unity recompile --strict` for compile errors and warnings. `unity test` for logic (exit code 8 means tests ran and failed). Then a Play-mode check through `eval` for behavior. It loops on its own until all three pass.
6. **Claude reports** what it built, what it verified, and what you should try by hand.
7. **You playtest.** Press Play in the editor. Judge feel, readability, fun.
8. **Fix.** If you give feedback, then Claude goes back to step 2 to fix. 
8. **Accept.** Once you're happy with the changes, tell it to Claude, e.g. using the following prompt: "Mnnn is accepted. Update 3-plan.md: 1. Tick Mnnn as done. 2. Under it, add a short "Notes" line: decisions we made, deviations from the plan, and anything deferred. 3. If what we learned affects upcoming milestones, propose changes to them, but don't apply them yet." Once you are happy with the changes, commit and push, and move on to the next milestone.
