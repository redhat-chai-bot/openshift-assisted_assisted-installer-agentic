# Periodic↔presubmit correlation

Read when flagging that a broken periodic and a broken presubmit are the
"same test". Prefer the job-name parsing in
`assisted-installer-prow-job-debugger` (`references/gcs-and-config.md`) when
that skill is loaded; the rule below is the workflow's required matching
contract.

## Job-name shape

OpenShift CI job names typically look like:

```text
{pull|periodic|branch|postsubmit}-ci-<org>-<repo>-<branch>-<test-as>
```

Examples:

- `periodic-ci-openshift-assisted-service-master-e2e-metal-assisted`
- `pull-ci-openshift-assisted-service-master-e2e-metal-assisted`

Org may be `openshift` or `openshift-assisted` (or another org from the query
parameters). Multi-segment repos and branches are resolved by matching the
longest `\<repo\>-\<branch\>` prefix that corresponds to a known
`ci-operator` config, as in the debugger skill; the remainder is **test `as`**.

## Match key

Two jobs correlate when all of the following are equal after parsing:

| Field | Source |
|-------|--------|
| org | From the job-name prefix after `{pull\|periodic\|...}-ci-` |
| repo | Parsed repo segment |
| branch | Parsed branch segment (exact string; `master` ≠ `main`) |
| test `as` | Suffix after repo-branch |

Do **not** correlate solely on a raw string suffix of the full job name. Strip
the `{pull|periodic|branch|postsubmit}-ci-` prefix first, then compare the
`(org, repo, branch, test-as)` tuple.

If parsing is ambiguous, grep the full job name in `ci-operator/jobs/` (via
the debugger skill's config walk) before asserting a correlation. Unresolved
ambiguity → leave the correlation flag unset and note the uncertainty.

## How to use the flag

- For each broken **periodic**, mark whether a matching presubmit exists in the
  detection results (or would share the same match key).
- For each broken **presubmit**, mark whether a matching periodic appears in
  the failing periodic set.
- Prioritize investigation when both sides of a match key are broken; treat
  that as evidence of systemic breakage rather than a single PR.
