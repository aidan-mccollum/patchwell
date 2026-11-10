# Installing Patchwell

Patchwell is a hosted MCP server. There's nothing to download, build or run locally.

1. Ask the user for their Patchwell API key. It starts with `pw_live_`. If they don't have one, they can get a free key
   with $1 of credit (no card; one per email and one per network every 30 days, with a daily cap on free keys) by
   running this with their own email address:

   ```bash
   curl -X POST https://patchwell.io/v1/credits/trial -H "Content-Type: application/json" -d '{"email": "you@example.com"}'
   ```

   The key is in the `api_key` field of the answer. They can also get one at https://patchwell.io/quickstart.

2. Add the server to the MCP settings file. For Cline that's `cline_mcp_settings.json`:

   ```json
   {
     "mcpServers": {
       "patchwell": {
         "type": "streamableHttp",
         "url": "https://patchwell.io/mcp/",
         "headers": {
           "Authorization": "Bearer pw_live_YOUR_KEY"
         },
         "disabled": false,
         "autoApprove": []
       }
     }
   }
   ```

3. Check that the server shows five tools: `research`, `transcribe`, `clean_data`, `get_job_result` and `listen_clip`.
   Listing tools works without a key; calling one needs it.

Each call spends the key's credit: $0.15 per answered research question (declined ones are free), $0.14 per audio
hour (minimum $0.02 a file), $0.50 per 1,000 rows. Long jobs return a `job_id`; fetch the result with
`get_job_result`.
