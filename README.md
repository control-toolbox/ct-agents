# ct-agents

How AI agents work on the packages of the
[control-toolbox](https://github.com/control-toolbox) ecosystem: the behavior every
agent shares, the adapters for each harness, and recipes for recurring tasks.

| Repository | Question it answers |
| --- | --- |
| [`CTAppTemplate.jl`](https://github.com/control-toolbox/CTAppTemplate.jl), [`CTActions`](https://github.com/control-toolbox/CTActions) | what is a package built from, how does its CI run? |
| [`ct-conventions`](https://github.com/control-toolbox/ct-conventions) | what is right: structure, types, errors, tests, docs? |
| `ct-lint` *(planned)* | does the code follow these rules, and with which guarantee? |
| `ct-agents` (here) | who does what, in which order? |

An agent applies the conventions; it does not restate them. Every file here **cites**
rules by identifier (`MOD3`, `TST1`) and procedures by link into `ct-conventions`,
never copies them.

> **Status.** Documentation only: the agent base, one harness adapter, one setup note
> and one recipe, moved out of the former Handbook. Roles and workflows come later.

## Contents

| File | Content |
| --- | --- |
| [`base.md`](base.md) | Behavior shared by every agent, whatever the harness: reading the conventions, git, editing, asking vs proceeding |
| [`adapters/claude-code.md`](adapters/claude-code.md) | Claude Code specifics: background test runs with `Monitor` |
| [`setup/github-pat.md`](setup/github-pat.md) | GitHub MCP server and fine-grained personal access tokens |
| [`recipes/vitepress-migration.md`](recipes/vitepress-migration.md) | Migrating a package's docs from Documenter to DocumenterVitepress |

## Layout

```text
base.md        shared behavior, provider-independent
adapters/      one file per harness (Claude Code, …): tools and habits specific to it
setup/         one-time machine or account setup
recipes/       step-by-step procedures for recurring tasks, future roles
```

A **harness adapter** holds only what depends on the tool running the agent. Anything
that holds for every agent goes in `base.md`; anything that says what correct code is
goes in `ct-conventions`.
