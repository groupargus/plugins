---
name: insights
description: Scans every company in a Family active in the last 12 months with five fixed checks and points at the companies worth a look, each with the records behind it. Use when asked what to look into across customers, which accounts need attention, or whether anything is worth investigating.
allowed-tools: mcp__plugin_sunnny_sunnny__get_summary mcp__plugin_sunnny_sunnny__list_organizations mcp__plugin_sunnny_sunnny__get_organization_summary mcp__plugin_sunnny_sunnny__list_organization_records
---

# Sunnny Insights

In ChatGPT, `/insights` and `/overview` are the words `Insights` and `Overview`.

## Steps

1. **Text after the command.** None: run. A company name such as Adobe: reply only "For one company, run `/overview Adobe`. `/insights` on its own scans the whole portfolio." and call nothing. If unsure, it's other text. Other text: run, and after the opening sentence say `/insights` doesn't filter by Owner or date yet, so this covers the whole portfolio.
2. **Connector.** Call tools by bare name. With several Sunnny connectors, use the one named or ask which first, then use only that one.
3. **Connection.** Call `get_summary`. If it fails, say to reconnect and sign in. If empty, offer to help connect a system. Then stop.
4. **Today** is the run's date. Every window counts from it.
5. **Read.** Page `list_organizations` with `has_activity_since` today minus 12 months and `limit` 100. M is page 1's `filtered_count`. Per company: `get_organization_summary`, then nonzero `opportunities` and `tickets`. `meetings` only for a company in a multi-company Family with no opportunities or recent ticket. `custom-types` and a contract type's records only for a Check 2 candidate with no opportunity. `amount` is in minor units (cents for USD). Read whole Families in list order from page 1, never skipping or picking any, with no call budget of your own, until you can't go on.
6. **Rank** by each Check's lead rule, ties by name from A to Z. First leads, next 5 are similar. Keep the full list for "Ask me for the rest".

**Splitting the read.** When you can start sub-agents, one sub-agent pages `list_organizations` and returns M and a compact roster: per Family, each company's ID, name, parent, `last_activity_date`, whether it has a `source` entry, and its CRM `view_url`. Split the roster into batches of whole Families, about 35 Families each, and start one sub-agent per batch, at most 4 at a time. Give each step 5's reading rules and the Checks section verbatim, today's date, the connector, and its batch to read in place of the whole list. Each returns one compact row per candidate: Check, company, Family, the value it ranked on, and the dated, linked records behind it, with the stored priority of every unresolved ticket for Check 1 so the highest priority is judged across all batches. Retry a failed batch once. If it fails again, leave its companies out of N.

## The Checks

1. **Deal closing soon, open high-priority tickets.** An open deal closing within 60 days, at a company that itself has unresolved tickets at the highest priority. Lead: largest deal.
2. **Went quiet recently.** `last_activity_date` 3 to 12 months ago, with an opportunity or contract record. Lead: largest won total, 0 for none.
3. **Failure to launch.** A deal won 30 days to 12 months ago whose close date equals the company's `last_activity_date`. Lead: largest such deal.
4. **Buying signal, no opportunity added.** A ticket matching buy-more terms, opened over 60 days ago, with no opportunity created in its Family from its opened date to 60 days after. Lead: longest wait to the Family's next opportunity or to today, one per company.
5. **Families with potential whitespace.** In a Family of more than one company, a company with a `source` entry, no opportunities, and its own ticket or meeting in 12 months, while a sibling has a won deal. Lead: largest sibling win.

Compare stored values case-insensitively. Unresolved: status not meaning solved or closed. Highest priority: the top value that ticket's source stores, urgent over high over normal or medium over low. Won: a stage meaning won, such as `Closed Won`. Open: a stage not starting with "Closed". Buy-more terms, in subject plus description: "add seats", "upgrad", "enterprise tier", "more licenses", "expand", or "additional" then seats, users or licenses. Contract record: a Custom record whose type describes a contract or subscription.

Not run when `get_summary` lacks a Check's data: opportunities for 1, 3, 4 and 5, tickets for 1 and 4, tickets or meetings for 3 and 5. Name what to connect: a CRM for opportunities, support data such as Zendesk for tickets, a support or call system for tickets or meetings.

## Reply

Open with only "I ran all 5 checks over the M companies in Families active in the last 12 months on <connector>." (partial: "I read N of the M companies..."), plus step 1's sentence if any, and no other caveat. Then a rule, the five Checks, and nothing after.

Each Insight is a `### <n>. <Check>: <company>` heading, exactly three #, then bold-labeled Fact, Worth asking and Similar companies bullets, then `Explore deeper: /overview <company>`. A Check with no Insight gets a heading with no company and `- **Not run:** this check needs <data>. It would show <what>. Connect <system> to turn it on.` or `- **Found none:** I looked at <what> and found no <what>.`

- Add ` (<Family>)` after a company, in headings and Similar companies, only when its top-level company has another name.
- Fact: served data only, each record dated and under its own company, status and priority as stored. No inference.
- Worth asking: one or two questions, the only interpretation.
- Similar companies: up to 5, each with the value it ranked on, such as days waited, then "Ask me for the rest." if more exist.
- Link each company (its CRM entry), deal, meeting and ticket at first mention per Insight to `source.view_url`, placeholder hosts too. Null stays plain. No app.sunnny.com.
- No names of people, emails, phones, field codes or underscores. Concrete dates.

**Example.** Take every company and number from the data, never from this example:

```
### 4. Buying signal, no opportunity added: Deutsche Bank

- **Fact:** [Deutsche Bank](view_url) asked support to add seats in [#243 Add seats to the Deutsche Bank plan](view_url) on 2025-10-22. No opportunity was logged in its Family until [Deutsche Bank - Seat Expansion](view_url) ($230,000) on 2026-08-02, 284 days later.
- **Worth asking:** Did anyone on sales see #243 before August? Its description is about renewal pricing.
- **Similar companies:** [BMW of North America](view_url) (BMW Group), waited 85 days. Ask me for the rest.

Explore deeper: `/overview Deutsche Bank`
```
