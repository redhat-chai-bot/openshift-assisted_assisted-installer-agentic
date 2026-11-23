---
name: prow-job-jira-ticket-creation
description: Use when creating or proposing a MGMT Jira task for a broken Assisted Installer CI Prow job after investigation, with user approval before any create or update.
---

# CI Prow Job Jira Ticket Creation

Provide a unified Jira ticket procedure for broken CI Prow jobs. Invoke this
skill only when the user explicitly asks to create or update a ticket.
Evidence may come from a prior `detect-broken-jobs` report or from other
investigation; `detect-broken-jobs` must not call this skill automatically.

## Inputs and prerequisites

- Job type: `periodic` or `presubmit`.
- Job name, repo, and branch.
- Failure streak length and dates; for presubmit, distinct PRs in the streak.
- All Prow URLs in the streak (at least the most recent failure URL).
- Root cause analysis with evidence, severity, failure layer, and suggested fix
  directions. Optional: tier classification and cross-repo impact.
- Require authenticated Jira search and, after approval, create or update
  capability through the invoker's available providers. Prefer semantic Jira
  operations; use a CLI only when its syntax and authentication are known.
- Optional local path for saving the proposed or created ticket summary.

Return `blocked` if required investigation inputs or Jira access are missing.

## Boundaries and approval

Do **not** create or update Jira tickets automatically. Present the proposed
ticket (fields, title, description, remote links, and create-vs-update choice)
and wait for explicit user approval before any remote mutation. Silence is not
consent. Return `awaiting_approval` when a decision is pending.

Pre-check search is read-only. The only permitted writes after approval are the
approved Jira create or update (including remote links and labels) and an
optional local summary file.

## When to file

File a Jira task when:

- A job has been classified as Tier 1 (Broken) and has a clear,
  non-infrastructure, non-PR-specific root cause
- The investigation identifies actionable fix directions
- No existing Jira issue already tracks the problem (see Pre-Check below)

## Pre-Check: search for existing issues

Before creating a new ticket, search for existing issues. Prefer excluding
terminal statuses your Jira instance uses for done work (at least `Closed` and
`Resolved` when present):

- `project = MGMT AND text ~ "<job-name>" AND status not in (Closed, Resolved)`
- `project = MGMT AND text ~ "<root-cause-keyword>" AND status not in (Closed, Resolved)`

`text ~` is a fuzzy heuristic; job-name tokens can match unrelated issues.
Treat hits as candidates to review, not automatic duplicates. Confirm the
prescribed component and label exist in MGMT before relying on exact values; if
either is missing, stop and report `blocked` with the mismatch.

If a matching open issue exists, propose updating it instead of creating a
duplicate, and obtain approval for that update.

## Jira fields

| Field | Value |
|-------|-------|
| Project | MGMT |
| Type | Task |
| Component | Assisted-Installer CI Prow |
| Label | Assisted-Installer-CI-Prow |

### Title format

`CI: <short-job-identifier> broken -- <root cause summary>`

Keep the title compact. Do **not** include consecutive failure counts in the
title.

### Remote links

- Add at least one failed Prow job URL as a remote link on the ticket.
- Use the most recent failure URL from the streak.
- Title format: `Prow: <short-job-identifier> -- <date> failure`

## Description

Build the description from
[description-structure.md](references/description-structure.md). Exclude
PR-specific-only analysis, speculation without log evidence, and pure CI
infrastructure streaks (do not file those at all).

## Output and stopping conditions

Return the proposed or applied ticket summary (fields, description outline,
remote links, existing-issue matches, and create-vs-update decision). Optionally
write it to the requested local path.

- `complete`: approved create or update succeeded and was verified, or the user
  approved skipping because an existing issue already tracks the problem.
- `awaiting_approval`: proposal is ready and a user decision is pending.
- `declined`: user rejected create and update; no remote mutation occurred.
- `skipped`: filing criteria were not met (for example pure CI infrastructure
  failures, missing root cause, or PR-specific only).
- `blocked`: missing inputs, Jira access, pre-check failure, or an approved
  mutation could not be completed.
