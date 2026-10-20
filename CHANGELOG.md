# Changelog

All notable changes to ct-agents will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.3.0] - 2026-10-20

### ✨ New Features

- Phase 8 of [`recipes/release.md`](recipes/release.md) ends with `release.jl audit`
  (CTActions#120): once `status` says *done*, the agent checks the trail the release left
  (the comments of the issue, the one registration mention, the PR, the tag, the registry,
  the deleted branch) and reports a ❌ to the human, never repairing the trail. The guide
  tells a developer they can run it on any release.

## [0.2.0] - 2026-10-19

### 🔧 Changed

- The clone of ct-conventions in a package is named `.ct-conventions/` instead of
  `.conventions/`, as the clone of ct-agents is `.ct-agents/` (ct-conventions v0.7.1). The
  `AGENTS.md` template, its clone commands, the `.gitignore` and `_typos.toml` entries and
  the bump commands of [`recipes/pin-conventions.md`](recipes/pin-conventions.md) follow, with
  a section to move a package from the former name. The old name does not occur in the new
  one, so one `sed` does it.

## [0.1.0] - 2026-10-19

First tagged version. Until now the repository was read from `main`, so a change reached
every package at once and no report could say which version of a recipe it had applied. A
package now pins a tag, as it pins one of ct-conventions (see the README, *Versioning*).

### ✨ New Features

- [`base.md`](base.md), the behavior every agent shares; the harness adapter
  [`adapters/claude-code.md`](adapters/claude-code.md); the setup note
  [`setup/github-pat.md`](setup/github-pat.md).
- Recipes for harmonizing the repositories of the ecosystem
  ([`recipes/harmonize.md`](recipes/harmonize.md) and its themes: pinning the conventions,
  labels, issue templates, lint and automation) and for migrating a package's docs to
  DocumenterVitepress.
- [`recipes/release.md`](recipes/release.md), releasing a package, stable or beta, on top of
  the release tool of CTActions (`status`, `gate`, `ready`, `merge`, `register`), in
  `supervised` and `autonomous` modes, with the texts of the tracking issue
  ([`recipes/release-issue.md`](recipes/release-issue.md)); the guide for the developer
  ([`guides/release.md`](guides/release.md)) and the design behind it
  ([`design/release.md`](design/release.md)).
- A versioning of the repository: this changelog, SemVer tags, and a workflow that tags
  and releases on the push of a new `## [X.Y.Z]` section.
- The agent states, in the proposal of a release, the versions it applies: ct-agents,
  ct-conventions and the commit of CTActions.
