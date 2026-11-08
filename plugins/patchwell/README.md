# Patchwell plugin

Connects your coding agent to Patchwell's hosted MCP server. No local code, scripts or hooks: the plugin is one MCP
server entry.

## Tools

- `research`: answers a question from the web with a cited page for each sentence and a list of what the sources
  couldn't confirm. It won't look up private people.
- `transcribe`: audio or video URL to text with word timestamps, speaker labels, SRT and VTT.
- `clean_data`: CSV, Excel or JSON to clean rows plus a log of every changed value.
- `get_job_result`: fetches the result of a long job by its `job_id`.
- `listen_clip`: plays a moment of a transcribed recording inside the result card. Hosts that support cards keep it out
  of the model's tool list.

## Network and credentials

- The plugin talks to one endpoint: `https://patchwell.io/connect/` (MCP over streamable HTTP).
- Sign-in uses OAuth 2.1 with dynamic client registration and PKCE. The first time, run `/mcp` and pick `patchwell`.
  A page on patchwell.io asks for your Patchwell key or gives you a free one with $1 of credit (one email, no card).
  Your client stores the key as its access token.

## Cost

Each call spends prepaid Patchwell credit: $0.15 per answered research question (declined or unanswered ones are
free), $0.008 per audio minute, $0.50 per 1,000 rows. Listing the tools is free.

[Privacy](https://patchwell.io/privacy) · [Terms](https://patchwell.io/terms) · [Docs](https://patchwell.io/quickstart)
