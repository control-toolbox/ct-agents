# Changelog

All notable changes to ct-agents will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
