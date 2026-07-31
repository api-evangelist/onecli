---
name: Connect an app and enforce a policy rule
description: Discover available apps, connect one with BYOC credentials, verify the connection, and add a policy rule that governs how agents use it.
api: openapi/onecli-openapi-original.yml
operations: [listApps, connectApp, listConnections, createRule]
---

# Connect an app and enforce a policy rule

Use this to wire a downstream provider (GitHub, Slack, AWS, ...) into a project and
put guardrails on it before agents can call it.

## Auth
`Authorization: Bearer <oc_...>` (project key), or `oc_org_` + `X-Project-Id`.
Base URL `https://api.onecli.sh/v1`.

## Steps
1. **List apps** — `listApps` (`GET /apps`) to see every available provider with its
   current configuration and connection status.
2. **Connect the app** — `connectApp` (`POST /apps/{provider}/connect`) with the app's
   required credential fields (BYOC / direct credentials). A missing required field
   returns `400` ("{field} is required"); an unknown provider returns `404`
   ("Unknown provider: {provider}"). For OAuth apps, most connect in one click from the
   dashboard instead.
3. **Verify** — `listConnections` (`GET /connections`), optionally filtered by the
   `provider` query parameter, to confirm the connection exists.
4. **Govern it** — `createRule` (`POST /rules`) to add a policy rule. Rules control how
   agents interact with the service: `allow`, `block`, `rate_limit`, or manual approval.
   For `rate_limit` you must supply `rateLimit` and `rateLimitWindow`, else `400`
   ("rateLimit and rateLimitWindow are required when action is rate_limit").

## Notes
- Organization-level equivalents (`connectOrgApp`, `createOrgRule`) apply across all
  projects and require the admin/owner role (Cloud or self-hosted Enterprise).
- Tool-level permissions per provider are managed via `setRulePermissions`
  against the app's permission definition (`getPermissionDefinition`).
