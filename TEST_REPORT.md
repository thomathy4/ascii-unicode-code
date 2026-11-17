# Test report — ASCII/ANSI Code for Unicode and Sentential Logic

Test run: 2026-11-17
Code version: 1.0.0
Unicode version used: **18.0.0** (UnicodeData.txt, downloaded 2026-11-17)
Python version: **3.12.3**
System `unicodedata` version: 15.0.0 — note: the mapping and the
tests use the downloaded Unicode 18.0.0 data, not the system
`unicodedata` module. The system version is reported by the test
harness only as a cross-check; it does not affect the results.
Test harness: `test_ascii_unicode_code.py`
Command run: `python3 test_ascii_unicode_code.py`
Overall result: **PASS** (last line of the run: `OVERALL: PASS`)

## Summary counts

| Test | Form / mode | Pass | Fail |
|---|---|---|---|
| Assigned characters round-trip | ESC | 310,341 | 0 |
| Assigned characters round-trip | Visible | 310,341 | 0 |
| All scalar values round-trip | ESC | 1,112,064 | 0 |
| All scalar values round-trip | Visible | 1,112,064 | 0 |
| Logic inventory | both forms + compact aliases | 18 | 0 |
| Space / LF / CR / CRLF | both forms + named forms + aliases | 5 | 0 |
| ESC ↔ visible conversion | both ways, sample + all assigned | 310,342 | 0 |
| Negative tests | — | 12 | 0 |
| Variant ESC-only | spot check + logic inventory | 19 | 0 |
| Variant visible-only | spot check + logic inventory | 19 | 0 |

"Assigned characters" here means the 310,341 entries in
UnicodeData.txt 18.0.0 that are characters: assigned 172,808 +
control 65 + private-use 137,468. Surrogates (2,048) are listed
in UnicodeData.txt as ranges but are **not** characters, so they
are excluded from this count and are tested separately as
negative cases (see below).

"All scalar values" means every code point U+0000–U+10FFFF
except surrogates: 1,114,112 − 2,048 = 1,112,064. This includes
unassigned code points (801,657), noncharacters (66), private-use
and control characters. Every one of them round-tripped in both
forms. The full round-trip was run in chunks of 8,192 characters
— it was not skipped or sampled. Each chunk was encoded, checked
to be ASCII-only (max byte ≤ 0x7F), decoded, and compared
character-for-character with the input.

## What each test did

### (a) Assigned characters, ESC and visible

For each form, all 310,341 assigned characters were encoded
in chunks of 8,192, decoded, and compared with the input.
Pass: 310,341 per form. Fail: 0 per form.

### (b) All 1,114,112 code points where applicable

For each form, all 1,112,064 scalar values (every code point
except surrogates) were encoded in chunks of 8,192, checked
ASCII-only, decoded, and compared.
Pass: 1,112,064 per form. Fail: 0 per form.

### (c) Logic inventory

All 18 logic symbols (see `logic_mapping.csv` / `SYMBOLS.md`):
for each symbol, `encode_char` was checked in both forms,
both forms were decoded back to the symbol, and — where a
compact alias exists — encoding with `--logic` produced the
alias and decoding with `--logic` produced the symbol back.
Pass: 18. Fail: 0.

### (d) Space / LF / CR / CRLF

Five cases, each checked in both forms, plus named forms and
aliases:
1. space literal, 2. LF literal with `--preserve-newlines`,
3. CR encoded, 4. CRLF, 5. mixed `a b\nc\rd\r\ne`.
Named forms (`encode_char(' ')`, `encode_char('\n')`,
`encode_char('\r')`) were checked in both forms, and
`decode_visible("[SP][LF][CR][CRLF]", aliases=True)` was
checked to give `" \n\r\r\n"`.
Pass: 5. Fail: 0.

### (e) ESC ↔ visible conversion, both ways

One mixed sample (ASCII, space, ¬, ∧, →, 😀, a CJK character,
U+10FFFF, U+0001, U+007F) was converted ESC → visible →
ESC and compared byte-for-byte, and decoded from both forms.
Then all 310,341 assigned characters were converted ESC →
visible → ESC in chunks of 8,192 and compared.
Pass: 310,342 (310,341 assigned + 1 sample). Fail: 0.

Note: an earlier version of this test failed on the chunk
containing LF/CR, because `convert_esc_to_visible` used
`preserve_newlines=True` and turned an encoded LF into a
literal newline. That was a real bug, found by the test, and
was fixed in `ascii_unicode_code.py` (both convert functions
now use `preserve_newlines=False`). The counts above are from
the run after that fix. The fix is also recorded in `SPEC.md`
§12, deviation 2.

A second real bug was also found by the tests and fixed
before this run: the mapping generator's `parse_ucd` treated
the surrogate ranges in UnicodeData.txt as assigned
characters, so the assigned round-trip failed on the chunk
containing U+D800. The parser now skips surrogate ranges.
The counts above are from the run after that fix.

A third real bug was found by reading back the README/HOWTO
commands as a new user would, after the counts above were
first recorded: conversion was decode-then-re-encode with
`preserve_newlines=False`, so a literal trailing newline
became an encoded one and ESC → visible → ESC was not
byte-identical (`cmp` failed at the trailing LF). Conversion
is now token-by-token (see `SPEC.md` §4.3), and the
byte-identical check (`cmp` both directions, and with
`--no-preserve-newlines`) passes. The full test harness was
re-run after that fix and still reports OVERALL: PASS with
the same counts as above.

### (f) Negative tests

All 12 were required to raise an error, and all 12 did.
Pass: 12. Fail: 0.

1. Surrogate encode (`encode_text("\ud800")`) — rejected.
2. Surrogate decode, visible (`\uD800`) — rejected.
3. Surrogate decode, ESC (`ESC[UD800m`) — rejected.
4. Out-of-range decode, ESC (`ESC[U110000m`) — rejected.
5. Out-of-range decode, visible (`\U00110000`) — rejected.
6. Malformed ESC hex (`ESC[U00ZZm`) — rejected.
7. Malformed ESC, no terminator (`ESC[U00AC` with no `m`) — rejected.
8. Malformed `\u` short (`\u00`) — rejected.
9. Malformed `\U` short (`\U0001F6`) — rejected.
10. Non-ASCII byte in ESC-form input — rejected.
11. ESC-only mode rejects `--form visible` — rejected.
12. Visible-only mode rejects `--form esc` — rejected.

## Variant tests

Each variant was spot-checked with the mode flag, plus the
full 18-symbol logic inventory in that mode:

- **ESC-only** (`--mode esc-only`, `--form esc`): sample
  round-trip (1) + logic inventory (18) = pass 19, fail 0.
  Also checked in the negative tests: ESC-only rejects
  `--form visible` and `convert --to visible`.
- **Visible-only** (`--mode visible-only`, `--form visible`):
  sample round-trip (1) + logic inventory (18) = pass 19,
  fail 0. Also checked: visible-only rejects `--form esc`
  and `convert --to esc`.

The variant folders (`variants/esc-only/`,
`variants/visible-only/`) record these counts in their own
`VARIANT.md` files. The mapping CSVs are shared with the
primary Both build and were not regenerated for the variants.

## Layout and accessibility change (2026-11-17, after the counts above)

After the test run above, the documentation and CSVs were
reformatted to Thomas's layout rule: every symbol table has
Column 1 = Symbol and Column 2 = Definition in the same row,
with extra columns after the definition, and the CSVs were
reordered the same way (`unicode_mapping.csv`:
codepoint, definition, decimal, status, …;
`logic_mapping.csv`: symbol, definition, codepoint, …).
Every CSV row now has a non-empty definition in column 2
(verified: 1,114,112 rows, 0 empty; 18 logic rows, 0 empty).
Every Markdown symbol table was checked by a script that
parses table rows and verifies the definition cell is
present and non-empty in the immediately adjacent column:
all tables pass, 0 bad rows (see the layout check in the
final report). The full test harness was re-run after the
reformat and still reports OVERALL: PASS with the same
counts as above. The mapping file grew from about 65 MB to
about 105 MB because unassigned / surrogate / noncharacter
rows now carry a status definition instead of an empty name.

Accessibility (ISO/IEC 40500 / WCAG 2.0/2.1 AA and
ISO 9241-171, as they apply to documentation) is recorded
in `README.md`, "Accessibility and table layout": one
header row per table, no merged cells, no empty header
cells, nothing conveyed by colour alone, plain language,
consistent heading structure, descriptive link text. No
HTML version is produced.

## What was not tested

- The CLI was spot-checked by hand (see `README.md` /
  `HOWTO.md` examples), but the test harness tests the
  library functions, not every CLI flag combination.
- Terminal display of ESC-form output was not tested as a
  test case — it is documented as expected behaviour
  (a terminal interprets ESC sequences) in `README.md`.
- No performance / timing test was run beyond the full
  round-trip itself, which took about 9 seconds for the
  whole harness on this machine.
