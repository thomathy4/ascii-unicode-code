# Split mapping file

The full Unicode mapping, `unicode_mapping.csv` (1,114,112 rows + header,
~105 MB), exceeds GitHub's 100 MB per-file limit, so it is stored here as
37 plain-text chunks, each under 3 MB:

- `unicode_mapping.chunk01.csv` … `unicode_mapping.chunk37.csv`

The header row is in `chunk01` only. Concatenating the chunks in order
reproduces the original file exactly.

Note: an earlier 13-part split (`unicode_mapping.part01.csv` …
`part13.csv`, each ~9 MB) was prepared first, but the GitHub MCP push
path rejects ~9 MB requests with HTTP 413, so it was re-split into
these ~3 MB chunks. Do not use the `part*.csv` files — they are not
part of this scheme. One of them, `part13.csv`, was pushed to GitHub
in the first snapshot under the old scheme; it is retained on GitHub,
not deleted, per Thomas's instruction (2026-11-17). The old local
`part01`–`part12` files were never pushed and are retained locally.

Change note — 2026-11-17: the 13-part split is superseded by the
37-chunk split because a ~9 MB push failed with HTTP 413 while a
3.8 MB Batch A push succeeded. Reassembly:
`cat mapping/unicode_mapping.chunk*.csv > unicode_mapping.csv`, or
`python3 mapping/reassemble.py`, verified by SHA-256 and line count.

## Reassemble

From the repository root, either:

```sh
cat mapping/unicode_mapping.chunk*.csv > unicode_mapping.csv
```

or:

```sh
python3 mapping/reassemble.py
```

`reassemble.py` verifies the result: SHA-256
`2c9163c7c67b8adbab8697c38e580648433d73428c7a23945def5d1826c784ef`
and 1,114,113 lines including the header.

The same file can also be regenerated from scratch with
`python3 generate_mapping.py` (reads `UnicodeData.txt`, Unicode 18.0.0).
