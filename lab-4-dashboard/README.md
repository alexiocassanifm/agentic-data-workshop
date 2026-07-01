# Lab 4 — Dashboard

## Objective
Turn the data now sitting in your databases into a small set of business-relevant
charts, and confirm that every number on screen matches what is actually stored.
The point of this lab is not "make a pretty chart" — it is picking, before any
chart exists, which business questions matter enough to answer, and then
directing the agent to build exactly those and nothing else. By the end you
will have either a small running web dashboard or a single self-contained
HTML file, plus direct proof, via a database query, that its numbers are
correct.

## Duration
About 60 minutes: roughly 10 minutes deciding what to measure, 35–40 minutes
building and iterating with the agent, and 10–15 minutes verifying the result
against the database.

## Prerequisites
- The shared corpus at [`../corpus/`](../corpus/) — for reference only, you
  will not touch it directly in this lab.
- A populated Supabase project from [Lab 1](../lab-1-relational-supabase/README.md).
  This lab reads from the relational store you built there: customers,
  contacts, subscriptions, and whatever else your Lab 1 schema captured.
  Business metrics like MRR, churn, and counts by segment are naturally SQL
  aggregations, which is why this dashboard is scoped to the Supabase data
  rather than to MongoDB or the vector stores.
- By this point in the day you will also have been through Lab 2 (MongoDB)
  and Lab 3 ([semantic/vector search](../lab-3-vector-search/README.md)) on
  the same corpus. You do not need their output for this lab, but it is the
  same customers and subscriptions you have been comparing across backends
  all day.
- Confirm your agent still has a live MCP connection to that Supabase
  project. Ask it to check — the mcp-health-check skill does this, or you
  can just ask directly — it should be able to list your tables and report
  non-zero row counts.

## Guided steps
1. Do not ask for any chart yet. First decide which business questions this
   dashboard needs to answer — see "Explicit decision point" below — and
   write your final list down before moving on.
2. Tell your agent the list of questions/metrics you settled on. For each
   one, ask it to propose which table(s) and aggregation answer it, and what
   chart type (single number, bar, line, table) best represents it. Review
   this proposal before anything gets built; it is a plan, not a preview.
3. Choose which output you are building, based on the time you have left:
   - A small web app that queries Supabase live, each time the page loads.
   - A single self-contained HTML file that the agent queries Supabase to
     build once, with the results (and a small charting library) embedded
     directly in the file, so no live connection is needed to view it
     afterward.
   Tell your agent which one you want; do not let it default to whichever
   is easiest for it to produce.
4. For the web app variant: ask your agent to scaffold a minimal app, wire
   it to Supabase using your project's connection details — your Supabase
   project URL and access token — describing these to your agent by name
   rather than pasting them into a file it will show you, and run the app
   locally so you can view it in a browser.
   For the HTML artifact variant: ask your agent to run the queries for each
   metric now and generate one HTML file containing the results and the
   charts, then open that file in a browser.
5. Ask your agent to label every chart and number with the business
   question it answers, in plain language (for example "Active
   subscriptions" or "MRR by plan tier"), not with raw column or table
   names.
6. Review the first version together. Pick one or two concrete things to
   change — a grouping, a sort order, a missing filter, a second axis — and
   ask the agent to make that specific change. Repeat once or twice; resist
   the urge to keep adding new metrics at this stage.
7. Once you are satisfied with the layout, move to Verification below
   before calling the lab done.

## Explicit decision point
Before step 2 above, decide on 3 to 5 business questions this dashboard will
answer. Do this yourself, on paper or out loud with your agent as a
sounding board — do not ask the agent to propose the list and then simply
approve it. For each question, be able to say what decision it would inform
and roughly which data it depends on.

A menu to choose from — adapt it to whatever actually ended up in your
Supabase tables, and mix types freely:
- **A single headline number**: total current MRR; count of active
  customers; churn rate (churned customers versus active customers).
- **A breakdown by category**: MRR or customer count by plan tier;
  customers by industry; customers by country; customers by company size
  band.
- **A trend over time**: signups by month or quarter; MRR trajectory, if
  your schema tracks subscription history.
- **A risk or health signal**: proportion of customers on trial versus
  paying; count of contracts approaching renewal, if your Lab 1 schema
  modeled contract terms.

Deliberately pick a mix — at least one single number, one category
breakdown, and one trend — so the dashboard exercises more than one chart
type. Write your final 3–5 down before telling the agent to build anything.

## Verification
In the provider's UI: open your Supabase project's Table Editor, apply the
same filter as one of your dashboard metrics (for example, filter the
`subscriptions` table to `status = active` and read the row count off the
UI), for at least two of your metrics. No query typing required. If a
metric needs an aggregation the Table Editor cannot show directly (a sum
or a group-by), ask your agent for the exact query behind that number and
paste it into the SQL editor instead. Compare the number you get to the
number on the dashboard.

Via the agent: ask it to run a direct check comparing every dashboard
number against a fresh query against Supabase, and to report any mismatch
with both numbers side by side, not just "looks correct." This is the same
comparison the pipeline-verify skill performs after a population pipeline —
here you are pointing it at the dashboard instead of at the database alone.

Two numbers worth checking specifically, because they are easy to get
wrong:
- **Active subscriptions**: a direct `status = 'active'` count should
  return 156. If a chart or KPI reads a different number, it is almost
  certainly counting all 200 subscription rows rather than just the
  current ones.
- **Total customers**: 180. If a customer-count tile reads higher than
  that, it is counting subscription rows or contacts instead of distinct
  customers.

If any number does not match, ask the agent which query produced the
on-screen number and which query produced the verification number, then
have it explain the discrepancy before you accept the dashboard as correct.

## Known pitfalls
- **Double-counting historical subscription rows.** `subscriptions.csv` has
  200 rows but only 156 are currently active; the other 44 are churned,
  trial, or `upgraded` — a superseded historical row for a customer who has
  since changed plans (about 19 customers have more than one row). Any
  `SUM(mrr_usd)` or `COUNT(*)` grouped by plan that does not filter to
  `status = 'active'`, or that does not take only the latest row per
  customer, overstates revenue and customer counts.
- **Usage-log coverage gap.** Only 177 of the 180 customers have a
  usage-log file — 3 very recently signed customers have no usage history
  yet. Any usage-based metric, if your schema pulled usage data into
  Supabase, will be missing those 3 rows. That is a real, expected join
  gap, not a bug, but it is worth a one-line footnote on the dashboard
  rather than a silent omission.
- **Inherited messiness from customers.csv.** If `country` or `industry`
  values were not normalized during Lab 1, or were normalized in a way you
  do not remember, a "customers by country" chart can fragment into
  near-duplicate categories, such as `"US"` and `"United States"` as
  separate bars. Do not let the agent silently merge categories it invents
  on the spot while building the chart — either fix it at the source in
  Lab 1's tables, or flag the fragmentation visibly.
- **Charting before deciding.** If you let the agent propose metrics and
  charts before you have picked your questions, review time gets spent
  tuning colors and layout instead of checking whether the dashboard
  actually answers something. The order in the guided steps above is
  deliberate.
- **Credentials in the web app variant.** If you choose the live web app,
  make sure your agent reads your Supabase project URL and access token
  from an environment variable or local config it does not show you or
  commit anywhere, rather than writing them directly into a file. Ask it to
  confirm how it is storing them before you consider the app done.
- **The HTML artifact is a snapshot.** It is correct at the moment it is
  generated but will not reflect later changes to the database. If you
  update data in Supabase afterward, you have to ask the agent to
  regenerate the file; it will not refresh itself.

## Multi-tool notes
### Using Codex CLI or opencode instead
Neither tool has Claude Code's skill mechanism, so instead of invoking
mcp-health-check or pipeline-verify by name, ask your agent directly to
perform the same steps: "list the tables in my Supabase project and their
row counts" at the start, and "compare every number in the dashboard
against a fresh query against the database and show me both numbers" at
the end.

For the live web app variant, Claude Code can often run and preview a
local server for you inline. With Codex CLI or opencode you may need to
ask your agent for the local address it started the server on and open
that address yourself in a browser, then report back what you see so the
agent can keep iterating with you.

Credential handling works the same way in principle across tools: describe
your Supabase project URL and access token to your agent by name and let
it store them in whatever environment-variable or local-config mechanism
that tool uses, rather than pasting them into a source file.
