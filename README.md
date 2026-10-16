# Sunnny plugin for Claude

Connects Claude to your Sunnny data (CRM, support, and call records) and adds a skill that writes an executive overview of any customer.

The plugin is in beta. Sign-in only works for Sunnny accounts enabled for the beta. To get access, reach out to your contact at Sunnny.

## Install

Works in Claude on the web, the Claude desktop app, and Cowork.

1. Open **Customize → Plugins**, choose **Add → Add marketplace**, enter `groupargus/plugins`, and choose **Sync**.
2. On the **Discover** tab, find **Sunnny** and choose **Add**.
3. Open the plugin's **Connectors** tab, choose **Connect**, and sign in with your Sunnny account. If Claude asks you to finish connecting a connector, choose **Continue connecting**.

## Use it

You can ask for an overview of any customer organization with the `/overview` skill. For example: `/overview LVMH`.

You can also ask questions about your data, such as "What is the most recently closed deal?". If the data isn't in your account, Claude says so.

## Upgrading from the manual setup

If you set up Sunnny earlier with a custom connector and the `sunnny-claude-skill-upload.zip` skill, remove both after installing the plugin. Otherwise Claude shows the skill and the connector twice.

1. **Customize → Skills**: remove `sunnny-overview`.
2. **Customize → Connectors**: remove your custom Sunnny connector.
