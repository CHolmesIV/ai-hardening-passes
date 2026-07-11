# Audit Finding Template

Use one issue per root cause. Combine affected call sites when one shared fix closes them all.

## Title

`[Boundary] Imperative description of the required protection`

## Severity

`Critical | High | Medium | Low` with one sentence covering exploitability and impact.

## Evidence

- Affected code/config with file and line references
- The trust boundary crossed
- Preconditions required

Do not paste secrets, customer data, working credentials, or a destructive public exploit into an issue.

## Failure scenario

Describe the shortest reproducible path from attacker, user action, race, or failure to impact. Distinguish what was demonstrated from what was inferred.

## Impact

State the user, tenant, financial, availability, privacy, or operational consequence. Avoid generic "could be insecure" language.

## Recommended fix

Describe the smallest safe control and where it must be enforced. Include migration, rollout, compatibility, and cleanup concerns when applicable.

## Acceptance criteria

- Concrete behavior that must hold
- Negative or cross-boundary regression test
- Operational evidence required from staging or production

## Verification

Record commands/tests run, result, environment, commit reviewed, and any area not verified.

## Deduplication

Link related issues. Explain why this is a distinct root cause, or add the evidence to the existing issue instead of opening a duplicate.
