---
name: check-tickets
description: >
  Check Critical Manufacturing Customer Portal support tickets: list open or recent tickets, find a
  ticket by number or keywords, show its status, priority, owner and SLA dates, and summarise its
  conversation. Use for "show my open tickets", "what's the status of ticket 12345?", "any P1 tickets
  for ACME?", "summarise the conversation on #12345", or "did anyone already report this error?".
metadata:
  version: "0.1.0"
  author: "Critical Manufacturing"
  icon: "../../assets/logo.svg"
---

# Check Customer Portal tickets

Read support tickets from the Critical Manufacturing Customer Portal through the `cm-customerportal`
connector. Every tool acts as the signed-in portal user, so results are exactly the tickets that user
can see. This skill only reads. To draft a new ticket use `open-ticket`. To draft a reply use
`reply-to-ticket`.

## Tools

| Tool | Use it for |
|---|---|
| `search_tickets` | Lists and searches. Filters: `query`, `states`, `priority`, `openReason`, `createdSince`/`createdUntil`, `view`, `agents`, `isInternal`, `sortBy`. Paged with `totalRows`/`hasMore`. |
| `get_ticket` | One ticket: fields, SLA dates, linked work items, tags, and the list of articles. Accepts the ticket number (`12345` or `#12345`) or the internal id. |
| `get_ticket_conversation` | The whole thread as plain text, oldest first. |
| `get_ticket_article` | The full text of one article, by id from `get_ticket`. |
| `get_current_date_time` | Today's date. Call it before any relative date ("last week", "overdue"). |

## Who is asking

- **Customers** see the tickets of their own sites. Use the default `view: "All"`.
- **CM staff** may see thousands of tickets from many customers. If the request doesn't name a
  customer, site, ticket or keyword, ask which customer or site they mean instead of listing
  everything. Use `view: "Agent"` and `agents: ["<name>"]` for "my queue"-style questions only when
  the user says they are a support agent.
- Never mix tickets from different customers in one answer unless the user asked for a cross-customer
  view.

## How to answer common requests

- **"My open tickets" / "active tickets":** `states: ["Open", "Assessed", "Contained"]`,
  `sortBy: "LastModified"`. Without `states`, every state is returned, including closed ones.
- **A specific ticket:** `get_ticket` with the number. Fetch `get_ticket_conversation` only when the
  user wants the history or a summary, or when the latest article alone doesn't explain the status.
- **"Was this already reported?":** `search_tickets` with 2–4 key words, `view: "All"`, no state
  filter. Also suggest the `known-issues` skill, because product bugs are often tracked there.
- **Relative dates:** call `get_current_date_time` first, then use `createdSince`/`createdUntil`.

Use small pages (`pageSize` 10–20). Fetch more pages only when the user asks or the answer needs a
complete count. Use `totalRows` to report totals instead of paging through everything. Use at most 8
tool calls per request, unless the user asks for a full export.

## Presenting tickets

- For lists, use a compact table: ticket number, title, state/status, priority, site, owner, last
  modified. Sort the most urgent first (P1/P2, then waiting on the requester, then most recent).
- For one ticket, give its state and status, priority, type (`openReason`), site, context (product
  version), owner, SLA or due dates, linked work items, and a 2–4 line summary of where it stands and
  who has to act next.
- Call out tickets that are **waiting on the user** (a status such as "Waiting For Information" or "Waiting For Requester"),
  that have passed an SLA date, or that are P1/P2.
- Link each ticket as `https://portal.criticalmanufacturing.com/Entity/Ticket/<id>`, using the `id`
  returned by the tools.
- Quote the ticket conversation sparingly. Summarise it, and don't repeat credentials, tokens or
  personal data that appear in articles.

## Limits

- The connector cannot create tickets, post replies, change status or attach files. Never say a
  ticket was changed.
- If a ticket is not found, say so and offer a keyword search. Don't guess at ticket numbers.
