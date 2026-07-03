---
name: backend-compare
description: Use when building the Bonus lab's self-contained HTML artifact that scores Supabase (Lab 1) against MongoDB (Lab 2) on the student's own weighted criteria and concrete, cited observations from those two labs — not for the Qdrant-vs-pgvector comparison built inside Lab 3.
---

# Backend Compare

This skill builds the Bonus lab's closing artifact: a single self-contained
HTML file that scores Supabase (relational, Lab 1) against MongoDB
(document, Lab 2) on criteria the student defined themselves, with every
score traced to something concrete that actually happened in those two
labs on the shared Larkspur Flow corpus.

Scope: this skill only compares Supabase (Lab 1) against MongoDB (Lab 2).
It has nothing to do with the separate Qdrant-vs-pgvector comparison built
directly inside Lab 3 — do not pull that comparison into this artifact,
and do not let this skill be invoked as a stand-in for it.

This skill's job is enforcement first, generation second. Two gates must
pass, in order, before any backend is scored. Do not skip or reorder them,
and do not let the student talk you past them by asking to "just see how
it looks so far" — that request is itself the failure mode this skill
exists to prevent.

## Gate 1 — criteria and weights must already exist, and must be usable

Ask the student whether they already have a written note listing their
final evaluation criteria and numeric weights (the decision point in the
Bonus lab README asks them to produce this before involving you in any
scored way).

- If no such note exists yet: stop here. Do not discuss, score, or rank
  either backend. Tell the student, in plain language, that you need
  their criteria and weights first, and that you can help them brainstorm
  a longlist of candidate dimensions (things like: looking up a customer's
  full history in one place, keeping billing and contracts consistent,
  handling the messy `customers.csv` fields, modeling the nested
  `usage-logs`, how naturally a realistic business query can be written,
  how much friction came up while building each pipeline) — but make clear
  this is a menu to choose from, not a scored verdict, and that picking
  the final 4–6 criteria and assigning weights is their call to make, not
  yours.
- Once the student has a candidate list, check it is actually usable
  before accepting it:
  - **4 to 6 criteria.** Fewer than 4 is usually too coarse to be
    informative; more than 6 tends to dilute the weights past the point of
    a meaningful comparison. Flag it if the count is outside that range
    and ask whether they want to trim or expand.
  - **Numeric weights that sum to one fixed, stated total** (100 is a
    reasonable default to suggest). Reject labels like "High / Medium /
    Low" — they cannot be multiplied into a single comparable score. Ask
    the student to convert them to numbers before continuing.
  - If weights don't sum to the stated total, point out the gap and ask
    the student to adjust — do not silently rescale their numbers for
    them.

## Gate 2 — ask, explicitly, whether this was decided before or after seeing any comparison

This question must always be asked out loud, every time, even if the
student already hands you a criteria note unprompted — do not assume it
was done in the right order just because a note exists.

Ask plainly: *"Did you write these criteria and weights before you saw any
scored comparison, ranking, or side-by-side result for Supabase versus
MongoDB — or after?"* (Having already built both backends in Lab 1 and Lab
2 is fine and expected; the question is specifically about whether a
*scored comparison* between them existed yet when the weights were fixed.)

- **If before:** proceed. Note in the artifact, plainly, that criteria and
  weights were fixed before any scoring took place.
- **If after:** stop and explain, in plain language, why this matters:
  criteria chosen after seeing which backend comes out ahead tend to
  rationalize a decision already made rather than test one — the numbers
  will look like an evaluation but won't function like one. Give the
  student two options and let them choose:
  1. Go back and write a fresh set of criteria and weights now, before you
     show them anything scored, treating the ones they had as void; or
  2. Proceed anyway, but only if they explicitly say so — and if they do,
     the artifact you build must carry a clearly visible caveat at the top
     stating that criteria were fixed after seeing results, so anyone
     reading it later knows to discount the exercise's evaluative claim
     accordingly.
  Do not proceed silently either way — record which option was chosen and
  reflect it in the artifact.

## Gathering evidence

Once both gates pass, assemble two evidence briefs — one for each
backend — before scoring anything. If the student or an earlier part of
the conversation already supplied these, use them; otherwise gather them
yourself by inspecting the live backends and ask the student to confirm
they still match what they remember from Lab 1 and Lab 2.

Each evidence brief needs the same kind of concrete material, at
comparable depth on both sides:

- **Lab 1 (Supabase) brief:** the schema as it actually exists right now —
  tables, keys, relationships and constraints; the alternative structure
  that was considered and rejected during schema design, and the stated
  reasoning; one or two queries that were genuinely natural to write; one
  that was awkward or needed a workaround; and any concrete
  pipeline-verification results or friction points from Lab 1 (row counts,
  parse failures, constraint rejections).
- **Lab 2 (MongoDB) brief:** the collections and document shapes as they
  actually exist right now; the embed-vs-reference decisions that were
  actually made (including the alternative considered and rejected); one
  or two queries that were natural, one that was awkward; index decisions
  and what they made fast or possible; and concrete pipeline-verification
  results or friction points from Lab 2.

If one brief is noticeably thinner than the other — fewer concrete
details, no rejected alternative, no specific query — say so plainly to
the student before scoring anything, and ask them to help you deepen the
thin side (or re-inspect the live backend yourself) rather than let a
shallow brief quietly bias every score toward the backend with the richer
writeup.

If the connections are stale or the student is unsure the evidence still
reflects the current state of either backend, re-check the live tables and
collections (row/document counts, a quick inspection of structure) before
relying on recollection from an earlier conversation.

## Scoring — every cell cites a specific observation, or it doesn't get scored

For each criterion, score both Supabase and MongoDB on a consistent 1–5
scale (1 = poor fit for this criterion on this corpus, 5 = strong fit).
For every single score:

- Attach one specific, concrete citation drawn from that backend's
  evidence brief: a table or collection name, an actual constraint or
  index, a specific query and how it read, a concrete number from a
  pipeline-verification check, a specific friction point that came up
  while building it. A citation must point at something that actually
  happened in this workshop's Lab 1 or Lab 2 on this corpus — never a
  generic claim about Postgres or MongoDB in general that could have come
  from a blog post. State each citation as a plain-language description of
  the concrete fact (name the table, constraint, or collection; describe
  what the query did; give the number) — never paste a raw error message,
  stack trace, or code/log excerpt into the artifact itself.
- If you cannot find or construct a concrete citation for a cell, do not
  invent one. Either ask the student for the missing observation, or leave
  that cell explicitly marked as not scored (state plainly why — "no
  observation available for this criterion" — rather than filling it with
  a plausible-sounding guess).
- Do not reuse the exact same citation for more than one or two cells on
  the same backend. If you find yourself needing to, that is a sign the
  evidence brief for that backend is too shallow, not that the backend
  genuinely ties across every criterion — say so to the student instead of
  papering over it.

Compute, for each backend, the weighted total: for every criterion,
multiply its score by its weight, then sum across all criteria. The
recommendation in the final artifact must follow arithmetically from this
table — it must not introduce any new factor, consideration, or claim that
was never one of the student's stated criteria.

## Building the artifact

Produce one self-contained HTML file: all styling inline or in a single
embedded `<style>` block, no external stylesheets, fonts, scripts, or CDN
references, so it opens correctly in a browser with no network connection.
Include, in this order:

1. A short header stating what is being compared (Supabase from Lab 1 vs.
   MongoDB from Lab 2, on the Larkspur Flow corpus) and, prominently and
   impossible to miss, whether criteria and weights were fixed before or
   after seeing any comparison (including the caveat text from Gate 2 if
   applicable).
2. A table of the student's criteria and their weights, exactly as given.
3. The full scored table: one row per criterion, with Supabase's score and
   citation and MongoDB's score and citation side by side, plus the
   weighted contribution of each cell.
4. A totals row with each backend's weighted total.
5. A closing recommendation paragraph naming which backend the totals
   favor for this use case, referencing only the criteria already scored
   above — no new argument smuggled in at the end.

Save the file inside the Bonus lab's folder (`bonus-compare-backends/`),
named `comparison.html` unless the student asks for a different name, and
tell the student where you saved it.

## Before calling it done — reconcile and report back in plain language

Before presenting the artifact as finished, check it yourself and report
the results to the student in plain, non-technical language (no raw logs,
no stack traces, no code):

- Count the criteria the student set, and count the citations in the
  artifact. Every criterion scored against every backend needs its own
  citation, so these should reconcile one-to-one, with no cell left
  uncited (aside from any cell you explicitly marked as not scored, which
  you should call out by name).
- Confirm no single observation was reused as the citation for more than
  one or two rows on the same backend; flag it plainly if it was.
- Re-read the closing recommendation on its own and confirm it follows
  from the scored table above it and doesn't introduce anything that
  wasn't one of the stated criteria.
- Tell the student, in a few plain sentences: where the file was saved,
  which backend the totals favor and by how much, whether criteria were
  fixed before or after seeing results, and whether any cells were left
  uncited or any evidence brief looked thin. Invite them to open the file
  and read it end to end before treating it as final.
