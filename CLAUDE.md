## Project Overview
This repo is a curated list of skills that are meant to be used for the [melon](https://github.com/playsthisgame/melon) CLI. The skills are to then be saved in a yaml file called `index.yaml` in the root of this repo. The format of the `index.yaml` file is as follows:

```yaml
skills:
  - name: github.com/playsthisgame/skills/agentic-spec-dev
    description: "Spec-driven development workflow for AI coding agents"
    author: playsthisgame
    tags: [workflow, spec, planning]
    featured: true

  - name: github.com/anthropics/claude-code-skills/git-workflow
    description: "Opinionated git workflow — branching, commits, and PR conventions"
    author: anthropics
    tags: [git, workflow]
    featured: true
```

the tags will be keywords that are descriptive to what the skill does. The name in the `index.yaml` must be unique, so that this is a unique set of skills.

For now all of the skills will have `featured: true`

the github path used in the name must be compatible with how melon installs skills, the details can be found in the commands section of the README https://github.com/playsthisgame/melon/blob/main/README.md#commands

you should use the available skills to complete your task.