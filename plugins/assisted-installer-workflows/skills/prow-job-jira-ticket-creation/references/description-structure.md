# Jira description structure

Use this outline when drafting the ticket description.

## Summary

One paragraph: which job, which repo/branch, job type (periodic/presubmit),
failure streak length, and current status. For presubmit jobs, include the
number of distinct failing PRs. State clearly whether the root cause is
PR-specific or systemic.

## Failure series

- Start date of the first failure after a successful run
- Recovery date if applicable
- Table of the streak with: date/time, state, duration, classification
  (for presubmit: include PR number)
- Include 1-2 successes before the streak for context

## Job details

- Full job name
- Repo and branch
- Platform and infrastructure type
- OCP version (if applicable)

## Failing runs

Links to all Prow URLs of the failing runs in the streak.

## Failure analysis

- Severity: Low / Medium / High
- Failed step name
- Exit condition
- Layer: Infrastructure / Test / Product

If multiple failures have different root causes, document each in its own
Failure Analysis section with separate evidence and fix suggestions.

## Root cause

Detailed explanation with evidence -- relevant log excerpts in code blocks.

For presubmit jobs, include:

- **Why This Test Specifically**: Cross-job comparison table. Identify the
  configuration differentiator.
- **Regression Signal** (if applicable): Compare with older OCP version
  counterparts.
- **Cross-Repo Impact**: List other repos where the same test is also failing.

## Failure type breakdown

Summary table: failure type, count, percentage, duration signature
(short < 60 min, medium 60–200 min, long > 200 min).

## Suggested fix

Numbered list of actionable investigation/fix paths.

## References

Prow URLs for the latest failures on each affected repo.

## What to exclude

- **PR-specific build failures** -- for presubmit, mention them in the streak
  table but do not include in root cause analysis.
- **Speculation** -- only include failure modes confirmed by log evidence.
- **CI infrastructure transient issues** -- if all failures are pure infra
  problems, do not file a ticket at all.
