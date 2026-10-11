# derive_more-impl 2.1.1 compatibility patch

Source: the published crates.io `derive_more-impl` 2.1.1 archive, with its original MIT/Apache licensing retained. Only `src/constructor.rs` is modified: named constructors emit Rust field shorthand instead of repeating `field: field`. The generated constructor API, constness, argument types, field values and ordering are unchanged.

Rust Clippy 1.99.0 reports `redundant_field_names` for the original generated constructors despite their external macro origin. See [upstream issue 17525](https://github.com/rust-lang/rust-clippy/issues/17525). This source fix preserves strict lint checks instead of disabling the warning for authored code.

Original constructor source SHA256: `5752133576cd1ea50a9c8de544e5ecba7ca5300e7a7213a82f4cecfc2fbe6b22`.
