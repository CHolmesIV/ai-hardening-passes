---
name: security-pass
description: Run a pre-launch security pass on an AI-built app. Use when the user says "security pass", "harden my app", "audit my auth", "is my login secure", or is about to ship an app with authentication to real users. Checks session storage, server-side authorization, signup verification, auth rate limiting, password policy, and leaked secrets. Demands evidence for each, not reassurance.
---

# Security Pass

You are running a pre-launch security pass. Your job is to VERIFY, not to reassure. For every item below: inspect the actual code/config, state PASS or FAIL with the evidence (file, line, setting, or test output), and for every FAIL, implement the fix or hand the user a precise work order. Never mark an item PASS without showing the evidence.

Work through all six. Report as a table at the end: item, verdict, evidence, fix applied or owed.

## 1. Session token storage
Find where the session token/credential lives after login. localStorage or a JS-readable cookie = FAIL (XSS anywhere means session theft). PASS = httpOnly cookie or a managed auth provider handling sessions. Evidence: the storage call or Set-Cookie header.

## 2. Server-side authorization
For every privileged/admin/role-gated endpoint: the role must be loaded server-side (from the database or verified session), never trusted from the request body, query, or an unverified token claim. Grep for role checks; trace where the role value originates. Evidence: the check + origin. Then write or run a test proving a forged/low-privilege request gets 403.

## 3. Signup identity
Email verification must be on. If the auth provider supports 2FA, it should at least be available to users. Evidence: the provider setting name/value or the code path enforcing it.

## 4. Auth endpoint rate limiting
Login, signup, and password reset must be rate limited (provider-side counts if a managed provider fronts them, but verify, don't assume). Evidence: limiter config, then prove it: fire rapid repeated attempts and show the 429/lockout.

## 5. Password policy
Minimum length enforced and, where the provider supports it, breached-password checking enabled. Evidence: setting or validation code.

## 6. Secrets in the repo
Scan the working tree AND git history for API keys, tokens, connection strings, private keys. Report every hit (file, line, what it appears to be). Anything real found must be ROTATED, not just deleted. Deletion without rotation is not a fix, and say so explicitly.

## Output contract
Finish with:
1. The six-row verdict table.
2. Fixes applied this session (diffs summarized).
3. Remaining owed items with exact work orders the user can run later.
Do not soften FAILs. An app that fails items 1, 2, or 6 is not ready for real users; say that plainly.
