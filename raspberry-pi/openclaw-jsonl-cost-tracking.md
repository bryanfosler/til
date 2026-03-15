# OpenClaw session JSONL already tracks cost per message

OpenClaw writes session data to JSONL files at `~/.openclaw/agents/main/sessions/*.jsonl`.
Each `type: "message"` entry includes a fully-computed cost breakdown — no need to
reconstruct spend from raw tokens and pricing tables.

```json
{
  "type": "message",
  "message": {
    "role": "assistant",
    "model": "gpt-5.4",
    "usage": {
      "input": 9355,
      "output": 211,
      "cacheRead": 4096,
      "cacheWrite": 0,
      "totalTokens": 13662,
      "cost": {
        "input": 0.023,
        "output": 0.003,
        "cacheRead": 0.001,
        "cacheWrite": 0,
        "total": 0.0275
      }
    }
  }
}
```

Sum `message.usage.cost.total` across all assistant messages in the current month's
session files to get total spend — `timestamp` on each entry is ISO-8601 UTC.
