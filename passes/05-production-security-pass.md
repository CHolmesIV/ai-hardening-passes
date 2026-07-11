# The Production Security Pass

The first security pass protects the front door. This pass checks the doors AI-built apps forget exist: tenant identifiers in URLs, file uploads, outbound webhooks, background workers, billing events, and deletion promises.

Run it by trust boundary, not file by file. For each item, demand a code path, a concrete failure scenario, and a regression test. A scanner alert without a reachable path is a lead, not a finding.

## 1. Tenant isolation and object ownership

Authentication answers who the caller is. It does not prove they own the record in `/api/items/{id}`. Trace every read, update, delete, export, download, and "mark read" operation from request to query. Every query must scope by tenant and, for user-private records, by user. Test with two tenants and two users in the same tenant.

**Work order:** "Inventory endpoints that accept a record id. Prove each query constrains the record by the authenticated tenant and correct user scope. Add cross-tenant and same-tenant-cross-user tests for reads and writes."

## 2. Outbound requests and SSRF

User-controlled URLs for webhooks, imports, previews, or attachments can turn your server into a proxy to cloud metadata, localhost, or private services. HTTPS alone is not enough. Validate the destination, resolve DNS, reject loopback/private/link-local/reserved addresses, re-check every redirect, and never attach a platform secret to an untrusted host.

**Work order:** "Trace every outbound HTTP request and identify which URL parts a user or upstream API controls. Add a shared egress policy and tests for localhost, private IPv4/IPv6, DNS rebinding, redirects, and credentials leaking across hosts."

## 3. Uploads, downloads, and resource exhaustion

A size check after `read()` is not a size limit. Stream uploads and downloads, enforce limits before and during transfer, cap decompression and archive entry counts, validate content rather than trusting extensions, and clean up stored objects when database work fails. Include aggregate limits when assembling email or ZIP attachments.

**Work order:** "Find every file read, upload, archive, parser, and attachment path. Prove bounded memory, bounded bytes, bounded expansion, safe filenames, and cleanup on failure. Add an oversized streaming test and an archive-bomb test where relevant."

## 4. Webhooks, OAuth callbacks, and replay

Verify signatures against the raw, size-capped body before parsing. Reject stale timestamps and persist event ids so retries are idempotent. OAuth state must be short-lived, signed, single-use, tied to the initiating user/session, and validated at callback. Redirect URIs must be exact allowlisted values.

**Work order:** "For every public callback, show body limits, signature/state verification, freshness, replay handling, and identity binding. Test invalid signatures, replayed events, expired state, and a callback initiated by one user but completed by another."

## 5. Background jobs and delivery idempotency

`SELECT queued` followed by `UPDATE running` is a race unless the claim is atomic. Long jobs need an owner, lease, and heartbeat. A crashed worker must not let a stale attempt overwrite a newer result. External side effects such as email, payments, or webhooks need stable idempotency keys and a durable outbox or equivalent claim protocol.

**Work order:** "Run two workers against the same queue. Prove only one claims each job, leases cannot reap healthy work, stale workers cannot finalize, retries do not repeat committed side effects, and result persistence is atomic."

## 6. Billing and entitlements

Webhook delivery order is not business-event order. Persist event ids and provider timestamps or versions, ignore stale events, and verify that a delete/update targets the currently active subscription. Make checkout creation idempotent. Enforce seats, quotas, and usage caps atomically at the write that consumes them, not only in the UI.

**Work order:** "Replay billing events out of order and concurrently. Prove stale or duplicate events cannot downgrade current state, checkout retries cannot create parallel subscriptions, and two simultaneous quota-consuming requests cannot both pass the final slot."

## 7. Deletion, retention, and storage lifecycle

Database cascades do not delete object storage, search indexes, logs, provider data, or backups. Map every user and tenant artifact. Product promises, privacy copy, retention policy, lifecycle rules, and actual deletion code must agree. Document what is deleted now, asynchronously, or only after backup expiry.

**Work order:** "Delete a seeded tenant and produce an artifact-by-artifact receipt covering the database, object storage, caches, search, integrations, logs, and backups. Reconcile every result with the published privacy and retention language."

## 8. Browser and deployment boundary

Verify CSP, HSTS, cookie flags, CORS, framing protection, request-body limits, proxy timeouts, and rate limits in the response actually served by production. Review service privileges, secret-file permissions, backup failure reporting, restore tests, dependency scanning, and rollback. Repository config is not proof that the live edge uses it.

**Work order:** "Capture production response headers and effective proxy/service configuration. Compare them with source control, then test rate limits, body limits, backup alerts, a restore, and rollback. Report drift explicitly."

## 9. Client races and sensitive state

Fast navigation and changing filters can let stale responses overwrite the current screen. Abort obsolete requests or tag them with the state that initiated them. Keep secrets and sensitive records out of browser persistence, URLs, analytics, and error telemetry. Test rapid tenant switching, record switching, and slow responses.

**Work order:** "Throttle the API and rapidly switch tenant, record, filters, and preview. Prove old responses cannot render into new context and sensitive values do not enter browser storage, URLs, logs, or analytics."

## Output contract

Use the [audit finding template](../templates/audit-finding.md). Finish with:

1. A boundary-by-boundary PASS, FAIL, or NOT VERIFIED table with evidence.
2. Findings ranked by exploitability and impact, not by how easy they are to fix.
3. Existing issue links and duplicates rejected.
4. Tests run, tests omitted, and anything that requires live-environment verification.
5. Positive controls that were inspected and held, so the next reviewer does not repeat the same work.

Never claim a full security audit if you only ran static checks or reviewed one layer. Say exactly what was and was not verified.
