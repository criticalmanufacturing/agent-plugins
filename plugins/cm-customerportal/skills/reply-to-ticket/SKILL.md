---
name: reply-to-ticket
description: >
  Draft a reply to an existing Critical Manufacturing Customer Portal support ticket: read the ticket
  and its conversation, work out what is being asked or what is still open, and write a clear reply
  for the user to post in the portal. Use for "reply to ticket 12345", "answer support's question on
  #12345", "send an update on our P2 ticket", or "follow up on the ticket that's waiting for us".
metadata:
  version: "0.1.0"
  author: "Critical Manufacturing"
  icon: "../../assets/logo.svg"
---

# Reply to a Customer Portal ticket (draft)

The `cm-customerportal` connector **cannot post replies**. This skill reads the ticket, drafts the
reply, and gives the ticket link so the user can post it in the portal. Never say or imply that a
reply was sent.

## Step 1 — Read the ticket (2–3 tool calls)

1. `get_ticket` with the ticket number (`12345` or `#12345`). If the user doesn't know the number,
   find it with `search_tickets` (key words, plus `states: ["Open","Assessed","Contained"]` when they
   mean an active ticket). Confirm which ticket it is if more than one matches.
2. `get_ticket_conversation` for the thread. If it is `truncated`, use `get_ticket_article` for the
   articles the reply depends on.

Then, in 2–4 lines, summarise for the user where the ticket stands, the last message, and what is
being asked of them (for example logs, a reproduction, a confirmation that a fix works, or permission
to close).

## Step 2 — Collect what's missing

Ask only for facts the reply needs and the user hasn't given: answers to the questions in the thread,
test results, versions, timestamps, and whether the issue still happens. Don't invent results or
claim tests were done.

### Documented answers (optional)

When the reply has to explain how the product works, such as a how-to, configuration or expected
behaviour (typically CM staff answering a customer's question), support it with the official
documentation instead of memory. Put the citations in the reply, so the customer gets links they can
follow.

This step needs the **cm-gpt** plugin's connector (`search_documentation`, `get_adjacent_chunks`).
If those tools aren't available, skip the step and don't mention it. When they are:

- Build the search the way the cm-gpt `ask` skill does: `userQueryToEmbed` is 3–8 specific terms
  (module, feature, step or error names), and must **not** contain `Critical`, `Manufacturing` or
  `MES`. Use `docSources: []`, set `docVersions` to the major.minor version from the ticket's Context (empty if
  unknown), and use `maxNumberOfChunksToRetrieve` 2–3.
- Run at most **3** `search_documentation` calls here. `get_adjacent_chunks` calls don't count.
- Use only what the documentation says. If nothing relevant comes back, say that no documentation
  was found. Never invent steps, settings or URLs.
- Cite each source used as `[Title - DocSource (DocVersion)](SourceUrl)`. Omit `(DocVersion)` if it
  is empty.
- Don't use documentation to answer questions about the customer's own environment, data or
  timeline. Only the ticket, the user and the portal can answer those.

## Step 3 — Draft

- Address the last message directly. Answer every open question in the order it was asked.
- Keep it short and factual: what was done, what was observed, what is attached, and what the user
  needs next.
- If the issue is resolved, say so and state whether the ticket can be closed.
- Match the language of the thread unless the user asks otherwise.
- If documentation was used, end the reply with a short "References" list of its citations.
- Present the reply in a copy-ready block, followed by a list of files to attach in the portal, if
  any.

## Step 4 — Hand off

Once the user approves the draft, give the ticket link,
`https://portal.criticalmanufacturing.com/Entity/Ticket/<id>`, and tell them to post the reply there.
Offer to re-check the ticket later with `check-tickets`.

## Safety

- Never put passwords, tokens, connection strings or licence keys in a reply. Replace them with
  `<redacted>`.
- CM staff replying on behalf of support: don't commit to fix versions, dates or priorities that
  aren't already on the ticket or its linked work items (`get_ticket` returns them).
