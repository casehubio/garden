---
id: PP-20260526-6d39e5
title: "Opaque cross-module identifiers must be stored unchanged — never parsed by the receiver"
type: rule
scope: universal
applies_to: "Any module that receives an identifier string whose format is defined and owned by another module"
severity: important
violation_hint: "A module parses or splits an identifier string to extract sub-components,
  using a format defined in a different module — coupling the receiver to the sender's
  internal structure"
created: 2026-05-26
---

When a module receives an identifier whose format is owned by another module (e.g. a
caller reference, correlation ID, or external system reference), it must store and
propagate it unchanged. Parsing the internal structure — extracting sub-components,
splitting on delimiters, matching against a regex — couples the receiving module to a
format it does not own. If the owning module renames, restructures, or extends the
format, the parsing code in the receiver silently breaks or produces wrong output with
no compile-time signal. The receiving module's responsibility is to store the reference
faithfully and document that the format is opaque to it. Consumers who need the
constituent parts must obtain them from the owning module's API or parser, not by
re-implementing the format themselves.
