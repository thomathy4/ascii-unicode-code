# Split mapping file

The full Unicode mapping, `unicode_mapping.csv` (1,114,112 rows + header,
~105 MB), exceeds GitHub's 100 MB per-file limit, so it is stored here as
13 plain-text chunks, each under 9 MB:

- `unicode_mapping.part01.csv` … `unicode_mapping.part13.csv`

The header row is in `part01` only. Concatenating the parts in order
reproduces the original file exactly.

## Reassemble

From the repository root, either:

```sh
cat mapping/unicode_mapping.part*.csv > unicode_mapping.csv
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
