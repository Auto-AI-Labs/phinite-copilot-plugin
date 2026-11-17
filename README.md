# Phinite plugin for GitHub Copilot

Build, ship, run and debug **Phinite agent graphs** from GitHub Copilot (Copilot CLI and VS Code agent mode). The
plugin connects Copilot to the hosted **Phinite MCP server** (`https://mcp.dev.phinite.ai/mcp`, streamable HTTP, OAuth) and adds the
Phinite playbook as skills, rules and the `phinite-builder` agent. It uses the portable Agent Plugins 1.0 layout
(`plugin.json` + `mcp.json` + `skills/`), with Copilot's agent and rules under `com.github.copilot/`.

## Install

- **Copilot CLI:**
  ```text
  copilot plugin marketplace add Auto-AI-Labs/phinite-copilot-plugin
  copilot plugin install phinite@phinite
  ```
  Or straight from the repo: `copilot plugin install Auto-AI-Labs/phinite-copilot-plugin`.
- **VS Code:** enable `chat.plugins.enabled`, then run **Chat: Install Plugin from Source** and enter
  `https://github.com/Auto-AI-Labs/phinite-copilot-plugin` (or a local path to this folder).

You need Python 3.9+ for the project script. No Phinite token is stored in the repo.

To use another server (production or a local one), change `url` in `mcp.json`.

## Sign in

The server supports OAuth discovery and dynamic client registration, so there is nothing to configure. In VS Code,
start **phinite** from the MCP Servers list and allow the sign-in; in Copilot CLI, use `/mcp`. Or run
`/phinite-login`. A browser opens on the Phinite login (Google, GitHub or email); pick the org and the workspace.

## Use

Each task is a skill you can run as a slash command, or just ask for it:

| Skill | What it does |
|---|---|
| `/phinite-login` | Sign in, check the sign-in, or switch workspace |
| `/phinite-new-use-case` | Scaffold a use-case project and design its graphs with you |
| `/phinite-deploy-project` | Dry run, then deploy: tools upserted (published when changed), graphs created or updated |
| `/phinite-test-graph` | Chat or run the graph autonomously, read the log, diagnose |
| `/phinite-ship-graph` | Publish → build → environment → trigger or Chat API → API key → call it from outside |
| `/phinite-explain-run` | Explain a run: tool calls, inputs, outputs, variables, errors |
| `/phinite-evaluate-graph` | Evaluate with LLM judges: cases, metrics, failures and fixes |

The `phinite-builder` agent takes a whole use case end to end (`/agent` → phinite-builder in the CLI, or pick it in
the VS Code agents list).

## What's inside

- **Skills** (the same as the Claude plugin's, from the Phinite playbook): `phinite-design`, `phinite-ship`,
  `phinite-debug`, `phinite-api` and `phinite-project`, plus `phinite-quality` for evaluations (LLM judges,
  custom metrics, production scoring).
- **A local project script**, `skills/phinite-project/scripts/phinite_project.py` (Python 3.9+, standard library
  only). It scaffolds use-case folders from `templates/`, runs `graph_spec.py` on your machine, bundles the project
  and sends files through the server's upload links. The hosted server can't read your disk.
- `tests/`: the offline tests for that script: `python -m unittest tests/test_offline.py`.

The MCP tools and the rules they apply are listed in the
[phinite-mcp README](https://github.com/Auto-AI-Labs/phinite-mcp#tools).

## Security notes

- **No secrets on disk or in the chat.** The client stores and refreshes the OAuth token; the assistant never sees
  it. Env values go from your `.env` straight to an upload link and never pass through the chat.
- **Upload links** are valid for 15 minutes and for one purpose. The project script accepts only https links on
  `*.phinite.ai` (or the `PHINITE_MCP_URL` host, or localhost) with the `/u/<ticket>` path, so a link from a file,
  web page or message is refused.
- **Hidden files** (`.env`, `.git`…) in folders are never sent, and `KNOWLEDGE` files must be inside the project.
- **Env variables are workspace-wide**: prefix a use case's own secrets (`MYUC_SLACK_TOKEN`), and the assistant asks
  before overwriting a name that already exists.
- **`graph_spec.py` is code**: `deploy` runs it on your machine. Deploy only projects you wrote or trust.
