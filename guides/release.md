# Releasing a package with an AI agent

For a developer who wants to release a control-toolbox package, stable or beta, with an
agent doing the work and the checks. You say what you want; the agent proposes a plan,
asks two questions, and runs the release; tools, not the agent's care, decide whether
each step is allowed. The choices behind it are in [`design/release.md`](../design/release.md);
the procedure the agent follows is [`recipes/release.md`](../recipes/release.md).

## In short

1. Open your agent (Claude Code) in the clone of the package.
2. Say, in any language: *"prepare a stable release"* or *"prepare a beta release"*.
   If you do not say which, the agent asks. It never guesses.
3. Read its proposal (the version, what changes, what it will do, what you will do) and
   answer two questions: is the version right, and `supervised` or `autonomous`?
4. Follow the tracking issue it opens. In `supervised` mode you merge the release PR and
   give the go for the registration; in `autonomous` mode you do nothing unless it stops.
5. If the package has `Release.yml`, the proposal also asks **who does phases 2 to 8**: the
   agent, or the `Release` workflow of CTActions. With the workflow, the session writes the
   request (`.releases/vX.toml`: the facts, a verdict on each candidate change) and stops;
   the workflow does the rest and comments on the issue. See [Who does what](#who-does-what).

## Before you start

The packages released this way are **CTBase, CTModels, CTSolvers, CTFlows, CTDirect,
CTParser, CTLie and OptimalControl**. Check:

- **The recipe is pinned.** `AGENTS.md` names the ct-agents tag it applies ("Pinned version:
  ct-agents vX.Y.Z") and the agent reads the recipe from a git-ignored clone in
  `.ct-agents/`, as it reads the conventions from `.ct-conventions/`. Its proposal states the
  versions applied (ct-agents, ct-conventions, CTActions): check them. A package that does
  not pin ct-agents yet is read from `main`.
- **The package has the release workflows**: `ReleaseGate.yml`, `ReleaseTag.yml`, a
  `Breakage.yml` on `breakage-suite.yml`, a `Documentation.yml` that accepts
  `workflow_dispatch`, and the label callers (`Labels.yml`, or the older `pull_request`
  guards). The eight packages above have them. Without `ReleaseGate.yml`
  only `supervised` mode is offered.
- **Your machine**: `git`, Julia 1.10 or later, the GitHub CLI logged in (`gh auth status`)
  with write access to the repository, and a clone of
  [CTActions](https://github.com/control-toolbox/CTActions) next to the packages, up to
  date (the agent runs its release tool from there). Claude Code reads the package's
  `AGENTS.md`, which points to the recipe.
- **For a beta only**: the `ct-registry` registry installed in your Julia depot and SSH
  access to `control-toolbox/ct-registry`; the agent registers with `LocalRegistry`.
- **`main` is up to date and green**, with no unpushed commits. The agent checks and
  stops otherwise.
- **For `autonomous` mode**: your account can merge the release PR. If the repository
  requires a review before merging, `gh pr merge` is refused unless you may bypass it;
  use `supervised` then.

## Stable or beta

It is your choice, every time.

| | Beta | Stable |
| --- | --- | --- |
| Version | `X.Y.Z-beta`, on a fresh triple | `X.Y.Z` |
| Registered in | `ct-registry` | General |
| GitHub release | pre-release | latest release |
| Extra check | none | the package must resolve from General alone (`REL4`): a stable that needs a beta of a dependency cannot be installed by users |

The number follows SemVer as Pkg reads it: for `0.Y.Z` a breaking change bumps `Y`, a
feature or a fix bumps `Z`; from `1.0` it is major, minor, patch. A beta is never reused:
`0.4.3-beta` is followed by `0.4.4`, not `0.4.3`. The rules are `REL1`–`REL7` in
[ct-conventions](https://github.com/control-toolbox/ct-conventions/blob/main/rules/release.md);
the agent computes the number, and the gate refuses another one.

## The two modes

| | `supervised` | `autonomous` |
| --- | --- | --- |
| Merging the release PR | you | the agent, with `release.jl merge` only, after the tool checked what ran |
| Registration comment (stable) | after your go | without asking |
| `[merge approved]` on General, when betas explain a version jump | you, or the agent after your go | the agent |
| Stops | three times: after the proposal, before the merge (you merge), before the registration comment | once, after the proposal (version and mode), then only when something fails |

A first release: use `supervised`. Each mode has run once: `autonomous` on CTBase v0.30.12
(with Claude Code), `supervised` on CTModels v0.20.0 (with another agent). Between the
stops, the agent goes on by itself: opening the tracking issue, the release PR and waiting
for the checks do not need your go.

With `Release.yml` in the package, the same two modes apply to the workflow: the
approvals of the merge and of the registration are clicks on the run
([Who does what](#who-does-what)).

Whatever the mode, the agent always asks you for the kind (if you did not give it), the
version and the mode, and asks you before posting the registration comment on a
commit instead of the issue (when `main` moved after the merge).

## What happens

| # | Phase | Who, when the agent finishes | What |
| --- | --- | --- | --- |
| 0 | Preflight | agent, read only | `main` up to date and green, the state of the last release, something to release, `REL4` |
| 1 | Proposal | agent, then you | version and why, changes, plan, open PRs, the modes. **You answer.** |
| 2 | Tracking issue | agent | `[RELEASE] vX.Y.Z`, with a first comment |
| 3 | Release PR | agent | `Project.toml`, `CHANGELOG.md`, `BREAKING.md`; the release gate runs locally, then the PR gets its run labels |
| 4 | Checks | CI | tests, documentation preview, breakage report, the gate; the tool reads what actually ran |
| 5 | Merge | agent or you | `release.jl merge`, at the exact commit it checked |
| 6 | Tag and release | `ReleaseTag` workflow | the tag on the merge commit, the GitHub release with *What's Changed*, the docs of the tag |
| 7 | Registration | agent | stable: one comment, then General's AutoMerge; beta: `LocalRegistry` into ct-registry |
| 8 | Close | agent | the final comment, the issue closed, the branch deleted, then `release.jl audit` |

When the workflow finishes, phases 0 to 3 are the same and phases 4 to 8 are its:
the next section says who does what.

## Who does what

Phases 2 to 8 are mechanical, and an agent that waits for a CI of tens of minutes burns its
context for nothing. When the package has `Release.yml` (CTActions
[`release.yml`](https://github.com/control-toolbox/CTActions/blob/main/docs/workflows/release.md)),
you can hand them to a workflow. The hand-over is the **request**, `.releases/vX.toml`: the
agent writes it from the facts of the tool (`release.jl facts`), and `release.jl request`
checks it, commits it on `release/vX` and starts the workflow. The workflow acts only for a
release manager (the organization variable `CT_RELEASE_MANAGERS`, with the `admin` or
`maintain` role on the repository). Without `Release.yml`, the agent does it all, as above.

**The agent holds no power over the release once it has handed over**: the PR, the merge and
the registration are done by three jobs of the workflow, in a GitHub Environment whose
secrets hold the key of the App `ct-release`.

### `supervised`

| Phase | Agent (session) | Workflow | You |
| --- | --- | --- | --- |
| 0. Preflight | `release.jl facts`: state of `main`, versions, `REL4`, `REL7`, the candidates; open PRs | | |
| 1. Proposal | writes it | | **validate the version; choose stable or beta, the mode, who does the rest** |
| Request | reviews each candidate, writes `.releases/vX.toml` and `vX.md`, runs `release.jl request`, **stops** | | |
| 2. Tracking issue | | opens it from the request | |
| 3. Release PR | | waits for the CI of `main`, then release files, gate, PR, label `release: supervised` | **click *Review deployments***, a first time |
| 4. Checks | | waits for the CI, reads `release.jl ready`, comments ✅ or ❌ | |
| 5. Merge | | asks for an approval, then `release.jl merge` as the App | **click *Review deployments*** (or merge by hand: the workflow sees it) |
| 6. Tag and release | | `ReleaseTag` tags and releases; comments | |
| 7. Registration | | asks for an approval, then posts the registration comment once (stable, as the owner of `CT_RELEASE_REGISTRATION_TOKEN`) or registers in ct-registry (beta), follows General's PR | **click *Review deployments***; **comment `[merge approved]` on General's PR** if the version jumps over betas |
| 8. Close | | final comment, issue closed, branch deleted, `release.jl audit` | |

The comment of each stop says what to click, with the link to the run that waits.

### `autonomous`

| Phase | Agent (session) | Workflow | You |
| --- | --- | --- | --- |
| 0, 1, request | as in `supervised` | | validate the proposal (version, mode) |
| 2 to 4 | | as in `supervised` | |
| 5. Merge | | `release.jl merge`, **no approval** (environment `release-autonomous`) | |
| 6. Tag and release | | as in `supervised` | |
| 7. Registration | | registration comment, **no approval**; follows General's PR | **`[merge approved]`** if the version jumps over betas |
| 8. Close | | as in `supervised` | |

### The registration comment

The registration bot takes a comment from a **public member** of the organization, not from
a GitHub App: so the comment is posted with `CT_RELEASE_REGISTRATION_TOKEN`, the token of a
release manager (a secret of the environment of each package, set by CTActions'
`scripts/release/setup.sh`). Without it, or if the bot still refuses, the workflow stops
with the comment ready to copy, the bot's name written `@'JuliaRegistrator`: **remove the
apostrophe** and post it once. Never write the bot's name in any other comment, even quoted:
it triggers a registration wherever it appears.

### What stays with you in both modes

- Validating the proposal.
- The decisions the workflow never takes: `main` moved after the merge (the registration
  comment must go on the merge commit, `release.jl register --on-commit`); `[merge approved]`
  on General's PR; an error answer of the registration bot (never posted again by the
  workflow: you post the comment); a check that failed; `status` saying *blocked*. The
  workflow comments `⏸ waiting for you` or ❌ on the issue, with the reason and the output of
  `status`, and goes on when the facts change.

Other actors: the CI of the package produces what `ready` reads; the registration bot and
General's AutoMerge open and merge General's PR, which the workflow follows without acting on it.

## Following it

Everything is on the **tracking issue**. The agent posts one comment per phase, never
editing an earlier one, and each comment repeats the list of steps: ✅ done, ❌ the step
that stopped the release (with the reason), nothing on a step not reached. The first
comment is the proposal you validated; the last one is the list all ✅ with a link per
step.

You can also ask the tool where a release stands, from any machine:

```bash
julia --startup-file=no CTActions/scripts/release/release.jl status --repo control-toolbox/CTBase.jl [--version 0.30.12]
```

It reads GitHub and the registries, not a memory, and says the phase, the next action and
what blocks it. Once the release is done, `release.jl audit --repo … --version …` checks
that it left the trail the recipe asks for (a comment per phase, one registration mention,
the PR merged without auto-merge and with release files only, the tag, the registry, the
branch deleted); the agent runs it at the close, and you can run it on any release, whoever
did it. It cannot see who acted.

## When the agent stops

It stops, and comments on the issue, when:

- the kind was not given (it asks);
- `status` says *blocked* (for instance a version registered without a tag);
- `main` is not green, or the local `main` is ahead;
- there is nothing to release: changes to tests, CI or the changelog alone are not a
  release (`REL7`), changes to `src/`, `ext/`, `docs/` or the dependencies are;
- a stable does not resolve from General alone (`REL4`): a dependency needs its stable
  release first;
- a check of the release PR fails, or a job it needs did not run;
- the registration bot answers with an error, or does not answer;
- anything the recipe does not cover.

## When the workflow stops

It comments on the tracking issue and stops, once per reason, when `status` says *blocked*,
a check failed, or a job the release needs did not run (**❌**); and it waits (**⏸ waiting
for you**) for the decisions of the list above. Fix the cause, then the next event, the
hourly run or a run of `Release.yml` by hand goes on from the facts. To finish a
release by hand instead (a session, a human), remove the label `release: …` from the PR
first, then follow [Resuming](#resuming).

## Resuming

Nothing is remembered: the state is recomputed from GitHub and the registries. If a
session ends, a runner is down or `main` moves, start a new session in the package and say
*"resume the release of vX.Y.Z"*; the agent runs `status` and goes on from the phase it
reports. Each action checks that it is not already done, so nothing is created twice: an
issue, a PR, a tag; and the registration comment is never posted twice.

## What you may see and is not a problem

- **Red lines in the breakage report that were already there.** The report compares each
  downstream package with the run on the base commit; a package that already failed
  there is shown ℹ️ and says nothing about your release. Only a package that passed on the
  base and fails with your release matters, and then the version must be breaking.
- **While you have not merged, the tracking issue stays at phase 4, waiting**, even when
  the checks are green and `release.jl ready` says ready: it is your turn. The CI takes
  tens of minutes (the macOS jobs last the longest); the agent waits, it does not cancel
  anything.
- **The documentation preview needs its label.** If the repository has the label
  `documentation push preview`, the agent adds it to the release PR; the gate reports it
  with ℹ️ when it is missing, it does not block.
- **Missing run labels show as ℹ️ in the release gate of the PR**, not as ❌: the gate runs
  once per label added, so its first runs see some labels only. What counts is
  `release.jl ready`, which reads what actually ran and asks for every job a label gates.
- **The documentation of a tag** is built after the tag; it takes some twenty minutes.
  If it fails, the release is not undone: the agent reports it and the docs are rerun.
- **General's AutoMerge** needs some minutes (about ten in the one release run so far) and
  can take longer. A version jump needs `[merge approved]`.

## Doing it by hand

Every step has a command, and the agent uses no other:

```bash
TOOL="julia --startup-file=no CTActions/scripts/release/release.jl"
$TOOL status   --repo control-toolbox/CTBase.jl [--version X.Y.Z]
$TOOL gate     --base origin/main --labels "<run labels>" <clone of the release branch>
$TOOL ready    --repo control-toolbox/CTBase.jl --pr <N>
$TOOL merge    --repo control-toolbox/CTBase.jl --pr <N> --confirm
$TOOL register --repo control-toolbox/CTBase.jl --version X.Y.Z [--post]
```

`ready` says whether the PR may be merged from what ran on its head commit; `merge`
runs `ready` again and merges only a `release/v…` branch, at the commit it checked;
`register` posts the registration comment once and refuses to post twice. Never merge a
release PR with GitHub's auto-merge, and never write the registration bot's mention by
hand: it triggers a registration wherever it appears.

## Where to read more

| | |
| --- | --- |
| The procedure the agent follows | [`recipes/release.md`](../recipes/release.md) and the texts [`recipes/release-issue.md`](../recipes/release-issue.md) |
| Why it is built this way, and its limits | [`design/release.md`](../design/release.md) |
| The rules `REL1`–`REL7` | [ct-conventions `rules/release.md`](https://github.com/control-toolbox/ct-conventions/blob/main/rules/release.md) |
| The tool, the gate, the tag, the breakage suite | [CTActions `docs/workflows/`](https://github.com/control-toolbox/CTActions/tree/main/docs/workflows): `release-gate.md`, `release-tag.md`, `breakage-suite.md` |
| The workflow that runs a release from its request: the jobs, the modes, the App, the token, setting it up | [CTActions `docs/workflows/release.md`](https://github.com/control-toolbox/CTActions/blob/main/docs/workflows/release.md) |
| Who acts in a release (the App, the tokens) and why | [CTActions `docs/design/release-identities.md`](https://github.com/control-toolbox/CTActions/blob/main/docs/design/release-identities.md) |
| The two registries | [ct-registry `RELEASING.md`](https://github.com/control-toolbox/ct-registry/blob/HEAD/RELEASING.md) |
