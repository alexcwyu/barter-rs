# barter-rs: development workflow and layout

<!-- current-documentation-provenance -->
Documentation reviewed: 2026-10-13 · Git source base: `fff97c6925e2e08012f5243675e7057457f5d506` · [Current architecture and feature guide](ayu-develop.md).
Historical execution commands/results below retain their original dates and scope; this metadata update is not a new application test run.
<!-- /current-documentation-provenance -->


Use branch `feature/ayu_develop`. The actual upstream default branch and all fetched branch/tag identities are recorded in the maintenance sync evidence. Merge existing work; do not reset divergent branches or replace conflicting tags.

## Source synchronization

```sh
git fetch origin
git fetch upstream
git switch feature/ayu_develop
git merge origin/feature/ayu_develop
# Merge the actual default branch documented in the sync result.
```

This maintenance used namespaced remote tags before comparing tag identities. Missing upstream tags/branches were pushed to the fork; divergent branches were merged when conflict-free, and conflicting identities were retained for explicit reconciliation. The feature branch contains the default branch plus the maintenance changes. Source snapshot and architecture are documented in [ayu-develop.md](ayu-develop.md).

## Repository layout

- `docs/ayu-develop.md` and `docs/ayu-develop.html`: engineering guide with architecture, workflow, class and sequence diagrams and implementation links.
- `docs/development.md`: toolchain, layout and contributor workflow.
- `docs/validation.md`: exact executed commands, outcomes and limitations.

## Native Rust toolchain

Rust stable1.99.0 is installed through rustup. Compile all applicable targets and run the existing test suite. Optional Python features require the validated managed interpreter through `PYO3_PYTHON`. Source locks are checked in where appropriate for application/workspace reproducibility; build outputs remain ignored.

## Verification and documentation

Use the commands and results in [validation.md](validation.md). Markdown is the editable source of the guide; HTML embeds rendered SVG diagrams for offline browsing. Keep actual source links valid after moving modules. Preserve existing cases/assertions while correcting incompatible APIs, genuine resource leaks or invalid fixtures. No additional test cases were requested or added.

## Historical reference

The [previous development guide](archive/development-before-maintenance.md) is retained for source history. Its retired APIs and earlier toolchains are not the current branch instructions.

## Current compiler compatibility

Run `cargo build --workspace --all-targets`, `cargo test --workspace`, `cargo fmt --all --check` and `cargo clippy --workspace --all-targets -- -D warnings`. Existing example/benchmark fixtures are included in the strict lint scope.

The latest derive_more-impl 2.1.1 source is licensed and patched only to emit named constructor initializers in field shorthand form. This addresses the documented [Clippy 1.99 external macro false positive](https://github.com/rust-lang/rust-clippy/issues/17525), preserving constructor signatures, field values and constness. [Patch provenance](../vendor/derive_more-impl/AYU-PATCH.md) identifies the upstream source and exact change. Existing unit-success branches, fixed fixture arrays and type aliases were adapted to current lints without adding cases or weakening value assertions.
