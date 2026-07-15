# Advanced Music Genre Sorting - n8n Workflow

![Workflow canvas](workflow-canvas.png)

Production workflow running self-hosted (n8n on Docker / Synology DS923+).

## What it does
Watches a Dropbox folder for incoming audio promos, classifies genre using a
hybrid strategy, and auto-sorts files with confidence-based routing.

## Architecture
- Triple triggers: Manual Trigger, 15-min schedule, and webhook ingest - all converge on the same entry point
- Paginated Dropbox listing (handles folders beyond the API's single-page limit) with duplicate detection against already-sorted output
- Python audio analysis (ID3 tags, BPM via librosa) as primary genre detection
- Groq LLM fallback (Llama 3.3 70B via Groq's OpenAI-compatible API) for low-confidence tracks (below 0.6)
- Confidence routing: 0.7+ auto-sort, 0.4-0.7 needs-review, below 0.4 manual review
- Dropbox folder creation and file moves with retry/backoff
- Source-folder cleanup with an explicit safety guard that refuses to delete the root library folder, even if path resolution goes wrong
- Self-hosted monitoring: a webhook-triggered HTML dashboard (Chart.js) reading from a JSONL log, plus a dedicated error-logging path (no external notification service)
- Error path with continueOnFail on non-critical nodes

## Ops notes
Survived and was repaired through: volume migration (Docker), Dropbox OAuth
re-auth, path prefix fixes, pagination bugs on large folders, and a
cleanup-loop re-entrancy bug.

Built and iterated using AI-assisted development (Claude, Groq API tooling).
