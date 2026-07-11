# The Security Pass

Your AI just added a login screen. It looks legit. It is also, right now, the easiest thing in your app to break. AI-generated auth ships with the same five gaps almost every time, because the happy path works and nobody asked about the unhappy ones.

Run every item. Demand evidence, not "should be fine."

## 1. Where does the session token live?

If the answer is localStorage, any cross-site-scripting hole anywhere in your app hands out sessions. You want httpOnly, Secure cookies, or an auth provider (Clerk, Auth0, Supabase Auth) that manages sessions for you.

**Work order:** "Show me exactly where the session token is stored and how it is sent on each request. If it is readable from JavaScript, move sessions to httpOnly cookies and show me the Set-Cookie header."

## 2. Where does the authorization check happen?

Hiding the admin button is not an admin check. Every privileged action must be verified on the server, from the database, on every request. Never from a role field the client sent.

**Work order:** "For every admin or role-gated endpoint, show me the server-side check and where the role is loaded from. If any check trusts a value from the request or token without re-reading it server-side, fix it and add a test that a forged token gets a 403."

## 3. Can anyone sign up as anyone?

No email verification means I can register as your customer. No 2FA option means one leaked password is game over for that account.

**Work order:** "Turn on email verification for signups. Add or enable optional two-factor auth. Show me the settings or the code path that enforces both."

## 4. Are the auth endpoints rate limited?

Login and password reset with no rate limit is an open invitation to brute force. This one is a config change, not a project.

**Work order:** "Add rate limiting to login, signup, and password reset. Show me the limiter config and prove it: script 50 rapid attempts and show me the 429s."

## 5. What stops a garbage password?

Minimum length, and a check against known-breached passwords. Most auth providers have a toggle for this. Flip it.

**Work order:** "Enforce a password policy and enable compromised-password checking. Show me the setting."

## The bonus item almost everyone misses

Secrets in the repo. AI assistants paste API keys into code, comments, and test files constantly. Before you push anything public:

**Work order:** "Scan the entire repo history for anything that looks like a secret: API keys, tokens, connection strings, private keys. List every hit with file and line."

Then rotate anything it finds. Finding it is not the fix. Rotating it is the fix.
