---
name: Provision a OneCLI agent with scoped credentials
description: Create an agent, store a secret in the vault, assign it to the agent, and lock the agent to only its assigned secrets.
api: openapi/onecli-openapi-original.yml
operations: [createAgent, createSecret, updateAgentSecrets, updateAgentSecretMode, regenerateAgentToken]
---

# Provision a OneCLI agent with scoped credentials

Use this to onboard a new AI agent so it can reach an external service through the
OneCLI gateway without ever seeing the raw key.

## Auth
All calls use `Authorization: Bearer <oc_...>` (project key). With an `oc_org_` key,
add `X-Project-Id: proj_...`. See `authentication/onecli-authentication.yml`.
Base URL: `https://api.onecli.sh/v1` (Cloud) or `http://localhost:10254/v1` (self-hosted).

## Steps
1. **Create the agent** — `createAgent` (`POST /agents`). The `identifier` must be
   1-50 chars, lowercase alphanumeric + hyphens, starting with a letter or number.
   A duplicate identifier returns `409` ("An agent with this identifier already exists").
2. **Store the credential** — `createSecret` (`POST /secrets`). Provide the host/path
   patterns the gateway should match; secret values are never returned on read.
   A `hostPattern` with a protocol or path prefix returns `400`.
3. **Assign the secret to the agent** — `updateAgentSecrets`
   (`PUT /agents/{agentId}/secrets`) with the full list of secret IDs. Unknown IDs
   return `400` ("One or more secrets not found").
4. **Lock down access** — `updateAgentSecretMode`
   (`PATCH /agents/{agentId}/secret-mode`) with `mode: selective` so the agent may use
   only its explicitly assigned secrets (default `all` grants every project secret).
5. **Issue the token** — `regenerateAgentToken`
   (`POST /agents/{agentId}/regenerate-token`); the previous token is invalidated
   immediately. Give this token to the agent runtime (`onecli run -- <agent>`).

## Notes
- No idempotency key: rely on unique identifiers; duplicates surface as `409`.
- Errors: read `error` when it is a string, else `error.message`
  (see `errors/onecli-error-codes.yml`).
