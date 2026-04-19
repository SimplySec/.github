# 🚀 API Security Assessment Guide

A structured approach to testing REST and GraphQL APIs, based on the **OWASP API Security Top 10**.

> [!TIP]
> APIs are the backbone of modern apps. Unlike web browsers, API clients (like `curl` or Postman) don't enforce security UI—so you must test the raw endpoints.

## 1. Discovery & Inventory
You cannot secure what you don't know exists.
*   **Documentation:** Review OpenAPI/Swagger docs or GraphQL schemas (`/graphql`, `/introspection`).
*   **Shadow APIs:** Look for older versions (`/v1/`, `/v2/`) or test endpoints (`/staging`, `/debug`) that might have weaker security.
*   **Parameters:** Identify all input types (JSON, XML, Query Params, Headers).

## 2. Broken Object Level Authorization (BOLA / IDOR)
**The #1 API Risk.** Can User A access User B's data by changing an ID?
*   **Test:** Replace your resource ID (`/api/users/105/orders`) with another ID (`/api/users/106/orders`).
*   **Verify:** Does the API return a `403 Forbidden` or `404 Not Found`? If it returns a `200 OK` with User B's data, it's vulnerable.

## 3. Authentication & JWT Security
*   **Token Strength:** Check if JWTs are signed with a weak secret or if the `alg` header can be set to `none`.
*   **Expiration:** Ensure tokens expire and cannot be reused indefinitely.
*   **Sensitive Data:** Never store PII (email, passwords) in the JWT payload (it's only Base64 encoded, not encrypted).

## 4. Mass Assignment
Can you update fields that should be read-only?
*   **Test:** When updating your profile (`PATCH /api/user/me`), try adding `"is_admin": true` or `"balance": 9999` to the JSON body.
*   **Verify:** Ensure the server ignores or rejects unauthorized fields.

## 5. Rate Limiting & Resource Consumption
Prevent DoS and brute-force attacks.
*   **Thresholds:** Send 100 requests in rapid succession. Does the API return `429 Too Many Requests`?
*   **Pagination:** Check if you can request massive datasets (e.g., `?limit=999999`) to crash the server.

## 6. Business Logic & Sensitive Flows
*   **Multi-step:** Can you skip a payment step by calling the `/order/complete` endpoint directly?
*   **Integrity:** Can you send a negative quantity in a shopping cart request to receive a refund?

---
*Return to [SimplySec Playbook](../README.md)*
