---
name: mcp-health-check
description: Use whenever the student asks to run the mcp-health-check skill, check or verify their Supabase, MongoDB, or Qdrant connections, troubleshoot a backend that seems disconnected or behaving oddly, or confirm what a backend currently contains (empty or populated) at any point in the workshop. Read-only - never creates, writes, or deletes anything in any backend.
---

# MCP Health Check

## Purpose

This skill checks every backend this workshop uses through MCP and reports,
in plain language, whether each one is connected and what you can currently
see in it. The student directing you is a data scientist or data engineer,
not a developer - they will never read raw logs or stack traces, so every
finding you report gets translated into plain language before it reaches
them, including failures.

This skill only reads. It never creates, writes, updates, or deletes
anything in any backend, and it never uses a write-only tool "just to see
if it works." If a check can only be done by writing something, say so and
stop - don't do it anyway.

## When this applies

- Right after connecting a new backend in the lab that sets it up
  (Supabase in Lab 1, MongoDB in Lab 2, Qdrant in Lab 3), to confirm the
  connection actually works before moving on.
- At the start of any later lab, to confirm the connection that lab needs
  is still alive.
- Any time the student reports something looks wrong, or asks whether a
  backend is connected, or what it currently holds.

This same check is meant to be re-run all day, and again if the student
redoes the workshop at home later. Don't assume "empty" is always the
correct or expected answer - early in the day it is, later in the day it
usually isn't. Report what you actually find and let the student judge
whether that matches where they are.

## Scope: three backends checked directly, one covered indirectly, one not checked at all

- **Supabase** (Postgres, tables) - checked directly, Step 1 below.
- **MongoDB Atlas** (collections) - checked directly, Step 2 below.
- **Qdrant Cloud** (points inside collections) - checked for connection
  only, Step 3 below; it cannot be checked the same way as the other two,
  and that's expected, not a gap in this skill.
- **pgvector** - not a separate connection. It's an extension plus a table
  living inside the same Supabase Postgres project, so once it exists it
  shows up naturally inside the Step 1 check. Don't treat it as a fourth
  backend to connect to.
- **Voyage AI** - has no MCP server in this workshop at all; there is
  nothing to check here. If the student asks about it, say so plainly and
  point them to the Voyage AI dashboard, which is the only place its API
  key usage is visible.

## Step 1 - Supabase

Ask the Supabase MCP connection to list every table in the connected
project.

- If it responds: report the table names plainly, or state clearly that
  the project currently has no tables of the student's own.
- If the student specifically asks whether the pgvector extension itself
  is enabled: that detail lives in the Supabase dashboard's Database >
  Extensions page, not in anything this connection's table-listing tool
  exposes. Say so rather than guessing from the table list alone.
- If it fails or times out, don't guess at why - go to Diagnosing a
  problem below.

## Step 2 - MongoDB

Ask the MongoDB MCP connection to list databases first.

- Right after connecting MongoDB in Lab 2, expect to see only MongoDB's
  own built-in system databases (things like admin, local, config) and no
  database of the student's own yet. Report this directly - don't lump system databases in
  with "your data" as if they were something the student created, and
  don't go looking for collections inside a database that doesn't exist
  yet.
- Once a workshop database exists (from a later lab), list the collections
  inside it and report whatever now exists. If a document count is easy to
  get for a collection, include it; if a count call fails or is
  unavailable, say plainly that the count couldn't be retrieved rather
  than showing the raw error.

## Step 3 - Qdrant (this one behaves differently - read carefully)

This workshop's Qdrant MCP connection exposes exactly two tools: one to
store a point, one to find/query. There is no tool to list collections or
count points - don't hunt for one under a different name. (These MCP
servers are young and still changing, so if the exact tools available ever
look different from this description, treat that as reason to check the
current docs, not as license to invent a listing call that isn't there.)

- The only thing you can confirm through this connection is that it's
  configured and its tools are available to you - that is what "connected"
  means here. You cannot confirm through this connection alone whether the
  cluster is empty or populated, or what collections it holds.
- Report this honestly rather than papering over it: state whether the
  connection is alive, then tell the student plainly that collection
  contents and point counts can only be confirmed in the Qdrant Cloud
  dashboard's Collections page, and point them there.
- Never call the store tool to "test" the connection - that writes a real
  point into a real collection and is not a read-only action. The fact
  that the server's tools are listed and available to you already is the
  strongest confirmation you're allowed to give.

## Step 4 - Diagnosing a problem

When a check fails or looks wrong, give one plain-language likely cause and
one concrete next step, drawn from the causes actually seen in this
workshop rather than a generic "check your credentials":

- **MongoDB hangs or times out with no clear error.** The single most
  common cause is the Atlas Network Access allowlist - the current IP (or
  0.0.0.0/0) hasn't been added. Next step: open MongoDB Atlas in the
  browser, go to Network Access, and add the current IP or confirm
  0.0.0.0/0 is present.
- **A connection that worked earlier is dead now.** Free tiers idle out:
  an inactive Supabase project can pause after about a week, and an idle
  Qdrant free cluster can suspend on a similar timescale. Next step: open
  that provider's dashboard and resume or unpause the project or cluster
  before troubleshooting anything else.
- **A server just added doesn't show up at all.** Config changes need a
  reload or restart of the agent session to take effect. Next step: reload
  or restart the agent, then re-run this check.
- **Something that should be empty already has tables, collections, or
  data in it.** This usually means the wrong project or cluster is
  connected - a leftover from an earlier attempt, or a different project
  than the one intended. Next step: check the exact project reference,
  connection string, or cluster URL configured against the one the student
  meant to use.
- **A backend was never configured at all.** Say so plainly and point back
  to that backend's own "Backend setup" section - Supabase is in Lab 1,
  MongoDB is in Lab 2, Qdrant is Part 0 of Lab 3 - rather than assuming a
  credentials problem.
- **Credentials are rejected outright** (MongoDB authentication failure,
  Qdrant API key rejected). Likely a typo when the password was pasted
  into the connection string, or an API key that wasn't copied correctly
  the one time it was shown. Next step: reset or regenerate the credential
  in the provider's dashboard, then reconnect.

If none of these match what you're seeing, say plainly that the cause
isn't obvious from what you can see, describe the symptom in plain
language, and suggest the student check that provider's current setup docs
rather than inventing a cause you're not sure of.

## Step 5 - Report to the student

Give one short block per backend, in this shape:

- **Supabase:** connected or not; then either the list of tables, or "no
  tables yet."
- **MongoDB:** connected or not; then either the list of collections (with
  counts if available), or "no collections of your own yet - only
  MongoDB's system databases."
- **Qdrant:** connected or not; then a reminder that contents can only be
  confirmed in the Qdrant Cloud dashboard.

For example:

> Supabase: connected. No tables yet - the project is empty.
> MongoDB: connected. No collections of your own yet - only MongoDB's own
> system databases are present.
> Qdrant: connected - the server's tools are available. I can't tell from
> here whether the cluster is empty or populated; check the Collections
> page in the Qdrant Cloud dashboard to confirm.

If something failed, replace that backend's line with the plain-language
cause and next step from Step 4, not a raw error message.

## What never to do under this skill

- Never create, write, update, or delete anything in any backend - this
  skill only reads.
- Never call Qdrant's store tool, or any write-capable tool on any backend,
  as a way of testing whether a connection works.
- Never show the student a raw error message, stack trace, or unexplained
  technical log line - translate every failure into a plain-language likely
  cause before reporting it.
- Never report "empty" as though it's always the expected state, and never
  report "populated" as though it's always a problem - state what you
  actually found and let the point in the day decide whether that's right.
- Never guess at a cause you're not reasonably confident in; say plainly
  that the cause isn't clear rather than inventing one.
