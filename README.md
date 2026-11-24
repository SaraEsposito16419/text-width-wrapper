# text-width-wrapper

Break text into lines no wider than a target column count, respecting word boundaries and East Asian character widths.

```rust
use text_width_wrapper::{wrap, char_width};

let lines = wrap("hello world", 5);
assert_eq!(lines, vec!["hello", "world"]);

// CJK ideographs occupy two terminal columns each, so only two fit in five.
let lines = wrap("\u{4e00}\u{4e01}\u{4e02}", 5);
assert_eq!(lines, vec!["\u{4e00}\u{4e01}", "\u{4e02}"]);

assert_eq!(char_width('a'), 1);
assert_eq!(char_width('\u{4e00}'), 2);
```

## Why this exists

Wrapping text by `char` count is wrong the moment a string leaves ASCII. A CJK
ideograph renders two columns wide in a terminal, so a line of ten ideographs
is twenty columns, not ten. The usual fix is to pull in a Unicode width crate,
but that brings a data table that changes with every Unicode release and a
dependency graph for a function that should be a dozen lines.

This crate takes the opposite trade-off. It ships a hand-written width table
covering the ranges that appear in real text — combining marks and format
characters as zero, East Asian Wide and Fullwidth as two, everything else as
one — and accepts that a few exotic ranges (emoji, historic scripts) are
counted as width one. That is stable, auditable, and has no data files to go
stale. If you need full UAX #11 coverage, use a crate that ships the data.

## The awkward edge

A character that is itself wider than the target width cannot fit on any line.
`wrap` emits it alone rather than dropping it, so no text is lost, but the
resulting line is wider than requested. This is the only case where the output
exceeds `width`, and it is deliberate.

## API

- `wrap(text: &str, width: usize) -> Vec<String>` — break `text` into lines of
  at most `width` columns. Existing newlines force a break and are not emitted;
  runs of spaces and tabs collapse to a single space between words. A `width`
  of zero returns an empty vector.
- `char_width(c: char) -> usize` — the number of terminal columns occupied by
  `c`: 0 for combining marks and zero-width characters, 2 for East Asian Wide
  and Fullwidth, 1 otherwise.

Both are exported from the crate root.
