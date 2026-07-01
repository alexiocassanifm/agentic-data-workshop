---
name: pipeline-verify
description: Use once a population or sync pipeline has finished loading corpus records into any destination store (Supabase/Postgres tables, MongoDB collections, Qdrant points, or a pgvector table) to check what actually landed against the source files and report counts, a field-by-field sample, and any discrepancies in plain language.
---

Run this after a pipeline has already populated, or re-populated, a
destination store from the shared corpus. The job is to tell the student,
in plain language and with concrete numbers, whether the destination
actually matches the source — never a bare "looks good" or "looks fine."
Work through the steps below in order, and do not skip to a summary
verdict without doing the counting and sampling first.

## 1. Establish the source ground truth, freshly

For every entity in scope for this verification, count the actual source
records right now — do not reuse a number quoted earlier in the session or
recalled from a README.

- CSV files: count data rows, excluding the header.
- PDF, text, or JSON files: count files actually present in the folder
  right now.
- If an entity's source is split across more than one file (for example, a
  contract's terms live in a PDF, but which customers are even eligible
  for one is determined by a plan tier in a CSV), count it the way it
  genuinely appears in the source, not by assumption.

State each source count out loud before moving on — for example, "180
customer rows in customers.csv" or "500 support ticket files in
support-tickets/."

## 2. Confirm where each entity actually landed

Before comparing counts, confirm — from the schema or model that was
actually chosen and built earlier in this lab, not a default assumption —
where each source entity's data now lives in the destination:

- **Relational store:** which table, and whether the entity is one row
  per source record or several (subscription history, for instance, is
  legitimately more rows than customers).
- **Document store:** which collection, and whether the entity is its own
  document, an array or field embedded inside a parent document (contacts
  embedded inside a customer document, for example), or intentionally not
  present as a separate structure at all.
- **Vector store (Qdrant or pgvector):** which collection or table holds
  the entity, and confirm each record is a point with a payload (Qdrant)
  or a row with a vector column plus metadata columns (pgvector), not a
  plain copy of the source text.

Also list, one line each, every transformation the student explicitly
decided on during modeling: reformatted dates, canonicalized country
values, the ID used as the join key, chunking rules, the embedding model
and its vector dimensionality. These declared rules are what step 5 uses
to tell an intended change apart from a real error. If no record of these
decisions is available, ask the student directly what was decided rather
than guessing.

## 3. Compare counts, entity by entity

For each entity in scope:

- Get the current count in the destination: row count in a table, document
  count in a collection (including embedded sub-records, counted wherever
  they actually live), point count in Qdrant, row count in pgvector.
- Compare it to the fresh source count from step 1.
- If it matches exactly, record a pass for that entity.
- If it doesn't match, check whether the gap is explained by a
  cross-entity coverage fact that was already known before the pipeline
  ran — not every customer has a contract, not every customer has a
  usage-log file, a subscriptions table legitimately has more rows than
  customers. That kind of explanation only covers a gap between two
  *different* entities (customers versus contracts, customers versus
  usage logs); it never covers a shortfall between the source count and
  the destination count for the *same* entity. If a source count and its
  own destination count disagree — 500 support ticket files but 480 rows
  in the destination, 50 contract PDFs but 47 contract rows — that is
  always a real discrepancy to investigate (a parse failure, a dropped
  record during load), never something to explain away as "not every
  customer has one."
- When the gap is a genuine cross-entity coverage fact, state the reason
  in one plain sentence and record it as an explained gap: not a silent
  pass, and not an unexplained failure.
- When the gap has no such cross-entity explanation, record it as a real
  discrepancy to report in step 6.

Never collapse everything into a single combined total when more than one
entity is in scope. Give a separate count comparison for each entity.

## 4. Sample records at random and compare field by field

- Pick between 5 and 10 source records at random — genuinely randomize the
  selection, not the first N rows or files encountered — spanning more
  than one entity if more than one is in scope.
- For each sampled source record, look it up in the destination using its
  clean, prefixed ID (`customer_id`, `contact_id`, and so on) as the join
  key. These IDs are never messy in the source corpus, so a failed lookup
  means the destination record is genuinely missing, not that the ID
  needs cleaning first.
- Lay the source and destination values side by side, field by field.
- For an entity stored as a vector, "field by field" means: confirm the
  stored metadata or payload (source ID, `customer_id`, and whichever
  type-specific fields were chosen — priority and status for a ticket, the
  account executive for a note, the subject line for an email) matches
  the source file, and confirm a vector is present with the dimensionality
  recorded in step 2. Do not attempt to diff the vector's numbers against
  the source text — that comparison has no plain-language meaning.
  Instead, confirm the source text that was supposed to be embedded is
  actually the text in the file.

## 5. Classify every difference found

For every field-level difference from step 4, and every count gap from
step 3, classify it as exactly one of:

- **Expected transformation** — the destination value differs from the
  source in a way the student explicitly decided on in step 2 (a
  reformatted date, a canonicalized country name, a normalized casing).
  State the specific rule that explains it.
- **Missing** — a source record has no matching destination record at
  all.
- **Duplicated** — the same source record's ID appears more than once in
  the destination in a way the chosen model doesn't account for. A
  customer legitimately having several subscription history rows is not a
  duplicate; the same subscription ID inserted twice is.
- **Mismatched** — a destination value differs from the source and
  doesn't match any declared transformation rule: a company name still
  showing its original encoding artifact when the plan was to fix it, a
  number stored as text instead of a number, a value that is simply
  wrong.

Do not let "expected transformation" become a place to hide a real
problem. If the destination value doesn't actually match what the
declared rule says it should produce, classify it as mismatched instead.

## 6. Report the result in plain language

Produce a report the student can read without any database or technical
background — plain language and concrete numbers throughout, never raw
query output or error text. Structure it as:

- **One line per entity**, giving the source count, the destination
  count, and the outcome — for example: "Customers: 180 in the source
  file, 180 in the customers table — match." or "Usage logs: 180
  customers, 177 usage-log documents — 3 customers have no usage history
  yet, which is expected." or "Contacts: 350 in the source file, 347 in
  the destination — 3 missing, see below."
- **The sample comparison**: for each of the 5–10 sampled records, a
  short side-by-side of the fields checked, each flagged as matching, an
  expected transformation, or a problem.
- **A discrepancy list**: every missing, duplicated, or mismatched item
  found, each with its ID, the concrete source value and destination
  value, and a one-sentence plain-language explanation of what's wrong.
- **A final verdict**, entity by entity and overall: PASS if every
  entity's counts reconcile (matched or explained) and every sampled
  record checks out; FAIL otherwise, naming exactly which entities and
  records need attention next.

If this verification is being run against a dashboard's on-screen numbers
rather than a database directly, apply the same method: treat each chart
or figure as the destination, run the query that should produce that
number fresh against the database, and report it the same way — value
compared, match or mismatch, plain-language explanation, and an explicit
verdict per number.
