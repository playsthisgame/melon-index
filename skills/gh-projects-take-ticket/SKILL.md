---
name: gh-projects-take-ticket
description: Take the next Ready ticket from the current repository's GitHub Projects (v2) board (not Jira, Linear or other trackers), implement it in an isolated git worktree, run the project's CI checks, and open a pull request. Use when the user asks to take, pick up, grab, or work the next ticket, issue, or board item, or names a specific issue number to work on from the board.
---

# Take ticket (GitHub Projects)

Pick up one ticket from the repository's GitHub Projects (v2) board, implement
it in an isolated git worktree, and open a pull request. Move the board item
through Ready → In progress → In review along the way.

If the user named an issue number, work that issue. Otherwise pick from the
Ready column.

## GitHub access

Use whichever is available, in this order:
1. A GitHub MCP server (tools for projects, issues and pull requests).
2. The `gh` CLI (`gh project item-list`, `gh project item-edit`,
   `gh issue view`, `gh issue comment`, `gh pr create`).

If neither is available or authenticated, stop and tell the user.

## 0. Load the board config

Look for a "GitHub project board" section in the repository's agent
instructions file: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`,
`.github/copilot-instructions.md`, or whichever one this agent reads. It should
look like this:

```markdown
## GitHub project board

Project: <name>, user project #<number> (private), owner: <owner>

Status field: <field-id>

Option IDs:
- Backlog:     <id>
- Ready:       <id>
- In progress: <id>
- In review:   <id>
- Done:        <id>

Priority field: <field-id> (P0 <id>, P1 <id>, P2 <id>)   # optional

Worktree dir: .worktrees   # optional
```

Use these IDs directly. Do not re-query the field list.

If no such section exists, query the project's fields once. Then stop and give
the user a filled-in block to paste into their instructions file, so later runs
don't have to query. Do not continue without it. If the repository is linked
to more than one project, ask which one.

Option names are case-sensitive ("In progress", "In review").

## 1. Pick the ticket

If the user named an issue, find its board item. Otherwise:
- List project items with Status = Ready. If there are none, stop and say so.
  Do not fall back to Backlog.
- Pick the oldest. If Priority is set, prefer P0 > P1 > P2.

## 2. Claim it

Set Status to In progress before touching any code.

## 3. Read and sanity-check

Read the issue body and comments in full. If the acceptance criteria are
ambiguous, set Status back to Ready, comment on the issue explaining exactly
what's underspecified, and stop. Do not guess at intent.

## 4. Create an isolated worktree

Do NOT modify the primary checkout. Never run `git checkout` or `git switch`
in it.

```sh
git fetch origin
DEFAULT=$(git symbolic-ref --short refs/remotes/origin/HEAD | sed 's#^origin/##')
WT_DIR=<worktree dir from config, default .worktrees>
git worktree add "$WT_DIR/<issue-number>" -b <type>/<issue-number>-<slug> "origin/$DEFAULT"
```

- `<type>` is feat, fix, chore, docs or refactor, based on the issue.
- `<slug>` is a short kebab-case summary of the issue title.
- If `$WT_DIR` isn't already gitignored (`git check-ignore -q "$WT_DIR/x"`),
  add it to `.git/info/exclude`. That file is local, so nothing is committed.

Do all remaining work inside the worktree.

## 5. Set up the worktree

Gitignored files (dependencies, build output, `.env`, generated code) are not
carried over. Run the project's install, setup and build steps. Take them from
the agent instructions file, or failing that from the CI workflows in
`.github/workflows/`, or the Makefile, `package.json`, justfile or README.

If setup needs secrets or files you don't have (such as `.env`), say what's
missing instead of inventing values.

## 6. Implement and verify

Before writing code, read any specs, design docs or contribution guidelines
the instructions file points to for the area you're changing. Then implement.

Run the same checks CI runs. Take them from `.github/workflows/` if present,
otherwise from the instructions file (formatting, lint and the full test
suite). If any fail and you can't fix them, stop and report. Do not open a PR
with failing checks.

## 7. Open the PR

Commit with a clear message. Push with `-u`. Open a PR against the default
branch with `Closes #<issue-number>` in the body, plus a short summary of the
change and how it was verified.

## 8. Hand off

Set Status to In review. Report the PR URL and the worktree path.

## Rules

- Never commit or push to the default branch.
- Never close the issue. Never set Done.
- Do not remove the worktree. Leave it for review.
- One ticket per run.
