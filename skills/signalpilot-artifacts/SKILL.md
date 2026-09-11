---
name: signalpilot-artifacts
description: "Load when you need to download the files a SignalPilot agent saved (charts, CSVs, reports, dashboards). Covers list_artifacts, download_artifacts, the single-use POST download, and file checks."
---

# Download SignalPilot artifacts

The SignalPilot agent saves files in the chat. Examples: `revenue.png`, `result.csv`,
`report.html`, `artifacts/sales.dashboard.json`. You download them with two MCP tools and
one HTTP request per file.

## Procedure

### Step 1. List the files

1. Call `list_artifacts` with `thread_id`.
2. Read `artifacts`. Each entry has `artifact_id`, `filename`, `kind`, `byte_size`, `sha256`.
3. If the list is empty and the run is `completed`, tell the user the agent saved no files.

### Step 2. Get download links

1. Call `download_artifacts` with `thread_id` and `artifact_ids`.
2. Send at most 20 ids in one call. Keep the total under 100 MiB.
3. Each entry now has `download_url` and `expires_at`. The link is valid for two minutes.
   Each link works one time only.

### Step 3. Download each file

Do this at once, before the links expire.

1. Split `download_url` at the `#` character. The left part is the URL. The right part is
   the token.
2. Send one HTTP POST to the URL. Header: `Content-Type: application/json`.
   Body: `{"token": "<the token>"}`. Do not send a `Range` header. Do not add an
   `Authorization` header. The token is the credential.
3. Save the response body to the target folder under `filename`.

Example with `curl`:

```bash
curl -sS -X POST "<url before #>" \
  -H "Content-Type: application/json" \
  -d '{"token":"<token>"}' \
  -o "<folder>/<filename>"
```

Use the same shape with a Python script if `curl` is not available:

```python
import json, sys, urllib.request
url, token, out = sys.argv[1], sys.argv[2], sys.argv[3]
req = urllib.request.Request(url, data=json.dumps({"token": token}).encode(),
                             headers={"Content-Type": "application/json"}, method="POST")
with urllib.request.urlopen(req, timeout=60) as r, open(out, "wb") as f:
    f.write(r.read())
```

### Step 4. Check the files

1. Compare the file size with `byte_size`.
2. If you can, compare the SHA-256 hash with `sha256`.
3. If a download failed or the size is wrong, call `download_artifacts` again for that id
   and repeat Step 3. A used or expired link does not work a second time.

## Rules

- Treat the token as a secret. Do not print it in the report or the chat.
- Do not open `download_url` with a GET request. GET returns a web page, not the file.
- Keep the original `filename`. The report refers to it.
- `kind` tells you how to use the file: `image` goes in the report as a picture, `data`
  is a table, `markdown` and `html` are text you can read, `dashboard` is a SignalPilot
  dashboard definition.
