# Lab 0 - Setup: Tools and Your Coding Agent

This is the only lab of the day that happens mostly outside the agent.
You'll install git, get a local copy of this repository, and install and
start an AI coding agent inside it. That's all this lab does - no backend
accounts yet. Each later lab creates and connects the one backend it
actually needs, right before you use it: Lab 1 sets up Supabase, Lab 2
sets up MongoDB, Lab 3 sets up Qdrant and Voyage AI.

If MCP is new to you: it's a standard way for a coding agent to talk
directly to an external tool - a database, a vector store, an API - the
same way it already talks to files on your disk. Once a server is
"connected" (starting in Lab 1), you describe what you want in plain
language ("list the tables in my Supabase project") and the agent picks
the right tool on its own. You never call these tools yourself.

## Objective

By the end of this lab you will have:

- git installed and a local copy of this repository on your machine;
- one of the three supported coding agents (Claude Code, Codex CLI, or
  OpenCode) installed and running, started from inside this repository;
- an understanding of what this repo's skills are and that they need no
  separate setup from you.

## Prerequisites

- A computer with a terminal/shell and an internet connection.
- An email address, for the coding agent's own sign-in - you'll create
  the four backend accounts later, one per lab, as each becomes relevant.

## Install git and get the repo

If you're not sure whether you already have git, open a terminal and run
`git --version` - if that prints a version number, skip to cloning below.

**Install git**, if you don't already have it:

- macOS: `brew install git`, or install Xcode Command Line Tools with
  `xcode-select --install`, which includes git.
- Windows: download and run the installer from
  [git-scm.com/download/win](https://git-scm.com/download/win).
- Linux (Debian/Ubuntu): `sudo apt install git`. Other distributions: see
  [git-scm.com/download/linux](https://git-scm.com/download/linux).
- Full instructions for any platform: [git-scm.com/downloads](https://git-scm.com/downloads).

**Clone the repository:**

```
git clone https://github.com/alexiocassanifm/agentic-data-workshop.git
```

This creates a folder named `agentic-data-workshop`. `cd` into it - every
command and every agent session for the rest of this workshop happens
from inside this folder.

```
cd agentic-data-workshop
```

## Install your coding agent

Pick the agent you're using and follow that track's install instructions,
then start it from inside this repository and log in when prompted.
Nothing below needs a backend account yet - that starts in Lab 1.

### A note on where MCP servers live

Starting in Lab 1, each server you add can be stored privately on your
machine, or checked into this repository so the setup travels with it.
This workshop's later labs default to the private option, which is the
lower-friction choice for a one-day workshop. One rule holds regardless of
which you pick, for every backend you'll connect: a secret (a connection
string, an API key) must never appear as a literal value in a file that's
part of this repository - only as a reference to an environment variable
set outside it.

### A note on skills

Besides the MCP servers you'll connect starting in Lab 1, this repo also
ships five small "skills" under [`.claude/skills/`](../.claude/skills) -
`mcp-health-check`, `schema-proposal`, `pipeline-verify`,
`backend-compare`, and `dashboard-build` - that later labs ask you to
invoke by name (for example, "Run the mcp-health-check skill"). Unlike
the MCP servers, skills need no setup from you: they're just files
already checked into this repo, and your agent discovers them
automatically the moment it starts inside this folder - there's nothing
to add, connect, or authenticate.

Discovery does differ slightly by tool, which matters for how later
labs' prompts are worded:

- **Claude Code** and **opencode** both read `.claude/skills/` directly,
  so every skill just works once you start either tool inside this
  repository. The one thing to do on Claude Code specifically: the first
  time it opens this folder, accept its one-time workspace trust prompt
  - skills, like any other project file, only activate once you do.
- **Codex CLI** supports the same open skill format too, but looks for
  skills under `.agents/skills/` instead of `.claude/skills/` - this repo
  ships an identical copy of all five skills there as well, so every
  skill just works on Codex CLI too, the same way, by name.

Now jump to your track: [Claude Code](#track-claude-code) ·
[Codex CLI](#track-codex-cli) · [OpenCode](#track-opencode)

## Track: Claude Code

**Install**, if you don't already have it:

- macOS / Linux / WSL: `curl -fsSL https://claude.ai/install.sh | bash`
- Windows (PowerShell): `irm https://claude.ai/install.ps1 | iex`
- Homebrew: `brew install --cask claude-code`
- WinGet: `winget install Anthropic.ClaudeCode`
- Full instructions and troubleshooting: [code.claude.com/docs/en/overview](https://code.claude.com/docs/en/overview)

Start it with `claude` inside this repository and log in when prompted.
Once it's running, move on to [Lab 1](../lab-1-relational-supabase/README.md),
which walks you through creating a Supabase account and connecting it.

## Track: Codex CLI

**Install**, if you don't already have it:

- macOS / Linux: `curl -fsSL https://chatgpt.com/codex/install.sh | sh`
- Windows (PowerShell):
  `powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"`
- npm: `npm install -g @openai/codex`
- Homebrew: `brew install --cask codex`
- Full instructions: [developers.openai.com/codex/quickstart](https://developers.openai.com/codex/quickstart)

Start it with `codex` inside this repository and sign in when prompted.

A Codex-specific note before you start: adding an MCP server through Codex
always writes into your personal `~/.codex/config.toml`, not into this
repository - there's no scope choice to make. If you want a
project-shared setup instead, once you get to Lab 1 you can ask your agent
to hand-create or edit a `.codex/config.toml` file at the repo root
instead; Codex only loads a project-level config once you've marked this
folder as trusted.

Once it's running, move on to [Lab 1](../lab-1-relational-supabase/README.md),
which walks you through creating a Supabase account and connecting it.

## Track: OpenCode

**Install**, if you don't already have it:

- Install script: `curl -fsSL https://opencode.ai/install | bash`
- npm: `npm install -g opencode-ai`
- Homebrew: `brew install anomalyco/tap/opencode`
- Windows: `scoop install opencode` or `choco install opencode`
- Download page and full instructions: [opencode.ai/download](https://opencode.ai/download),
  [opencode.ai/docs](https://opencode.ai/docs)

Start it with `opencode` inside this repository and sign in when prompted.
Once it's running, move on to [Lab 1](../lab-1-relational-supabase/README.md),
which walks you through creating a Supabase account and connecting it.

## Known pitfalls

- **Cloning over SSH instead of HTTPS fails without a GitHub SSH key set
  up.** The HTTPS URL above works with no authentication for this public
  repository - stick with it unless you already have SSH access to GitHub
  configured.
- **Config changes don't take effect until you reload or restart your
  agent.** This applies to every MCP server you add in later labs - if a
  newly added server doesn't show up, that's usually why.
- **Codex's server-add step always targets your personal config, never
  this repository.** There's no scope choice - if you want the setup
  checked in, you (or your agent) have to hand-edit a project
  `.codex/config.toml` instead.
- **Secrets never belong in a file that's part of this repository.**
  Every backend you connect from Lab 1 onward hands you at least one
  secret (a connection string or an API key). Store each one in a
  password manager or a private notes file outside this repo, and never
  type one as a bare command-line argument - it can end up saved in your
  shell history.
- **These MCP servers are young and still changing.** They're updated
  frequently even past their first stable release; if a tool name, flag,
  or exact config field doesn't match what a later lab describes, check
  that provider's current docs - the overall workflow will still hold
  even if a detail has moved.
