# SUNNNY plugins

Connects your AI assistant to your SUNNNY data (CRM, support, and call records) and adds a skill that writes an executive overview of any customer.

## Install in Claude

Works in Claude on the web, the Claude desktop app, and Cowork.

1. Open Customize → Plugins. On the Discover tab, find SUNNNY and choose Add.
2. Open the plugin's Connectors tab, choose Connect, and sign in with your SUNNNY account. If Claude asks you to finish connecting a connector, choose Continue connecting.

## Install in ChatGPT

Find SUNNNY in ChatGPT's plugin directory and add it. When ChatGPT asks you to connect, sign in with your SUNNNY account.

## Usage

You can ask for an overview of any customer organization with the /overview skill. For example: /overview LVMH.

You can also ask questions about your data, such as "What is the most recently closed deal?". If the data isn't in your account, the agent says so.

## Data

The plugin connects to the SUNNNY MCP server at https://api.sunnny.com/mcp. Your ability to install plugins is dependent on your user role permissions within your Claude or ChatGPT plan.

To answer a question, the agent sends read-only queries to that server and gets back records from your SUNNNY account, such as companies, contacts, deals, meetings, calls, and support tickets. The plugin can't create, change, or delete records.

You sign in through SUNNNY’s OAuth page at auth.sunnny.com, so the plugin never sees your password. The plugin stores no data itself and sends data to no other service.
