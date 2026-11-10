## Project Overview

This repo holds **skill packs** and a few first-party **skills** for the [melon](https://github.com/playsthisgame/melon) CLI.

The searchable skill catalog is **not** here. It lives in melon's own `index.yaml` (https://github.com/playsthisgame/melon/blob/main/index.yaml). Add catalog entries there, not in this repo.

## Layout

- `packs/<name>/melon.yaml`: a pack, i.e. a manifest with `type: pack` whose `dependencies` are the skills it installs.
- `packs/<name>/SKILL.md`: optional. Its presence (plus `entrypoint: SKILL.md`) makes the pack a hybrid pack, placed as a skill in addition to its members.
- `skills/<name>/SKILL.md`: skills maintained in this repo.

## Rules for packs

- Every dependency path must be a GitHub directory containing a `SKILL.md`, in the form melon installs: `github.com/owner/repo/path/to/skill`.
- Constraint: use `^<latest semver tag>` only if the skill exists **at that tag**. Otherwise use the repo's default branch name (usually `main`). melon resolves the highest matching tag, so a path that only exists on `main` will fail with a semver constraint.
- Use ranges, never exact versions.
- No two members across all packs may share a final path segment, because melon names the symlink after it.
- A pack may depend on another pack in this repo (`github.com/playsthisgame/melon-index/packs/<name>`) with a `^<current repo version>` constraint.
- When adding or changing a pack, update the pack table in `README.md`.

## Releases

Tags are repo-wide (`vX.Y.Z`): every pack and skill shares the version. Adding packs or members is a minor bump. Removing members or skills is a major bump. Bump `version:` in each pack's `melon.yaml` to match the tag.
