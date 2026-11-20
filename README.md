# AI Hardening Passes

Checklists for anyone shipping an AI-built app to real users.

I run a government contracting business and I build the software that runs it with AI, daily. These are the passes I actually run on that code before real money touches it. Not theory. The failure modes below are the ones AI-generated code ships with by default, because nobody asked it not to.

The good news: every fix here is promptable. You do not need to write the code yourself. You need to know what to ask for. That is the whole game.

## The passes

| Pass | What it catches |
|---|---|
| [Security pass](passes/01-security-pass.md) | The login screen you just added is the softest target in your app |
| [Scale pass](passes/02-scale-pass.md) | The first 50 concurrent users kill the database, not the hundredth |
| [Performance pass](passes/03-performance-pass.md) | Slow by default: unpaginated tables, no caching, main-thread everything |
| [Role prompts](passes/04-role-prompts.md) | How to make the AI review its own work like a senior engineer instead of agreeing with you |
| [Production security pass](passes/05-production-security-pass.md) | The failures beyond login: tenant isolation, SSRF, queues, webhooks, billing, deletion, and resource limits |

## Use it as a skill

`skills/security-pass/` is a ready-to-install [Claude Code skill](https://docs.claude.com/en/docs/claude-code) version of the security pass. Drop it into your project's `.claude/skills/` directory (or `~/.claude/skills/` for all projects), then say "run a security pass" in Claude Code. It audits your actual code and demands evidence for every item instead of reassurance. The other passes are prompt checklists you can paste directly.

## How to use these

Run each pass against your app before launch, then again any time the AI builds a new surface. Each item is phrased so you can paste it into your AI session as a work order. Make the AI show you evidence, not reassurance: the test, the config line, the header in the response.

For a full-repository review, start with the short security pass, then run the production security pass by trust boundary. Record findings with the [audit finding template](templates/audit-finding.md) so another engineer can reproduce, prioritize, and close them without rereading the entire audit. See a [worked example](templates/example-finding.md).

## Who this is for

Solo builders and small teams shipping AI-written code to production. If you have a security team, they already know this. If you ARE the security team, this is for you.

MIT licensed. Use it, fork it, ship safer.
