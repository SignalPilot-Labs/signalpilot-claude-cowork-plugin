---
name: signalpilot
description: "Load when the user asks for a SignalPilot analysis, report, chart, dashboard, or asks to run a SignalPilot agent. Covers the full flow: launch the agent, share the chat link, watch progress, collect the report and artifacts, write a local report."
---

# SignalPilot analysis

## The environment

SignalPilot is a governed data platform. It runs an analysis agent in a cloud
sandbox with access to the user's warehouse, dbt project, and knowledge base.
You reach it through the SignalPilot MCP server. You do not query the warehouse
yourself. The SignalPilot agent does the analysis. You launch it, watch it,
collect the results, and write the report.

The MCP tools you use:

- `run_signalpilot_agent` starts an analysis. It returns `thread_id`, `run_id`, `chat_url`.
- `wait_signalpilot_agent` waits for a result.
- `continue_signalpilot_agent` sends a follow-up or an answer to the same chat.
- `get_signalpilot_agent` reads status. Use it only to fetch the final summary.
- `list_artifacts` and `download_artifacts` get the files the agent saved.
- `cancel_signalpilot_agent` stops a run.

Do not call the other SignalPilot tools (`query_database`, `schema_*`, `dbt_*`) for an
analysis request. Delegate to the agent.

## Communication style

Use ASD-STE100 Simplified Technical English with the user. Short sentences. One
fact per sentence. Active voice. No em dashes. Give numbers as the agent gave them.
Do not change or round a number.

## Procedure

Follow these steps in order. Do not skip a step.

### Step 1. Launch

1. Write the task as one clear paragraph. Include the metric, the time range, the grain,
   and the output the user wants. Example: "Make one simple line chart of monthly revenue
   for the past 12 months. Save the chart as a PNG. Write a short summary of the trend."
2. Call `run_signalpilot_agent` with `task`. Do not pass `project_id`, `connection_name`,
   or `branch` unless the user names them. Saved defaults apply.
3. Set `client_request_id` to a new random string. Reuse the same value if you retry the
   launch. This prevents a duplicate run.
4. If the tool says setup is missing, tell the user to open Settings, then MCP Connect, and
   set the default project and connection. Stop.

### Step 2. Share the link now

Send the `chat_url` to the user in your next message, before you do anything else.
Say: "SignalPilot is working. Watch it here: <chat_url>".

### Step 3. Watch progress

1. Call `wait_signalpilot_agent` with `thread_id`, `run_id`, `mode="completion"`.
2. The call returns after 25 seconds at most. Read `status` and `next_action`.
3. If `heartbeat` is true, call `wait_signalpilot_agent` again with the same `thread_id`,
   `run_id`, and `after_sequence` = the returned `next_sequence`. Repeat until the status is
   `completed`, `failed`, `cancelled`, or `input_required`.
4. Tell the user one short line for each new stage you see in `events`, for example "The agent
   is reading the schema" or "The agent is building the chart". Do not report every event.
5. Do not call `get_signalpilot_agent` in a loop. Do not launch a second agent for the same task.

### Step 4. Handle questions

If `status` is `input_required`:

1. Show the `question` text to the user.
2. Wait for the user's answer.
3. Call `continue_signalpilot_agent` with `thread_id` and the answer as `task`.
4. Go back to Step 3.

### Step 5. Collect the result

When `status` is `completed`:

1. Read `summary`. This is the agent's final report in Markdown.
2. If `summary` is empty, call `get_signalpilot_agent` with `thread_id`, `run_id`, and
   `after_sequence=0`. Read `summary` from that result.
3. Follow the `signalpilot-artifacts` skill to download every saved file.

If `status` is `failed`, show `error` to the user and stop. Do not retry by yourself.

### Step 6. Write your report

1. Create a folder `signalpilot-reports/<YYYY-MM-DD>-<short-task-name>/`.
2. Put the downloaded files in that folder.
3. Write `report.md` in that folder with these sections, in this order:
   1. `# <task name>`
   2. The `chat_url` on one line.
   3. `## SignalPilot report`: paste `summary` unchanged.
   4. `## Charts and files`: embed each image with `![name](filename)` and list every
      other file with its size.
   5. `## Notes`: your own observations, if any. Mark them as yours.
4. Show the user the path to `report.md` and the list of files.

## Claude Code notes

- Invoke this skill with `/signalpilot-agent:signalpilot` or let Claude Code load it.
- Tool names appear as `mcp__plugin_signalpilot-agent_signalpilot__<tool>`.
- If a tool returns "Authentication required", tell the user to run `/mcp`, select
  `signalpilot`, and sign in. Then continue.
- `wait_signalpilot_agent` returns in 25 seconds or less. Call it again for a heartbeat.

## Rules

- Share `chat_url` immediately after the launch. Always.
- Never change a number, label, or date from the SignalPilot report.
- Never paste the raw `download_url` into your report. The links expire in two minutes.
- One task, one agent. Use `continue_signalpilot_agent` for follow-ups.
- If the user asks to stop, call `cancel_signalpilot_agent`.
