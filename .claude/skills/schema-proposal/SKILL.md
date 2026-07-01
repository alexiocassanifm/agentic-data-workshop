---
name: schema-proposal
description: Use before creating any table, collection, or index from source files, when the student asks the agent to propose, design, or review a data model — a relational schema (entities, fields, keys, foreign keys) for a target like Supabase/Postgres, or a document/collection structure (embed-vs-reference calls) for a target like MongoDB. Produces a structured proposal with a rejected alternative and open questions, and stops for the student's decisions rather than building anything.
---

# Schema Proposal

## Purpose

This skill structures how you (the agent) present a data model proposal to
a student before any table, collection, or index gets created. The student
directing you is a data scientist or data engineer, not a developer — they
will never read raw logs, stack traces, or code. Everything you produce
under this skill is written in plain, everyday language, organized so a
non-technical reader can follow it and make a decision from it.

The single most important rule: **you propose, you do not decide.** A
schema proposal always ends in a list of open questions, and you stop and
wait for the student to answer them. You never pick the "obviously
reasonable" option on the student's behalf, and you never proceed to
building anything until the student has responded.

## When this applies

Use this skill whenever you are asked to propose, draft, or review a data
model from a set of source files, for either of two target paradigms:

- **Relational target** (e.g. Supabase/Postgres): the proposal covers
  entities, fields, inferred types, primary keys, foreign keys, and
  nullability.
- **Document target** (e.g. MongoDB): the proposal covers candidate
  collections and, for every relationship between entities, an explicit
  embed-vs-reference call.

If it is not clear from context which target paradigm the proposal is for,
ask the student directly before doing anything else: "Is this proposal for
a relational database with tables and foreign keys, or a document database
where I need to decide what gets embedded versus what gets its own
collection?"

## Step 1 — Read before proposing

Before drafting anything, read the actual source material in full — do not
propose a model from assumptions or from memory of a similar dataset.

- For structured files (CSVs, spreadsheets): read the columns, and sample
  enough real rows to see the actual range of values per column, not just
  the header names.
- For semi-structured files (JSON, nested logs): read at least one full
  example end to end so you understand the real nesting depth, not a
  guessed shape.
- For unstructured documents (PDFs, plain-text notes, emails, tickets):
  open a handful of examples and note what information is present in the
  text even though it has no fixed field structure.

Note anything messy or inconsistent as you go — mixed date formats, blank
values, inconsistent casing, encoding artifacts, fields that only some
records have. You will need this later when flagging open questions; do
not quietly normalize or fix any of it yourself at this stage.

## Step 2 — Draft the proposal

Write the proposal as plain prose and simple lists — no code blocks full
of raw schema syntax, no database-specific jargon left unexplained. Cover
the following four parts, in order, every time:

### Part A — Candidate entities and fields

For each entity you propose, state in plain language:

- What real-world thing it represents (e.g. "a customer company," "a
  single support ticket").
- Its candidate fields, each with an inferred type described in plain
  words ("a date," "a whole number," "free text," "one of a fixed set of
  categories") based on the values you actually sampled in Step 1, not a
  default guess.
- Which source file(s) each field comes from, including when a field is
  extracted from an unstructured document rather than read directly from
  a column (call this out explicitly — extracted values carry more
  uncertainty than a column that already exists).
- Anything messy or inconsistent you noticed in that entity's source
  data, described in plain terms (e.g. "about a tenth of the rows spell
  the country name differently than the rest").

### Part B — Relationships or embed-vs-reference calls

**For a relational target:** for every pair of related entities, state the
relationship in plain language (e.g. "one customer can have many support
tickets"), the field that would connect them, and whether that connection
should be enforced as a database-level foreign key. Flag, but do not
silently resolve, anything that affects this: what should happen if the
parent record is deleted, and whether the relationship is ever optional
(some customers may have zero related records of a given kind).

**For a document target:** for every relationship between entities, make
an explicit embed-or-reference call, and justify it against this concrete
rule: data that is small, bounded in size, and almost always read together
with its parent is a good candidate to embed; data that is unbounded,
keeps growing independently, or is often accessed on its own is a better
candidate to keep as its own collection referenced by an ID field. State
the call and the one-sentence reason for every relationship — never leave
one relationship's treatment implicit because a nearby one was covered.

### Part C — An alternative considered and set aside

State at least one genuinely different structural alternative you
considered — a different entity boundary, a different table or collection
split, a different embed/reference choice for at least one relationship —
and explain in one or two plain-language sentences why you are not
recommending it. This must be a real alternative you weighed, not a token
strawman; if there is a real tradeoff between two defensible structures,
say so honestly rather than presenting your recommendation as the only
reasonable option.

### Part D — Open questions for the student to decide

Close every proposal with an explicit, numbered list of the decisions the
student needs to make before anything gets built. A good open question:

- Names a real tradeoff in plain language (what you gain and what you
  give up with each option), not just "should I do X?" with no context.
- Is grounded in something you actually observed in the source data during
  Step 1, not a generic best-practice question.
- Covers, at minimum, whatever you were unable to resolve on your own:
  which fields should be required versus optional, how to handle any
  messy or inconsistent values you noticed, where table or collection
  boundaries should sit if more than one grouping is defensible, whether
  any field should be constrained as unique, and — for a document
  target — which of the two structural options in Part B the student
  actually wants for each relationship you flagged as a genuine tradeoff.

Do not include questions you can answer confidently yourself from the data
you already read; reserve this list for calls that legitimately require
the student's judgment.

## Step 3 — Stop and wait

After presenting Parts A through D, stop. Do not create any table,
collection, index, or constraint, and do not start writing a data-loading
pipeline. Tell the student plainly that you are waiting for their answers
to the open questions in Part D before doing anything else.

If the student answers only some of the open questions, restate which
ones are still unanswered rather than assuming a default for them and
moving on.

## Step 4 — Confirm and record the decision

Once the student has answered the open questions:

1. Repeat their decisions back in plain language, one at a time, so they
   can catch a misunderstanding before anything is built (e.g. "So: no
   contract row means no contract for that customer, not a placeholder
   row — and `contacts.email` should be unique. Confirmed?").
2. Save a short plain-language record of the final model — the entities,
   the relationship or embed-vs-reference calls, the alternative you set
   aside, and their decisions with the reasons behind them — as a note in
   the current lab folder, and tell the student where you saved it. Later
   comparison work will refer back to this record, so produce it by
   default rather than only on request; ask first only if the student has
   indicated they don't want extra files written.
3. Only after this confirmation step should you move on to actually
   creating tables, collections, or any other structure — and only
   following the decisions the student gave you, not a fresh set of
   choices invented at build time.

## What never to do under this skill

- Never present a single option as if there were no alternative worth
  mentioning.
- Never resolve an open question yourself and only mention it in passing
  after the fact — surface it before you build, not after.
- Never show raw error output, stack traces, or unexplained technical
  syntax to the student; translate anything technical into plain
  language, including when reporting problems you hit while reading the
  source files.
- Never treat "the student didn't object" as the same thing as "the
  student decided." If they haven't explicitly answered an open question,
  it is still open.
