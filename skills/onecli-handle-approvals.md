---
name: Handle manual-approval requests
description: Long-poll for pending manual-approval requests and approve or deny them so held agent requests are forwarded or blocked.
api: openapi/onecli-openapi-original.yml
operations: [listPendingApprovals, submitApprovalDecision]
---

# Handle manual-approval requests

When a policy rule requires manual approval, the gateway holds the agent's outbound
request and creates a pending approval. Use this to build an approver loop.

## Auth
`Authorization: Bearer <oc_...>` (project key), or `oc_org_` + `X-Project-Id`.
Base URL `https://api.onecli.sh/v1`.

## Steps
1. **Poll for requests** — `listPendingApprovals` (`GET /approvals`). This long-polls:
   it returns immediately if any approvals are pending, otherwise holds the connection
   open up to `timeoutSeconds` until one arrives or the poll times out. Loop it.
2. **Decide** — `submitApprovalDecision` (`POST /approvals/{id}/decision`) to approve or
   deny. Approving forwards the held upstream request; denying blocks it. Scope the call
   to the request's project with a project key or an `X-Project-Id` header.

## Notes
- To watch every project in an organization at once, use `listOrgPendingApprovals`
  (`GET /org/approvals`) with an `oc_org_` key (admin/owner role).
- The poll returning empty after `timeoutSeconds` is normal — reissue it.
- Errors follow the standard envelope (`errors/onecli-error-codes.yml`).
