# Recipe: release a package

How an agent releases one control-toolbox package, stable or beta, from "prepare a
release" to the closed tracking issue. What a correct release is — its number, its betas,
its registration — is in
[`rules/release.md`](https://github.com/control-toolbox/ct-conventions/blob/main/rules/release.md)
of ct-conventions (`REL1`–`REL7`), and how the two registries work is in
[`ct-registry/RELEASING.md`](https://github.com/control-toolbox/ct-registry/blob/HEAD/RELEASING.md).
This file says who does what, in which order. The texts the agent posts are in
[`release-issue.md`](release-issue.md).

## Principles

1. **Checked by tools, not by care.** Every rule that a tool can decide is decided by the
   release tool of CTActions,
   [`scripts/release/release.jl`](https://github.com/control-toolbox/CTActions/blob/main/docs/workflows/release-gate.md#the-release-tool),
   and every check that must hold before a merge is a **required check** of `main`: the
   release gate (`REL1`–`REL7`) and the breakage report (`REL1` against the base). Merging
   is done by GitHub's auto-merge, which waits for them. An agent that skips a step, or a
   human who forgets a script, cannot merge a release that breaks a rule.
2. **The state comes from the facts.** `release.jl status` computes where a release stands
   from GitHub and the registries — never from memory, never from a comment. Every session
   starts with it, and every outward action is preceded by it. A release interrupted by
   anything (a session that ends, a bot that fails, a runner down, `main` that moves) is
   resumed by running it again.
3. **Every action is "ensure".** Before doing something, check that it is not done: an
   issue, a PR, a tag that exists is reused or reported, never created twice. The
   registration comment is never posted twice (`REL6`).

Meant for local and remote mode alike: in remote mode, the orchestration runs the same
commands.

## Trigger

The human asks, in any language, to prepare a release of a package: "prepare a stable
release", "prepare a beta". In remote mode (to come), a comment `/release stable` or
`/release beta` on an issue of the package.

**Stable or beta is the human's choice. Never infer it** — not from the current version,
not from the branch, not from the changes. If the request does not say, ask, and do nothing
else until answered.

## Inputs

- `ROOT` — the directory that contains the packages, `CTActions` and `.worktrees/`.
- `PKG` — the package directory under `ROOT`; `NAME` — its registered name; `REPO` — its
  GitHub repository.
- `KIND` — `stable` or `beta`, given by the human.
- `TOOL` — `julia --startup-file=no "$ROOT/CTActions/scripts/release/release.jl"`, from an
  up-to-date clone of CTActions (`git -C "$ROOT/CTActions" pull`).

## Modes

The human chooses the mode in phase 1:

| | `supervised` | `autonomous` |
| --- | --- | --- |
| Merging the release PR | the human merges, or enables auto-merge | the agent enables auto-merge; GitHub merges when the required checks pass |
| Posting the registration comment | after the human's go | without asking |
| `[merge approved]` on General (`REL3`) | the human, or the agent after the human's go | the agent, if betas explain the jump |
| Everything else | same in both modes | same in both modes |

In `supervised` mode the agent stops at every checkpoint marked **⏸** below and waits for
a go. In `autonomous` mode it goes on, except on a stop condition.

**`autonomous` needs the protections.** Offer it only when the repository has them — else
only `supervised` (say why):

```bash
gh api "repos/$REPO" --jq .allow_auto_merge                                   # true
gh api "repos/$REPO/branches/main/protection" --jq '[.required_status_checks.strict, [.required_status_checks.checks[].context]]'
# strict true, and the contexts include the release gate and the breakage report
```

**Always asked, whatever the mode:** the kind (stable or beta, when not given), the
version (phase 1), the mode (phase 1).

## Ground rules

- **One release at a time** per package.
- **`status` first.** `$TOOL status --repo "$REPO" [--version "$VERSION"]` at the start of
  every session and before every outward action. Its phase and next action rule; when it
  says *blocked*, stop and report its blockers.
- **The tracking issue is the trace.** Every phase ends with **one new comment** on it,
  never an edit of an earlier one, ending with the marker of
  [`release-issue.md`](release-issue.md#the-state-marker), for the reader: the state is
  recomputed by `status`, not read from the marker.
- **The registration bot is never mentioned, except once** (`REL6`). Not in the issue
  description, the tracking comments, the PR, a commit message, the GitHub release or a
  report to the human; never for a beta. The registration comment builds the mention when
  it is posted (see [`release-issue.md`](release-issue.md#registration-comment)).
- Work in a fresh worktree from `origin/main`, never in `$ROOT/$PKG` (as in
  [`harmonize.md`](harmonize.md#b-worktree)): `$ROOT/.worktrees/$PKG-release`, branch
  `release/v$VERSION`.
- Past entries of `CHANGELOG.md` and `BREAKING.md` are not edited.
- English everywhere (`GEN1`). One line per paragraph in issue and PR texts.

## Phases

| # | Phase | Who acts | Checked by | ⏸ in `supervised` |
| --- | --- | --- | --- | --- |
| 0 | Preflight | agent, read only | `status`, the commands below | — |
| 1 | Proposal | agent, then the human | — | **always**, both modes |
| 2 | Tracking issue | agent | `status` | — |
| 3 | Release PR | agent | `release.jl gate`, locally then as a required check | — |
| 4 | Checks | CI | required checks | — |
| 5 | Merge | GitHub (auto-merge) or the human | branch protection | the human merges |
| 6 | Tag and GitHub release | the `ReleaseTag` workflow | `status` | — |
| 7 | Registration | agent (stable: one comment; beta: LocalRegistry) | `status` | ⏸ before the comment |
| 8 | Close | agent | `status` | — |

### 0. Preflight

Read only: nothing is written, so nothing is posted if the human stops at phase 1.

1. **Kind.** Known, or ask (see [Trigger](#trigger)).
2. **Status.** `$TOOL status --repo "$REPO"`. It must be phase 0 (nothing in progress, the
   last release complete). Any other phase is a release in progress: resume it from that
   phase instead of starting another. *Blocked* (e.g. the last version registered without
   tag) is a stop: report it.
3. **`main`.** Up to date and green:

   ```bash
   git -C "$ROOT/$PKG" fetch origin
   git -C "$ROOT/$PKG" rev-list --left-right --count main...origin/main   # "0 0", or behind only
   sha=$(git -C "$ROOT/$PKG" rev-parse origin/main)
   gh run list -R "$REPO" --commit "$sha" --json workflowName,status,conclusion \
     --jq '.[]|"\(.workflowName)\t\(.status)\t\(.conclusion)"'
   ```

   A local `main` ahead of `origin/main` is a stop (ask: unpushed commits). Every run on
   `origin/main` is `completed` with `success` or `skipped`.
4. **Breakage baseline.** The release PR compares its breakage with a run on its base:
   check there is one for `sha` (`gh run list -R "$REPO" --workflow Breakage.yml --commit
   "$sha"`); if not, start it now (`gh workflow run Breakage.yml -R "$REPO" --ref main`),
   it runs during the proposal.
5. **Open pull requests**, for the proposal: `gh pr list -R "$REPO" --state open --json
   number,title,isDraft,author`.
6. **Versions and changes.** The tags, the package in both registries, the changelog since
   the reference (`LAST_STABLE` for a stable, `LAST` for a beta): `[Unreleased]`, the
   `💥`/`✨` sections, the merged PRs, the files changed since the reference tag. The
   number follows `REL1`–`REL3`; the gate computes the same expected version and will
   refuse another. **Something to release (`REL7`):** a change under `src/`, `ext/`,
   `docs/`, or in `Project.toml` beyond the version; tests, CI or the changelog alone are
   not a release: stop and say so.
7. **Stable only — `REL4`.** `$TOOL registrable` on the `Project.toml` of `origin/main`:

   ```bash
   d=$(mktemp -d); git -C "$ROOT/$PKG" show origin/main:Project.toml > "$d/Project.toml"
   $TOOL registrable "$d/Project.toml"
   ```

   If it fails, stop: a dependency needs its stable release first; the report names the
   bound and the betas.
8. **Mode available.** Whether `autonomous` can be offered (see [Modes](#modes)).

### 1. Proposal ⏸

Present the [proposal](release-issue.md#proposal) (in remote mode, as a comment on the
issue of the trigger): the version and its reasoning, the recap of the changes, what the
agent will do, what the human will do, the open PRs, the jump if any, the modes available.
Then ask, and **wait**:

1. Is the version right?
2. `supervised` or `autonomous`?

A different version given by the human must still pass the gate: say which rule it would
break, if any.

### 2. Tracking issue

Ensure: `status` says phase 0 or 2 for `VERSION`; if the issue exists, reuse it.

```bash
gh issue create -R "$REPO" --title "[RELEASE] v$VERSION" --label release --body-file "$BODY"
```

`$BODY` is the [issue description](release-issue.md#issue-description), **not** the
organization's template. First comment: the decisions (kind, version and its reasoning,
mode), the proposal as validated. In remote mode the issue of the trigger is the tracking
issue: edit its title and description instead.

### 3. Release PR

Ensure: `status` says phase 3; if a `release/v$VERSION` PR exists, continue with it.

In the worktree (`git -C "$ROOT/$PKG" worktree add "$ROOT/.worktrees/$PKG-release" -b release/v$VERSION origin/main`):

1. `Project.toml`: `version = "$VERSION"`, nothing else.
2. `CHANGELOG.md`: the `[Unreleased]` entries move under `## [$VERSION] - YYYY-MM-DD`
   (`CHG2`, `✅ Compatibility` last); the merged PRs missing are added; an empty
   `## [Unreleased]` stays on top.
3. `BREAKING.md`: the breaking changes with their migration block (`CHG3`), or a
   `## Non-breaking note ($VERSION)` (`CHG4`).
4. **The labels.** The run labels of the package's workflows, except `run probe`:

   ```bash
   grep -rhoE "labels\.\*\.name, '[^']+'" .github/workflows | sed -E "s/.*'([^']+)'/\1/" | sort -u
   ```

   **GPU:** `run ci kkt-runner` if the package has a kkt job and `kkt` is online; else
   `run ci occidata-runner` if it has an occidata job and `occidata` is online; if a GPU job
   exists and neither runner is online, stop.

   ```bash
   gh api orgs/control-toolbox/actions/runners --jq '.runners[]|"\(.name)\t\(.status)\t\(.busy)"'
   ```

   Plus `documentation push preview` when the repository has it.
5. **The gate, locally**, before anything is pushed — the same command the required check
   runs:

   ```bash
   git commit -am "chore: prepare $NAME v$VERSION release"
   $TOOL gate --base origin/main --labels "<the labels, comma-separated>" .
   ```

   Fix what it refuses (it says which rule), or stop if the fix is not a release-file edit.
6. Push; open the PR; add the labels **after** creation (the label-gated callers react to
   `labeled`); in `autonomous` mode, enable auto-merge:

   ```bash
   gh pr create -R "$REPO" --base main --head "release/v$VERSION" \
     --title "chore: prepare $NAME v$VERSION release" --body-file "$PR_BODY"
   gh pr edit "$PR" -R "$REPO" --add-label "<label>" ...
   gh pr merge "$PR" -R "$REPO" --auto --merge           # autonomous only
   ```

   `$PR_BODY`: what the release contains in two or three lines, `Refs #<issue>` (never
   `Closes`), the generation note.

### 4. Checks

Wait for every check of the PR to end (Claude Code: see
[`adapters/claude-code.md`](../adapters/claude-code.md#waiting-for-pull-request-checks)),
then comment the [checks summary](release-issue.md#checks-summary): the gate's table, the
breakage report (its release check included), every other check. A failed check is a stop
in both modes: the gate and the breakage report say which rule; fix the release files and
push, or report.

### 5. Merge

- **`autonomous`:** nothing to do: GitHub merges when the required checks pass and the
  branch is up to date with `main` (`strict`). If `main` moved, update the branch
  (`gh pr update-branch`), and the checks run again.
- **`supervised` ⏸:** say that the PR is ready, and wait for the human to merge it or to
  enable auto-merge.

`status` then says phase 6. Comment: the merge commit.

### 6. Tag and GitHub release

Done by the `ReleaseTag` workflow of the package, on the push of the merge: it tags the
merge commit, creates the GitHub release (notes since the previous stable for a stable,
`REL5`; a beta is a pre-release), and runs the documentation workflow on the tag. Wait for it, then `status`: phase 7. If it did not run or failed, run it
again by hand — it never tags twice:

```bash
gh workflow run ReleaseTag.yml -R "$REPO" --ref main
```

Check that the documentation workflow of the tag succeeded (`gh run list -R "$REPO"
--branch "v$VERSION"`). If it did not run (the tag exists, so a rerun of `ReleaseTag` has
nothing left to do), dispatch it: `gh workflow run Documentation.yml -R "$REPO" --ref
"v$VERSION"`. A failure there is reported; it does not stop the release.
Comment: the release URL.

### 7. Registration

Ensure: `status` says **phase 7, todo**. That answer already checks what must hold: the
tag on the merge commit, the GitHub release, `main` still at the merge commit (the bot
registers the head of the default branch), no registration mention on the issue yet, no
General PR. Any other answer: follow it, or stop.

#### Stable — General

**`supervised` ⏸:** ask the human for the go. Post the
[registration comment](release-issue.md#registration-comment), **once**. Then `status`
again, after a minute:

- *waiting* on a General PR: note it;
- *blocked* "the registration comment is posted but General has no PR": read the bot's
  answer on the issue; **never post again**; report to the human, who decides.

If the gate reported a **jump** (`REL3`): in `supervised` mode, ask the human to comment
`[merge approved]` on the General PR, or for the go to do it; in `autonomous` mode, post it
(the gate has checked that betas explain the jump). If AutoMerge does not take the comment
into account, report it.

Wait for the General PR to be merged (Claude Code: see the adapter), then `status`: phase
8 once General holds the version, with the tree of the tag.

#### Beta — ct-registry

No comment, no mention. From the merge commit, with `LocalRegistry` in a shared
environment, ct-registry installed in the depot and SSH access to it:

```bash
git -C "$ROOT/.worktrees/$PKG-release" checkout --detach "$MERGE_SHA"
julia --startup-file=no --project=@ct-release -e '
  using Pkg; Pkg.add("LocalRegistry"); using LocalRegistry
  register(ARGS[1]; registry="ct-registry", repo=ARGS[2])' \
  "$ROOT/.worktrees/$PKG-release" "git@github.com:$REPO.git"
```

Then `status`: phase 8 once ct-registry holds the version, with the tree of the tag. In
remote mode, this step runs in a job that has the `SSH_KEY` secret.

Comment: the General PR and its merge, or the ct-registry commit.

### 8. Close

Ensure: `status` says phase 8, todo. Final comment ([template](release-issue.md#final-comment)):
the checklist, every link, what was left as is and why, the follow-ups (downstream
`[compat]`: CompatHelper opens those PRs). Then:

```bash
gh issue close "$ISSUE" -R "$REPO"
git -C "$ROOT/$PKG" worktree remove --force "$ROOT/.worktrees/$PKG-release"
git -C "$ROOT/$PKG" branch -D "release/v$VERSION"
git -C "$ROOT/$PKG" push origin --delete "release/v$VERSION" 2>/dev/null   # if the merge kept it
```

`status` must now say *done*. Report to the human: the version, the links, the stops and
what was skipped.

## Prerequisites in a package

What the recipe relies on; without them, only `supervised` mode, and the gate runs only
locally:

- the callers `ReleaseGate.yml`, `ReleaseTag.yml` and `Breakage.yml` on
  `breakage-suite.yml` (see CTActions' [workflows](https://github.com/control-toolbox/CTActions/blob/main/docs/README.md));
- `Documentation.yml` accepts `workflow_dispatch` (`ReleaseTag` runs it on the tag);
- `allow_auto_merge` on, and `main` protected with `strict` required checks: the release
  gate, the CI and documentation aggregators (`ci-ok`, `docs-ok`), the breakage report —
  see CTActions' [Required checks](https://github.com/control-toolbox/CTActions/blob/main/docs/conventions/required-checks.md).

## Stop and report when

In both modes, and whatever was validated before:

- the kind was not given (ask: never infer it);
- `status` says *blocked*;
- `main` is not green, or a local `main` is ahead;
- nothing to release (`REL7`), or a stable does not resolve from General alone (`REL4`);
- the local gate refuses the PR and the fix is not a release-file edit;
- a check of the PR fails;
- a GPU job exists and no GPU runner is online;
- the bot answers the registration with an error, or does not answer;
- anything not covered by this recipe.

Whenever you stop, comment on the tracking issue (once it exists) with the reason, the
output of `status`, and the state of the worktree: the next session resumes from there.
