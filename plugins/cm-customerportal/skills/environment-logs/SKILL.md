---
name: environment-logs
description: >
  Check Critical Manufacturing customer environments in the Customer Portal (DevOps Center): find an
  environment, show its status, version, packages, licence and installed apps, and retrieve and
  explain its last deployment log, such as why a deployment failed or only partially succeeded. Use
  for "why did our production deployment fail?", "get the deployment logs of env X", "which
  environments failed to deploy?", or "what version is ACME's staging on?".
metadata:
  version: "0.1.0"
  author: "Critical Manufacturing"
  icon: "../../assets/logo.svg"
---

# Customer environments and deployment logs

Read DevOps Center customer environments through the `cm-customerportal` connector, acting as the
signed-in user. The only logs available are the **last deployment log** of an environment. The
connector has no MES runtime or application logs. Say so if the user asks for them, and suggest the
environment's own observability tools.

This skill only reads. It cannot redeploy, terminate, change parameters, or install or uninstall
apps.

## Tools

| Tool | Use it for |
|---|---|
| `list_customer_environments` | Find environments: `customer`, `site` (exact, with fuzzy fallback reported in `fuzzyMatch`), `search`, `statuses`, `types`, `scope` (`Infrastructure` default or `Standalone`, which are disjoint), `onlyMine`, `includeAgentStatus` (slower). |
| `get_customer_environment` | Details: status, type, URL, deployment target, packages with versions and licence, installed apps, audit dates. Parameters and tokens are never returned. |
| `get_customer_environment_apps` | Apps and their install status, for App Framework environments. |
| `get_deployment_logs` | Last deployment log. `mode: "Issues"` (default) keeps the failing steps with context. `mode: "Full"` gives everything. `maxLength` 2000–60000. Secret-looking values are masked. |

## Finding the environment

- If the user gives a name, use `search` (plus `customer`/`site` when known). If nothing comes back
  in the default scope, retry once with `scope: "Standalone"`.
- If the tools report `fuzzyMatch`, tell the user which customer or site names were actually used.
- If several environments match, list them (name, customer, site, type, status, version) and ask
  which one. Don't pick one yourself.
- "Which environments failed?": `statuses: ["DeploymentFailed", "DeploymentPartiallySucceeded"]`,
  scoped to the customer or site. CM staff without a customer named: ask first, or show the page
  totals only.
- Link environments as `https://portal.criticalmanufacturing.com/Entity/CustomerEnvironment/<id>`.

## Explaining a deployment

1. `get_customer_environment` for context (status, version, packages, last change).
2. `get_deployment_logs` with `mode: "Issues"`. Only use `mode: "Full"` if the issue excerpt doesn't
   show the cause, or the user asks for the full log.
3. Report:
   - **Outcome**: status, and when it ran.
   - **Failing step(s)**: step name and the exact error lines, quoted briefly.
   - **Likely cause**: your reading of the error. Label it clearly as an interpretation, not a
     confirmed root cause.
   - **Documented guidance** (optional, see below): what the official documentation says about the
     failing step or error, with citations.
   - **Next steps**: things the user can check or fix, preferring documented ones. If it looks like
     a product defect, suggest `known-issues` to look it up and `open-ticket` to report it with the
     log excerpt.
4. If `truncated` is true, say that the earliest part of the log was cut off.

Use at most 6 Customer Portal tool calls per request.

## Documented guidance from CM GPT (optional)

After you have the failing step and error text, look them up in the official CM documentation.

This step needs the **cm-gpt** plugin's connector (`search_documentation`, `get_adjacent_chunks`).
If those tools aren't available, skip the step and don't mention it. When they are:

- Build the search the way the cm-gpt `ask` skill does: `userQueryToEmbed` is 3–8 specific terms
  (module, feature, step or error names), and must **not** contain `Critical`, `Manufacturing` or
  `MES`. Use `docSources: []`, set `docVersions` to the major.minor version from the environment's package version (empty if
  unknown), and use `maxNumberOfChunksToRetrieve` 2–3.
- Run at most **2** `search_documentation` calls here. `get_adjacent_chunks` calls don't count.
- Use only what the documentation says. If nothing relevant comes back, say that no documentation
  was found. Never invent steps, settings or URLs.
- Cite each source used as `[Title - DocSource (DocVersion)](SourceUrl)`. Omit `(DocVersion)` if it
  is empty.
- Search with the failing step name and distinctive words from the error, plus the deployment
  target if known (e.g. `envmanager installation failed Kubernetes Remote target`). Generic errors
  return generic documentation (DevOps Center, deployment targets, Infrastructure Agent). Present
  that as general checks, not as the cause of this failure. Don't dump whole logs into the chat. Quote only the relevant
lines, and never reproduce values that look like secrets even if they weren't masked.

## Agent status

With `includeAgentStatus: true`, each infrastructure reports `Connected`, `Disconnected`,
`NoInfrastructureAgent` or `CouldNotRetrieve`. A disconnected agent is a common reason why on-premises
deployments get stuck. Mention it when relevant.
