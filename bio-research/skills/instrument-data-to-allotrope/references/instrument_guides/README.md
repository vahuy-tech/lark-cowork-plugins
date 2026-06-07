# Instrument-specific guides

Add one Markdown note per instrument/vendor here (e.g. `agilent-openlab.md`,
`thermo-chromeleon.md`) capturing its raw export quirks: file format, header
layout, units, timezone handling, and known field-mapping gotchas.

The `instrument-data-to-allotrope` skill checks this folder during its
pre-parsing checklist (SKILL.md → "Review instrument-specific guide") before
writing a custom parser. Empty is fine — it just means no instrument-specific
guidance exists yet; the skill proceeds from the ASM schema overview.
