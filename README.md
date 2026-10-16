# Sunnny plugins

Connects your AI assistant to your Sunnny data (CRM, support, and call records) and adds a skill that writes an executive overview of any customer.

The plugin is in beta. Sign-in only works for Sunnny accounts enabled for the beta. To get access, reach out to your contact at Sunnny.

## Install in Claude

Works in Claude on the web, the Claude desktop app, and Cowork.

1. Open **Customize → Plugins**. On the **Discover** tab, find **Sunnny** and choose **Add**.
2. Open the plugin's **Connectors** tab, choose **Connect**, and sign in with your Sunnny account. If Claude asks you to finish connecting a connector, choose **Continue connecting**.

If Sunnny doesn't appear on the **Discover** tab, add its marketplace first. Choose **Add → Add marketplace**, enter `groupargus/plugins`, choose **Sync**, and then follow step 1.

### Upgrading from the manual setup

If you set up Sunnny in Claude earlier with a custom connector and the `sunnny-claude-skill-upload.zip` skill, remove both after installing the plugin. Otherwise Claude shows the skill and the connector twice.

1. **Customize → Skills**: remove `sunnny-overview`.
2. **Customize → Connectors**: remove your custom Sunnny connector.

## Install in ChatGPT

Coming soon.

## Use it

You can ask for an overview of any customer organization with the `/overview` skill. For example: `/overview LVMH`.

You can also ask questions about your data, such as "What is the most recently closed deal?". If the data isn't in your account, the assistant says so.

## Data

The plugin connects to Sunnny's MCP server at `https://api.sunnny.com/mcp`. To answer a question, the assistant sends read-only queries to that server and gets back records from your Sunnny account, such as companies, contacts, deals, meetings, calls, and support tickets. The plugin can't create, change, or delete records.

You sign in through Sunnny's OAuth page at `auth.sunnny.com`, so the plugin never sees your password. The plugin stores no data itself and sends data to no other service.
