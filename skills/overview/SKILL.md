---
name: overview
description: Executive summary of one company and its subsidiaries from the Sunnny database, delivered as a five-theme report. Use when asked for an overview of, or to summarize, brief on, or analyze, a company, customer, or account, including account reviews, QBR prep, customer health checks, or "what do we know about" a company.
argument-hint: [company name]
allowed-tools: mcp__plugin_sunnny_sunnny__get_summary mcp__plugin_sunnny_sunnny__list_organizations mcp__plugin_sunnny_sunnny__get_organization mcp__plugin_sunnny_sunnny__get_organization_summary mcp__plugin_sunnny_sunnny__list_organization_records mcp__plugin_sunnny_sunnny__list_contact_records
---

# Sunnny Overview

Act as a senior business operations analyst delivering a live briefing to the CEO about one company. Your evidence is the Sunnny database, read through the Sunnny MCP tools, holding unified customer data from CRM, support, and communication systems (Salesforce, HubSpot, Zendesk, Jira, Slack, Gong). Your deliverable is a written report, not a spreadsheet.

Every claim traces to a field in the dataset. Where the data is missing, the report says so explicitly (see the theme gates below) instead of filling the gap.

## Process

1. **Confirm which company.** Search for the requested name with `list_organizations`, whose `filter` matches any company in a family and returns each matching family whole, top-level company first. Count matches by the families in `organizations`, never by `filtered_count`, which counts companies. One family: proceed. Several families: list their top-level names and ask which one before running. Subsidiaries of one family don't count as separate matches — the family step handles those.
2. **Resolve the family of companies.** The step 1 result already holds the whole family tree, from the top-level company down; call `get_organization` only when you start from an organization ID. Expand the analysis to the full corporate family. Record lists cover one company only, with no roll-up from subsidiaries, so call `get_organization_summary` for each company in the tree first and let its counts drive the rest. Skip a company whose contacts, tickets, meetings, and opportunities are all zero: if Marriott International has 3 active subsidiaries behind an empty Starwood holding node, analyze the 3 and bypass the node. For the rest, call `list_organization_records` for each kind with a nonzero count, and for `custom-types` always, since the summary doesn't count custom records. Done when every subsidiary that has data is in scope.
3. **Map the data.** Accounts and Contacts are the core entities. Meetings, calls and conversations, deals, and tickets are historical activity belonging to those companies and people, never free-standing facts. Reach a contact's own meetings, tickets, and deals with `list_contact_records`, passing the contact's `id`; deals list their contacts by source-system ID, so match those against each contact's `source_ids`. Custom record types hold what the standard kinds don't, often contracts and surveys: kind `custom-types` lists them, and passing a type's `id` as `type_id` returns its records, with fields as key and value pairs under `properties`. Read the fields the records actually return rather than guessing names. Money amounts (`value`, `annual_revenue`) are in cents, so divide by 100; values inside custom records are already formatted. Ticket status and priority spellings differ by source system, so compare them case-insensitively and count `new` with `open` as unresolved.
4. **Compute portfolio baselines.** The Comparative Lens needs something to compare against: `get_summary` counts the tickets, meetings, opportunities, and contacts across the whole portfolio (use `merged_count` for contacts). For the number of families, page through `list_organizations` with `has_activity_since` set to an early date such as 2000-01-01 and count the entries in `organizations`. Neither `total_count` nor `filtered_count` works: both count companies, not families. Most families have no activity at all, so dividing by every family shrinks the average about tenfold. Divide the counts by the active families for per-family averages, such as tickets or meetings per family. The tools can't produce portfolio-wide rates or dollar figures, so compare open rates, pipeline, and booked revenue against the family's own history instead, such as this year against last. Show the math when you cite one.
5. **Gate each theme** against its Minimum Required Inputs (see the theme table), then write it in two voices.
6. **Write the Short Summary last**, once the themes are done, ending on The Read.
7. **Run the pre-flight checklist** before delivering. Done when every box passes.

## Report shape

```
# {Company} Organization Summary
## Short Summary
  (one line per theme, labeled with these short names instead of the full
   theme names: Overview / Revenue / Engagement / Support / Product Activity.
   Each carries its headline number, and the summary closes on The Read)
## Theme 1 … Theme 5, each with:
  Key Insights / Supporting Facts / Comparative Lens / Data Sources
```

**The Read** closes the Short Summary and appears nowhere else — the report ends on Theme 5, not on a trailing verdict. It is judgment across all five themes: the theme lines report, The Read commits, naming the real risk even when it is not the obvious one. From a real report: "The Read: an average-sized account with above-average support friction, a go-live that may never have happened, and no executive above EVP. The $105,000 open deal is not the risk — the $407,500 already booked is, with no usage data to say whether they use what they bought."

## The five themes and their gates

Exactly these five, in this order. A theme passes its gate when enough of its listed inputs exist to support analysis; it fails when they are missing, empty, or too sparse.

| Theme | Question it answers | Minimum Required Inputs | Insights to consider |
|---|---|---|---|
| 1. Customer Overview | Quick overview of this customer | Account profile (industry, employees, revenue; segment if a source has it); company tree (conglomerate or independent); account type (customer, prospect, lead); plan or product, usually from a contracts custom record | Recently new customer; prospect subsidiaries inside a large company tree |
| 2. Customer Value | What is this customer worth to me | Contract value and end dates, usually from a contracts custom record; opportunity pipeline with dollar value, stage, close dates; total across parent and subsidiaries | Upcoming contract expirations; deal close dates; deals at risk from support ticket volume |
| 3. Customer Engagement Level | How strong and strategic is the relationship | Engagement volume (meetings, calls, conversations); NPS/CSAT or surveys, usually from a survey custom record; contact count, seniority distribution (C-level through Director) from contact and meeting-participant titles, functional diversity from contact departments | Low or dropping engagement; who shows up in meetings and deals; missing senior contacts; recurring conversation topics |
| 4. Customer Support | How is the support experience | Tickets with priority and status; CSAT/NPS or SLA metrics; community engagement | Volume and trends; recurring ticket themes; high-priority or urgent tickets; bad sentiment |
| 5. Product Activity | Is the customer getting value | Product telemetry (DAU/WAU/MAU, logins, feature usage) OR a usage proxy (API calls, exports) | Low or declining usage; power users; active users vs. licensed seats; unused features |

Calibration from a real run: meetings plus contact depth passed the Engagement gate even with NPS absent (the absence was declared in the facts); support-ticket anecdotes alone failed the Product Activity gate.

### When a theme passes

Write 3–5 Key Insights, each backed by Supporting Facts, and every insight carrying a date, name, dollar value, or quoted comment. Add a Comparative Lens where data permits: a historical trend, peer benchmark, or portfolio average.

Inside a passing theme, name the standard inputs that are absent ("no contract end dates in the dataset, so churn can't be measured") rather than silently skipping them.

### When a theme fails

Set Key Insights to exactly `Insufficient Data`. In Supporting Facts, bullet the missing inputs and the nearest available signal, if any. Leave the Comparative Lens empty and record Data Sources as none available. A theme is fully written or explicitly empty, never half-filled.

## Two voices

Key Insights are full, natural sentences you would say aloud in a briefing — the strategic read. Supporting Facts are short, data-dense bullets — the evidence trail, with record IDs, source fields, and deep links to records where available.

### Exemplar (Customer Support, adapted from a real report)

Key Insights:
- Support is this family's weak spot: 11 of its 20 tickets are still open, and 20 tickets is eight times the portfolio average of 2.5 per family.
- Tickets are aging badly — 8 of the 11 open ones have been sitting for six months, and the oldest, Sephora asking to add seats to its plan, has been open since June 13, 2025.
- Two of the stalled tickets are customers trying to give us money: that Sephora add-seats request and Moet's high-priority enterprise-tier upgrade from May 2026.

Supporting Facts:
- 20 tickets total: 11 open, 9 solved (55% open). By company: LVMH 1 of 4 open; Moet 2 of 4; Sephora 5 of 8; Tiffany 3 of 6
- Oldest open ticket: Sephora "Add seats to the Sephora plan" #143, opened 2025-06-13, ~13 months (deep link to the ticket)
- No CSAT, sentiment, or SLA metrics in the dataset

Comparative Lens: Active families average 2.5 tickets (674 tickets across the 267 families with any activity). This family has 20, and its oldest open ticket predates every one of its expansion deals.

Data Sources: Zendesk Tickets

Note the split: "11 of 20 tickets open, eight times the portfolio average" appears as analysis in the insight and as the raw count in the facts — the insight says what it means, the fact makes it checkable.

## Conflicting and duplicated data

Source systems overlap. When the same deal, contact, or account appears in two systems with conflicting values, count it once, state the conflict, and give the range instead of a single false-precision number ("booked value is $1.43M to $4.23M depending on which system's records you trust; each initial license has a duplicate Salesforce/HubSpot pair"). Surfacing a data-quality problem is itself an insight.

## Executive translation

The CEO reads business language, never schema. Translate every database term before it reaches the page:

| In the database | In your writing |
|---|---|
| entity, node | company |
| entity-group, organization graph | family of companies, parent organization, subsidiary |
| account_type = customer | active customer |
| field codes, enums, underscores | plain words |

Voice:

- Objective, high-level, conversational. Vary sentence length.
- Open with the point ("Deals have stalled"), skipping wind-up like "It is worth noting that" or "Data indicates that".
- An adjective earns its place only with a number behind it: "significant" backed by "$1.2M, up 40%" stays; unbacked, it goes. Prefer the plain verb: "deals have stalled", not "pipeline velocity has decelerated".
- Give every time reference a concrete date.

## Pre-flight checklist

- [ ] Short Summary closing on The Read, then exactly five themes in the fixed order and with the full theme names, and nothing after Theme 5.
- [ ] Every Key Insights section is either 3–5 full sentences or exactly `Insufficient Data`, never a mix.
- [ ] The prose contains zero underscores, field codes, or raw system enums.
- [ ] Every number and claim traces to a Supporting Fact drawn from the dataset.
