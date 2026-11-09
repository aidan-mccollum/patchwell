<p align="center"><img src="patchwell.png" width="96" alt="Patchwell"></p>

<h1 align="center">Patchwell</h1>

<p align="center">Cited web research, transcription with word timestamps, and spreadsheet cleaning with a change log, for AI agents.<br>
Remote MCP server and plain HTTP API. Pay per call, no subscription.</p>

<p align="center"><a href="https://patchwell.io">patchwell.io</a> · <a href="https://patchwell.io/quickstart">Quickstart</a> · <a href="https://patchwell.io/docs">API reference</a> · <a href="https://registry.modelcontextprotocol.io/v0/servers?search=io.patchwell">MCP Registry</a></p>

---

## Tools

| Tool | What it returns | Price |
|---|---|---|
| `research` | An answer where each sentence cites the page it came from, what the sources couldn't confirm, and a confidence grade. Topics, markets, products, companies, regulations, published research. Declines questions about private individuals and about anyone's personal details; public figures' public roles and statements are fine. | $0.15 per answered question. Declined or unanswered: free |
| `transcribe` | Text, word-level start/end times and confidence, who said each word (Speaker 1, 2, ..., or a name when the recording says it in a set phrase like "My name is ...", with the quote as evidence), speaker turns, sentence segments, SRT and VTT with speaker tags. Audio or video URLs up to 3 hours. Stock phrases Whisper invents over silence ("Thanks for watching!") and repeated loops are removed, each with a flag holding the original text and timestamps. | $0.008 per audio minute (min. $0.02 a file) |
| `clean_data` | Clean rows from a CSV, Excel or JSON file, plus a log of every changed value, a review list, detected column types, data quality before and after, a JSON Schema and a reusable recipe. Exports to CSV and Excel. | $0.50 per 1,000 rows |
| `get_job_result` | The result of a long job (anything over about 100 seconds returns a `job_id`). | Free |

From our own runs: a 44-minute interview took 85 seconds end to end, speaker labels included (one run, GPU start included). On 12 public recordings held out from our tuning (AMI, VoxConverse, Earnings-21 and SIMSAMU; 32 in all, 20 used for tuning), 97.1% of words went to the right speaker. That's a word-level measure, not DER, and words where two people overlap aren't scored. A messy 352-row customer file took about 2 seconds to clean, with 45 duplicates merged and 1,429 values fixed, each one logged. A research question typically takes 50 to 100 seconds (median about 70 seconds, up to about 3 minutes), plus 20 to 40 seconds when the model has to start after a quiet period.

## Get a key

Free key with $1 of credit (more on launch day). It needs an email but no card. You get one per email and one per network every 30 days, and only a set number go out each day:

```bash
curl -X POST https://patchwell.io/v1/credits/trial \
  -H "Content-Type: application/json" -d '{"email": "you@example.com"}'
```

Or buy credit at [patchwell.io](https://patchwell.io/#pricing). Keys look like `pw_live_...`.

## Connect

**Claude (web and desktop) or ChatGPT**

Add a custom connector with the address `https://patchwell.io/connect/`. The app opens a Patchwell page where you paste
your key or get a free one, then sends you back. In Claude it's under Customize, then Connectors, then Add. In ChatGPT, use
developer mode.

**Claude Code**

Install the plugin, then run `/mcp`, pick `patchwell` and sign in. A Patchwell page asks for your key or gives you a
free one.

```
/plugin install patchwell --marketplace aidan-mccollum/patchwell
```

Or connect with a key you already have:

```bash
claude mcp add --transport http patchwell https://patchwell.io/mcp/ --header "Authorization: Bearer YOUR_KEY"
```

**Gemini CLI**

```bash
gemini extensions install https://github.com/aidan-mccollum/patchwell
```

Then run `/mcp auth patchwell` inside Gemini CLI to sign in.

**Cursor, Windsurf and other clients that take JSON config**

```json
{
  "mcpServers": {
    "patchwell": {
      "type": "http",
      "url": "https://patchwell.io/mcp/",
      "headers": { "Authorization": "Bearer YOUR_KEY" }
    }
  }
}
```

**Claude Desktop** (through the `mcp-remote` bridge)

```json
{
  "mcpServers": {
    "patchwell": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://patchwell.io/connect"]
    }
  }
}
```

The first time Claude Desktop starts the bridge, a Patchwell page opens in your browser and asks for your key or gives
you a free one. The key stays out of this file.

**Plain HTTP**

```bash
curl https://patchwell.io/v1/research \
  -H "Authorization: Bearer $PATCHWELL_KEY" -H "Content-Type: application/json" \
  -d '{"ask": "Which open-source licenses allow commercial use?"}'
```

## What a research answer contains

| Field | What it holds |
|---|---|
| `answer` | The answer. Each sentence ends with the URL of the page it came from, in brackets |
| `sources` | The pages the answer rests on, with their titles |
| `gaps` | What the sources couldn't confirm, and how many claims the check dropped |
| `confidence` | `high`, `medium` or `low` |
| `billing` | What the call cost: $0.15 for an answer, nothing when it declines or finds no sourced answer |

Before you see the answer, a second step checks each claim against the page it cites. The quote behind a claim has to appear on that page (string matching, no model involved), and then the same model judges whether the quote supports the claim. Claims that fail are dropped and counted in `gaps`. A claim that passes means the page says it, not that it's true, so open the cited page before you rely on anything important.

## Also on Apify

The same tools run as Apify Actors, billed through your Apify account at Apify per-event prices, which can differ from the direct API:
[research](https://apify.com/patchwell/patchwell-research) ·
[transcribe](https://apify.com/patchwell/patchwell-transcribe) ·
[clean data](https://apify.com/patchwell/patchwell-clean-data)

## Built for automations

- **Retries never charge twice:** send an `Idempotency-Key` header; a retry gets the first response back.
- **Webhooks:** add `callback_url` and get the result POSTed when it's done, signed with HMAC-SHA256 and retried for about two hours.
- **Presets:** save a configuration once (`PUT /v1/presets/<name>`) and send `"preset": "<name>"`.
- **Exports:** `POST /v1/export?format=docx` (or `srt`, `vtt`, `txt`, `md`, `csv`) turns a transcript into a file.
- **Speaker labels you can check:** every word carries `spk_p_wrong`, the chance its speaker label is wrong (checked on recordings held out from our tuning), and `speaker_review` lists the spots worth a listen with links that open the file at that second.
- **Stable results:** every result has `schema_version`; within v1, fields are only added.

- **Talk-time analytics:** each speaker's share, pace, turns, longest stretch, questions and interruptions.
- **Rename speakers** in the Claude card, or with `POST /v1/rename` and `speaker_names` on exports.
- **Templates:** n8n workflows (tested inside n8n) and a schedulable podcast-to-notes script at [patchwell.io/automations](https://patchwell.io/automations).

How these numbers were measured, and where the tools fall short: [patchwell.io/trust](https://patchwell.io/trust).

## Privacy and limits

- Patchwell deletes input files (audio, video, spreadsheets) after each call. It keeps a long job's result up to 7 days for pickup, and the full response to a request sent with an `Idempotency-Key` for 24 hours, so a retry gets the same answer.
- Research questions aren't kept unless you turn that on: the box when you buy credit, the account setting (`POST /v1/account/privacy`), or the header `X-Patchwell-Retain: on`. Kept questions stay 30 days with contact details removed, and Patchwell uses them only to find where the tool goes wrong.
- No model API company (OpenAI, Anthropic and the like) receives your questions: the model runs on GPUs Patchwell rents. Serper and Tavily receive the search queries the tool writes from your question, and Wikipedia's search and the GDELT news index receive its topic for lookups. Every provider is listed in the [privacy policy](https://patchwell.io/privacy).
- Research declines questions about private individuals and about anyone's personal details (where they live, contact details, family, health, finances, tracking someone). Public figures' public roles and statements are fine. If a key keeps sending declined questions, Patchwell pauses research on it for 24 hours.
- Speaker labels use voice embeddings computed in memory during one call and never stored. Send `"speakers": false` to skip them.
- The tools refuse URLs that point at private or internal network addresses.
- If Patchwell shuts down, you get at least 30 days' notice and a refund of all unused credit you paid for. Good-faith testing, including trying to break the refusals, is welcome: see the [terms](https://patchwell.io/terms#testing).

## About this repository

This repo holds the MCP Registry entry (`server.json`), the Claude Code plugin (`plugins/patchwell`), the Gemini CLI
extension (`gemini-extension.json`), the Cursor plugin (`.cursor-plugin/`) and these docs. The service runs at
patchwell.io.

Questions: patchwell.support@gmail.com
