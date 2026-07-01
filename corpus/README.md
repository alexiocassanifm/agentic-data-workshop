# Larkspur Flow Data Corpus

This is the shared data corpus used throughout the workshop. It represents
the customer data of **Larkspur Software, Inc.**, the maker of **Larkspur
Flow**, a B2B SaaS platform that mid-market and enterprise operations,
support, and sales teams use for ticketing, workflow automation, reporting,
and third-party integrations (Salesforce, Slack, and others).

The same 180 fictional customer companies, their contacts, and their
subscriptions run through every file in this corpus. Every other entity
(contracts, support tickets, sales notes, usage logs, onboarding emails)
refers back to those customers by the same `customer_id`. That single,
consistent key is what makes it possible to join, cross-reference, and
reconcile data across every format in this folder, regardless of which
backend or tool you use to explore it.

Nothing here needs to be cleaned up before you can start working with it.
Some of it is genuinely messy on purpose (see "What's messy" below), and
the rest is clean and consistent. Knowing which is which is the point of
the exercise: it should inform how you model each piece, not force you to
fix anything first.

## The ID scheme

Every entity has its own prefixed, zero-padded identifier, and `customer_id`
is the thread that ties everything together:

| Entity | ID format | Example |
|---|---|---|
| Customer | `CUST-####` | `CUST-0034` |
| Contact | `CONT-####` | `CONT-0212` |
| Subscription | `SUB-####` | `SUB-0087` |
| Contract | `CTR-####` | `CTR-0019` |
| Support ticket | `TCK-#####` | `TCK-00317` |
| Sales note | `SN-####` | `SN-0102` |
| Onboarding email | `EML-####` | `EML-0055` |

`customer_id` appears, unmodified and always in the clean `CUST-####` form,
in: `customers.csv`, `contacts.csv`, `subscriptions.csv`, in the header of
every support ticket and sales note, inside the JSON body of every usage
log, and it can be resolved from the filename of every usage log and
contract file. It is never abbreviated, reformatted, or left blank
anywhere in this corpus, even in the one file (`customers.csv`) that is
otherwise deliberately messy — the identifier column is the one thing kept
completely clean so that joins always work. Contacts, sales reps, account
executives, and the customer's own contract signatory are all drawn from
the same underlying pool of contact records in `contacts.csv`, so a name
that appears in a support ticket or a sales note is a real contact you can
also find by `customer_id` in that file.

## Entities

### 1. `customers-export/customers.csv` — 180 rows
One row per customer company. Columns: `customer_id`, `company_name`,
`industry`, `company_size_band`, `country`, `signup_date`, `plan_tier`.
This is the one file in the corpus that is intentionally messy (see below).

### 2. `customers-export/contacts.csv` — 350 rows
One or two named contacts per customer (VP/Director/Manager-level roles).
Columns: `contact_id`, `customer_id`, `full_name`, `role_title`, `email`.
Clean and consistent — no missing values, one date format (there are no
dates in this file), no casing issues.

### 3. `customers-export/subscriptions.csv` — 200 rows
The billing/plan history behind each customer. Columns: `subscription_id`,
`customer_id`, `plan`, `mrr_usd`, `start_date`, `end_date`, `status`.
There are more rows than customers because some customers have more than
one row over time — an upgrade or downgrade history, ending with their
current plan. `status` is one of `active`, `churned`, `trial`, or
`upgraded` (a historical row that was superseded by a later plan change).
`end_date` is blank for rows that are still in effect. This file is clean:
one consistent ISO date format, no missing values, no casing issues. The
`plan_tier` shown in `customers.csv` always matches that customer's most
recent row here — the two are consistent, though they live in different
files, which is itself a useful thing to notice when deciding what belongs
in one table versus another.

### 4. `contracts/` — 50 PDF files
Short (1–2 page) Master Subscription Agreements, one per contract, named
`CTR-####_<company-slug>.pdf`. Each PDF is a real, generated PDF document
containing: the parties and effective date, a term-length and renewal
clause, a pricing table (plan tier, seat count, monthly fee, payment
terms), a service-level clause (an uptime percentage that varies by
contract), a data-residency clause (varies: US, EU, or unspecified), and
one additional custom clause that varies contract to contract (seat
flexibility, onboarding support, data export rights, SSO support, and so
on). Only customers on the Professional or Enterprise plan have a
contract, which mirrors how these agreements would actually be used in
practice; not every customer in `customers.csv` will have a matching file
here. The monthly fee shown in each contract matches that customer's
`mrr_usd` in `subscriptions.csv`, so the two can be cross-checked against
each other.

### 5. `support-tickets/` — 500 plain text files
One file per ticket, named `TCK-#####.txt`. Each has a short header
(`Ticket ID`, `Customer ID`, `Contact`, `Priority`, `Status`, `Created`)
followed by a free-text subject and body written the way a real support
conversation reads. The tickets cluster around 12 recurring topics, so
that a semantic/vector search exercise later in the day produces
meaningful, coherent neighborhoods rather than noise:

1. SSO / login failures
2. Invoice / billing discrepancies
3. API rate limiting
4. Data export bugs
5. Onboarding friction
6. Reporting / analytics requests
7. Performance / latency complaints
8. Integration failures (Salesforce, Slack)
9. Security / compliance questions
10. Cancellation / churn-risk signals
11. Mobile app bugs
12. Permissions / access-control confusion

Within a topic the wording varies (different customers describe the same
kind of problem differently, in different tones — calm, frustrated,
urgent, or just curious), which is what makes this useful for embeddings
and semantic search rather than keyword matching alone.

### 6. `sales-notes/` — 150 plain text files
CRM-style call and meeting notes written by account executives, named
`SN-####.txt`, in free-text prose rather than a form. Each has a short
header (account, contact, account executive, date, meeting type) followed
by the note itself. Beyond sales-specific topics — renewal risk, expansion
opportunities, competitor mentions, pricing pushback — a good number of
notes also relay the same product friction that shows up in the support
tickets (a customer mentioning SSO frustration to their account exec, for
instance). Those notes deliberately reuse language from the matching
ticket topic, so they land near that topic's neighborhood in a semantic
search rather than off on their own.

### 7. `usage-logs/` — 177 JSON files
One file per customer with enough billing history to have usage data,
named `usage-<customer_id>.json` (for example `usage-CUST-0034.json`). A
handful of very recently signed-up customers don't have a file yet, which
is itself a realistic edge case worth noticing when you join this against
`customers.csv`. Each file is a nested document: account metadata, a
usage period, and a `daily_usage` array where every day nests its own
array of per-feature records (`feature_name`, `event_count`,
`unique_users`), drawn from a fixed set of 14 product features (dashboard
views, automation workflows, API access, SSO login, mobile app, and so
on). Not every customer uses every feature — the available features
depend on plan tier, and which of those they actually touch varies day to
day. This nested, one-record-per-day-per-feature shape (rather than one
flat row per feature) is deliberate: it's a natural example for comparing
"nested documents" against "normalized rows and references" later in the
day.

### 8. `emails/` — 124 plain text files
Onboarding-journey emails, named `EML-####.txt`, formatted as a
From/To/Subject/Date header block followed by a body. They're sent from a
Larkspur Flow customer success representative to a real contact at the
customer, and follow the onboarding journey: welcome, setup steps,
check-in, an upsell nudge for growing accounts, and a renewal reminder for
longer-tenured ones. Not every customer receives every stage — the number
of emails a given customer has depends on how far along their journey is.

## What's messy, and what's clean

**Deliberately messy — `customers-export/customers.csv` only:**
- **Date formats in `signup_date` are mixed.** Most rows use `YYYY-MM-DD`,
  a meaningful minority use `MM/DD/YYYY`, and a couple of rows spell the
  date out (`7 Mar 2024`). Any tool or agent that assumes a single date
  format will misparse some of these.
- **A handful of missing values** in non-critical columns: a few blank
  `industry` values, a few blank `company_size_band` values, and one blank
  `signup_date`. Nothing critical (no missing `customer_id`, no missing
  `plan_tier`) is ever blank.
- **Inconsistent capitalization and abbreviation in `country`.** Most rows
  are consistent ("United States", "United Kingdom", "Germany"), but a
  minority use different casing ("australia", "DENMARK") or abbreviations
  ("US", "UK") for the same country.
- **Two rows have an encoding artifact in the company name.** Two
  customers have accented characters in their names (a French and a
  German company); in `customers.csv` those names show up with a mangled,
  mojibake-style encoding, the way accented text sometimes looks after
  being read with the wrong character encoding. Everywhere else those two
  companies appear in the corpus, their names are written correctly — the
  artifact is confined to this one file, which is itself worth noticing.

**Clean and consistent everywhere else:** `contacts.csv`,
`subscriptions.csv`, every contract PDF, and every JSON usage log use one
consistent date format, complete non-key fields, and consistent
capitalization. `customer_id` itself is never messy anywhere in the
corpus, including in `customers.csv` — it is always the clean `CUST-####`
form, precisely so that every join across every file and format still
works.

**Unstructured by design, not "messy":** support tickets, sales notes, and
onboarding emails are free-text documents, not tables. They are internally
consistent (a small fixed header followed by natural-language prose) but
have no fixed schema for the body content itself — extracting anything
structured out of them (a sentiment, a topic, a mentioned feature name, a
renewal date) is a genuine task, not a formatting problem to fix first.

**Semi-structured by design:** the usage logs are valid, well-formed JSON,
but nested rather than flat — a single file mixes one-to-many
relationships (customer to days, day to per-feature records) in a single
document. Deciding whether that nesting is a good fit as-is, or whether it
should be flattened into separate rows/tables, is the discussion point
that file is meant to prompt.
