# Obsidian Tasks + Dataview: A PM's Reference

**Date:** 2026-05-23

Two community plugins that turn Obsidian into a real task management system: **Tasks** gives every checkbox a structured metadata layer (due dates, priorities, recurrence), and **Dataview** lets you query that metadata across your entire vault like a database.

---

## Tasks Plugin

### Why it earns its place

Vanilla Obsidian checkboxes are dumb — tick it, it's done. Tasks adds structure to every `- [ ]` item so you can filter, surface, and resurface tasks from anywhere in the vault. The killer use-case: write a task in your project note, surface it on your Daily Note via a query, and it stays in sync because it lives in one place.

### Emoji syntax cheat sheet

Every field goes inline on the task line, after the description:

| Emoji | Field | Example |
|-------|-------|---------|
| 📅 | Due date | `📅 2026-05-30` |
| ⏳ | Scheduled (plan to work on it) | `⏳ 2026-05-27` |
| 🛫 | Start (earliest you can work on it) | `🛫 2026-05-25` |
| ➕ | Created date (auto-added) | `➕ 2026-05-23` |
| ✅ | Done date (auto-added on completion) | `✅ 2026-05-23` |
| 🔁 | Recurring | `🔁 every week` |
| 🔺 | Highest priority | — |
| ⏫ | High priority | — |
| 🔼 | Medium priority | — |
| (none) | Normal priority | — |
| 🔽 | Low priority | — |
| ⏬ | Lowest priority | — |

Full example:
```
- [ ] Finalize Berntsen interview deck ⏫ 📅 2026-05-28 🔁 every week
```

### Recurring task patterns

After `🔁`, anything that starts with "every" is valid:

```
🔁 every day
🔁 every weekday
🔁 every week on Monday
🔁 every month on the 1st
🔁 every 2 weeks
🔁 every month on the last Friday
🔁 every week when done          ← next due = completion date + 7 days (not fixed anchor)
```

When you check off a recurring task, Tasks marks the original done and creates a new task above it with the next date. **Important:** recurring tasks need at least one date field (📅, ⏳, or 🛫) or the recurrence is inert.

### Querying tasks

Embed a query anywhere with a ` ```tasks ``` ` code block. Each line is an implicit AND:

**All incomplete tasks due this week:**
````
```tasks
not done
due in this week
```
````

**Overdue tasks, sorted by priority:**
````
```tasks
not done
due before today
sort by priority
sort by due
```
````

**High-priority tasks not yet started:**
````
```tasks
not done
priority is above medium
```
````

**Everything in a specific project folder:**
````
```tasks
not done
path includes Projects/Berntsen
```
````

**Tasks tagged #waiting, grouped by file:**
````
```tasks
not done
tags include #waiting
group by filename
sort by due
```
````

**This week's tasks, grouped by due date:**
````
```tasks
not done
due in this week
group by due
sort by priority
```
````

Filters are case-insensitive as of Tasks v5.2.0. Implicit AND between lines — no `AND` keyword needed.

---

## Dataview Plugin

### Why it earns its place

Dataview treats every note as a database row and every YAML frontmatter key / inline field as a column. The killer use-case: pull a list of all session logs you tagged `#cairn` this week without maintaining any manual index.

### Query structure

Embed with a ` ```dataview ``` ` code block. SQL-like but not SQL:

```
<QUERY TYPE> [fields]
FROM <source>
WHERE <condition>
SORT <field> ASC|DESC
GROUP BY <field>
LIMIT <n>
```

Only the Query Type is required. Sources can be a folder path (quoted), a tag, or a link.

### Query type quick reference

| Type | Output |
|------|--------|
| `LIST` | Bullet list of matching pages |
| `TABLE col1, col2` | Table with columns |
| `TASK` | Interactive task checklist |
| `CALENDAR due` | Calendar dots by date |

### Useful vault queries

**Recent session logs tagged with a project:**
````
```dataview
LIST
FROM "AI Knowledge/Claude Code/sessions"
WHERE contains(tags, "#cairn")
SORT file.ctime DESC
LIMIT 10
```
````

**All notes modified this week:**
````
```dataview
TABLE file.mtime as "Modified", tags
FROM ""
WHERE file.mtime >= date(today) - dur(7 days)
SORT file.mtime DESC
```
````

**Incomplete tasks across the vault (grouped by file):**
````
```dataview
TASK
WHERE !completed
GROUP BY file.link
```
````

**Incomplete tasks from a specific folder only:**
````
```dataview
TASK
FROM "Projects"
WHERE !completed
SORT due ASC
```
````

**Notes with a specific frontmatter status:**
````
```dataview
TABLE status, file.ctime as "Created"
FROM "Projects"
WHERE status = "active"
SORT file.ctime DESC
```
````

---

## Combining Tasks + Dataview in a Daily Note

This is where they shine together. The pattern: Tasks owns the task markup; Dataview handles cross-vault aggregation.

**Option 1 — Native Tasks query in Daily Note (recommended)**

Paste this in your Daily Note template:

````
## Today's Tasks

### Due Today
```tasks
not done
due today
```

### This Week
```tasks
not done
due in this week
not due today
group by due
```

### Overdue
```tasks
not done
due before today
sort by priority
```
````

The Tasks plugin renders these live. Checking a task here updates it in the source file.

**Option 2 — Dataview pulls session logs linked to today**

Useful for the review section of a Daily Note:

````
```dataview
LIST
FROM "AI Knowledge/Claude Code/sessions"
WHERE file.day = date(today)
```
````

**Critical gotcha for recurring tasks:** Do NOT use a `TASK` Dataview query to check off recurring tasks. Dataview will mark it done but will NOT generate the next occurrence. Navigate to the source file and use Tasks' native toggle instead.

---

## Gotchas

**Dataview is community-maintained — updates can lag.** If Obsidian updates break something, check the [Dataview GitHub issues](https://github.com/blacksmithgu/obsidian-dataview/issues) before assuming your query is wrong.

**Emoji collision on mobile.** iOS and Android keyboard autocorrect can substitute similar-looking emoji for Tasks' specific ones. The Tasks emoji shorthand (typing `due` and letting autocomplete replace it) is more reliable than copying emoji from other apps — copied emoji can contain Unicode Variation Selectors that Tasks won't parse.

**Dataview TASK queries are slow on large vaults.** Each query re-indexes on load. On a vault with hundreds of notes, a dashboard with 5+ TASK queries will noticeably lag. Scope queries tightly with `FROM "folder"` rather than scanning the whole vault.

**Dataview doesn't know about recurrence.** Checking a `🔁` recurring task from a Dataview TASK block adds a done date but skips creating the next occurrence. This is a known limitation: Tasks and Dataview handle checkbox events independently.

**DataviewJS triggers a security warning.** If you use the more powerful `dataviewjs` blocks (JavaScript API), Obsidian will warn you on first load. DQL (plain `dataview` blocks) does not trigger this.

**`date(today)` is Dataview syntax, not Tasks syntax.** In Tasks queries use natural language (`due today`, `due in this week`). In Dataview queries use `date(today)` and `dur()` for date math. They are not interchangeable.

---

## Sources

- [Tasks Emoji Format — Official Docs](https://publish.obsidian.md/tasks/Reference/Task+Formats/Tasks+Emoji+Format)
- [Tasks Filters Reference](https://publish.obsidian.md/tasks/Queries/Filters)
- [Tasks Grouping Reference](https://publish.obsidian.md/tasks/Queries/Grouping)
- [Tasks Recurring Tasks Guide](https://publish.obsidian.md/tasks/Getting+Started/Recurring+Tasks)
- [Dataview Query Structure](https://blacksmithgu.github.io/obsidian-dataview/queries/structure/)
- [Tasks + Dataview Integration (Official)](https://publish.obsidian.md/tasks/Other+Plugins/Dataview)
