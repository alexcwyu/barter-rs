# barter-rs: validation results

<!-- current-documentation-provenance -->
Documentation reviewed: 2026-10-13 · Git source base: `fff97c6925e2e08012f5243675e7057457f5d506` · [Current architecture and feature guide](ayu-develop.md).
Historical execution commands/results below retain their original dates and scope; this metadata update is not a new application test run.
<!-- /current-documentation-provenance -->


All owned native targets build; the complete existing suite, formatting check and strict all-target Clippy pass.

## Verified scope

224 existing tests passed;3 pre-existing documentation examples remain ignored. No new cases were added.

Latest licensed derive_more-impl2.1.1 emits equivalent field shorthand to address the Clippy1.99 external-macro false positive. Existing example/benchmark fixtures and unit-success branches were adapted to current lints; constructor values/signatures and meaningful existing assertions are preserved. Strict Clippy uses -D warnings, while stable rustfmt reports existing nightly-only imports_granularity configuration warnings.

## Environment

Rust stable1.99.0 on macOS ARM

## Executed checks

Exploratory failures remain visible. Use later successful runs only for the scope they actually exercised; builds, syntax checks and benchmark smoke runs are distinct from complete test suites.

| Check | Exit | Duration | Command |
| --- | ---: | ---: | --- |
| build | 0 | 145.02s | `cargo build --workspace --all-targets` |
| clippy-constructor-source | 101 | 7.43s | `cargo clippy --workspace --all-targets -- -D warnings` |
| clippy-current-modules | 101 | 5.87s | `cargo clippy --workspace --all-targets --locked -- -D warnings` |
| clippy-existing-cases | 101 | 1.29s | `cargo clippy --workspace --all-targets --locked -- -D warnings` |
| clippy-final | 101 | 6.98s | `cargo clippy --workspace --all-targets --locked -- -D warnings` |
| clippy-final-existing-workspace | 0 | 4.54s | `cargo clippy --workspace --all-targets --locked -- -D warnings` |
| clippy-machine-fixes | 0 | 22.82s | `cargo clippy --fix --workspace --all-targets --allow-dirty --allow-staged -- -D warnings` |
| clippy-post-fix-check | 101 | 7.49s | `cargo clippy --workspace --all-targets --locked -- -D warnings` |
| clippy-required | 101 | 10.84s | `cargo clippy --workspace --all-targets --locked -- -D warnings` |
| final-existing-build-tests | 1 | 1.57s | `sh -c /opt/homebrew/opt/rustup/bin/cargo fmt --all --check && /opt/homebrew/opt/rustup/bin/cargo build --workspace --all-targets && /opt/homebrew/opt/rustup/bin/cargo test --workspace` |
| final-formatted-build-tests | 0 | 445.17s | `sh -c cargo fmt --all --check && cargo build --workspace --all-targets && cargo test --workspace` |
| format-required | 0 | 1.03s | `cargo fmt --all -- --check` |
| test | 0 | 100.21s | `cargo test --workspace` |

## Portable evidence

[Executed command records](maintenance/index.html) · [Machine-readable verification](maintenance/verification.json) · [Engineering guide](ayu-develop.md) · [Development workflow](development.md)

Full original logs are retained in the workspace maintenance evidence; portable records contain their hashes, exact execution metadata and available outcome lines.
