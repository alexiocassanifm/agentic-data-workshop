# Lab 0 - Setup: Accounts and MCP Configuration

This is the only lab of the day that happens mostly outside the agent. You'll
create four accounts, collect the credentials each one gives you, and then
hand those credentials to your AI coding agent so it can reach each backend
directly through MCP (Model Context Protocol) servers. From Lab 1 onward, you
direct every backend in natural language and rarely open a provider's web UI
except to double-check what the agent did.

## Objective

By the end of this lab you will have:

- four accounts created: Supabase, MongoDB Atlas, Qdrant Cloud, and Voyage AI;
- three of them - Supabase, MongoDB, and Qdrant - connected to your agent
  through MCP servers, so the agent can query and later populate them;
- a Voyage AI API key created and stored safely for Lab 3 (it has no MCP
  server of its own - more on that below);
- confirmation, both in each provider's own web UI and by asking your agent,
  that every backend is reachable and currently empty.

## Duration

75-90 minutes.

## Prerequisites

- A terminal with one of the supported agents already installed and working:
  Claude Code (primary path in this README), OpenAI Codex CLI, or opencode.
  Installing the agent itself is outside the scope of this lab - sort that
  out first if you haven't already.
- Node.js 22.12 or later (needed to run the MongoDB MCP server) and `uv` /
  `uvx` (needed to run the Qdrant MCP server) available on your machine. If
  you're not sure whether you have them, the first guided step below checks
  for you.
- A web browser and an email address you're willing to use for four
  separate sign-ups.
- A password manager, or at minimum a private notes file kept outside this
  repository, to hold four sets of credentials.
- About 75-90 minutes of uninterrupted time. Four sign-ups back to back is
  more tiring than it sounds - see Known pitfalls.

This is the first lab of the day, so there is no previous lab to have
completed. If you want a preview of the corpus you'll be working with from
Lab 1 onward, skim `../corpus/README.md` - Lab 0 itself doesn't touch it.

## Guided steps

Work through one provider completely - account created, MCP server
connected, connection verified - before moving to the next, rather than
creating all four accounts up front. Steps marked "in your browser" are
manual; the rest are things to ask your agent to do - each of those steps
includes a ready-to-paste prompt in a quote block. Fill in the bracketed
blanks with your own values before sending it; the exact server/package
names are already correct, you don't need to know them yourself.

1. **Check your machine is ready.** Ask your agent to check that Node.js
   22.12 or later and `uv`/`uvx` are both available. If either is missing,
   ask your agent to walk you through installing it for your operating
   system before you continue.

   > **Prompt to give your agent:**
   > "Check whether Node.js 22.12 or later, and uv/uvx, are installed on
   > this machine. If either is missing, walk me through installing it
   > for my operating system, then confirm both are ready before we
   > continue."

2. **Supabase - create your account and project (in your browser).** Go to
   supabase.com, sign up, and create a new project: pick an organization, a
   project name, a database password (store it safely even though the MCP
   setup below won't need it directly), and a region. Wait for provisioning
   to finish, a couple of minutes.

3. **Supabase - connect the MCP server.** Ask your agent to add the
   Supabase MCP server using the hosted endpoint
   `https://mcp.supabase.com/mcp`, scoped to your project's reference ID
   (find it under Project Settings > General > Reference ID in the
   Supabase dashboard). This is one of the two places where scope matters -
   read the decision point below before you tell your agent to go ahead.

   > **Prompt to give your agent** (read the decision point below first,
   > then fill in the two blanks):
   > "Add an MCP server named `supabase`, in [PROJECT / LOCAL] scope,
   > using the hosted endpoint
   > `https://mcp.supabase.com/mcp?project_ref=<YOUR PROJECT REFERENCE>`.
   > Don't add any API key or access token - this server logs in through
   > a browser the first time we use it."

4. **Supabase - approve access.** Reload or restart your agent so the new
   server definition takes effect. It should open a browser window for
   you: log in to Supabase there and approve access to the organization
   that owns your project. No personal access token is needed for this
   flow - if something you're reading elsewhere tells you to generate one
   first, that's the older, no-longer-necessary method.

5. **Supabase - verify.** Ask your agent to list the tables in your
   Supabase project. It should report an empty project - no tables yet.
   Note in passing: you won't need a raw Postgres connection string for
   this MCP server, but it's worth locating anyway (Project Settings >
   Database > Connection string), since Lab 3's pgvector work talks to
   Postgres directly and will want it then.

   > **Prompt to give your agent:**
   > "List every table in my Supabase project through the MCP connection,
   > and tell me whether the project is empty."

6. **MongoDB Atlas - create your account and cluster (in your browser).**
   Go to mongodb.com/cloud/atlas, sign up, and create a free (M0) cluster.

7. **MongoDB Atlas - create a database user (in your browser).** Under
   Database Access, create a user with a username and password. Store the
   password safely - you'll need to paste it into the connection string.

8. **MongoDB Atlas - open network access (in your browser).** Under
   Network Access, add an entry for your current IP address. For a
   workshop laptop that might move between networks during the day,
   allowing access from anywhere (0.0.0.0/0) is the pragmatic choice - see
   Known pitfalls for what that trade-off costs you.

9. **MongoDB Atlas - get your connection string (in your browser).** Click
   Connect on your cluster, choose the driver connection string option,
   and copy the `mongodb+srv://...` string. Replace the password
   placeholder in it with your database user's actual password.

10. **MongoDB - connect the MCP server.** Ask your agent to add the
    MongoDB MCP server, giving it your connection string as the value of
    its connection-string setting. Tell your agent explicitly: this value
    must never be passed as a plain command-line argument, only as this
    setting's value - otherwise it can end up saved in your shell history.

    > **Prompt to give your agent** (fill in the blank first):
    > "Add an MCP server named `MongoDB`, in [PROJECT / LOCAL] scope,
    > running the command `npx` with arguments `-y
    > mongodb-mcp-server@latest`. Set an environment variable called
    > `MDB_MCP_CONNECTION_STRING` to this value: `<YOUR MONGODB ATLAS
    > CONNECTION STRING>`. Make sure that value is only ever stored as
    > this environment variable - never pass it as a command-line
    > argument, and never write it into any file that's part of this
    > repository."

11. **MongoDB - verify.** Reload your agent, then ask it to list your
    databases and collections. Right after cluster creation this should
    show no collections of your own - only MongoDB's own system databases.

    > **Prompt to give your agent:**
    > "List the databases and collections you can currently see in
    > MongoDB."

12. **Qdrant Cloud - create your account and cluster (in your browser).**
    Go to cloud.qdrant.io, sign up, and create a free cluster.

13. **Qdrant Cloud - get your URL and API key (in your browser).** From
    the cluster's Data Access / API Keys area, generate an API key and
    copy it immediately - it's shown once - along with the cluster's URL.

14. **Qdrant - connect the MCP server.** Ask your agent to add the Qdrant
    MCP server with your cluster's URL and API key. Don't worry about
    naming a collection yet - Lab 3 is where the real one gets created.

    > **Prompt to give your agent** (fill in the two blanks first):
    > "Add an MCP server named `qdrant`, in [PROJECT / LOCAL] scope,
    > running the command `uvx` with the argument `mcp-server-qdrant`.
    > Set two environment variables: `QDRANT_URL` to `<YOUR QDRANT CLOUD
    > CLUSTER URL>`, and `QDRANT_API_KEY` to `<YOUR QDRANT CLOUD API
    > KEY>`. Don't set a collection name - we'll create the real one in
    > Lab 3."

15. **Qdrant - verify.** Reload your agent and ask it to confirm the
    Qdrant MCP server connected. This server only exposes a store tool and
    a find tool - there's no "list collections" tool - so the strongest
    confirmation that it's empty and ready is the Qdrant Cloud dashboard
    itself (see Verification below).

    > **Prompt to give your agent:**
    > "Confirm the qdrant MCP server is connected and ready to use."

16. **Voyage AI - create your account and API key (in your browser).** Go
    to voyageai.com, sign up, and from the dashboard generate a new API
    key. Copy it immediately.

17. **Voyage AI - store the key for later.** There's no MCP server for
    Voyage AI and nothing to configure right now: Lab 3 is where this key
    gets used, through a small script your agent writes at that point.
    For now, just keep the key alongside your other three secrets (see
    Known pitfalls) so you can hand it to your agent again when Lab 3
    asks for it.

18. **Run the full health check.** Ask your agent to run the
    `mcp-health-check` skill (if you're on Claude Code), or, on Codex CLI
    or opencode, ask it directly to check each of the three connections.
    Confirm it reports Supabase, MongoDB, and Qdrant all connected and
    empty.

    > **Prompt to give your agent** (Claude Code):
    > "Run the mcp-health-check skill."
    >
    > **Prompt to give your agent** (Codex CLI / opencode):
    > "Check my Supabase, MongoDB, and Qdrant connections and tell me,
    > for each one, whether it's connected and what it currently sees."

## Explicit decision point

### Where do your MCP server definitions and secrets live?

When you ask your agent to add each MCP server (steps 3, 10, and 14), it
will either default to a scope or ask you which one you want. Don't just
accept whatever it proposes first - decide on purpose, and give your agent
an explicit instruction each time.

You have two real options:

- **Project scope.** The server definitions live in a config file at the
  root of this repository, alongside the labs themselves. That's
  convenient if you want this setup to travel with the repo - to another
  machine, or to a teammate. But it comes with a rule: no literal secret
  value may ever be written into that file, since it's meant to be safe to
  keep in this repository. MongoDB's connection string and Qdrant's API
  key must instead be kept in an environment variable or your agent's own
  secret store, with the config file only referencing it by name - which
  means you now have one more thing to manage: where that actual value
  lives on your machine.

- **Local (or user) scope.** The server definitions live in your agent's
  own private settings, entirely outside this repository. They're never
  committed, never shared, and never seen by anyone else who might later
  look at this repo. That means you can be more relaxed about where the
  literal secret value sits, since it's private to your machine by
  construction - but it also means this setup won't follow you if you
  switch laptops or hand the repo to someone else.

For reference, here is roughly what the three server entries look like in
a Claude Code-style config once configured, regardless of which scope you
pick:

```
{
  "mcpServers": {
    "supabase": {
      "type": "http",
      "url": "https://mcp.supabase.com/mcp?project_ref=your-project-reference"
    },
    "MongoDB": {
      "command": "npx",
      "args": ["-y", "mongodb-mcp-server@latest"],
      "env": {
        "MDB_MCP_CONNECTION_STRING": "<your MongoDB Atlas connection string>"
      }
    },
    "qdrant": {
      "command": "uvx",
      "args": ["mcp-server-qdrant"],
      "env": {
        "QDRANT_URL": "<your Qdrant Cloud cluster URL>",
        "QDRANT_API_KEY": "<your Qdrant Cloud API key>"
      }
    }
  }
}
```

Notice that Supabase's entry carries no secret at all - the OAuth flow
handles that - while Mongo's and Qdrant's entries do. That asymmetry is
exactly why this decision matters more for those two than for Supabase: in
**local scope**, the two angle-bracketed placeholders above are literal
values, typed directly into your agent's private config. In **project
scope**, they must instead be a reference to an environment variable you
set outside this repository - the file that's actually written to disk in
this repo never contains the value itself, only the name of where to find
it. The exact reference syntax depends on which agent you're using; ask
your agent to show you what it wrote once it's done, so you can confirm
which of the two you ended up with.

For a one-day workshop you plan to redo at home on the same laptop, local
scope is the lower-friction choice. If you'd rather keep this repository as
a portable personal reference across machines, project scope with
externalized secrets pays for itself. Either is defensible - the point is
to be able to say which one you picked and why.

## Verification

**In each provider's web UI:**

- Supabase dashboard: project status is "Active"; Table Editor shows no
  tables.
- MongoDB Atlas: cluster status is healthy; Database > Collections shows
  no collections of your own (only the system databases MongoDB creates
  automatically).
- Qdrant Cloud: cluster dashboard shows zero collections and a healthy
  status.
- Voyage AI dashboard: your new API key is listed, with no usage yet.

**Via the agent:**

- On Claude Code, ask your agent to run the `mcp-health-check` skill. It
  should report, in plain language, that Supabase, MongoDB, and Qdrant are
  all connected, and that each currently looks empty.
- On Codex CLI or opencode, there's no equivalent built-in skill - ask your
  agent directly: "Check my Supabase, MongoDB, and Qdrant connections and
  tell me, for each one, whether it's connected and what it currently
  sees." The outcome you're looking for is the same.
- A concrete check to run either way: ask your agent to list every table
  in Supabase and every collection in MongoDB, and to confirm the Qdrant
  connection is alive. None of the three should show any data yet - if one
  does, you're either looking at the wrong project/cluster, or something
  from a previous attempt at this lab is still sitting there.

## Known pitfalls

- **MongoDB Atlas network access is the single most common stall.**
  Without your current IP (or 0.0.0.0/0) added under Network Access, the
  connection simply hangs or times out, with no clear error at the MCP
  level. If MongoDB verification is stuck, check this first. The trade-off
  with 0.0.0.0/0: it opens the cluster to connections from any address, so
  the username and password in your connection string become the only
  thing standing between your data and the internet - fine for a
  throwaway workshop cluster you'll tear down afterward, not something to
  carry over into a real project.
- **Four different secrets, four different handling rules.** Supabase's
  login is OAuth, so there's no key to store for it at all. MongoDB's
  connection string, Qdrant's API key, and Voyage's API key are all plain
  secrets you copy once and must store safely. Never let any of them end
  up in a file that's part of this repository, and never type one as a
  bare command-line argument - it can end up saved in your shell history.
- **Four sign-ups back to back is genuinely tiring.** Finish one account
  completely - create it, connect its MCP server, verify it - before
  starting the next, rather than creating all four accounts up front and
  configuring everything at the end.
- **Config changes don't take effect until you reload or restart your
  agent.** If a newly added server doesn't show up, that's usually why.
- **Supabase's recommended setup is OAuth-only - no connection string, no
  personal access token.** Older guides describing a personal-access-token
  flow are describing a legacy method you don't need here. If what you see
  doesn't match this README, check the current docs at
  supabase.com/docs/guides/getting-started/mcp.
- **The Qdrant MCP server can't list or count anything.** It only exposes
  a store tool and a find tool, so don't expect your agent to "list
  collections" the way it can for Supabase and MongoDB. Treat the Qdrant
  Cloud dashboard as the source of truth for "is it empty and ready."
- **Free tiers idle out.** An inactive Supabase project can pause after
  about a week, and an idle Qdrant free cluster can suspend on a similar
  timescale. If you come back to redo this workshop at home later and a
  connection that used to work suddenly doesn't, check whether the
  project or cluster needs to be resumed in its web UI before you start
  troubleshooting the MCP configuration itself.
- **Keep manual tool-call approval switched on** in your agent for the
  rest of the workshop. Supabase's own documentation notes that data
  returned from a query can contain text designed to redirect the agent -
  approval prompts are your safety net. None of these backends should
  ever be pointed at real production data.
- **These MCP servers are young and still changing.** Both the Supabase
  and MongoDB servers are pre-1.0 and evolving; if a tool name, flag, or
  exact config field doesn't match what's described here, check the
  current docs linked in each provider's guided step above - the overall
  workflow will still hold even if a detail has moved.

## Multi-tool notes

### Using Codex CLI or opencode instead

The account creation and browser steps above (2, 6-9, 12-13, 16) are
identical no matter which agent you use. Only how you register each MCP
server, and how you check that it worked, changes.

**Codex CLI**

- Ask Codex to add each server the same way you would Claude Code; under
  the hood it can do this via `codex mcp add`, or by editing the
  `mcp_servers` tables in `~/.codex/config.toml` (global) or a project
  `.codex/config.toml` (only loaded once Codex has marked this project as
  trusted - if a server you added seems to be ignored, check that first).
- Supabase maps to a remote entry with `url = "https://mcp.supabase.com/mcp"`
  under `[mcp_servers.supabase]`. Whether your Codex version walks you
  through the same browser OAuth flow, or expects a bearer token instead,
  check the current docs at developers.openai.com/codex/mcp - if OAuth
  isn't supported in your version, fall back to a Supabase personal access
  token supplied via a bearer-token environment variable, plus your
  project reference in the url.
- MongoDB and Qdrant map to local entries under `[mcp_servers.mongodb]` /
  `[mcp_servers.qdrant]`, each with a command, its arguments, and an `env`
  table carrying the same connection string, URL, and API key described
  above.
- Verify with `codex mcp list` from the shell, or `/mcp` (`/mcp verbose`
  for more detail) inside a session.
- There's no `mcp-health-check` skill in Codex - ask your agent directly:
  "Check my Supabase, MongoDB, and Qdrant connections and tell me what
  each currently sees."

**opencode**

- Ask your agent to add each server to `opencode.json` (project root) or
  `~/.config/opencode/opencode.json` (global), under a top-level `"mcp"`
  object.
- Supabase maps to a `"type": "remote"` entry with `"url":
  "https://mcp.supabase.com/mcp"` - opencode auto-detects the OAuth flow.
- MongoDB and Qdrant map to `"type": "local"` entries. Two naming details
  differ from the Claude Code shape shown above: the command is a single
  array (for example `["npx", "-y", "mongodb-mcp-server@latest"]`) rather
  than separate command/args fields, and environment variables go under
  the key `"environment"`, not `"env"`.
- Verify with `opencode mcp list` from the shell, `opencode mcp debug
  <name>` if something looks connected but isn't behaving as expected, or
  `/mcp` inside a session.
- As with Codex, there's no built-in health-check skill - ask your agent
  directly to check each connection and report what it sees.
