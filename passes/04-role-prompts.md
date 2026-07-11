# Role Prompts: Make the AI Argue With Its Own Code

The default mode of every coding assistant is agreeable. It builds what you asked, tells you it went great, and moves on. The fix is to make it wear a different hat and attack its own output. These are the roles I actually assign, with what each one is for. Write your own wording; the role and the demanded deliverables are what matter.

## The auditor

New session, fresh eyes: "You are a senior engineer who just inherited this codebase and trusts nothing. Reverse-engineer the architecture, then report: bad decisions, duplicated logic, bottlenecks, scaling risks, maintenance traps. Do not change behavior. Rank the problems." Run this monthly and after any big AI-built feature. A fresh session matters: the session that wrote the code will defend it.

## The security engineer

"Audit this app like a paid security review. Vulnerabilities, auth flaws, injection risks, data exposure, infrastructure gaps. For each: severity, a concrete attack scenario, and the fix." The attack scenario is the key demand. "This could be vulnerable" is noise; "here is the request I would send and what it returns" is a finding. Pair it with the [security pass](01-security-pass.md).

## The debugger

For production issues: "Treat this as a live outage. Before touching anything: what does this code actually do, what is the real root cause, why does the failure happen, what edge cases hide nearby. Then the most robust fix, not the fastest one. Do not guess." The "do not guess" clause is doing real work. Without it you get plausible fixes for the wrong bug.

## The performance engineer

"Profile this app like it is about to take serious traffic. Find bottlenecks, wasted renders, expensive operations, memory leaks. Report numbers first, then a ranked fix list." Numbers first. Otherwise you get vibes.

## The tech lead

The most underused one. Before building anything: "Act as the tech lead responsible for maintaining this product for five years. Before writing code: ask clarifying questions, challenge weak decisions, flag scaling risks, propose the simpler approach if one exists. Then give me the tradeoff analysis and the plan." This turns the model from a code generator into something that pushes back, which is the thing you actually needed.

## The architect, the frontend engineer, the DevOps engineer

Same pattern, scoped: design the system before building the minimum version of it; build UI components with loading, empty, and error states handled instead of just the happy path; prepare the deployment with CI, monitoring, and a rollback story. Every role prompt follows one formula:

**Seniority + specific responsibility + explicit deliverables + a constraint ("do not change behavior", "numbers first", "ask before building").**

## The rule that makes all of these work

One role per session, run it against work the session did not produce, and demand artifacts: a ranked list, a test, a config, a number. If the output is a compliment, the prompt failed.

For a large codebase, divide the review by trust boundary rather than arbitrary folders: identity and authorization, tenant data, file and network I/O, asynchronous work, billing, frontend state, and operations. A cheaper model can inventory routes and trace call paths; the senior reviewer should validate exploitability, reject speculative findings, set severity, and own the final recommendation. Do not let multiple reviewers file the same root cause as separate issues.

**Second-eye work order:** "Review code you did not write. Make no changes on the first pass. For each finding, provide the affected boundary, concrete failure or attack path, code evidence, user impact, smallest safe fix, and a regression test. Check existing issues before filing. If you cannot demonstrate impact from the code or a test, label it a question, not a vulnerability."
