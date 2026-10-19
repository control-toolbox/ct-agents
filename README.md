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
> and the recipes. Roles and workflows come later; the release recipe is written so that
> its phases can become one.

## Two modes, one base

An agent runs in one of two modes, and every role must work in both:

| | Local | Remote |
| --- | --- | --- |
| Where | a session on a developer's machine (Claude Code, …) | a CI runner (GitHub Actions + an agent engine) |
| Who drives | the developer, turn by turn | an orchestration script, from an issue or a queue |
| Approval | asked in the session before any commit or push | given by human review of a draft PR |
| Trace | the conversation, plus the plan | issue, branch, run log, draft PR, summary comment |

What changes between the modes is **who approves and where the trace lives**, not
what the agent does: [`base.md`](base.md), the roles and the recipes are shared, and the
mode-specific parts are confined to [`adapters/`](adapters/) and to the orchestration.
A role written for one mode only is a defect.

Today only the local mode is described. The remote mode — orchestration, queue, run
log — comes with the first roles; `ocourses/agents` is its reference implementation.

## Contents

| File | Content |
| --- | --- |
| [`base.md`](base.md) | Behavior shared by every agent, whatever the harness: reading the conventions, git, editing, asking vs proceeding |
| [`adapters/claude-code.md`](adapters/claude-code.md) | Claude Code specifics: background test runs with `Monitor` |
| [`setup/github-pat.md`](setup/github-pat.md) | GitHub MCP server and fine-grained personal access tokens |
| [`recipes/harmonize.md`](recipes/harmonize.md) | Harmonizing the repositories of the ecosystem: shared protocol (issues, worktree, PR narration), the [issue description](recipes/harmonize-issue.md), and the themes — [`pin-conventions.md`](recipes/pin-conventions.md), [`labels.md`](recipes/labels.md), [`issue-templates.md`](recipes/issue-templates.md), [`lint-and-automation.md`](recipes/lint-and-automation.md) |
| [`recipes/vitepress-migration.md`](recipes/vitepress-migration.md) | Migrating a package's docs from Documenter to DocumenterVitepress |
| [`guides/release.md`](guides/release.md) | **For a developer**: how to release a package with an agent — what to say, the two modes, what happens, what to do when it stops, and the commands by hand |
| [`design/release.md`](design/release.md) | Why the release is built this way (state from the facts, rules as scripts, no auto-merge, breakage as evidence) and its known limits |
| [`recipes/release.md`](recipes/release.md) | Releasing a package, stable or beta, on top of CTActions' release tool (`status`, `gate`, `ready`, `merge`, `register`): proposal, tracking issue, release PR, checked merge, tag, registration; `supervised` and `autonomous` modes. Texts: [`release-issue.md`](recipes/release-issue.md) |

## Layout

```text
base.md        shared behavior, provider-independent
adapters/      one file per harness (Claude Code, …): tools and habits specific to it
setup/         one-time machine or account setup
recipes/       step-by-step procedures for recurring tasks, future roles
guides/        for a developer: how to ask an agent for a task, and what to expect
design/        why a task is built the way it is, and its limits
```

A **harness adapter** holds only what depends on the tool running the agent. Anything
that holds for every agent goes in `base.md`; anything that says what correct code is
goes in `ct-conventions`.
