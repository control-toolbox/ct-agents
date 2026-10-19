# Releasing with an agent: the choices

Why the release of a control-toolbox package is built the way it is. For the person who
maintains it or wants to change it; a developer who just releases reads
[`guides/release.md`](../guides/release.md). The procedure itself is
[`recipes/release.md`](../recipes/release.md), the rules are `REL1`–`REL7` of
[ct-conventions](https://github.com/control-toolbox/ct-conventions/blob/main/rules/release.md),
the tools are documented in
[CTActions `docs/workflows/`](https://github.com/control-toolbox/CTActions/tree/main/docs/workflows).
This file does not repeat them: it says what each choice protects from.

## Goal

A release that an agent can run, locally today and from an issue comment later, and that
a human can trust without watching it: a wrong number, a stable that cannot be installed,
a second registration, a tag that does not match the registered tree must be **impossible
to get through**, not merely unlikely. So the agent is the hands; the decisions that a
tool can take are taken by a tool.

## The choices

**1. The state comes from the facts, never from memory.** `release.jl status` computes the
phase from GitHub (issues, PRs, tag, release, General's PR, the bot's answer) and the
registries. A session that ends, a runner that is down, a bot that fails, a `main` that
moves: run `status` again and go on. The marker at the end of each comment is for the
reader; nothing reads it back.

**2. Every action is "ensure".** Before creating something, check that it is not there: an
issue, a PR, a tag. A release can therefore be interrupted anywhere and resumed, and a
retry cannot duplicate. The one action that must happen exactly once, the registration
comment, is made by a command (`release.jl register`) that refuses unless `status` says it
is due, and checks afterwards that there is one.

**3. Rules are scripts.** `REL1`–`REL7` are decided by `release.jl gate`: the number
(SemVer as Pkg reads it, the suffix, a fresh triple, the expected version), the changelog
and BREAKING entries, a diff limited to the release files, something to release, the run
labels, and for a stable that it resolves from General alone. It runs locally before the PR
is pushed and again in the PR. An agent that is wrong is stopped by the gate; a gate that
is wrong is a bug to fix in one place.

**4. No auto-merge.** GitHub's auto-merge trusts check *names*. A workflow gated by a
label runs once per label added, a job skipped because the label added was another one
reports success, and a workflow whose jobs were all skipped has the conclusion
`skipped`: so a check can look green while the tests failed or never ran. The merge goes
through `release.jl merge` instead, which reads, per workflow, the latest run that
actually executed jobs on the head commit, requires every job a needed label gates to have
run and passed (one GPU runner is enough, `kkt` first), requires the gate and the breakage
report, then merges with `--match-head-commit`: GitHub refuses if the head moved since the
check. It merges `release/v…` branches only. This replaces any repository setting (no
required status checks to maintain, whatever the branch protection).

**5. Breakage is evidence for the number.** A change that breaks a downstream package is
acceptable, but then the version must say so (`REL1`). The breakage suite compares, on a
release PR, the **stable** column (the last release of each downstream package, betas
included) between the PR and a run on the base commit: ✅ then ❌ means the release breaks
it, and a non-breaking number fails. A package that already fails on the base is
reported and counts for nothing. The base run is made on every push to `main`, which is
why a release PR without one fails instead of passing blind. The downstream packages
are **derived** from the `Project.toml` files of the organization, not listed, so that a
new consumer cannot be forgotten.

**6. The bot is mentioned once.** The registration bot of General acts on its mention
wherever it appears, including an issue description, a PR or a commit message. `REL6`
forbids it everywhere except the one registration comment, and the gate checks the PR text
and the commits. The comment is built by the tool, so the mention exists in no file and in
no command of the recipe. If `main` moved after the merge, the bot would register the head
of the default branch, not the tagged commit: the comment is then posted on the merge
commit, with the human's go in both modes.

**7. The tag and the release are made by a workflow.** On the push of the merge,
`ReleaseTag` tags the merge commit and creates the GitHub release (notes since the
previous stable for a stable; a pre-release for a beta), then dispatches the documentation
on the tag: a tag created with `GITHUB_TOKEN` triggers no workflow, but `workflow_dispatch`
is allowed. The tag, the registry and the GitHub release must agree on the tree (`REL5`);
`status` refuses a version registered without a tag.

**8. Two registries, two paths.** A stable goes to General through its bot; a beta goes to
`ct-registry` with `LocalRegistry`, no comment, no mention. A beta is a tag on a fresh
triple and is never reused. A jump in General (a stable that skips versions) is allowed
only when betas explain it, and needs `[merge approved]`.

**9. Stable or beta is asked, never inferred.** The consequences of the wrong guess (a
version in General that nobody can use, or a beta where a stable was expected) are not
worth a heuristic.

**10. Docs-only changes justify a release; tests-only changes do not (`REL7`).** What the
users get is what is released.

**11. Two modes, one procedure.** `supervised` stops at the checkpoints; `autonomous`
goes on except on a stop condition and merges only through the tool. Both run the same
phases and the same commands, so that the remote agent (a GitHub bot triggered from an
issue comment) runs the same recipe with another driver.

**12. The procedure is versioned and its version is cited.** ct-agents has SemVer tags; a
package pins one in `AGENTS.md` and reads the recipes from a clone at that tag, as it does
for the conventions. A change to a recipe therefore reaches a package when it bumps its pin,
not at once, and a report can say which version of the procedure it applied. The cost is a
bump in each package when a recipe changes in a way that matters. The release tool (CTActions)
is not versioned yet: the proposal cites its commit.

## What each safeguard answers

| Safeguard | Failure it prevents, or one that happened |
| --- | --- |
| `status` from the facts | resuming from a wrong belief about what is done |
| GitHub's *list* API, not search | the search index lags behind new issues, so a second tracking issue was possible |
| The registered tree compared with the tag's | a registered version with no tag (found in CTParser's `v0.8.17-beta`) |
| `ready` reads runs and jobs | green checks from skipped label-gated jobs |
| `--match-head-commit` | a push between the check and the merge |
| One mention, counted afterwards | a double registration |
| `REL4` on the lowest and the newest Julia | a stable that only a beta dependency satisfies |
| The gate limits the diff to the release files | a release PR carrying other changes |
| Base run required | a breakage verdict made against nothing |

## Known limits

- **Each mode has run once**: `autonomous` on CTBase v0.30.12 (Claude Code), `supervised`
  on CTModels v0.20.0 (another agent, which found the recipe from `AGENTS.md` alone). Its
  feedback, step by step, is on CTModels.jl#476; it led to the three stops of `supervised`
  being stated, the wait being described, and the gate checking the encoding of the release
  files.
- **A failed documentation build of the tag** is reported, not retried: it must be rerun
  by hand (`gh workflow run Documentation.yml --ref vX.Y.Z`).
- **The gate is strict on the expected version.** A number the maintainer prefers but the
  rules do not give is refused; changing the number means changing the changelog first.
- **Not covered**: the first registration of a new package in General, and the reasons
  AutoMerge may refuse a PR beyond the version jump. Who may post `[merge approved]` on
  a General PR has not been verified.
- **`enforce_admins` is off** on the packages: an administrator can merge around the
  tool. The tool protects the agent and the routine, not against a deliberate bypass.
- **The consumers are read from the default branch** at the time of the run; a package
  that does not declare a dependency in its `Project.toml` (an application's docs
  environment) is declared by hand in `targets`.
- **Remote mode** (a GitHub bot, then an agent triggered from an issue comment) is
  designed for but not built: the orchestration would run the same commands, the beta
  registration needing the `SSH_KEY` secret and no secret being exposed to the agent.
