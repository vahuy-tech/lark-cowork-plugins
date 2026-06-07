# Reference ASM examples

Drop known-good **Allotrope Simple Model (ASM)** JSON files here, named by
instrument/technique (e.g. `agilent-hplc-1290.json`, `biotek-synergy-h1.json`).

The `instrument-data-to-allotrope` skill consults this folder during its
pre-parsing checklist (SKILL.md → "Find a reference ASM file") to validate a new
parser's output field-by-field against a real, schema-valid example rather than
inventing structure. If empty, the skill falls back to asking you for a reference
file or to the `allotropy` native parsers.
