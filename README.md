# Wise Owl MCP server

[Wise Owl](https://wiseowl.cat/) is a daily to-do list for software engineers: jot down the reviews, interruptions and small things of the day, and whatever you don't finish moves to tomorrow on its own. Its MCP server lets Claude, ChatGPT, Cursor, VS Code or any MCP client read and update your list. Ask in plain words: "what's still open this week?", "add the follow-ups from this PR for tomorrow", "plan my day".

```
https://api.wiseowl.cat/mcp
```

It's a remote server (Streamable HTTP): there's nothing to install or run. This repository holds only its documentation; the server itself isn't open source.

The setup for each assistant, in English, Catalan and Spanish, is also at **[wiseowl.cat/en/connect](https://wiseowl.cat/en/connect/)**.

<!-- Demo GIF: an assistant planning the day with Wise Owl (about 20 seconds). Add it here, e.g. ![Claude planning the day with Wise Owl](demo.gif) -->

## You need an account

The assistant works with the list your Wise Owl account keeps in sync, so you need a free account: [sign up at wiseowl.cat](https://wiseowl.cat/signup). Without an account, Wise Owl still works in your browser, but your list never leaves it, so no assistant can reach it.

## Set it up

Claude and ChatGPT sign in with OAuth: they only need the address, and you log in to Wise Owl and allow them, with no keys to copy. For Claude Code, Cursor, VS Code, Windsurf and other clients, [create an access token](#create-an-access-token) and add it with the same address.

### Claude (web and desktop)

1. In Claude, open **Settings → Connectors** and add a custom connector.
2. Paste Wise Owl's address: `https://api.wiseowl.cat/mcp`
3. Log in to Wise Owl and allow access. No keys to copy: it's OAuth.

The steps are the same in the Claude desktop app.

### ChatGPT

ChatGPT connects with the same address: add a custom connector with it, then log in to Wise Owl and allow it.

### Create an access token

Claude Code, Cursor, VS Code and Windsurf connect with a token:

1. Log in to [Wise Owl](https://wiseowl.cat/login) and open **User configuration → Connected assistants**.
2. Write the assistant's name and click **Create token**.
3. Copy the token, which starts with `wo_pat_`, right away: it won't be shown again. It can read and change your tasks, and it expires after 90 days.

Treat it like a password, and keep it out of files you commit. In the examples below, put your token in place of `wo_pat_…`.

### Claude Code

```sh
claude mcp add --transport http wiseowl https://api.wiseowl.cat/mcp --header "Authorization: Bearer wo_pat_…"
```

When you create a token, Wise Owl shows this command with the token already in it.

### Cursor

Add Wise Owl to `~/.cursor/mcp.json` for every project, or to `.cursor/mcp.json` in one project:

```json
{
  "mcpServers": {
    "wiseowl": {
      "url": "https://api.wiseowl.cat/mcp",
      "headers": { "Authorization": "Bearer wo_pat_…" }
    }
  }
}
```

### VS Code

Add Wise Owl to `.vscode/mcp.json` in your project, or to your user configuration (**MCP: Open User Configuration** in the Command Palette). VS Code asks for the token when the server first starts, so it stays out of the file:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "wiseowl-token",
      "description": "Wise Owl access token",
      "password": true
    }
  ],
  "servers": {
    "wiseowl": {
      "type": "http",
      "url": "https://api.wiseowl.cat/mcp",
      "headers": { "Authorization": "Bearer ${input:wiseowl-token}" }
    }
  }
}
```

### Windsurf

Windsurf, now called Devin Desktop, reads its MCP servers from `mcp_config.json`. Add Wise Owl there:

```json
{
  "mcpServers": {
    "wiseowl": {
      "serverUrl": "https://api.wiseowl.cat/mcp",
      "headers": { "Authorization": "Bearer wo_pat_…" }
    }
  }
}
```

### Other MCP clients

Any client that supports Streamable HTTP can connect: give it the address and, if it can't sign in with OAuth, the header `Authorization: Bearer wo_pat_…` with your token.

## Tools

The tools follow the same rules as the app: unfinished tasks carry over to today, and each board (a separate list, like "Work" and "Personal") keeps its own days. The read tools cover every board unless the assistant names one. Dates are `YYYY-MM-DD` or today / tomorrow / yesterday, in your time zone.

**Read**

| Tool | What it does |
| --- | --- |
| `list_boards` | Your boards, and how many tasks are open on each today. |
| `get_day` | A day's tasks, with their status and how many times each was carried over. Opening today first brings over what's unfinished, as the app does. In Claude, the day shows up as a list you can tick right in the chat. |
| `list_days` | The tasks of a range of days, up to 31. |
| `search_tasks` | Finds tasks by their text, and if you like by status or between two dates. |
| `get_stale_tasks` | The open tasks that keep being carried over: candidates to do now, break down or drop. |
| `get_summary` | How many tasks were done, carried over, dropped or left open in a period of up to 92 days, in total and per day. |

**Write**

| Tool | What it does |
| --- | --- |
| `add_tasks` | Adds up to 50 tasks at once, for today or a later day, on the board you say. |
| `set_status` | Marks a task done, open again, won't do, or for the next day. |
| `edit_task` | Rewrites a task's text. |
| `move_task` | Plans an open task for another day, like pushing it in the app. |
| `delete_task` | Deletes a task from every day it's on. It's flagged as destructive, so a well-behaved assistant asks you first. |

**Prompts**

| Prompt | What it does |
| --- | --- |
| `plan_my_day` | Looks at today and the tasks that keep moving, and proposes a plan without changing anything until you say so. |
| `weekly_review` | Reviews the last seven days: what you got done, what kept moving and what to change. It doesn't change any task. |

## Permissions

There are two permissions, or scopes:

- `tasks:read`: read your tasks and days.
- `tasks:write`: add, change, move and delete your tasks.

When you connect an assistant with OAuth, Wise Owl shows you what it asks for, and you allow or deny it. An access token has both.

Assistants can read and change tasks and nothing else: not your password, your account or your billing. Your settings list what each assistant added, rewrote, moved or deleted, with an Undo next to each change.

## Revoke access

In Wise Owl, open **User configuration → Connected assistants**. Click **Disconnect** next to an assistant you connected with OAuth, or **Revoke** next to an access token. It stops working at once, and an assistant you disconnect has to ask you again to come back. Deleting your account revokes them all.

Removing Wise Owl from the assistant's settings doesn't revoke its access on its own: do it in Wise Owl too.

## Privacy

- An assistant sees the text of the tasks it reads, and your AI provider handles that text under its own terms. Connect only the assistants you trust with it.
- Wise Owl's logs record which tool an assistant called, how long it took and whether it worked, never the arguments or the replies, which hold your task text. The list of changes in your settings keeps what each assistant changed, so you can undo it.
- Without an account, your tasks stay in your browser's local storage and never reach Wise Owl's servers.

The full policy is at [wiseowl.cat/en/privacy-policy](https://wiseowl.cat/en/privacy-policy/).

## Links

- How to connect each assistant: https://wiseowl.cat/en/connect/
- Wise Owl: https://wiseowl.cat/
- Feedback and support: hola@wiseowl.cat

## License

The documentation in this repository is under the [MIT License](LICENSE).
