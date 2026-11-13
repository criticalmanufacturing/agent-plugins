---
name: known-issues
description: >
  Search Critical Manufacturing known issues (product bugs tracked by CM) in the Customer Portal:
  check whether a problem is already known, in which versions it was detected, and in which release
  it is fixed or planned. Use for "is this a known bug?", "known issues for MES 11.1.3", "is work
  item 123456 fixed?", or "which version fixes the SPC export problem?".
metadata:
  version: "0.1.0"
  author: "Critical Manufacturing"
  icon: "../../assets/logo.svg"
---

# Customer Portal known issues

Search known issues through the `cm-customerportal` connector, acting as the signed-in user. Read
only. The connector cannot vote, follow or reply.

## Tools

| Tool | Use it for |
|---|---|
| `search_known_issues` | `query` (relevance-ranked; a number also matches the work item id), `detectedVersions`, `versions` (fixed/planned release), `tags`, `sortBy`, `view` (`Public` default; `Internal` requires the InternalKnownIssues role), `includeBody`. Paged. |
| `get_thread` | One known issue: state, tags, linked work item (type, state, release/detected version), votes, description and replies (paged with `replyPage`). |

## How to search

- Use 2–4 distinctive key words from the symptom or error message. Don't paste whole sentences. If
  nothing matches, try once more with different words (synonyms, or the entity or module name). Use
  at most 3 searches per question.
- Add `detectedVersions` when the user gave their version, and `versions` when they ask what a
  release fixes.
- Only use `view: "Internal"` if the user asks for internal known issues. If it's rejected, say they
  don't have access.
- Open the 1–3 most relevant results with `get_thread` before claiming a match. Titles alone can
  mislead.

## Presenting

- For each relevant issue: title, state, detected version, fix/planned release, work item id, and a
  one-line summary of the symptom and any workaround from the description or replies.
- Link each issue as `https://portal.criticalmanufacturing.com/KnownIssues/<id>`.
- Be explicit about the match quality: "this matches your symptom", "this is related but not the
  same", or "no known issue found". Don't overstate.
- If the issue is fixed in a release, say which one. Don't say it's in a later version unless the
  data shows it.
- If nothing matches and the problem looks like a defect, suggest `open-ticket`.
