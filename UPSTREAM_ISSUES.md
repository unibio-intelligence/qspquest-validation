# Upstream observations and domain boundaries

No upstream issue or pull request has been filed yet. A tested patch is in
`upstream-patches/rxode2/`. These observations apply to the pinned package
versions, not every version of nlmixr2/rxode2. Per-case results are in the result
reports; the runners regenerate the references.

## rxode2 5.1.8: infusion `podo()` differs between readout and dynamics

An independent synthetic reproducer doses 10 units into `depot` over three time
units. The model reports `podo(depot)` as 10, while `d/dt(integrated_history) =
podo(depot)` integrates 10/3. At time four, the integrated value is approximately
13.3333; integrating the reported amount would give 40.

Run `Rscript scripts/probe_rxode2_infusion_history.R`. The discrepancy also
appears in the artificial infusion stress condition of
`Cirincione_2017_exenatide`. It is not a clinical infusion regimen claim.

As a workaround, an extra reference run replaces `podo(depot)` in the R equations
with a step covariate holding the administered amount. The helper
`scripts/nlmixr_dose_history_reference.R` accepts one individual with explicit
depot doses in time order and finite amounts, and rejects ambiguous simultaneous
doses and resets. QSPQuest's equations, parameters and dosing are unchanged. The
original discrepancy and the workaround match are reported as separate rows.
The patch removes two event-handler assignments that overwrite the administered
amount with the internal infusion rate. New runs of patched rxode2 and the current QSPQuest engine match on all 404 valid requests with BDF. Nine analytic
history tests cover bolus, duration/rate infusions, bioavailability, unequal
repeat/overlapping doses, compartments, subjects and no-dose zero-history policy.
See `upstream-patches/rxode2/README.md` for the patch and reproduction commands.

## nlmixr2lib 0.3.2: inhibition template IC50 identities

The `r0/rmax/crmax` input-inhibition template defines
`Imax=(R0-Rmax)/R0` and computes the numerator
`Rmax-(1-Imax)*R0`, which is identically zero. The output-inhibition template
uses `Imax=(Rmax-R0)/Rmax` and computes
`R0-(1-Imax)*Rmax`, also identically zero. Parameter substitution cannot make
these identities positive. The formulas overwrite separately declared `imax`.
The precursor template inherits the resulting zero half-maximal inhibitory
concentration (IC50).

With zero exposure, the inhibition fraction becomes an undefined 0/0. QSPQuest
rejects it; rxode2's safe division can return a finite value instead, which is not
a solution of the original equations. These template defects are candidates for
an upstream report with the algebra, original sources and examples.

The precursor templates also refer to `kin` while defining `kin` without an
independent declaration and omit drug-free stationary initial conditions. The
stimulation defaults yield a negative half-maximal effective concentration
(EC50) and dose-dependent poles. Both
engines fail at the two attempted pole-crossing conditions. Do not substitute
an arbitrary positive EC50 and call the result the same source model.

## Mathematical boundaries rather than solver bugs

- `Xie_2019_agomelatine`: logarithmic readout at zero concentration is undefined.
  rxode2's safe-log policy floors the value; QSPQuest rejects log(0). A linear
  concentration readout is a separate, well-defined observation contract.
- Two tumor-growth templates have finite-time blow-up before the requested
  endpoint of 24. Continuing beyond the pole has no finite mathematical solution.
  One R run returns missing values; another returns a very large finite number.
  A finite CSV entry is not sufficient evidence of a valid continued solution.
- `Cirincione_2017_exenatide` without an administration: `podo()`/`tad()` history
  is absent. The original R CSV contains missing values after the initial row,
  despite a successful process exit. QSPQuest rejects undefined history. A real
  zero-amount administration supplies history and is a different event contract.

The 16 rejected requests are listed as negative cases. They are not counted as
passes, and they are not rerun with different parameters or doses.
See `NEGATIVE_CONDITIONS.md` for the exact requests and actual R outcomes.
