# Lab 3 — Semantic Search: Qdrant and pgvector

## Objective

In Lab 1 you designed and enforced a relational schema on Supabase. In Lab
2 you modeled the same corpus with no schema at all, in MongoDB. This lab
introduces a third paradigm, and it is not "the same corpus again, another
tool": there is no schema to design here. The free-text sources you've
mostly set aside so far — support tickets, sales notes, onboarding emails
— become embeddings, numeric vectors a database can compare for closeness
of *meaning* rather than exact value. A good query for "customers upset
about login problems" should surface tickets, notes, and emails that never
use those exact words.

You generate those embeddings explicitly, through your agent, using Voyage
AI — not by leaning on a black-box default embedder built into a database
tool — so you see and control the one step that makes semantic search work
at all.

Then you run the exercise twice, on the same search question, in two
architecturally different places: Qdrant, a database built for nothing but
vectors, reached through its official, close-to-batteries-included MCP
server; and pgvector, an extension that turns the very same Postgres
project from Lab 1 into a vector store, where you control the embeddings,
the index, and the query directly. Neither is the automatically correct
answer. By the end of this lab you will have loaded the same vectors into
both, run the same queries against both, and be ready to argue, from what
you actually observed, which one you would put into production for this
use case.

> **Method note.** Qdrant's official MCP server is close to
> batteries-included: point it at your cluster, and a single tool call
> embeds text and stores it, creating the collection for you if needed.
> pgvector is the opposite: you enable an extension, choose an index type,
> write the table, and construct the similarity query yourself, right
> alongside the relational data from Lab 1. Neither is "the way" to do
> semantic search — a dedicated vector database earns its place when
> vector search is a first-class, high-volume workload; extending the
> database you already operate earns its place when vectors are one more
> thing living next to data you already query relationally. Deciding
> which is true for Larkspur is the actual point of doing this lab twice.
> The written verdict you produce at the end is this lab's own
> comparison — a counterpart to the Bonus lab's Supabase-vs-MongoDB
> artifact, not an input to it.

## Duration

Approximately 90 minutes:

- ~10 min: connection check and picking what to embed
- ~15 min: generating embeddings explicitly via Voyage AI, on a small pilot batch
- ~10 min: the Qdrant ingestion-path decision, below
- ~20 min: loading both destinations (Qdrant and pgvector) and scaling to the full corpus
- ~15 min: pipeline verification on both sides
- ~20 min: side-by-side semantic queries and the closing decision-point writeup

## Prerequisites

- **Lab 0 completed**, specifically: a live Supabase MCP connection
  pointing at your Lab 1 project, a live Qdrant Cloud MCP connection with
  write access (not read-only), and a Voyage AI API key your agent can
  use. See [`../lab-0-setup/README.md`](../lab-0-setup/README.md).
- **Lab 1 completed.** This lab writes pgvector data into the same
  Supabase/Postgres project you built in Lab 1 — the new vectors will
  live alongside the `customers`, `contacts`, and `subscriptions` tables
  you already created. See
  [`../lab-1-relational-supabase/README.md`](../lab-1-relational-supabase/README.md).
- **Lab 2 (MongoDB) is not required for this lab.** Nothing here depends
  on what you built there, though it's assumed you've done it as part of
  the same day.
- The shared corpus, described in full in
  [`../corpus/README.md`](../corpus/README.md). This lab uses only the
  free-text sources:
  - `corpus/support-tickets/` — 500 files, `TCK-#####.txt`
  - `corpus/sales-notes/` — 150 files, `SN-####.txt`
  - `corpus/emails/` — 124 files, `EML-####.txt`

  774 documents in total. The customer CSVs, contract PDFs, and usage-log
  JSON from Lab 1 and Lab 2 are not used in this lab.
- No prior experience with vector databases, embeddings, or SQL is
  required. Everything below is done by describing what you want to your
  agent.

## Guided steps

1. **Check both connections.** Ask your agent to run the
   `mcp-health-check` skill (or the manual equivalent) and confirm: the
   Supabase MCP connection points at your Lab 1 project, the Qdrant MCP
   connection points at your Qdrant Cloud cluster and has write access,
   and a Voyage AI API key is actually available to it.

2. **Pick what to embed, and start small.** Ask your agent to list the
   support tickets, sales notes, and emails folders with their counts
   (774 files combined), then propose a small pilot batch — for example
   15–20 tickets, 10 notes, and 8 emails — to prove the whole pipeline
   end to end before committing to embedding all 774.

3. **Settle chunking and metadata.** Ask your agent to confirm that each
   file is short enough to embed whole, as a single chunk, without
   splitting (flag anything unusually long), and to propose the small set
   of metadata fields to carry alongside every vector: source type,
   source ID, `customer_id`, and one or two useful fields per type
   (priority and status for tickets, account executive for notes, subject
   line for emails).

4. **Pick and record the embedding model.** Ask your agent to check
   Voyage's current documentation for the recommended general-purpose
   text embedding model, and to report back the exact model name and the
   vector dimensionality it produces. Write that number down — you'll
   need it to set up both destinations correctly.

5. **Generate embeddings for the pilot batch.** Ask your agent to call
   the Voyage embeddings API directly on the pilot documents — not
   through any database's built-in embedder — batching many documents
   into each API call rather than sending one call per document. Ask it
   to confirm the number of documents embedded and the resulting vector
   dimensionality match what you expect.

6. **Stop here and work through Part A of the decision point below**
   before your agent writes a single vector into Qdrant.

7. **Ask your agent to create the Qdrant collection** — sized to the
   dimensionality from step 4, with cosine distance — and load the pilot
   vectors and metadata into it, using the ingestion path you chose in
   Part A.

8. **Ask your agent to enable pgvector** on the Lab 1 Supabase project
   (if it isn't already), create a new table for these embeddings sized
   to the same dimensionality, load the pilot vectors and metadata into
   it, and then add an appropriate index. Ask it to explain briefly why
   it picked the index type it did — and if it picks an index type that
   trains on existing data (IVFFlat), confirm it's building that index
   after the rows are loaded, not on an empty table.

9. **Once both pilot loads check out, scale up to the full 774
   documents**, batching the Voyage calls again to stay within its rate
   limits.

10. **Ask your agent to verify what actually landed**, on both sides,
    against the source files. In Claude Code this is what the
    `pipeline-verify` skill is for; on other tools, ask directly for the
    same comparison — counts against the source folders, a field-by-field
    spot check on a handful of records, and a clear pass or fail with
    concrete numbers.

11. **Run the comparison queries.** Ask your agent to run the same
    semantic query against both backends — something like "customers
    frustrated about single sign-on breaking their login" — and present
    the top results from each side by side, with a short snippet and a
    similarity score. Then ask it to run a second, cross-genre query that
    deliberately spans support tickets and sales notes (the corpus
    reuses SSO-frustration vocabulary across both), and check whether
    each backend surfaces more than one document type, not just the most
    literal match.

12. **Ask your agent to summarize what it noticed** operating each
    backend during this lab — which one required more explicit decisions
    from you, which one hid more machinery, how latency and result
    quality compared. This is the raw material for Part B of the
    decision point below.

## Explicit decision point

### Part A — while building: how do your Voyage vectors actually get into Qdrant?

Ask your agent, before it stores anything: if you ask it to store a
document's text in Qdrant using the store tool, will it use the Voyage
embedding you already generated, or compute its own? The honest answer
for the official Qdrant MCP server is the latter — its store tool always
embeds text itself, with its own built-in model, and there is no way to
hand it a vector you already computed. That gives you two real options;
decide before your agent writes a single point:

- **Option 1 — let Qdrant embed for you.** Ask your agent to store each
  document's text directly through the Qdrant MCP tool and accept the
  vectors it produces with its own built-in embedder. This is genuinely
  the batteries-included path and the fastest way to a working
  collection — but it means Qdrant's vectors are not the Voyage vectors
  sitting in pgvector. The two backends will be answering the same query
  in two different embedding spaces, so a side-by-side comparison tells
  you something about the two systems in general, but not a clean
  apples-to-apples test of "same embeddings, different store."
- **Option 2 — keep control of the embedding.** Ask your agent to write
  the Voyage vectors you already generated directly into Qdrant, going
  around the store tool's own embedding step (it does this by calling
  Qdrant's API directly with the vector, rather than handing it raw
  text). This preserves a true apples-to-apples comparison with pgvector,
  at the cost of stepping outside the tool's simplest path. If you choose
  this, you must also retrieve using that same direct path — not the
  Qdrant MCP's own query tool — for every query in this lab. Mixing an
  externally supplied vector on the way in with the tool's own built-in
  embedder on the way out compares two different embedding spaces, and
  will produce meaningless or outright broken results.

State out loud to your agent which option you're taking, and why, before
letting it proceed.

### Part B — after you've seen both: where should Larkspur's semantic search actually live?

Once both stores are loaded, verified, and queried, write down — in a
scratch note your agent keeps for you, the same way you did in Lab 2 —
one paragraph answering: for Larkspur's own support, sales, and email
corpus, would you recommend building semantic search on top of the
Postgres project you already operate (pgvector), or standing up Qdrant as
a dedicated store? Ground the answer in at least two concrete things you
personally saw in this lab, not database theory in the abstract — for
example, how much you had to configure by hand on each side, how the two
result sets actually differed on the same query, how filtering by
metadata (`customer_id`, priority, source type) felt in each, or how much
control you had over the index and the embedding itself. There is no
single correct answer here; the discipline is citing your own
observations, not defaulting to whichever backend felt more polished on
first contact.

## Verification

**In the Qdrant Cloud UI:**

- Open your cluster, go to Collections, and open the collection you
  created. Confirm the points count matches the number of documents you
  embedded (774, or your pilot batch's count if you haven't scaled up
  yet).
- Check the collection's configured vector size and distance metric. If
  you chose Option 2 in Part A, this should match the Voyage
  dimensionality you recorded earlier, with cosine distance. If you chose
  Option 1, the collection was sized and created automatically by
  Qdrant's own built-in embedder, using its own model's dimension — that
  is expected, not a sign anything went wrong; the Voyage-dimension check
  applies to the pgvector side only in that case.
- Open a few individual points and confirm the payload holds the
  metadata you decided on (source type, source ID, `customer_id`, and so
  on) and reads back correctly for a document you recognize.

**In the Supabase UI:**

- Table Editor: confirm the new embeddings table exists with the
  expected row count, and Database → Extensions shows `pgvector` enabled.
- Open the SQL editor (or ask your agent to do it for you) and look at
  one row: confirm the vector column is not null and has the length you
  expect for the Voyage model you used.

**Via your agent:**

- Ask it to count points in Qdrant and rows in the pgvector table and
  compare both to the number of source files you embedded — the same
  comparison the `pipeline-verify` skill runs — and report a clear pass
  or fail with numbers, not "looks about right."
- Ask it to spot-check a handful of records by source ID against the
  original file, confirming the metadata matches (for example, a
  ticket's priority and status in the stored payload match what's
  actually written in the `.txt` file).
- Ask it to re-run the two semantic queries from the guided steps against
  both backends one more time, and present the results side by side,
  exactly as you'll want them for the decision-point writeup above.

## Known pitfalls

- **Voyage's rate limit, hit the hard way.** Without a payment method on
  file, Voyage's API throttles to a handful of requests per minute.
  Embedding 774 documents one call at a time at that rate can take hours.
  Always ask your agent to batch many documents into each API call, and
  pilot on a small subset before committing to the full corpus.
- **Vector-space mismatch between store and retrieve.** If you chose
  Option 2 in the decision point (writing Voyage vectors directly into
  Qdrant), you must also query directly, the same way, every time.
  Falling back to the Qdrant MCP's own query tool "just for a quick
  check" re-embeds your query text with its own built-in model and
  compares it against vectors from a different embedding space — results
  will look wrong, or the query will simply fail, for reasons that have
  nothing to do with semantic search itself.
- **A read-only MCP connection quietly removes the write tools.** If
  your Qdrant or Supabase MCP connection from Lab 0 was configured
  read-only, the tools needed to create a collection or table, or write
  points or rows, may not be available at all, or will fail partway
  through. Confirm write access on both sides before starting.
- **Dimension or distance mismatch.** The pgvector column always needs
  the exact dimensionality your Voyage model actually produces. The
  Qdrant collection needs it too, but only if you chose Option 2 in Part
  A — under Option 1, Qdrant sizes its own collection to its built-in
  model's dimension, and that's the expected, correct outcome, not a
  bug. Whichever path you're on, keep the distance metric (cosine)
  consistent everywhere you compare results, or the comparison isn't
  really comparing the same thing.
- **Mismatched chunking between the two backends.** If one backend ends
  up with whole files as chunks and the other with paragraph-split
  chunks, the two result sets no longer describe the same units of text.
  Keep chunking identical on both sides.
- **Cluster or project idle suspension.** Free-tier Qdrant Cloud clusters
  and Supabase projects can pause themselves after roughly a week of no
  activity. If you're repeating this lab at home some days after Lab 0,
  check both dashboards are awake before asking your agent to do
  anything.
- **Treating the pilot batch as the final answer.** The pilot exists to
  catch mistakes cheaply. Make sure your agent actually scales up to the
  full 774 documents — or explicitly tells you why it didn't — before you
  run the comparison queries; a comparison built on a partial, arbitrary
  slice of the corpus isn't a fair one.

## Multi-tool notes

### Using Codex CLI or opencode instead

Neither Codex CLI nor opencode has Claude Code's skill mechanism, so
there's no `mcp-health-check` or `pipeline-verify` skill to invoke by
name. Every outcome in this lab is reachable by asking directly:

- Instead of `mcp-health-check`, ask your agent to check its Supabase and
  Qdrant connections and report, in plain language, what it can currently
  see on each (tables or collections, row or point counts, read versus
  write access).
- Instead of `pipeline-verify`, ask: "Compare the number of documents I
  embedded against the point count in Qdrant and the row count in the
  pgvector table, spot-check a handful of records field by field against
  the original files, and report pass or fail with concrete numbers."
- Confirm both your Supabase and Qdrant MCP connections were configured
  for whichever tool you're using back in Lab 0 — Codex CLI and opencode
  each use their own configuration file format for MCP servers, and the
  connection step differs slightly between tools, but every step in this
  lab works identically once both connections are live.
- Option 2 in the decision point (writing Voyage vectors directly into
  Qdrant, bypassing the store tool's own embedding step) is not a
  Claude-Code-specific technique — it's your agent calling Qdrant's API
  directly on your behalf, which works the same way regardless of which
  agent is driving.
