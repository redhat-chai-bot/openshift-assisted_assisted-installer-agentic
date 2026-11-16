# Investigation and CI-infra filtering

Read when investigating prioritized Tier 1 jobs. Requires
`assisted-installer-prow-job-debugger`. Filtering pure CI infrastructure
issues needs root-cause evidence from that skill; it is not a separate
pre-investigation phase.

## Per-job order

For each prioritized job (longest streak first):

1. Analyze the most recent failure with `assisted-installer-prow-job-debugger`.
   Follow that skill's methodology and report contract.
2. Check 4–5 build logs across the streak when durations or error patterns
   diverge.
3. Decide whether the streak is pure CI infrastructure (mark
   `suitable_for_ticket=false`) or a product/test issue (continue).
4. Compare related jobs and identify configuration differentiators.
5. Produce root cause, evidence, severity (Low/Medium/High), layer
   (Infrastructure / Test / Product), suggested fixes, and whether the job is
   suitable for a later user-initiated ticket (`suitable_for_ticket`).

## CI infrastructure skip rule

Mark `suitable_for_ticket=false` when **all** examined failures in the streak
share the same pure CI infrastructure root cause (for example Hive cluster
pool exhaustion, build-pod init hung) with no underlying product or test
issue. Verify by checking runs before the streak for a different root cause.
If evidence is insufficient to decide, keep investigating rather than marking
unsuitable. This skill never files tickets; the flag only guides a later
separate `prow-job-jira-ticket-creation` call if the user requests one.

## Duration signatures

Classify run duration to spot mixed root causes in one streak:

| Signature | Duration | Use |
|-----------|----------|-----|
| Short | < 60 min | Often provisioning, setup, or early script failure |
| Medium | 60–200 min | Compare error patterns; do not assume short/long cause |
| Long | > 200 min | Often timeout or late-phase install/test failure |

If short and long failures both appear in the streak, investigate both modes
and document separate root causes when they differ.

## Presubmit error-pattern hints

When classifying presubmit (or mixed) streaks, search build logs for:

- `OFCIR ACQUIRE ERROR` -- bare-metal resource provisioning timeout
- `cluster operator .* is degraded` -- operator bootstrap issue
- `cluster operator .* is not available` -- operator not reaching Available
- `make: ***` -- build failure (often PR-specific)
- `ssh: Could not resolve hostname` -- infrastructure provisioning failure

PR-specific build failures belong in the streak table but not in systemic root
cause analysis.

## Cross-job comparison

Compare pass rates for related jobs (same repo, branch, infrastructure type):

- All bare-metal jobs fail similarly → systemic infra issue
- Only this test fails → test configuration issue
- Sibling topology with different config passes → configuration-specific
  regression

Check differentiators: `AGENT_ISOLATED_NETWORK`, registry settings, topology,
OCP payload version counterparts, install method (agent ISO, ZTP, IPI).
