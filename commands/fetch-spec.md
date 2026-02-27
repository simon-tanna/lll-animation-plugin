---
description: Fetch the LLL Animation Workshop spec from Notion
argument-hint: "[task-number-or-section]"
---

Use the plugin's Notion MCP server to fetch the LLL Animation Workshop specification.

The canonical spec page ID is `30dced2a7e13804eb784d06ef1b9e6a9`.

Use the Notion fetch tool with id `30dced2a7e13804eb784d06ef1b9e6a9` to retrieve the full spec page.

If the fetch fails (authentication error, page not found, or network issue), inform the user and suggest:
- Checking their Notion connection with `/mcp`
- Verifying they have access to the LLL Animation Spec page
- Trying again after re-authenticating

After a successful fetch:

1. If `$ARGUMENTS` is provided, locate the section or task matching the argument (e.g., "1.3", "Task 2.1", "Phase 2", "boundaries") and present only that portion with its acceptance criteria, hints, and gotchas.
2. If no argument is provided, present a structured overview of the spec:
   - Workshop title and summary
   - Phase 1 tasks (numbered list with one-line descriptions)
   - Phase 2 tasks (numbered list with one-line descriptions)
   - Any recent changes or updates noted on the page

Present the output as clean, readable markdown. Do not dump raw Notion API responses.
