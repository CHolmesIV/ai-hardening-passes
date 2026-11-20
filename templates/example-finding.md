# Example Finding (worked)

A filled-in version of [audit-finding.md](audit-finding.md). The app, files, and numbers are made up. Use it to see the level of detail to aim for.

## Title

`[Tenant isolation] Scope invoice reads and downloads to the caller's organization`

## Severity

High. Any logged-in user can read another customer's invoices by changing an id in the URL, and ids are sequential.

## Evidence

- `src/routes/invoices.ts:42` loads the invoice with `db.invoice.findUnique({ where: { id } })` and never checks `organizationId`.
- `src/routes/invoices.ts:71` (PDF download) has the same lookup.
- Boundary crossed: one tenant's data served to another tenant's session.
- Preconditions: any valid account, including a free trial signup. No special role.

## Failure scenario

Demonstrated in staging: user A (org 1) requests `GET /api/invoices/1041`, which belongs to org 2, and gets a 200 with the full invoice body. Incrementing the id walks the whole table. Inferred, not tested: the same gap on `PATCH /api/invoices/:id`, which uses the shared lookup helper.

## Impact

Customer names, addresses, line items, and amounts for every tenant are readable by any signed-up user. Privacy exposure and likely a contract and breach-notification problem.

## Recommended fix

Add `organizationId` from the verified session to every invoice query, in one shared `getInvoiceForOrg(session, id)` helper, and return 404 (not 403) on a miss so ids are not confirmed to exist. Replace the direct lookups in the read, download, and update routes. No migration needed. Switch the public id to a UUID as a later hardening step, not as the fix.

## Acceptance criteria

- A user in org 1 gets 404 for any invoice id in org 2, on read, download, and update.
- Regression test: two orgs, two users, cross-org request on each route fails.
- Staging log shows no cross-org 200s after the deploy.

## Verification

Reproduced on staging at commit `a1b2c3d` with two test accounts. Not verified: the export-to-CSV job and the admin impersonation path.

## Deduplication

Related: none open. Distinct from any rate-limiting issue because the missing control is ownership, not volume. If a similar gap is found on `/api/customers`, add it here as an affected call site, since the shared helper closes both.
