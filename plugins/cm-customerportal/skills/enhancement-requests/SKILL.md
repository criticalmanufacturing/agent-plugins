---
name: enhancement-requests
description: >
  Search Critical Manufacturing enhancement requests (product feature requests voted by customers) in
  the Customer Portal: find whether a feature was already requested, its state, votes, queue position
  and planned or delivered release. Use for "has anyone requested X?", "status of our enhancement
  requests", "most voted feature requests", or "what's planned for 11.2?".
metadata:
  version: "0.1.0"
  author: "Critical Manufacturing"
  icon: "../../assets/logo.svg"
---

# Customer Portal enhancement requests

Search enhancement requests through the `cm-customerportal` connector, acting as the signed-in user.
Read only. The connector cannot create, vote, follow or reply. Point the user to the portal page to
vote or comment, or to `open-ticket` (type `Enhancement Request`) to request something new.

## Tools

| Tool | Use it for |
|---|---|
| `search_enhancement_requests` | `query` (relevance-ranked; a number also matches the work item id), `author` (`Mine`, `MyCompany`), `followedByMe`, `repliedByMe`, `tags`, `versions` (delivered/planned release), `sortBy` (e.g. `MostVoted`, `QueuePositionAscending`, `ReleaseNewest`), `includeBody`. Paged. Internal requests are included automatically for users allowed to see them. |
| `get_thread` | One request: state, linked work item, queue position, votes (`includeVoters` only for allowed internal users), description and replies. |

## Common requests

- **"Was X already requested?":** `query` with 2–4 key words. Open the best 1–3 with `get_thread`
  before saying it's the same request. Use at most 3 searches.
- **"Our requests":** `author: "MyCompany"` (or `Mine`), `sortBy: "Newest"`.
- **"What's planned for version V?":** `versions: ["V"]`, `sortBy: "QueuePositionAscending"`.
- **"Top requests":** `sortBy: "MostVoted"` with a small page.

## Presenting

- Use a table: title, state, votes, queue position, planned/delivered release, work item id.
- Link each request as `https://portal.criticalmanufacturing.com/EnhancementRequests/<id>`.
- Report the planned release and queue position exactly as the portal gives them. A planned release
  is not a delivery commitment. Say so if the user is relying on it.
- Only CM staff who are allowed to see votes get voter details. Never list voters' companies to a
  customer user.
