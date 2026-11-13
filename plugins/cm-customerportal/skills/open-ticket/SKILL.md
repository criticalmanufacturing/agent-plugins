---
name: open-ticket
description: >
  Prepare a new Critical Manufacturing Customer Portal support ticket: collect the required details,
  check for existing tickets and known issues first, and produce a ready-to-paste draft (title, type,
  site, product version, environment, impact/urgency and description) with the portal link to submit
  it. Use for "open a ticket", "raise a support case", "report this MES bug to CM", or "I need to
  request a service from CM support".
metadata:
  version: "0.1.0"
  author: "Critical Manufacturing"
  icon: "../../assets/logo.svg"
---

# Open a Customer Portal ticket (draft)

The `cm-customerportal` connector **cannot create tickets**. This skill prepares a complete,
well-structured draft and checks for duplicates. The user then submits it themselves at
`https://portal.criticalmanufacturing.com/Support/Tickets` (the "new ticket" wizard). Never say or
imply that a ticket was created, submitted or sent.

## Step 1 — Collect the details

The portal's new ticket wizard asks for these fields. Ask only for what the conversation hasn't
already given, in a single short message.

| Field | Required | Notes |
|---|---|---|
| Title | Yes | One line, specific: symptom + where it happens. |
| Type | Yes | `Bug`, `Question`, `Service Request` or `Enhancement Request`. Suggest one from the description. |
| Customer and site | Yes | If the user can see several sites (CM staff, multi-site customers), ask which one. Don't guess. |
| Date of occurrence | Yes | When the problem happened. Call `get_current_date_time` to resolve "today"/"yesterday". |
| Context | Recommended | Product and version, e.g. `MES 11.1.3`. |
| Environment type and customer environment | Recommended | Production / Staging / Testing / Development and the environment name. `list_customer_environments` can find it. |
| Impact and urgency | Recommended | How many users/lines are affected and how fast it must be solved. The portal derives the priority from these. Don't promise a priority. |
| Description | Yes | See the template below. |
| CC, tags, attachments | Optional | Remind the user to attach logs or screenshots in the portal. The connector can't attach files. |

## Step 2 — Check for duplicates (max 3 portal searches)

Before drafting, run:

1. `search_tickets` with 2–4 key words from the symptom, no state filter, `view: "All"`.
2. `search_known_issues` with the same key words (and `detectedVersions` if the version is known).
3. For an enhancement request, also run `search_enhancement_requests`.

If something closely matches, show it with its link (tickets: `/Entity/Ticket/<id>`, known issues:
`/KnownIssues/<id>`, enhancement requests: `/EnhancementRequests/<id>`). Ask whether the user still
wants a new ticket or would rather follow up on the existing one (`reply-to-ticket`), follow the
known issue, or vote for the request.

### Questions: check the documentation first

For `Question` tickets (and how-to or configuration questions filed as other types), check whether
the answer is already documented before drafting. If it is, give the documented answer with its
citations, and ask whether the user still wants to open the ticket. If they do, add the links they
already checked to the description, so support doesn't send the same pages back.

This step needs the **cm-gpt** plugin's connector (`search_documentation`, `get_adjacent_chunks`).
If those tools aren't available, skip the step and don't mention it. When they are:

- Build the search the way the cm-gpt `ask` skill does: `userQueryToEmbed` is 3–8 specific terms
  (module, feature, step or error names), and must **not** contain `Critical`, `Manufacturing` or
  `MES`. Use `docSources: []`, set `docVersions` to the major.minor version from the ticket's Context (empty if
  unknown), and use `maxNumberOfChunksToRetrieve` 2–3.
- Run at most **2** `search_documentation` calls here. `get_adjacent_chunks` calls don't count.
- Use only what the documentation says. If nothing relevant comes back, say that no documentation
  was found. Never invent steps, settings or URLs.
- Cite each source used as `[Title - DocSource (DocVersion)](SourceUrl)`. Omit `(DocVersion)` if it
  is empty.
- Don't search documentation for `Bug`, `Service Request` or `Enhancement Request` tickets unless
  the user asks.

If a deployment failed, `get_deployment_logs` (see the `environment-logs` skill) can give the exact
error text for the description. Mask any secret-looking values.

## Step 3 — Draft

Present the draft in a copy-ready block:

```
Title: <title>
Type: <Bug | Question | Service Request | Enhancement Request>
Customer / Site: <customer> / <site>
Date of occurrence: <YYYY-MM-DD HH:MM, timezone>
Context: <MES x.y.z>
Environment: <type> — <environment name>
Impact: <…>    Urgency: <…>

Description
- What happened: <observed behaviour, exact error message>
- Expected: <expected behaviour>
- Steps to reproduce: <1., 2., 3.>
- Scope: <how many users/resources/lines, since when, workaround?>
- Recent changes: <deployments, customisation, configuration>
- Attachments to add in the portal: <logs, screenshots, exported data>
```

Leave out any section the user has no information for. Don't invent steps, errors or versions. Write
in English unless the user asks for another language.

## Step 4 — Hand off

- Ask the user to review the draft. Revise it until they're happy.
- Then give the submission link `https://portal.criticalmanufacturing.com/Support/Tickets`, and tell
  them to fill in the wizard with the draft and attach their files.
- After they submit, offer to check the new ticket with the `check-tickets` skill.

## Safety

- Never include passwords, tokens, connection strings or licence keys in a draft. If the user pastes
  one, replace it with `<redacted>` and tell them.
- Don't promise response times or priorities. These come from the customer's support contract and
  the portal's SLA rules.
