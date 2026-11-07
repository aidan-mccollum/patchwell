<p align="center"><img src="patchwell.png" width="96" alt="Patchwell"></p>

<h1 align="center">Patchwell</h1>

<p align="center">Three tools for AI agents that show their work: cited web research, transcription with word timestamps, and spreadsheet cleaning with a change log.<br>
Remote MCP server and plain HTTP API. Pay per call, no subscription.</p>

<p align="center"><a href="https://patchwell.io">patchwell.io</a> · <a href="https://patchwell.io/quickstart">Quickstart</a> · <a href="https://patchwell.io/docs">API reference</a> · <a href="https://registry.modelcontextprotocol.io/v0/servers?search=io.patchwell">MCP Registry</a></p>

---

## Tools

| Tool | What it returns | Price |
|---|---|---|
| `research` | An answer where every sentence cites the page it came from, the claims that couldn't be verified, and a confidence grade. Topics, markets, products, companies, regulations, published research. Does not look up private people. | $0.15 per answered question. Declined or unanswered: free |
| `transcribe` | Text, word-level start/end times and confidence, sentence segments, SRT and VTT. Audio or video URLs up to 3 hours. Whisper's invented phrases and repetition loops are flagged, never silently edited. | $0.008 per audio minute |
| `clean_data` | Clean rows from a CSV, Excel or JSON file, plus a log of every changed value, a review list, detected column types, a JSON Schema and a reusable recipe. | $2 per 1,000 rows |
| `get_job_result` | The result of a long job (anything over about 100 seconds returns a `job_id`). | Free |

Measured on real files: a 44-minute interview transcribed in 49 seconds; a 352-row messy customer file cleaned in 2.2 seconds (45 duplicates merged, 1,435 values fixed, every one logged); a research question answered in about a minute.

## Get a key

Free $1 key, no card, one per email:

```bash
curl -X POST https://patchwell.io/v1/credits/trial \
  -H "Content-Type: application/json" -d '{"email": "you@example.com"}'
```

Or buy credit at [patchwell.io](https://patchwell.io/#pricing). Keys look like `pw_live_...`.

## Connect

**Claude Code**

```bash
claude mcp add --transport http patchwell https://patchwell.io/mcp/ --header "Authorization: Bearer YOUR_KEY"
```

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
      "args": ["-y", "mcp-remote", "https://patchwell.io/mcp/", "--header", "Authorization: Bearer YOUR_KEY"]
    }
  }
}
```

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

Each claim is checked against the text of the page it cites before you see it. Claims the page doesn't support are removed and counted in `gaps`.

## Also on Apify

The same tools run as Apify Actors, billed through your Apify account:
[research](https://apify.com/patchwell/patchwell-research) ·
[transcribe](https://apify.com/patchwell/patchwell-transcribe) ·
[clean data](https://apify.com/patchwell/patchwell-clean-data)

## Privacy and limits

- Audio and files are processed in temporary containers and discarded. Patchwell keeps no copies.
- Research questions are kept for 30 days with contact details removed, to improve accuracy. Switch it off per key at [patchwell.io/privacy](https://patchwell.io/privacy), or per request with the header `X-Patchwell-Retain: off`.
- Research declines questions about private individuals (addresses, phone numbers, relatives, profiles). Repeated attempts pause the key.
- URLs pointing at private or internal network addresses are refused.

## About this repository

This repo holds the public listing: the MCP Registry entry (`server.json`) and these docs. The service runs at patchwell.io.

Questions: patchwell.support@gmail.com
