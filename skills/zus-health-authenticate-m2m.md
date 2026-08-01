---
name: Authenticate a backend service to the Zus API
description: Obtain and reuse an OAuth 2.0 bearer token for machine-to-machine access to the Zus FHIR and platform APIs.
api: authentication/zus-health-authentication.yml
operations: [create-app-client, exchange-token-1]
---

# Authenticate a backend service to the Zus API

Use this when a backend system (no human in the loop) needs to call the Zus FHIR REST API
(`https://api.zusapi.com`) or the GraphQL FQS API (`https://fqs.zusapi.com/query`).

## Steps

1. **Provision an M2M App Client.** Create an app client (`create-app-client`) to receive a
   `clientId` and `clientSecret`. Store the secret in a secrets manager; rotate it with the
   rotate-app-client-secret operation when needed.
2. **Request an access token** using the OAuth 2.0 client-credentials flow with your `clientId` and
   `clientSecret`. The token is a bearer JWT valid for **1 hour**.
3. **Cache and reuse the token** until it expires. The token endpoint is rate-limited to **200
   requests per hour per App Client** — exceeding it returns `429`. Never fetch a fresh token per
   API call.
4. **Call the API** with `Authorization: Bearer <token>`. The same token works for both the FHIR
   REST API and the GraphQL FQS endpoint.
5. **(End-user apps)** To let your own users act without a second login, exchange their IdP ID token
   for a Zus access token via `exchange-token-1` (OIDC token exchange).

## Rules

- Reuse tokens; a `429` means you are minting tokens too often.
- A `401` means the token is missing or expired — refresh and retry once.
- A `403` means the app client lacks the role/permission or treatment relationship for that data.
