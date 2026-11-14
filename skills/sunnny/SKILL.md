---
name: sunnny
description: Lists the Sunnny skills and checks the Sunnny connection. Use when the user types Sunnny on its own, asks what Sunnny can do, or asks which Sunnny skills are available.
allowed-tools: mcp__plugin_sunnny_sunnny__get_summary
---

# Sunnny

Show the user which Sunnny skills they can run here, once you know Sunnny answers.

1. **Check the connection.** Call `get_summary` once.
   - The call fails or asks for sign-in: tell the user Sunnny isn't connected and to reconnect it and sign in again. Stop here.
   - It returns an empty object: no systems are connected to Sunnny yet. Say so, and offer to walk them through connecting one. Then continue to step 2.
   - Anything else: say in one line that Sunnny is connected.
2. **List the skills.** Show every row of the table below, with its command and what it gives the user, worded as written.

| Skill | Run it with | What it gives you |
|---|---|---|
| Overview | `/overview [company name]` | An executive report on one company and its subsidiaries, covering overview, revenue, engagement, support, and product activity. |
