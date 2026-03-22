# SQLite `localtime` Returns UTC When Your Server Is UTC

When you use `strftime('%H', timestamp/1000, 'unixepoch', 'localtime')` in SQLite to bucket events into local hours, it converts to the *server's* local timezone — not the user's browser timezone.

If your server runs UTC (Raspberry Pi default), `'localtime'` **is** UTC. So a user in CDT (UTC-5) doing work at 9am will see their events grouped into hour 14 on a chart, and their morning appears empty.

## The Fix: Return UTC Buckets, Let the Browser Convert

Instead of asking SQLite for a local hour string, return a UTC millisecond timestamp floored to the hour:

```sql
-- Before (broken on UTC servers)
SELECT strftime('%H', ts_ms/1000, 'unixepoch', 'localtime') AS hour

-- After (timezone-safe)
SELECT (ts_ms / 3600000) * 3600000 AS bucketTs
```

Then in the browser, generate local-time display slots and map each to its UTC bucket:

```javascript
const localMid = new Date();
localMid.setHours(0, 0, 0, 0);

for (let h = 0; h <= currentHour; h++) {
  const slotMs = localMid.getTime() + h * 3600000;
  const utcBucket = Math.floor(slotMs / 3600000) * 3600000;
  const record = dataByBucket[utcBucket] || { costUsd: 0 };
  // render slot at label `h + ":00"`
}
```

## Rule of Thumb

**Servers speak UTC. Browsers speak local time.** Never do timezone-to-local conversion on the server for data that will be displayed to users in potentially different timezones. Return UTC timestamps and let the client handle display conversion.
