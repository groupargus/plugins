---
name: overview
description: Executive summary of one company and its subsidiaries from the Sunnny database, delivered as a five-theme report. Use when asked for an overview of, or to summarize, brief on, or analyze, a company, customer, or account, including account reviews, QBR prep, customer health checks, or "what do we know about" a company.
argument-hint: [company name]
allowed-tools: mcp__plugin_sunnny_sunnny__get_summary mcp__plugin_sunnny_sunnny__list_organizations mcp__plugin_sunnny_sunnny__get_organization mcp__plugin_sunnny_sunnny__get_organization_summary mcp__plugin_sunnny_sunnny__list_organization_records mcp__plugin_sunnny_sunnny__list_contact_records
---

# Sunnny Overview

Act as a senior business operations analyst delivering a live briefing to the CEO about one company. Your evidence is the Sunnny database, read through the Sunnny MCP tools, holding unified customer data from CRM, support, and communication systems (Salesforce, HubSpot, Zendesk, Jira, Slack, Gong). Your deliverable is a written report, not a spreadsheet, posted as your chat reply. After the report, add a one-line offer to put the report into a document the user can share, and make that document only when they ask for one.

Every claim traces to a field in the dataset. Where the data is missing, the report says so explicitly (see the theme gates below) instead of filling the gap.

## Process

1. **Confirm which company.** Search for the requested name with `list_organizations`, whose `filter` matches any company in a family and returns each matching family whole, top-level company first. Count matches by the families in `organizations`, never by `filtered_count`, which counts companies. One family: proceed. Several families: list their top-level names and ask which one before running. Subsidiaries of one family don't count as separate matches — the family step handles those.
2. **Resolve the family of companies.** The step 1 result already holds the whole family tree, from the top-level company down; call `get_organization` only when you start from an organization ID. Expand the analysis to the full corporate family. Record lists cover one company only, with no roll-up from subsidiaries, so call `get_organization_summary` for each company in the tree first and let its counts drive the rest. The summary doesn't count custom records, so also call `list_organization_records` with kind `custom-types` for every company in the tree. Skip a company whose contacts, tickets, meetings, and opportunities are all zero and that has no custom record types: if Marriott International has 3 active subsidiaries behind an empty Starwood holding node, analyze the 3 and bypass the node. For the rest, call `list_organization_records` for each kind with a nonzero count, and fetch the records of each custom type listed. Done when every subsidiary that has data is in scope.
3. **Map the data.** Accounts and Contacts are the core entities. Meetings, calls and conversations, deals, and tickets are historical activity belonging to those companies and people, never free-standing facts. Reach a contact's own meetings, tickets, and deals with `list_contact_records`, passing the contact's `id`. For a deal's contacts, pass each one's `contact_id`, and if `contact_id` is null that person didn't match a Sunnny contact. Custom record types hold what the standard kinds don't, such as contracts, invoices, surveys, and license usage, and each one feeds one or more themes (see Routing custom types below): kind `custom-types` lists them, and passing a type's `id` as `type_id` returns its records, with the type's fields as top-level keys next to `id` and the other keys Sunnny adds, which start with `__`. Read the fields the records actually return rather than guessing names. Money fields (`value`, `annual_revenue`) hold an `amount` in the currency's smallest unit next to its `currency`, so divide by 100 for currencies with cents, such as USD and EUR, and check the currency before converting any other; values inside custom records are already formatted. Ticket status and priority spellings differ by source system, so compare them case-insensitively and count `new` with `open` as unresolved.
4. **Compute portfolio baselines.** The Comparative Lens needs something to compare against: `get_summary` counts the tickets, meetings, opportunities, and contacts across the whole portfolio (use `merged_count` for contacts). For the number of families, call `list_organizations` once with `has_activity_since` set to an early date such as 2000-01-01 and `limit` 1, and read `filtered_family_count`. Neither `total_count` nor `filtered_count` works: both count companies, not families. Most families have no activity at all, so dividing by every family shrinks the average about tenfold. Divide the counts by the active families for per-family averages, such as tickets or meetings per family. The tools can't produce portfolio-wide rates or dollar figures, so compare open rates, pipeline, and booked revenue against the family's own history instead, such as this year against last. Show the math when you cite one.
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
  Key Insights / Comparative Lens
## Supporting Facts and Sources
  (one sub-heading per passing theme, in theme order, holding that
   theme's Supporting Facts and its Data Sources)
```

**The Read** closes the Short Summary and appears nowhere else — the report ends on Supporting Facts and Sources, not on a trailing verdict. It is judgment across all five themes: the theme lines report, The Read commits, naming the real risk even when it is not the obvious one. From a real report: "The Read: an average-sized account with above-average support friction, a go-live that may never have happened, and no executive above EVP. The $105,000 open deal is not the risk — the $407,500 already booked is, with no usage data to say whether they use what they bought."

## The five themes and their gates

Exactly these five, in this order. A theme passes its gate when enough of its listed inputs exist to support analysis; it fails when they are missing, empty, or too sparse.

| Theme | Question it answers | Minimum Required Inputs | Insights to consider |
|---|---|---|---|
| 1. Customer Overview | Quick overview of this customer | Account profile (industry, employees, revenue; segment if a source has it); company tree (conglomerate or independent); account type (customer, prospect, lead); plan or product, usually from a contracts, subscriptions, or licenses custom type; go-live status from a projects custom type | Recently new customer; prospect subsidiaries inside a large company tree; onboarding that never went live |
| 2. Customer Value | What is this customer worth to me | Contract value and end dates, usually from a contracts or subscriptions custom type; opportunity pipeline with dollar value, stage, close dates; quotes, orders, and invoices custom types; total across parent and subsidiaries | Upcoming contract expirations; deal close dates; deals at risk from support ticket volume; downgrades or cancellations; overdue invoices |
| 3. Customer Engagement Level | How strong and strategic is the relationship | Engagement volume (meetings, calls, conversations); NPS or relationship surveys, usually from a surveys custom type; campaign, webinar, event, or QBR custom types; contact count, seniority distribution (C-level through Director) from contact and meeting-participant titles, functional diversity from contact departments | Low or dropping engagement; who shows up in meetings, events, and deals; missing senior contacts; recurring conversation topics; overdue QBR actions |
| 4. Customer Support | How is the support experience | Tickets with priority and status; CSAT or SLA metrics, often from a surveys custom type; community engagement, usually from a community or forum custom type | Volume and trends; recurring ticket themes; high-priority or urgent tickets; bad sentiment; unanswered community questions |
| 5. Product Activity | Is the customer getting value | Product telemetry (DAU/WAU/MAU, logins, feature usage), usually from a product activities or licenses custom type, OR a usage proxy (API calls, exports) | Low or declining usage; power users; active users vs. licensed seats; usage against plan limits; unused features; most-requested features |

Calibration from a real run: meetings plus contact depth passed the Engagement gate even with NPS absent (the absence was declared in the facts); support-ticket anecdotes alone failed the Product Activity gate.

### Routing custom types

Customers name their own custom types and fields, so recognize each type by what its name and fields describe rather than by exact spelling: a type with an amount, a due date, and a paid or unpaid status is an invoice whether it's called "Invoices" or "AR Ledger".

The table gives each type's main theme, where it usually does its work. Its fields can also answer any other theme's question, and they count as Minimum Required Inputs there: with no contracts, a licenses or billing type can supply plan type and seats for Customer Overview and ARR or MRR for Customer Value. A licenses type alone can pass Product Activity, but Feedback/Requests can't: like ticket anecdotes, it needs usage data beside it. Cite a type in the Data Sources of every theme it feeds, by its display name.

Give each fact its full evidence in one theme, the one whose question it answers most directly, and refer to it in a phrase elsewhere ("Pro plan, 120 seats"). When two types disagree on the same fact, such as a contract saying Enterprise and a license saying Pro, treat it as conflicting data (see below) rather than picking one.

| Custom type | Main theme | What to compute |
|---|---|---|
| Contracts, Subscriptions/Billing | 1. Customer Overview (plan or product) and 2. Customer Value | Contract value and ARR or MRR; renewals or end dates within 90 days; upgrades, downgrades, and cancellations |
| Quotes/CPQ | 2. Customer Value | Open quotes; discount depth (list price against net); quotes about to expire; the linked deal |
| Sales/Purchase Orders, Invoices | 2. Customer Value | Booked amount; overdue or unpaid invoices |
| Projects (onboarding, implementation) | 1. Customer Overview; fees in 2. Customer Value | Go-live status; slipped milestones against the planned end |
| Campaigns, Webinars & Events | 3. Customer Engagement Level | Reply and attendance rates; who engaged, by title; no-shows |
| Surveys (NPS, CSAT, product feedback) | NPS and other surveys in 3. Customer Engagement Level; CSAT in 4. Customer Support | Scores and their trend; detractor comments, quoted |
| Business reviews (QBRs) | 3. Customer Engagement Level | Date of the last review; customer goals; agreed actions now overdue |
| Community/Forum activity | 4. Customer Support | Post volume and trend; unanswered questions; sentiment |
| Feedback/Requests | 5. Product Activity | Most-requested product areas; vote counts |
| Product activities, Licenses/Usage | 5. Product Activity | Active users against seats bought; login trend; usage against plan limits |

A type that matches no row goes to the theme whose question its fields answer. When no theme fits, name it in Customer Overview's Supporting Facts as present but not analyzed, with its record count.

### When a theme passes

Write 3–5 Key Insights, each backed by Supporting Facts, and every insight carrying a date, name, dollar value, or quoted comment. Name a person alongside their company and, where the record has one, their job title, and never include an email address or phone number. Add a Comparative Lens where data permits: a historical trend, peer benchmark, or portfolio average. The theme itself holds only Key Insights and the Comparative Lens. Its Supporting Facts and Data Sources go under its sub-heading in Supporting Facts and Sources at the end of the report.

Inside a passing theme, name the standard inputs that are absent ("no contract end dates in the dataset, so churn can't be measured") rather than silently skipping them.

### When a theme fails

Set Key Insights to exactly `Insufficient Data`, followed by one line naming the missing inputs and the nearest available signal, if any. Leave the Comparative Lens empty, and leave the theme out of Supporting Facts and Sources. A theme is fully written or explicitly empty, never half-filled.

## Two voices

Key Insights are full, natural sentences you would say aloud in a briefing — the strategic read. Supporting Facts are short, data-dense bullets — the evidence trail, with record IDs, source fields, and links to records, gathered at the end of the report so the themes read cleanly.

Link every company, deal, meeting, and ticket the report names, at its first mention in each section (the Short Summary, each theme, and each theme's Supporting Facts), as a Markdown link on its name or ticket number pointing to the record's `source.view_url`. A company has one `source` per system, so link its CRM entry (Salesforce or HubSpot). A record whose `view_url` is null stays unlinked.

### Exemplar (Customer Support, made-up family)

The companies and numbers below are invented to show the shape. Never reuse them in a report.

Key Insights:
- Support is this family's weak spot: 11 of its 20 tickets are still open, and 20 tickets is eight times the portfolio average of 2.5 per family.
- Tickets are aging badly — 8 of the 11 open ones have been sitting for six months, and the oldest, Brightline asking to add seats to its plan, has been open since June 13, 2025.
- Two of the stalled tickets are customers trying to give us money: that Brightline add-seats request and Halvorsen Beverages' high-priority enterprise-tier upgrade from May 2026.

Comparative Lens: Active families average 2.5 tickets (674 tickets across the 267 families with any activity). This family has 20, and its oldest open ticket predates every one of its expansion deals.

Later, under the Customer Support sub-heading of Supporting Facts and Sources:

Supporting Facts:
- 20 tickets total: 11 open, 9 solved (55% open). By company: Halvorsen Group 1 of 4 open; Halvorsen Beverages 2 of 4; Brightline 5 of 8; Calder 3 of 6
- Oldest open ticket: Brightline "Add seats to the Brightline plan" #143, opened 2025-06-13, ~13 months (deep link to the ticket)
- No CSAT, sentiment, or SLA metrics in the dataset

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

- [ ] Short Summary closing on The Read, then exactly five themes in the fixed order and with the full theme names, then Supporting Facts and Sources grouped by theme, and nothing after it.
- [ ] Every theme holds only Key Insights and a Comparative Lens, with its Supporting Facts and Data Sources in the closing section.
- [ ] Every Key Insights section is either 3–5 full sentences or exactly `Insufficient Data` with its missing-inputs line, never a mix.
- [ ] The prose contains zero underscores, field codes, or raw system enums.
- [ ] Every number and claim traces to a Supporting Fact drawn from the dataset.
- [ ] Every company, deal, meeting, and ticket links to its `view_url` at its first mention in each section, unless that is null.
- [ ] Every custom type found in the family appears in a theme's Data Sources or in Customer Overview as not analyzed.
- [ ] Every person named carries their company and, where the record has one, their job title, and the report contains no email addresses or phone numbers.
