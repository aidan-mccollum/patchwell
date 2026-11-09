<p align="center"><img src="patchwell.png" width="96" alt="Patchwell"></p>

<h1 align="center">Patchwell</h1>

<p align="center">Cited web research, transcription with word timestamps, and spreadsheet cleaning with a change log, for AI agents.<br>
Remote MCP server and plain HTTP API. Pay per call, no subscription.</p>

<p align="center"><a href="https://patchwell.io">patchwell.io</a> · <a href="https://patchwell.io/quickstart">Quickstart</a> · <a href="https://patchwell.io/docs">API reference</a> · <a href="https://registry.modelcontextprotocol.io/v0/servers?search=io.patchwell">MCP Registry</a></p>

---

## Tools

| Tool | What it returns | Price |
|---|---|---|
| `research` | An answer where each sentence cites the page it came from, what the sources couldn't confirm, and a confidence grade. Topics, markets, products, companies, regulations, published research. Won't look up private people. | $0.15 per answered question. Declined or unanswered: free |
| `transcribe` | Text, word-level start/end times and confidence, who said each word (Speaker 1, 2, ..., or their name when the recording says it, with the quote as evidence), speaker turns, sentence segments, SRT and VTT with speaker tags. Audio or video URLs up to 3 hours. When Whisper invents a phrase or loops on a line, you get a flag with the original text and timestamps. | $0.008 per audio minute (min. $0.02 a file) |
| `clean_data` | Clean rows from a CSV, Excel or JSON file, plus a log of every changed value, a review list, detected column types, data quality before and after, a JSON Schema and a reusable recipe. Exports to CSV and Excel. | $0.50 per 1,000 rows |
| `get_job_result` | The result of a long job (anything over about 100 seconds returns a `job_id`). | Free |

Measured on real files: a 45-minute earnings call with 7 speakers took about 90 seconds, speaker labels included. On 24 public recordings (AMI meetings, VoxConverse debates and news, Earnings-21 calls), 97% of words were credited to the right speaker. A messy 352-row customer file took 1.7 seconds to clean, with 45 duplicates merged and 1,429 values fixed, each one logged. On real dirty datasets from data-cleaning research, it changed the meaning of 0.02% or fewer of the cells that were already correct. A research question takes about a minute.

## Get a key

Free $1 key, no card, one per email:

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
  -d '{"ask": "What does Stripe charge for online card payments in the US?"}'
```

## What a research answer looks like

Real output, shortened:

```json
{
  "answer": "Stripe charges 2.9% + 30 cents for standard online domestic card payments in the US [https://www.swipesum.com/insights/guide-to-stripe-fees-rates-for-2025][https://checkoutpage.com/blog/stripe-processing-fees]. PayPal's standard merchant fee is 3.49% + 49¢ per online transaction [https://merchantinsiders.com/blogs/paypal-fees/]. ...",
  "confidence": "medium",
  "gaps": ["2 claims left out because the sources found did not clearly support them."],
  "billing": { "price": 0.15 }
}
```

Before you see the answer, a separate step compares each claim with the text of the page it cites. It drops the claims that page doesn't support and counts them in `gaps`.

## Also on Apify

The same tools run as Apify Actors, billed through your Apify account:
[research](https://apify.com/patchwell/patchwell-research) ·
[transcribe](https://apify.com/patchwell/patchwell-transcribe) ·
[clean data](https://apify.com/patchwell/patchwell-clean-data)

## Built for automations

- **Retries never charge twice:** send an `Idempotency-Key` header; a retry gets the first response back.
- **Webhooks:** add `callback_url` and get the result POSTed when it's done, signed with HMAC-SHA256 and retried for about two hours.
- **Presets:** save a configuration once (`PUT /v1/presets/<name>`) and send `"preset": "<name>"`.
- **Exports:** `POST /v1/export?format=docx` (or `srt`, `vtt`, `txt`, `md`, `csv`) turns a transcript into a file.
- **Speaker labels you can check:** every word carries `spk_p_wrong`, the measured chance its speaker is wrong, and `speaker_review` lists the spots worth a listen with links that open the file at that second.
- **Stable results:** every result has `schema_version`; within v1, fields are only added.

- **Talk-time analytics:** each speaker's share, pace, turns, longest stretch, questions and interruptions.
- **Rename speakers** in the Claude card, or with `POST /v1/rename` and `speaker_names` on exports.
- **Templates:** n8n workflows (tested inside n8n) and a schedulable podcast-to-notes script at [patchwell.io/automations](https://patchwell.io/automations).

How these numbers were measured, and where the tools fall short: [patchwell.io/trust](https://patchwell.io/trust).

## Privacy and limits

- Patchwell processes audio and files in temporary containers and discards them. It keeps no copies.
- Patchwell keeps research questions for 30 days, with contact details removed, to improve accuracy. Switch that off per key at [patchwell.io/privacy](https://patchwell.io/privacy), or per request with the header `X-Patchwell-Retain: off`.
- Research declines questions about private individuals (addresses, phone numbers, relatives, profiles). If a key keeps asking, Patchwell pauses it.
- The tools refuse URLs that point at private or internal network addresses.

## About this repository

This repo holds the MCP Registry entry (`server.json`), the Claude Code plugin (`plugins/patchwell`), the Gemini CLI
extension (`gemini-extension.json`), the Cursor plugin (`.cursor-plugin/`) and these docs. The service runs at
patchwell.io.

Questions: patchwell.support@gmail.com
