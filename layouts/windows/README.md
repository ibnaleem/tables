# Windows keyboard conversion tables

This completes the **218-entry Microsoft Windows keyboard catalog** audited on
7 October 2026. Every entry is indexed in [COVERAGE.md](COVERAGE.md) and
[index.json](index.json). There are **488 generated table files**:

- **352 key-position tables:** 176 distinct non-identity maps, in both directions.
- **136 composition tables:** 68 distinct common dead-key maps, in both directions.
- **34 catalog entries reuse an identical definition.** The index points them to
  the canonical files instead of creating duplicate copies.
- **25 entries have US-identical visible base/Shift key characters.** Some still
  have dead keys and therefore have composition tables. A blank table filename
  in the index means no key-position conversion is needed, not a missing file.

All earlier Bulgarian and Netherlands variants are included in this catalog.
Original upstream files and the earlier 14-file addition are not modified.
The new files live under `layouts/windows/` so that their provenance and naming
do not collide with the mixed legacy files at `layouts/`.

## Installation and selection

Extract the package into the root of an `ibnaleem/tables` checkout, or simply use
the extracted `layouts/windows/` directory. Git is not required. The archive also
contains an optional patch; use the extracted files or the patch, not both.

1. Find the exact active Windows keyboard name or KLID in `COVERAGE.md`.
2. Select **Layout → US** or **US → Layout**.
3. Use the key-position file for physical key labels. Where a layout has dead
   keys, use its separate composition file for the supported accented text.
4. Use one file per conversion. Do not merge unrelated layouts, both directions,
   or a key-position file and its composition alternative.

Filenames include the canonical Windows KLID and a readable name. An alias may
point to another language's filename because their recorded definitions are
identical. The headers list every alias, and the coverage index preserves every
individual Windows name.

Example with current Hashcat table-attack support:

```sh
# US key sequence "azerty" becomes "qwerty" on legacy French AZERTY.
hashcat -a 5 --stdout words.txt layouts/windows/0000040c_french_legacy_azerty-reverse.table identity=0

# Reverse that conversion.
hashcat -a 5 --stdout words.txt layouts/windows/0000040c_french_legacy_azerty.table identity=0
```

`identity=0` disables the automatic unchanged choice for matched tokens. Unknown
input is still passed through. If one character could have come from several
physical keys, multiple candidates are intentional. This is physical-key
conversion, not translation or transliteration.

The format is UTF-8: one source token, one TAB, and one replacement per line.
Whitespace tokens and a source beginning with `#` are encoded using `$HEX[...]`.
There are no empty-replacement/deletion rules. Older Hashcat releases may not
support the current table attack. Some old repository headers show a different
command; these files follow the current upstream `-a 5` interface.

## Precisely what is covered

The scope is the same main-key base/Shift scope as the first Bulgarian/Dutch
package: **Caps Lock off; 49 positions in Microsoft's standard diagrams,
including Space and ISO scan code 0x56**. The catalog contains 21,364 such cells.
Of these, 20,614 have a recorded output; 750 are blank or nonprinting and are
explicitly omitted. Omitted states are not silently turned into deletion rules.
They cannot be simulated from these text tables; unmapped text passes through.

Physical positions are normalized by scan code. The ANSI/ISO backslash key can
move between the diagrams' rows, so the parser handles that explicitly rather
than treating Windows virtual-key names as fixed physical positions.

Multi-character outputs are preserved, including Arabic lam-alef and Indic
sequences. Supplementary Unicode characters are retained in full UTF-8, including
ADLaM and historic-script layouts. Unicode normalization is not performed:
precomposed and decomposed character sequences remain different byte strings.

**This is complete catalog coverage within that scope, not every behavior of
every keyboard.** It does not cover AltGr/Ctrl/Caps Lock layers, the numeric
keypad, language-switch keys, macOS/Linux definitions, vendor layouts, or extra
JIS/ABNT hardware keys outside the published 49-position diagrams. For example,
the Brazilian ABNT/ABNT2 catalog entries are covered only on those positions.

Japanese, Korean, the Chinese US entries and Tamil Anjal have reference-layer
characters matching US in these diagrams. This does **not** convert kana, kanji,
Hangul, Chinese IMEs or a phonetic input engine. Their contextual composition
requires a different mechanism; the index does not invent character mappings.

## Dead-key tables and text ambiguity

Key-position tables treat dead-key legends as spacing tokens, consistent with
the existing repository. They are not a simulation of composing accented text.

The `_composed.table` variants are **standalone alternatives** covering a dead
key followed by a directly available base/Shift character. The inverse direction
uses the documented dead key + Space spelling for a standalone accent whenever
that spelling is provided by the source definition.

The index records compositions requiring another modifier layer or a prior
composition; those are not fabricated. Repeated or dangling dead keys, arbitrary
dead-key state, and all possible spellings of a spacing accent are not modeled.
These limits also apply to mnemonic/phonetic layouts whose letter keys themselves
act as dead keys, such as Russian Mnemonic and Cherokee Phonetic.

Hashcat matches the longest source token at a position. Where a layout can emit
several Unicode characters with one key, a whole token may also have been typed
using several separate keys. A static table cannot recover every alternative
segmentation or the original lost key identity. These tables preserve the
published single-key tokens and document that ambiguity instead of asserting a
unique reversible text transformation.

The ordinary table attack supports tokens up to 32 source bytes / 255 replacement
bytes in the tested implementation. The special `--keyboard-layout-mapping`
option is narrower (four bytes and deterministic mappings) and is not suitable
for all these tables. Use the table attack unless you have checked those limits
for your chosen forward file. Reverse files are table-attack files.

## Verification

Every table was loaded by **Hashcat's actual, unmodified table loader and
tokenizer** from commit `bb52c06b9e4ac7439b4415b69d04e0b72e1192f9`.
An independent Python implementation and fixed examples were used for comparison.

Results:

- 488 files passed strict UTF-8, one-tab, duplicate, token-length and regeneration checks.
- 41,228 bidirectional checks against the published physical-key states.
- 23,092 checks of included composition pairs.
- 18 fixed examples for AZERTY, QWERTZ, Dvorak, Colemak, Cyrillic, Arabic,
  supplementary Unicode and Dutch accents.
- The native implementation matched all 51,850 test inputs and 53,291 output candidates.

Hashcat compiled successfully. Its full CLI could not initialize executable-path
detection in this environment. The native probe therefore calls the loader and
tokenizer directly, without changing their code. This is a native parser and
tokenizer integration test, **not a GPU attack test or live Windows driver test**.
See [VALIDATION.json](VALIDATION.json) for the recorded results.

Rebuild and run the portable checks from the repository root:

```sh
python3 tools/windows_layouts/build.py
python3 tools/windows_layouts/verify.py
```

Python 3.9+ and its standard library are sufficient. The factual snapshot in
`tools/windows_layouts/definitions.json` contains all source URLs and SHA-256
hashes. No network is needed to regenerate the tables.

To reproduce the optional native checks on Linux, build the pinned Hashcat
checkout with `make`, then:

```sh
python3 tools/windows_layouts/build_native_probe.py /path/to/hashcat --output /tmp/native_table_probe
python3 tools/windows_layouts/verify.py --native /tmp/native_table_probe
```

## Source and licensing

- [Repository base commit](https://github.com/ibnaleem/tables/tree/e7030fda096016a7abda31a67738b4e43c9748b1)
- [Microsoft Windows keyboard catalog](https://learn.microsoft.com/en-us/globalization/windows-keyboard-layouts)
- Per-layout Microsoft diagram links are included in `COVERAGE.md`, file headers,
  and the source snapshot. Retrieved 2026-10-07.
- [Hashcat table implementation at the tested revision](https://github.com/hashcat/hashcat/blob/bb52c06b9e4ac7439b4415b69d04e0b72e1192f9/src/feeds/feed_table.c)
- [Hashcat table documentation](https://github.com/hashcat/hashcat/blob/bb52c06b9e4ac7439b4415b69d04e0b72e1192f9/docs/hashcat-table.md)

The archive carries the original repository's GPL-3.0 license. No Hashcat binary
or source checkout is bundled; the optional probe compiles against a separately
obtained Hashcat checkout. Nothing has been pushed or submitted upstream.

## Credits

@YD
