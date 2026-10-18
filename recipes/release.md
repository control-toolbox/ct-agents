# Recipe: release a package

How an agent releases one control-toolbox package, stable or beta, from "prepare a
release" to the closed tracking issue. What a correct release is — its number, its betas,
its registration — is in
[`rules/release.md`](https://github.com/control-toolbox/ct-conventions/blob/main/rules/release.md)
of ct-conventions (`REL1`–`REL6`), and how the two registries work is in
[`ct-registry/RELEASING.md`](https://github.com/control-toolbox/ct-registry/blob/main/RELEASING.md).
This file says who does what, in which order. The texts the agent posts are in
[`release-issue.md`](release-issue.md).

Meant for local and remote mode alike: every phase starts from the state written in the
tracking issue, so a run can stop at any phase and be resumed by another session, or by
the orchestration of the remote mode.

## Trigger

The human asks, in any language, to prepare a release of a package: "prepare a stable
release", "prepare a beta". In remote mode (to come), a comment `/release stable` or
`/release beta` on an issue of the package.

**Stable or beta is the human's choice. Never infer it** — not from the current version,
not from the branch, not from the changes. If the request does not say, ask, and do nothing
else until answered.

## Inputs

- `ROOT` — the directory that contains the packages and `.worktrees/`.
- `PKG` — the package directory under `ROOT`; `NAME` — its registered name (the `name`
  of `Project.toml`); `REPO` — its GitHub repository.
- `KIND` — `stable` or `beta`, given by the human.

Found in phase 0: `LAST` (the last release, any kind), `LAST_STABLE` (the last version in
General), `VERSION` (the new one), `MODE`.

## Modes

The human chooses the mode in phase 1:

| | `supervised` | `autonomous` |
| --- | --- | --- |
| Merging the release PR | the human | the agent, under the conditions of [`base.md`](../base.md#git--commits) |
| Posting the registration comment | after the human's go | without asking |
| `[merge approved]` on General (`REL3`) | the human, or the agent after the human's go | the agent, if betas explain the jump |
| Everything else | same in both modes | same in both modes |

In `supervised` mode the agent stops at every checkpoint marked **⏸** below and waits for
a go. In `autonomous` mode it goes on, except on a stop condition.

**Always asked, whatever the mode:** the kind (stable or beta, when not given), the
version (phase 1), the mode (phase 1).

## Ground rules

- **One release at a time** per package: one open issue labelled `release`, one open
  `release/*` branch.
- **The tracking issue is the trace and the state.** Every phase ends with **one new
  comment** on it, never an edit of an earlier one, and every comment ends with the state
  marker of [`release-issue.md`](release-issue.md#the-state-marker). Resuming means
  reading the last marker.
- **The registration bot is never mentioned, except once** (`REL6`). Not in the issue
  description, the tracking comments, the PR, a commit message, the GitHub release or a
  report to the human; never for a beta. The registration comment of phase 7 builds the
  mention when it is posted (`printf`, see [`release-issue.md`](release-issue.md#registration-comment)),
  so no file and no command pasted in a comment contains it.
- Work in a fresh worktree from `origin/main`, never in `$ROOT/$PKG` (as in
  [`harmonize.md`](harmonize.md#b-worktree)): `$ROOT/.worktrees/$PKG-release`, branch
  `release/v$VERSION`.
- Past entries of `CHANGELOG.md` and `BREAKING.md` are not edited.
- English everywhere (`GEN1`). One line per paragraph in issue and PR texts.

## Phases

| # | Phase | Outward action | ⏸ in `supervised` |
| --- | --- | --- | --- |
| 0 | Preflight | none (read only) | — |
| 1 | Proposal | none | **always**, both modes: version and mode |
| 2 | Tracking issue | issue opened | — |
| 3 | Release PR | branch pushed, PR opened | — |
| 4 | Checks | none | — |
| 5 | Merge | PR merged | the human merges |
| 6 | Tag and GitHub release | tag, release | — |
| 7 | Registration | General: one comment; beta: ct-registry push | ⏸ before the comment |
| 8 | Close | issue closed | — |

### 0. Preflight

Read only: nothing is written anywhere, so nothing is posted if the human stops at phase 1.

1. **Kind.** Known, or ask (see [Trigger](#trigger)).
2. **`main` is up to date and green.**

   ```bash
   git -C "$ROOT/$PKG" fetch origin
   git -C "$ROOT/$PKG" rev-list --left-right --count main...origin/main   # "0 0", or behind only
   sha=$(git -C "$ROOT/$PKG" rev-parse origin/main)
   gh run list -R "$REPO" --commit "$sha" --json workflowName,status,conclusion \
     --jq '.[]|"\(.workflowName)\t\(.status)\t\(.conclusion)"'
   ```

   A local `main` ahead of `origin/main` holds unpushed commits the human may expect in the
   release: stop and ask. Every run on `origin/main` must be `completed` with `success` or
   `skipped`.
3. **No release in progress.**

   ```bash
   gh issue list -R "$REPO" --label release --state open
   gh pr list -R "$REPO" --state open --json number,headRefName \
     --jq '.[]|select(.headRefName|startswith("release/"))|.number'
   ```

4. **Open pull requests**, listed for the proposal (the human may want to wait for one):
   `gh pr list -R "$REPO" --state open --json number,title,isDraft,author`.
5. **Versions.** The tags, and the package in both registries (`L` is the uppercase first
   letter of `NAME`):

   ```bash
   git -C "$ROOT/$PKG" tag --sort=-v:refname | head -5
   curl -fsSL "https://raw.githubusercontent.com/JuliaRegistries/General/master/$L/$NAME/Versions.toml" | grep '^\['
   gh api "repos/control-toolbox/ct-registry/contents/$L/$NAME/Versions.toml" \
     -H "Accept: application/vnd.github.raw" | grep '^\['
   grep '^version' "$ROOT/$PKG/Project.toml"
   ```

   The `version` of `Project.toml` on `origin/main` is the last tag (`REL5`); if not, stop
   and report.
6. **Changes.** Since `LAST` for a beta, since `LAST_STABLE` for a stable:
   `git log --first-parent --oneline v$PREV..origin/main`, the PRs merged in that range,
   the `[Unreleased]` section of `CHANGELOG.md`, and the entries of `BREAKING.md` after
   `PREV`. A change is breaking when it has, or should have, a `💥 Breaking Changes` entry
   (`CHG3`); a merged PR missing from `[Unreleased]` is reported in the proposal.
7. **Stable only — `REL4`.** On the `Project.toml` of `origin/main`, with
   [CTActions' script](https://github.com/control-toolbox/CTActions/blob/main/docs/workflows/registrable.md#running-it-locally)
   from an up-to-date clone of CTActions (`git -C "$ROOT/CTActions" pull`):

   ```bash
   d=$(mktemp -d); git -C "$ROOT/$PKG" show origin/main:Project.toml > "$d/Project.toml"
   julia --startup-file=no "$ROOT/CTActions/scripts/registrable/registrable.jl" "$d/Project.toml"
   ```

   If it fails, stop: a dependency needs its stable release first. The report names the
   bound and the betas that satisfy it; put it in the proposal.
8. **Version** by `REL1` and `REL2`, and for a stable, **jump** or not by `REL3` (compare
   with `LAST_STABLE`; if a jump, check that every skipped triple is a beta in ct-registry).

### 1. Proposal ⏸

Present the proposal of [`release-issue.md`](release-issue.md#proposal) to the human (in
remote mode, as a comment on the issue of the trigger): the version and the reasoning that
gives it, the recap of the changes, what the agent will do, what the human will do, the
open PRs, the jump if any. Then ask, and **wait**:

1. Is the version right?
2. `supervised` or `autonomous`?

A different version given by the human must still satisfy `REL1`–`REL3`: if it does not,
say which rule it breaks, and ask again.

### 2. Tracking issue

```bash
gh issue create -R "$REPO" --title "[RELEASE] v$VERSION" --label release --body-file "$BODY"
```

`$BODY` is the [issue description](release-issue.md#issue-description), **not** the
organization's issue template. The title is right from the start since the version is
decided; if it changes later, `gh issue edit --title` and say so in a comment. First
comment: the decisions (kind, version and its reasoning, mode), with the proposal as
validated. In remote mode the issue of the trigger is the tracking issue: edit its title
and description instead.

### 3. Release PR

In the worktree (`git -C "$ROOT/$PKG" worktree add "$ROOT/.worktrees/$PKG-release" -b release/v$VERSION origin/main`):

1. `Project.toml`: `version = "$VERSION"`.
2. `CHANGELOG.md`: the `[Unreleased]` entries move under `## [$VERSION] - YYYY-MM-DD`
   (`CHG2`, sections in order, `✅ Compatibility` last); the merged PRs missing from
   `[Unreleased]` are added; an empty `## [Unreleased]` stays on top.
3. `BREAKING.md`: the breaking changes with their migration block (`CHG3`), or a
   `## Non-breaking note ($VERSION)` (`CHG4`).
4. Stable: run the `REL4` script again on the worktree.
5. One commit, `chore: prepare $NAME v$VERSION release`; push; open the PR, then add the
   labels **after** creation (the label-gated callers react to `labeled`, not `opened`):

   ```bash
   gh pr create -R "$REPO" --base main --head "release/v$VERSION" \
     --title "chore: prepare $NAME v$VERSION release" --body-file "$PR_BODY"
   # the run labels the workflows of this package react to
   grep -rhoE "labels\.\*\.name, '[^']+'" .github/workflows | sed -E "s/.*'([^']+)'/\1/" | sort -u
   gh pr edit "$PR" -R "$REPO" --add-label "<each run label>" --add-label "documentation push preview"
   ```

   The run labels differ between packages (`run ci kkt-runner`, `run breakage
   applications`, …): take them from the workflows, never from a fixed list, leaving out
   `run probe` (a probe is diagnostic only and gates nothing). A label named
   there but absent from the repository (`gh label list`) is a stop. `documentation push
   preview` is added when the package has it.

   `$PR_BODY`: what the release contains in two or three lines, `Refs #<issue>` (never
   `Closes`: the issue closes in phase 8), the generation note.

The `Registrable` check runs by itself on the PR (it touches `Project.toml`).

### 4. Checks

Wait for every check of the PR to end (Claude Code: see
[`adapters/claude-code.md`](../adapters/claude-code.md#waiting-for-pull-request-checks)):

```bash
gh pr checks "$PR" -R "$REPO" --watch --fail-fast
gh pr checks "$PR" -R "$REPO" --json name,state,link
```

Comment on the issue with the [checks summary](release-issue.md#checks-summary): every
check and its result, the breakage table the Breakage workflow posts on the PR, the link
to the documentation preview. A failed check is a stop, in both modes.

### 5. Merge

Check that `main` has not moved since the PR was opened: `gh api repos/$REPO/commits/main
--jq .sha` is the base of the PR (`gh pr view --json baseRefOid`). If it moved, stop: the
release would not contain what was checked.

- **`autonomous`:** merge only if every condition of [`base.md`](../base.md#git--commits)
  holds — the diff is `Project.toml`, `CHANGELOG.md`, `BREAKING.md` only
  (`gh pr diff "$PR" --name-only`), every check passed or was skipped, `main` has not
  moved — then `gh pr merge "$PR" -R "$REPO" --merge`.
- **`supervised` ⏸:** say that the PR is ready, and wait for the human to merge
  (`gh pr view "$PR" --json state,mergeCommit`).

Comment: the merge commit `MERGE_SHA`.

### 6. Tag and GitHub release

```bash
gh release create "v$VERSION" -R "$REPO" --target "$MERGE_SHA" --title "v$VERSION" \
  --generate-notes --notes-start-tag "v$PREV" <--latest | --prerelease>
gh release view "v$VERSION" -R "$REPO" --json body --jq .body > whats-changed.md
```

`PREV` is `LAST_STABLE` for a stable (`REL5`: the notes cover everything since the previous
stable, betas included), `LAST` for a beta. A stable is `--latest`, a beta `--prerelease`.
Check that the documentation workflow of the tag succeeded (`gh run list -R "$REPO"
--branch "v$VERSION"`); a failure there is reported, it does not stop the release.
Comment: the release URL, the notes.

### 7. Registration

#### Stable — General

Checks, each pasted in a comment **before** the registration comment:

1. The issue holds no mention yet. Show the search finds one where there is one — the last
   registered `[RELEASE]` issue — then run it on this issue:

   ```bash
   count() { gh issue view "$1" -R "$REPO" --json body,comments \
     --jq '[.body, .comments[].body] | map(select(test("@" + "JuliaRegistrator"; "i"))) | length'; }
   count "$PREVIOUS_RELEASE_ISSUE"   # 1 or more
   count "$ISSUE"                    # 0
   ```

2. `VERSION` is not in General's `Versions.toml`, and General has no open PR for it
   (`gh pr list -R JuliaRegistries/General --state open --search "$NAME v$VERSION in:title"`).
3. `main` is still `MERGE_SHA` (`gh api repos/$REPO/commits/main --jq .sha`): the bot
   registers the head of the default branch, and the tag points to `MERGE_SHA` (`REL5`).

Then, **`supervised` ⏸:** ask the human for the go. Post the
[registration comment](release-issue.md#registration-comment), **once**. Wait for the
bot's answer on the issue (a minute):

- *Registration pull request created*: note the General PR.
- an error, or no answer: **stop. Never post the comment a second time**; report to the
  human, who decides.

If phase 0 found a **jump** (`REL3`): in `supervised` mode, ask the human to comment
`[merge approved]` on the General PR, or for the go to do it; in `autonomous` mode, post it
if every skipped triple is a beta in ct-registry, else stop. If AutoMerge does not take the
comment into account, report it: who may approve is General's rule, not ours.

Wait for the General PR to be merged (it can take a while; Claude Code: see the adapter),
then check that `VERSION` is in General's `Versions.toml`. TagBot finds the tag already
there and creates nothing.

#### Beta — ct-registry

No comment on the issue, no mention. From the merge commit, with `LocalRegistry` in a
shared environment (not the package's), ct-registry installed in the depot and SSH access
to it:

```bash
git -C "$ROOT/.worktrees/$PKG-release" checkout --detach "$MERGE_SHA"
julia --startup-file=no --project=@ct-release -e '
  using Pkg; Pkg.add("LocalRegistry"); using LocalRegistry
  register(ARGS[1]; registry="ct-registry", repo=ARGS[2])' \
  "$ROOT/.worktrees/$PKG-release" "git@github.com:$REPO.git"
```

Check that `VERSION` is in ct-registry's `Versions.toml` (phase 0, item 5). In remote mode,
this step runs in a job that has the `SSH_KEY` secret.

Comment: the General PR and its merge, or the ct-registry commit.

### 8. Close

Final comment ([template](release-issue.md#final-comment)): the checklist, every link (PR,
GitHub release, registry), what was left as is and why, the follow-ups (downstream
`[compat]`: CompatHelper opens those PRs). Then:

```bash
gh issue close "$ISSUE" -R "$REPO"
git -C "$ROOT/$PKG" worktree remove --force "$ROOT/.worktrees/$PKG-release"
git -C "$ROOT/$PKG" branch -D "release/v$VERSION"
git -C "$ROOT/$PKG" push origin --delete "release/v$VERSION" 2>/dev/null   # if the merge kept it
```

Report to the human: the version, the links, the stops and what was skipped.

## Stop and report when

In both modes, and whatever was validated before:

- the kind was not given (ask: never infer it);
- `main` is not green, a local `main` is ahead, or a release is already in progress;
- the version of `Project.toml` on `main` is not the last tag;
- a stable does not resolve from General alone (`REL4`);
- a check of the PR fails, or `main` moved under the PR;
- a run label named in the workflows does not exist in the repository;
- the issue already holds a mention of the registration bot before phase 7, or the bot
  answers with an error, or does not answer;
- a jump not explained by betas (`REL3`);
- anything not covered by this recipe.

Whenever you stop, comment on the tracking issue (once it exists) with the reason, the
phase, and the state of the worktree, ending with the state marker: the next session
resumes from there.
