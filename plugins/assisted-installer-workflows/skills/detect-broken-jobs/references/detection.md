# Detection: periodic and presubmit

Read when classifying failing jobs in Phase 1. Apply only the scopes the
caller requested.

## Shared defaults

| Parameter | Default | Description |
|-----------|---------|-------------|
| Orgs | openshift, openshift-assisted | GitHub organizations |
| Repos | Repos containing "assisted" | Filter by team |
| Branches | All | Include all branches |

## Periodic jobs

| Parameter | Default | Description |
|-----------|---------|-------------|
| Window | 7 days | Lookback for execution history |
| Min consecutive failures | 2 | Minimum back-to-back failures to flag (user-configurable) |

For each `job_name` + branch (job_type = periodic):

1. **Consecutive failure tail**: order runs by timestamp DESC; count newest
   consecutive failures (streak breaks at first success).
2. **Overall success rate**: successes / total_runs over the window.
3. **Days since last success**.
4. **Most recent failure URL** and **all failing run URLs** in the streak.

If a job has only one execution in the window, extend lookback until a success
is found or the minimum consecutive-failure count is reached.

### Periodic tiers

| Tier | Name | Criteria | Meaning |
|------|------|----------|---------|
| **1** | Broken | Consecutive failure tail >= min consecutive failures (default 2) | Needs investigation |
| **2** | Highly Unstable | Success rate < 35% with 10+ runs in the window, not in Tier 1 | Unreliable but not a full consecutive-failure streak |

Note: with a 7-day window, once-daily jobs (~7 runs) cannot reach Tier 2's
10-run floor. Extend the window or lower the run floor when those jobs matter,
or rely on Tier 1 consecutive failures.

### Periodic output fields

Job name, repo, branch, streak length, pass rate, days since last success,
total runs, most recent failure URL, all streak URLs, correlated-presubmit
flag (see [correlation.md](correlation.md)).

## Presubmit jobs

| Parameter | Default | Description |
|-----------|---------|-------------|
| Window | 14 days | Lookback period |
| Min distinct PRs (Tier 1) | 3 | Distinct PRs in failure tail for Broken |
| Min distinct PRs (Tier 2) | 2 | Distinct PRs in failure tail for Needs Attention |

For each `job_name` + branch (job_type = presubmit), compute the same streak /
success-rate / URL fields as periodic, plus **distinct PR numbers** in the
consecutive failure tail.

### Presubmit tiers

| Tier | Name | Criteria | Meaning |
|------|------|----------|---------|
| **1** | Broken | Failure tail spans **3+** distinct PRs (or configured Tier 1 minimum) | Definitely broken -- no single PR caused this |
| **2** | Needs Attention | Failure tail spans **2+** distinct PRs (or configured Tier 2 minimum), not Tier 1 | Very likely broken |
| **3** | Highly Unstable | Success rate < 35% with 10+ runs, not in Tier 1/2 | Occasionally passes but effectively blocks PRs |

### Presubmit output fields

Same as periodic, plus distinct PRs in streak and
also-broken-as-periodic flag (see [correlation.md](correlation.md)).

## Optional exclusions

List excluded jobs separately with Prow URLs when the user asks to exclude:

- **lint / unit-test**: often dependency or formatting root causes
- **Image build jobs** (`images`, `mce-images`, `okd-scos-images`,
  `edge-images`): usually share one root cause -- investigate together

## Notes

- Image build jobs on the same repo/branch usually share one root cause.
- When a test has a sibling variant (e.g. `compact-ipv4` vs
  `compact-ipv4-iso-no-registry`), compare pass rates; configuration
  differences are often the differentiator.
- Presubmit tiers: Tier 1 catches fully blocked jobs; Tier 2 catches
  likely-broken jobs earlier; Tier 3 catches intermittent blockers that
  streak rules miss.
