---
name: melon
description: >
  Manage skills and agent dependencies using melon — a dependency manager for
  agentic markdown (skills, agents, and prompts). Use this skill whenever the
  user wants to install, add, remove, update, or search for skills with melon;
  when they ask about melon.yaml, melon.lock, or the .melon/ cache; when they
  want to set up a new project with melon init; when they ask how to share skills
  across their team or CI; or when they encounter any melon command or error.
  Also triggers when the user wants to manage AI tool skill dependencies across
  claude-code, cursor, windsurf, roo, or other supported agents.
compatibility:
  tools:
    - Bash
  dependencies:
    - melon
---

# melon

Melon is a dependency manager for agent skills. It resolves skill packages from
GitHub, caches them in `.melon/`, and symlinks them into the expected directory
for each AI tool (e.g. `.claude/skills/` for Claude Code). Works like npm or
Go modules but for markdown-based agent context.

## Installation

```bash
# npm (no Node.js runtime required after install — pure Go binary)
npm install -g @playsthisgame/melon

# or Go
go install github.com/playsthisgame/melon/cmd/melon@latest
```

Requires Git on `PATH`.

## Quick start

```bash
melon init          # scaffold melon.yaml + .melon/ cache directory
melon add github.com/playsthisgame/melon-index/skills/video-to-gif
melon install       # fetch, lock, and symlink skills into tool directories
```

## melon.yaml — the manifest

```yaml
name: my-agent
version: 0.1.0
description: "My coding agent"

dependencies:
  github.com/owner/repo/path/to/skill: "^1.2.0"   # semver constraint
  github.com/owner/repo/path/to/skill: "main"      # branch pin

tool_compat:           # where melon places skills
  - claude-code        # → .claude/skills/
  - cursor             # → .agents/skills/
  - windsurf           # → .windsurf/skills/
  - roo                # → .roo/skills/

# vendor: false        # opt out of committing .melon/ and symlinks to git
```

**Dependency name formats:**
- Full repo: `github.com/owner/repo`
- Monorepo subdirectory: `github.com/owner/repo/path/to/skill`
- GitHub web URL: `github.com/owner/repo/tree/main/path/to/skill` (the `tree/<branch>/` is stripped automatically)

**Version formats:**
- Semver constraint: `^1.2.0`, `~2.0.0`, `1.0.0`
- Branch pin: `"main"`

## Commands

### `melon init`

Scaffold a new manifest. Prompts for package name, type, and AI tools in use.

```bash
melon init
melon init --yes        # accept all defaults (scripting)
melon init --dir ./app  # initialize in a specific directory
```

### `melon install`

Resolve the dependency graph, fetch into `.melon/`, write `melon.lock`, symlink
into tool directories.

```bash
melon install
melon install --frozen    # fail if lock would change — use in CI
melon install --no-place  # fetch and lock only, skip symlinking
```

### `melon add`

Add a dependency to `melon.yaml` and install it. Resolves the latest semver tag
automatically if no version is given.

```bash
melon add github.com/owner/repo/path/to/skill           # latest tag → ^x.y.z
melon add github.com/owner/repo/path/to/skill@^1.0.0    # explicit constraint
melon add github.com/owner/repo/path/to/skill@main      # branch pin
```

### `melon remove`

Remove a dependency from `melon.yaml`, unlink its symlinks, and delete its
`.melon/` cache entry.

```bash
melon remove github.com/owner/repo/path/to/skill
```

### `melon update`

Update deps to the latest version within their existing constraints. Interactive
multi-select in a TTY; pass a name directly to target a specific dep.

```bash
melon update                                      # interactive picker
melon update github.com/owner/repo/path/to/skill  # target one dep
```

If a newer major version exists outside the current constraint, melon prints a
hint showing how to upgrade with `melon add`.

### `melon search`

Search the melon-index by keyword. In a terminal, results appear as an
interactive list — navigate with ↑↓, press Enter to select, and melon offers to
run `melon add` for you.

```bash
melon search git workflow
melon search spec planning
```

### `melon list`

Show installed skills and audit installation state.

```bash
melon list               # list all installed skills
melon list --pending     # show deps in melon.yaml not yet installed
melon list --check       # verify symlinks exist in all tool directories
```

### `melon outdated`

Show which deps have newer versions available. Exits with code 1 if any dep is
outdated — useful in CI checks.

```bash
melon outdated
```

### `melon info`

Show metadata (description, author, available versions) for a skill before
installing it.

```bash
melon info github.com/owner/repo/path/to/skill
```

### `melon clean`

Remove `.melon/` cache entries no longer referenced by `melon.lock`, and remove
their symlinks. Reclaims disk space after removing or upgrading deps.

```bash
melon clean
```

## Vendoring vs. gitignore management

**Default (`vendor: true`):** `.melon/` and generated symlinks are committed to
git. Teammates and CI get skills without running `melon install`.

**`vendor: false`:** Melon auto-manages a `.gitignore` block for `.melon/` and
all managed symlink paths, keeping it in sync as deps change. CI fetches fresh
on each run using the pinned versions in `melon.lock`.

## Lock file

`melon.lock` is generated by `melon install` and should always be committed.
It pins the exact version, git tag, repo URL, subdirectory, and a SHA-256 tree
hash for each dep — guaranteeing reproducible installs.

## Supported AI tools

| Tool | Skills directory |
|---|---|
| `claude-code` | `.claude/skills/` |
| `cursor` | `.agents/skills/` |
| `windsurf` | `.windsurf/skills/` |
| `roo` | `.roo/skills/` |
| `codex` | `.agents/skills/` |
| `opencode` | `.agents/skills/` |
| `gemini-cli` | `.agents/skills/` |
| `github-copilot` | `.agents/skills/` |
| `cline` | `.agents/skills/` |
| `amp` | `.agents/skills/` |

## CI pattern

```bash
# In CI — fail if melon.lock would change
melon install --frozen
```

## Common workflows

**New project setup:**
```bash
melon init
# edit melon.yaml to add deps
melon install
```

**Find and add a skill:**
```bash
melon search <keyword>     # find skills interactively
melon info <skill-path>    # inspect before installing
melon add <skill-path>     # add and install
```

**Keep skills up to date:**
```bash
melon outdated             # see what can be updated
melon update               # update interactively
```

**After cloning a repo that uses melon:**
```bash
melon install              # restores all skills from melon.lock
```
