<p align="center"><img src="patchwell.png" width="96" alt="Patchwell"></p>

<h1 align="center">Patchwell</h1>

<p align="center">Cited web research, transcription with word timestamps, and spreadsheet cleaning with a change log, for AI agents.<br>
Remote MCP server and plain HTTP API. Pay per call, no subscription.</p>

<p align="center"><a href="https://patchwell.io">patchwell.io</a> · <a href="https://patchwell.io/quickstart">Quickstart</a> · <a href="https://patchwell.io/docs">API reference</a> · <a href="https://registry.modelcontextprotocol.io/v0/servers?search=io.patchwell">MCP Registry</a></p>

---

## Tools

| Tool | What it returns | Price |
|---|---|---|
| `research` | An answer where each sentence cites the page it came from, what the sources couldn't confirm, and a confidence grade. Topics, markets, products, companies, regulations, published research. When the answer has to be worked out from several facts (an age at an event, a difference between two years), the first sentence states it and shows the arithmetic. Send `"depth": "deep"` to search about twice as wide on hard multi-step questions, or `"private": true` to keep the question inside Modal, the cloud Patchwell runs on (same price). Declines questions about private individuals and about anyone's personal details; public figures' public roles and statements are fine. | $0.02 per answered question, $0.06 with `"depth": "deep"`. Declined or unanswered: free |
| `transcribe` | Text, word-level start/end times and confidence, who said each word (Speaker 1, 2, ..., or a name when the recording says it in a set phrase like "My name is ...", with the quote as evidence), speaker turns, sentence segments, SRT and VTT with speaker tags. Audio or video URLs up to 3 hours, or up to 24 hours with `max_seconds` set that high. Stock phrases Whisper invents over silence ("Thanks for watching!") and repeated loops are removed, each with a flag holding the original text and timestamps. | $0.06 per audio hour (min. $0.01 a file) |
| `clean_data` | Clean rows from a CSV, Excel or JSON file, plus a log of every changed value, a review list, detected column types, data quality before and after, a JSON Schema and a reusable recipe. Exports to CSV and Excel. | $0.01 per 1,000 rows |
| `get_job_result` | The result of a long job (anything over about 100 seconds returns a `job_id`). | Free |

Measured quality, regenerated from the stored benchmark results on every deploy (a score taken on code that has since changed is withdrawn until it's re-measured). Each research question has a spending limit, so it never costs more than its price.

<!-- facts:quality-md -->
- **Transcription** (measured 2026-11-22): 98% of words right on clean read speech; 95% of words right on harder read speech; 91% of words right on parliament speeches; 85% of words right on earnings calls with accents; 85% of words right on crowded meetings. Measured on 5 public test sets, 489 recordings, 56 minutes of audio, with the same normalizer the public leaderboards use. Most misses are on overlapping speakers and phone-quality audio.
- **Research** (measured 2026-11-23): Simple factual questions (SimpleQA Verified, 100 public benchmark questions): when it answers, it's right 99% of the time (likely 94 to 100%), and it answers 92% of them; the rest come back unanswered, free; Questions that need several facts combined (FRAMES, 100 public benchmark questions): when it answers, it's right 79% of the time (likely 68 to 87%), and it answers 72% of them; the rest come back unanswered, free. A small test (100 questions each), so each figure has a range.
- **Data cleaning**: being re-measured after a change to the tool.
- **Speaker labels**: accuracy by kind of recording is on https://patchwell.io/trust.
<!-- /facts:quality-md -->

## Get a key

Free key with $1 of credit. It needs an email but no card. You get one per email and one per network every 30 days, and only a set number go out each day:

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
| `claims` | Each sentence again, with its page (URL and title) and the quote found on that page, so your code can check it without fetching anything |
| `sources` | The pages the answer rests on, with their titles |
| `gaps` | What the sources couldn't confirm, and how many claims the check dropped |
| `confidence` | `high`, `medium` or `low` |
| `billing` | What the call cost: $0.02 for an answer ($0.06 at `"depth": "deep"`), nothing when it declines or finds no sourced answer |

A worked-out claim (an age at an event, a difference between two years, who was older) carries `"derived": true`, `derived_from` (the facts it rests on) and `calc` (the arithmetic). Code re-checks that arithmetic using only the numbers and dates in those facts.

Before you see the answer, code checks each claim against the page it cites. The quote behind a claim has to appear on that page (the match ignores case, punctuation and spacing, and as a last resort accepts a passage holding most of the quote's words; the quote you get back is the page's own text). Then the same model judges twice, once plainly and once skeptically, whether the page supports the claim, and both must agree. The checker drops claims that fail and counts them in `gaps`. Last, a check asks whether the answer responds to exactly the question asked; if not, you get no answer and pay nothing. A passing claim tells you the page says it, so open the cited page before you rely on anything important.

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
- The research model, an open-weight model, runs at Sail Research, which keeps nothing after the request and never trains on it. Send `"private": true` and Sail never receives it: the model runs inside Modal, the cloud Patchwell runs on, which never trains on it and deletes it after processing. Wikipedia is searched first; the search providers listed in the privacy policy receive the search queries the tool writes from your question, and the GDELT news index receives its topic for lookups. Search results (titles, links, snippets) are cached for 3 days under a one-way fingerprint of the query. Every provider is listed in the [privacy policy](https://patchwell.io/privacy).
- Research declines questions about private individuals and about anyone's personal details (where they live, contact details, family, health, finances, tracking someone). Public figures' public roles and statements are fine, and so are widely known public or historical figures' encyclopedia facts (birth and death dates, age at an event, birthplace, career dates, an athlete's listed stats). If a key keeps sending declined questions, Patchwell pauses research on it for 24 hours.
- Speaker labels use voice embeddings computed in memory during one call and never stored. Send `"speakers": false` to skip them.
- The tools refuse URLs that point at private or internal network addresses.
- If Patchwell shuts down, you get at least 30 days' notice and a refund of all unused credit you paid for. Good-faith testing, including trying to break the refusals, is welcome: see the [terms](https://patchwell.io/terms#testing).

## About this repository

This repo holds the MCP Registry entry (`server.json`), the Claude Code plugin (`plugins/patchwell`), the Gemini CLI
extension (`gemini-extension.json`), the Cursor plugin (`.cursor-plugin/`) and these docs. The service runs at
patchwell.io.

Questions: patchwell.support@gmail.com
