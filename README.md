# QSPQuest validation

Numerical comparisons for [QSPQuest](https://github.com/unibio-intelligence/qspquest):
what was tested, what matched, and where the results apply. These results apply to
QSPQuest builds with the numerical identity in [environment.json](environment.json):
the same solver sources and locked dependencies. A release that changes those is
re-recorded. A match means agreement within stated tolerances.
QSPQuest now uses upstream Diffsol, which has no Rodas5P; native results that
used Rodas5P in earlier builds are now recorded with BDF.

| Test | Coverage | Result |
| --- | --- | --- |
| OMEX simulations | 8 models, 47 scenarios, BDF; six reference engines | All 47 scenarios matched each reference |
| Additional source presets | 597 fixed presets | All matched their retained reference trajectories |
| Upstream conditions | 59 model definitions, 420 conditions | 402 matched unmodified rxode2; 1 matched an explicit dose history; 1 rxode2 infusion discrepancy matched after a documented fix; 16 rejected (see below) |
| Individual fitting | 1,906 contracts from 21 datasets, run with BDF | All 1,906 fits converged and matched parameters, likelihoods and predictions |

These cohorts overlap; their counts are not additive. Agreement supports the
tested equations, inputs and domains. It does not establish biological validity,
clinical suitability, all-model fitting support or population-fitting equivalence.

## Start here

- **Inspect or replay one result:** [First replay](docs/first-replay.md) verifies
  the archive and runs one retained preset with the CLI.
- **Understand the tests:** [Methods](METHODS.md) gives sampling rules,
  tolerances, parameter roles and optional independent-reference reruns.
- **Inspect the evidence:** [Summary](results/summary.json),
  [simulation rows](results/simulations.csv) and [fitting rows](results/fitting.csv).
  Compressed reports in `results/` retain detailed metrics and engine identities.

## What the comparisons establish

Simulation comparisons use identical observation grids and compare every declared
state and output. The OMEX references are rxode2, mrgsolve, OrdinaryDiffEq,
RoadRunner, COPASI and AMICI. The additional source presets use retained
source-specific references; replaying them does not regenerate those references.

Individual fits use bounded Gaussian likelihoods with fixed nuisance parameters
and no estimated random effects. Every native fit must converge and satisfy
parameter and prediction limits of `1e-6 + 0.001*abs(reference)` and a normalized
negative-log-likelihood difference of at most `1e-4`.
[Five datasets remain outside the fitting claim](REMAINING_FITTING_CONTRACTS.md).
Population fitting is outside this comparison set.

The 16 rejected conditions cover undefined expressions, absent dose
history and trajectories without a valid finite solution. They are negative
outcomes, not simulation successes. See the
[condition-by-condition diagnostics](NEGATIVE_CONDITIONS.md).

An infusion-history discrepancy in rxode2 prompted a separate corrected-reference
comparison. Unmodified rxode2 failed seven of nine analytic history regressions;
the patch passed all nine. These regressions test the reference correction;
native agreement is assessed separately. The
[findings](UPSTREAM_ISSUES.md) preserve the original outcomes and the
[patch notes](upstream-patches/rxode2/README.md) describe reproduction.

## Scope

[Environment](environment.json) identifies the tested engine and reference
versions. These comparisons apply to that build and the stated inputs.

The COMBINE/OMEX archives are test fixtures. Native fitting and JSON model profiles
are identified explicitly. To find and use models, visit the companion
[model library](https://github.com/unibio-intelligence/qspquest-models).

Contributions are welcome: open an issue first, then follow the
[contributing guide](CONTRIBUTING.md). Validation contributions are
GPL-3.0-only; component notices remain applicable.
See [third-party notices](THIRD_PARTY_NOTICES.md).

## Acknowledgements

We gratefully acknowledge support from Anthropic's AI for Science rare disease
research grants.
