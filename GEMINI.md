# Patchwell

Patchwell adds paid tools. Each call spends the user's prepaid Patchwell credit, so use one only when the task needs it.

- `research`: answers a question from the web, with a cited page for each sentence and a list of what the sources
  couldn't confirm. Takes about a minute. $0.15 per answered question; declined or unanswered questions cost nothing.
  It won't look up private people.
- `transcribe`: turns an audio or video URL into text with word times, speaker labels, SRT and VTT. $0.008 per audio
  minute.
- `clean_data`: cleans a CSV, Excel or JSON file and returns the rows plus a log of every changed value. $0.50 per
  1,000 rows.
- Long jobs return a `job_id`. Call `get_job_result` with it until the status is `done`.

If the tools aren't connected yet, tell the user to run `/mcp auth patchwell`. A browser page asks for their Patchwell
key or gives them a free one with $1 of credit.
