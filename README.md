# Claude-learnings
# Multi-Agent Sub-Agents — Proof of Work

Two working demos of Claude Code sub-agents talking to each other.

- **Coder + Reviewer** — no hooks, delegation happens via proactive agent descriptions.
- **News-Gatherer + Emailer** — wired with a `SubagentStop` hook and the Gmail MCP connector.

## Layout

```
.claude/
├── agents/
│   ├── coder.md
│   ├── reviewer.md
│   ├── news-gatherer.md
│   └── emailer.md
├── memory/
│   ├── state.md          # coder/reviewer round log
│   ├── learnings.md      # past pitfalls
│   ├── news_draft.md     # verified stories from news-gatherer
│   └── email_ready.md    # final email drafted by emailer
└── settings.json         # SubagentStop hook config
```

## Prerequisites

- Claude Code installed
- Gmail account (only for Use Case 2)

## Use Case 1 — Coder + Reviewer

No setup beyond the two agent files.

```
claude
> Use the coder agent to create calc.py with a function divide(a, b) …
```

Coder writes the code → reviewer runs the tests → verdict lands in `.claude/memory/state.md`.

## Use Case 2 — News-Gatherer + Emailer

**1. Connect Gmail**

```
claude mcp add gmail
```

Sign in through the browser, grant send permission.

**2. Hook is already in `.claude/settings.json`**

```json
{
  "hooks": {
    "SubagentStop": [
      {
        "matcher": "news-gatherer",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "The news-gatherer subagent finished and wrote .claude/memory/news_draft.md. Now invoke the emailer subagent to read that file and create the Gmail draft."
          }
        ]
      }
    ]
  }
}
```

**3. Run**

```
claude
> Use the news-gatherer agent to collect today's top 10 verified AI news stories.
```

News-gatherer fetches + verifies stories → hook fires → emailer creates a Gmail draft → check `in:draft` in Gmail and hit **Send**.

## Notes

- Coder never runs tests — that's the reviewer's job.
- Reviewer never edits code — only verifies and reports.
- Emailer creates a *draft*, not a sent email. You send it.
- Both flows log to `.claude/memory/` so nothing is lost between rounds.
