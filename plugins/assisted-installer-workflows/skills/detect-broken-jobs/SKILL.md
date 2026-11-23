---
name: detect-broken-jobs
description: Use when detecting broken Assisted Installer periodic and presubmit Prow jobs and investigating root causes.
---

# Detect Broken Jobs

Orchestrate detection of broken Assisted Installer **periodic** and
**presubmit** Prow jobs and investigate prioritized failures. Own task
selection, sequencing, and aggregate results.

## Inputs and prerequisites

- Optional scope: `periodic`, `presubmit`, or both (default both).
- Optional query overrides for each job type (defaults in
  [detection.md](references/detection.md)).
- Optional local report path.
- Require CI job-history query access covering the requested windows (for
  example BigQuery or OpenSearch). Prefer semantic job-history operations; use
  a provider-specific CLI only when its syntax and authentication are known.
- Require `assisted-installer-prow-job-debugger` from `assisted-installer-skills`
  for per-job investigation, including CI-infra filtering that needs root-cause
  evidence. Load and use that skill; do not recreate its procedure. If it is
  unavailable, stop investigation (and any root-cause-dependent filtering) and
  report `blocked` with the missing prerequisite. Detection-only results may
  still be returned.

## Boundaries

Read-only aside from optional local report writes. Do not create, update, or
propose Jira tickets, and do not invoke `prow-job-jira-ticket-creation`. Ticket
filing is a separate user-initiated call after the user reviews this skill's
output.

## Procedure

1. **Detect** failing jobs for the requested scopes using
   [detection.md](references/detection.md). Correlate periodic↔presubmit using
   [correlation.md](references/correlation.md).
2. **Investigate** prioritized Tier 1 jobs (longest streaks first) using
   [investigation.md](references/investigation.md). CI-infra filtering happens
   during investigation once root-cause evidence is available; do not skip jobs
   as pure infra without that evidence.
3. **Stop** after returning the aggregate report.

## Output and stopping conditions

Return the detection tables (periodic and/or presubmit), correlation flags,
investigation findings (including whether each job is suitable for later
ticket filing), and optional local report. Include enough evidence that a
later `prow-job-jira-ticket-creation` call can reuse it without re-detecting.

- `complete`: detection finished for the requested scopes; every prioritized
  Tier 1 job is investigated or intentionally skipped as pure CI
  infrastructure or out of scope.
- `partial`: some prioritized jobs were investigated and others remain pending
  without a blocking prerequisite failure.
- `empty`: detection finished with no matching broken or unstable jobs.
- `blocked`: missing query access, required debugger skill, incomplete
  detection, or every intended investigation path failed.
