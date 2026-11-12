Lab 5 — Encoding forensics

 Diagnosis (before the fix)

SHA-256 of the original `staff-broken.csv` (taken before anything else):

```
919027f1f8c71d85cba3f12aad0f6c927d4e0450cfb2a2d63378a69f14cab270
```

`file` says (from `evidence/file-before.txt`):

```
staff-broken.csv: Non-ISO extended-ASCII text, with CRLF line terminators
staff-broken.csv: Non-ISO extended-ASCII text, with CRLF line terminators
```

`file` only tells me that this is single-byte text with CRLF; it cannot tell code pages apart, so it is the first witness, not the verdict.

First 16 bytes (from `evidence/hexdump-before.txt`):

```
00000000: d2e0 e12e b93b d4c8 ce3b c4ee ebe6 edee  .....;...;......
```

| Offset | Bytes               | What they are |
|--------|---------------------|---------------|
| 00–02  | `d2 e0 e1`          | Three Cyrillic letters, one byte each: Т а б. Not a BOM (a UTF-8 BOM would be `ef bb bf`, UTF-16LE `ff fe`). |
| 03     | `2e`                | `.` — ASCII full stop, so the header starts `Таб.` |
| 04     | `b9`                | `№` as a single byte in this code page (in UTF-8 it would take three bytes: `e2 84 96`). |
| 05     | `3b`                | `;` — the field separator is a semicolon. |
| 06–08  | `d4 c8 ce`          | Ф И О |
| 09     | `3b`                | `;` — second separator. |
| 0A–0F  | `c4 ee eb e6 ed ee` | Д о л ж н о — the start of `Должность`. |

The first non-ASCII character, `Т`, is the very first byte `d2`: it takes **one** byte here, whereas in UTF-8 it would take two (`d0 a2`). No line ending falls inside the first sixteen bytes; it is established in the line-ending section below.

 Encoding: RK1048 (KZ-1048)

Evidence:

- The bytes are Cyrillic letters as single bytes in the range 80–FF (no `d0 xx` / `d1 xx` pairs), so the file is a single-byte code page, not UTF-8.
- `iconv -f UTF-8 -t UTF-8 staff-broken.csv` fails with `illegal input sequence at position 0`: the file is not valid UTF-8 (and therefore not double-encoded UTF-8 either).
- I decoded the file with five candidates (`CP1251 CP866 KOI8-R RK1048 PT154`). CP866 and KOI8-R give noise (`╥рс.╣;╘╚╬`). CP1251, RK1048 and PT154 agree on every Russian letter, so the Kazakh letters decided it:

| Name in the file | CP1251 | PT154 | RK1048 |
|------------------|--------|-------|--------|
| 1001 | Ќасымова Јлия | Қасымова Јлия | **Қасымова Әлия** |
| 1003 | Нўрланов Єалымжан | Нўрланов Әалымжан | **Нұрланов Ғалымжан** |
| 1005 | Ґтебаев | Ҙтебаев | **Өтебаев** |

  Only RK1048 shows every Kazakh letter correctly (Қ, Ә, Ұ, Ғ, Ө, Ү, Һ, І). CP1251 and PT154 each get some of them wrong.
- `iconv -f RK1048 -t UTF-8 staff-broken.csv > /dev/null` finishes without an error, so RK1048 decodes the whole file, not only the first lines. I read all 25 names after decoding and every one is correct.

 Line ending: CRLF (Windows)

- `cat -A staff-broken.csv | head -3` shows `^M$` at the end of every line.
- `tr -cd '\r' < staff-broken.csv | wc -c` gives **26** CR bytes and `wc -l` gives **26** LF bytes (one header + 25 staff rows), so every line ends in CR LF. No mixed endings.

 BOM: absent

The file starts with `d2 e0 e1`, not `ef bb bf`, `ff fe` or `fe ff`.

 What each application was doing wrong

I did not open the broken file in the applications (the original stays untouched), so this is how each reader decides the encoding, not a screenshot of the broken result:

- **Excel (double-click):** the file has no BOM, so Excel has nothing that says "UTF-8" and falls back to a legacy single-byte code page of the system. That is not RK1048, so the Kazakh letters (and the other upper-half bytes) are mapped to the wrong characters.
- **LibreOffice Calc:** with Character set on *Automatic* it has to guess without a BOM; for a single-byte file the guess is a default code page rather than RK1048, so the result differs from Excel's, because each program assumes a different default.
- **cat in a UTF-8 terminal:** the terminal decodes every byte as UTF-8. Single bytes in 80–FF such as `d2 e0 e1` are not valid UTF-8, so they show as replacement symbols or garbage.

In all three cases the bytes were intact; the reader assumed a different encoding from the one the file was written in.

 Fix

Script: `scripts/fix.sh`

```bash
#!/usr/bin/env bash
# Lab 5 - convert staff-broken.csv from RK1048 to UTF-8 (with BOM, CRLF kept)
set -euo pipefail
SRC_ENC="RK1048"
IN=evidence/staff-broken.csv
OUT=evidence/staff-fixed.csv

{
  printf '\xEF\xBB\xBF'
  iconv -f "$SRC_ENC" -t UTF-8 "$IN"
} > "$OUT"

file -e csv "$OUT"
xxd -l 16 "$OUT"
```

Changed: encoding RK1048 → UTF-8; line ending kept as CRLF; BOM added.

- **BOM added** because of Excel: opened by double-click, Excel uses the BOM to recognise UTF-8; without it the CSV is read in a legacy code page again. LibreOffice and `cat` work either way, so Excel was the reader that decided it. The cost is that a script reading the first column header will see an invisible U+FEFF in front of `Таб.№`.
- **CRLF kept** because I changed only what was necessary for the names to read correctly. The cost is that a script on Linux sees a hidden CR at the end of every line: `awk -F';' '{ print $NF "|" }' staff-fixed.csv` prints `|ата приёма`, the `|` jumps to the start of the line and overwrites the first character. A script consumer would strip it with `sed 's/\r$//'`.

Only one BOM was written (the source had none): the fixed file starts `ef bb bf d0 a2` (BOM, then `Т`), not a double BOM.

The original is untouched: `sha256sum staff-broken.csv` after the conversion is still `919027f1f8c71d85cba3f12aad0f6c927d4e0450cfb2a2d63378a69f14cab270`.

## Verification

`file` after (from `evidence/file-after.txt`):

```
staff-fixed.csv: Unicode text, UTF-8 (with BOM) text, with CRLF line terminators
staff-fixed.csv: Unicode text, UTF-8 (with BOM) text, with CRLF line terminators
```

First 16 bytes of the fixed file:

```
00000000: efbb bfd0 a2d0 b0d0 b12e e284 963b d0a4  .............;..
```

BOM `ef bb bf`, then `Т а б .` as UTF-8 (`d0 a2 d0 b0 d0 b1 2e`), `№` as three bytes (`e2 84 96`), `;`, and the start of `Ф` (`d0 a4`).

Opened in:

- **Excel** for Mac 16.59 (Microsoft Excel для Mac, Office LTSC 2021) — opened by double-click on the file, not through the import wizard; all names correct, six columns.
- **LibreOffice Calc** 26.8.2.2 (AARCH64, macOS 15.2) — the Text Import dialog proposed Character set *Automatic* = Unicode (UTF-8) and separator `;`, which I accepted without changing anything (`evidence/calc-import-dialog.png`).
- **cat** on Ubuntu 24.04.5 LTS, over SSH — all 25 names correct.

Evidence of all three side by side: `evidence/opens-correctly.png`.
