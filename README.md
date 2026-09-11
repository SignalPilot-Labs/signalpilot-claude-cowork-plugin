# SignalPilot Agent plugin for Claude Code and Claude Cowork

Delegate a data analysis to a SignalPilot agent from Claude Code. The plugin bundles the
SignalPilot MCP server (OAuth sign-in, no API key) and two skills:

- `signalpilot`: launch the agent, share the chat link, watch progress, collect the report.
- `signalpilot-artifacts`: download the files the agent saved and check them.

## Install

```bash
claude plugin marketplace add SignalPilot-Labs/signalpilot-claude-cowork-plugin
claude plugin install signalpilot-agent@signalpilot-agent
```

Then sign in once: run `/mcp`, select `signalpilot`, choose Authenticate. Your browser opens
the SignalPilot sign-in. Pick your organization on the consent screen.

The MCP server is `https://gateway.signalpilot.ai/mcp`.

## Try it

```
Run a SignalPilot agent for a simple chart of the past year of revenue and write a report.
```

Claude Code shares the chat link at once, watches the run, downloads the chart and data,
and writes `signalpilot-reports/<date>-<task>/report.md`.

## Note on skill names

The sandbox plugin `signalpilot-dbt` also has a skill named `signalpilot`. If both plugins
are installed, use the namespaced form `/signalpilot-agent:signalpilot`.

## Requirements

- Claude Code 2.1.186 or newer (OAuth for HTTP MCP servers).
- A SignalPilot account with a default project and connection set under Settings, then
  MCP Connect.
