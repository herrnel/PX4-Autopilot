# AGENTS.md — herrnel/PX4-Autopilot (branch: `ai-grand-prix`)

You (the AI agent) are operating inside a **fork** of PX4-Autopilot. This
file is the contract for how to make changes safely.

## Where you are

- This directory is a git submodule of the parent project
  `AI-Grand-Prix` (`herrnel/Albatross`), checked out under
  `external/PX4-Autopilot`.
- Remotes:
  - `origin`  → `https://github.com/PX4/PX4-Autopilot.git` (read-only — upstream)
  - `myfork`  → `https://github.com/herrnel/PX4-Autopilot.git` (push target)
- Working branch: **`ai-grand-prix`** — every change goes here.
  Never commit directly to `main`; `main` stays clean so we can rebase.
- This fork itself contains a nested submodule (`Tools/simulation/gz`)
  that has been re-pointed at `herrnel/PX4-gazebo-models` on its own
  `ai-grand-prix` branch. See that submodule's AGENTS.md before editing
  worlds or models.

## Core rules

1. **Never `git reset --hard` or `git clean -fd`** without first running
   `git status` AND `git stash list` AND showing the user what would be
   lost. A previous re-clone of this submodule already destroyed local
   work once; the macOS/Homebrew fixes that survived are gold.
2. **Never edit on `main`.** Always work on `ai-grand-prix` (or a topic
   branch off it). Verify with `git branch --show-current` before edits.
3. **Never push to `origin`.** It's the upstream PX4 repo; you don't
   have write access. Push only to `myfork`.
4. **Commit early, commit often.** Anything uncommitted is one careless
   command from gone. If in doubt, `git add -A && git commit -m "WIP"`
   on a branch.
5. **Never re-clone this directory.** If it looks broken, ask the user;
   don't `rm -rf` and start over.

## Normal change workflow

When the user asks you to change PX4 (e.g. tweak an airframe, fix a
build flag, add a world target):

```bash
# 0. Confirm state
git status
git branch --show-current   # must be 'ai-grand-prix'

# 1. Make the edits with the edit/write tools (NOT sed/echo >>).

# 2. Commit
git add -A
git -c user.name="Nelson Herrera" -c user.email="ndanielherrera@icloud.com" \
    commit -m "<imperative subject>

<why, not what — the diff shows what>"

# 3. Push to fork
git push          # tracks myfork/ai-grand-prix
```

Then from the **parent repo** (`AI-Grand-Prix`):

```bash
scripts/bump-submodule.sh external/PX4-Autopilot "<same commit subject>"
```

That commits the new submodule SHA pointer in the parent and reminds
you to push it.

## Syncing with upstream PX4

Periodically (or when the user asks for a feature only available
upstream):

```bash
# From the parent repo:
scripts/sync-upstream.sh external/PX4-Autopilot main
```

That fetches `origin/main`, rebases `ai-grand-prix` onto it, and
force-pushes to `myfork` with `--force-with-lease`. If conflicts arise
the script stops and tells you what to do.

## What's in the fork (current value-adds)

These are the reasons this fork exists. Don't lose them.

- **macOS/Homebrew build fixes** (commit `3bedef9f`):
  - `Makefile`: auto-select `.venv-px4/bin/python3` if present
  - `ROMFS/.../px4-rc.gzsim`: pick gz-sim 8.x via `--force-version`
  - `cmake/px4_add_common_flags.cmake`: `-Wno-double-promotion`
  - `src/modules/simulation/gz_bridge/gz_env.sh.in`: Homebrew
    `DYLD_FALLBACK_LIBRARY_PATH` + `GI_TYPELIB_PATH`
  - `src/modules/simulation/gz_plugins/CMakeLists.txt`: locate
    Homebrew `qt@5`
  - `src/modules/simulation/gz_plugins/optical_flow/CMakeLists.txt`:
    `BUILD_RPATH`/`INSTALL_RPATH` for `libOpticalFlow`
- **Nested submodule pointer** (commit `980bc405`): `Tools/simulation/gz`
  now points at `herrnel/PX4-gazebo-models@ai-grand-prix`, which adds
  the Illini warehouse SITL world.

## When in doubt

Ask the user. Prefer reading (`git log`, `git status`, `git diff`,
`git remote -v`, `git branch -vv`) over destructive action.
