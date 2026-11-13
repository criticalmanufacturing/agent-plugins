---
name: licenses
description: >
  Review Critical Manufacturing software licenses in the Customer Portal: which licenses a customer
  or site has, which expire in the next 30, 60 or 90 days, which already expired, and what a license
  includes (modules, entity and property limitations, activations). Use for "which licenses expire
  soon?", "show ACME's production licenses", "is the 11.1 license still valid?", or "which modules
  are in license X?".
metadata:
  version: "0.1.0"
  author: "Critical Manufacturing"
  icon: "../../assets/logo.svg"
---

# Customer Portal licenses

Read software licenses through the `cm-customerportal` connector, acting as the signed-in user. This
skill only reads. Renewing, extending or changing a license is done by Critical Manufacturing. Point
the user to their CM contact or a `Service Request` ticket (`open-ticket` skill).

## Tools

| Tool | Use it for |
|---|---|
| `list_licenses` | Lists licenses: `query` (fuzzy free text), `sites`, `environmentType`, `version`, `states`, `includeExpired`, `sortBy` (`Site` or `ExpirationDate`). Paged. By default only Active and Frozen, non-expired licenses. |
| `get_license_modules` | Licensed modules of one license. |
| `get_license_entity_limitations` | Maximum instances per MES entity type. |
| `get_license_property_limitations` | Maximum instances per entity type with a given property value (`propertyValue` is the stored value, so enums are numeric). |
| `get_current_date_time` | **Always call first** for anything about expiry. |

Refer to a license by its `name` field, which is the name shown in the portal. Link to the portal's
license list at `https://portal.criticalmanufacturing.com/Licenses`.

## Who is asking

Customers see their own sites' licenses. CM staff can see thousands of licenses across all
customers. If no customer, site or license is named, ask which customer or site, unless the user
explicitly wants a portfolio-wide expiry report.

## Expiry tiers

Compute `daysLeft` = `expirationDate` − today (from `get_current_date_time`), in whole days, and
classify:

| Tier | Rule |
|---|---|
| Expired | `daysLeft` < 0 (only listed when `includeExpired: true`) |
| Critical | 0–30 days |
| Warning | 31–60 days |
| Upcoming | 61–90 days |
| OK | more than 90 days, or no expiration date |

Show the portal's own `expirationStatus` alongside as returned, but the tier above is the one the
skill reports.

For "what expires soon": call `list_licenses` with `sortBy: "ExpirationDate"` plus any site or
environment filters, and page until `expirationDate` passes the 90-day horizon or `hasMore` is false.
The list is sorted soonest first, so you can stop early. Use at most 5 pages. If there are more,
report the totals and offer to continue.

## Presenting

- Group by tier (Critical → Warning → Upcoming), then by site. Use a table with these columns:
  license name, site, environment type, product version, state, expiration date, days left,
  activations used.
- Flag **Production** licenses in Critical or Warning first. These are the ones that can stop a
  plant.
- Mention `renewalDate`/`renewalPeriodMonths` when they are present and relevant.
- Note `Frozen` licenses separately, because they are not in normal use.
- Modules and limitations: call the three detail tools only for the license(s) the user asks about.
  Don't call them for whole lists. Present modules as a sorted list and limitations as a table.

## Limits

- Don't state that a license *will* be renewed or that a renewal is in progress. The connector has no
  such data.
- A license that is valid for a version depends on its `versionValidation` mode. To answer "is there
  a license for 11.1?", use the `version` filter rather than comparing version strings yourself.
