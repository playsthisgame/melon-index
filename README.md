# melon-index

Skill packs and skills for [melon](https://github.com/playsthisgame/melon), the dependency manager for agent skills.

The searchable skill catalog lives in melon's [`index.yaml`](https://github.com/playsthisgame/melon/blob/main/index.yaml). This repo holds the **packs** (curated groups of skills that install together) and a few skills maintained here.

## Packs

A pack is a `melon.yaml` with `type: pack`. Its dependencies are the skills it installs. Packs need melon v0.5.0 or later.

```sh
melon info github.com/playsthisgame/melon-index/packs/spring-expert   # see what's inside
melon add  github.com/playsthisgame/melon-index/packs/spring-expert   # install it
```

| Pack | What it's for | Skills | Kind |
|---|---|---|---|
| [`agent-workflow`](packs/agent-workflow/melon.yaml) | A disciplined process for coding agents: brainstorm, plan, test-first, debug systematically, and verify before finishing (obra/superpowers) | 7 | pure |
| [`cloud-infra`](packs/cloud-infra/melon.yaml) | Infrastructure as code and containers: Terraform (official HashiCorp), Kubernetes, and Docker | 7 | pure |
| [`cloudflare-workers`](packs/cloudflare-workers/melon.yaml) | Build on Cloudflare: product selection, Workers best practices, Wrangler, Durable Objects, and the Agents SDK (official cloudflare/skills) | 5 | pure |
| [`dotnet-expert`](packs/dotnet-expert/melon.yaml) | C# and .NET: ASP.NET Core Web APIs, EF Core, performance, async, and testing (official dotnet/skills) | 7 | pure |
| [`expo-mobile`](packs/expo-mobile/melon.yaml) | React Native with Expo: routing, native UI, SDK upgrades, and EAS workflows (official expo/skills) | 6 | pure |
| [`go-expert`](packs/go-expert/melon.yaml) | Idiomatic Go: concurrency, patterns, and table-driven testing | 3 | pure |
| [`llm-apps`](packs/llm-apps/melon.yaml) | Build LLM applications: the Claude API, MCP servers, LangChain and LangGraph agents, and RAG | 6 | pure |
| [`nextjs-fullstack`](packs/nextjs-fullstack/melon.yaml) | Full-stack Next.js: everything in react-frontend plus App Router, Supabase, shadcn/ui, and Vercel deploys | 5 | pure |
| [`office-docs`](packs/office-docs/melon.yaml) | Create and edit Word, PDF, PowerPoint, and Excel files (anthropics/skills) | 4 | pure |
| [`pr-workflow`](packs/pr-workflow/melon.yaml) | Ship changes cleanly: find bugs, review code, write commits and PR descriptions, fix CI, and address review comments | 8 | pure |
| [`python-backend`](packs/python-backend/melon.yaml) | Python services: modern tooling (uv, ruff), type safety, async, FastAPI, Django, and pytest patterns | 7 | pure |
| [`react-frontend`](packs/react-frontend/melon.yaml) | Modern React frontends: Vercel's React guidance, composition patterns, design quality, TypeScript, and UI testing | 7 | pure |
| [`rust-expert`](packs/rust-expert/melon.yaml) | Rust: ownership-aware design, async patterns, testing, and common anti-patterns | 5 | pure |
| [`security-audit`](packs/security-audit/melon.yaml) | Security review toolkit: Semgrep and CodeQL scans, diff review, supply-chain risk, footguns, and threat modeling | 7 | pure |
| [`spring-expert`](packs/spring-expert/melon.yaml) | Build Spring Boot applications: project setup, idiomatic Spring, JUnit testing, Javadoc, API design, and PostgreSQL | 8 | hybrid |

- **Pure** packs only install their members.
- **Hybrid** packs also install a skill of their own. `spring-expert` adds an entry-point skill that tells the agent which member to use for each task.
- `nextjs-fullstack` includes the whole `react-frontend` pack, so packs can contain other packs.

Removing a pack removes its members too, unless something else still needs them:

```sh
melon remove github.com/playsthisgame/melon-index/packs/spring-expert
```

## Skills

| Skill | What it does |
|---|---|
| [`melon`](skills/melon/SKILL.md) | Manage agent skill dependencies with melon |
| [`video-to-gif`](skills/video-to-gif/SKILL.md) | Create high quality GIFs from videos |

```sh
melon add github.com/playsthisgame/melon-index/skills/video-to-gif
```

## Contributing a pack

1. Create `packs/<name>/melon.yaml`:

   ```yaml
   name: my-pack
   version: 1.1.0
   description: "One sentence on what the pack is for"
   type: pack
   tags: [example]
   dependencies:
     github.com/owner/repo/path/to/skill: "^1.2.0"   # a range if the repo has semver tags
     github.com/owner/other-repo/skills/thing: "main" # a branch if it doesn't
   ```

2. Use ranges (`^1.2.0`), not exact versions. Exact pins conflict with anyone who also depends on that skill directly.
3. Only use a semver range when the skill exists at that tag. Some repos tag rarely, and their newest skills only exist on `main`.
4. To make it a hybrid pack, add a `SKILL.md` to the pack directory and `entrypoint: SKILL.md` to its `melon.yaml`.
5. Check that it installs: `melon add github.com/<your-fork>/melon-index/packs/<name>@<your-branch>`.

## Versioning

Tags are repo-wide, so every pack and skill here shares one version. Each release is tagged `vX.Y.Z`. Adding a pack or a member is a minor bump. Removing a member, or anything that would break an existing install, is a major bump.
