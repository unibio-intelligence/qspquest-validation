# Test methods

This document describes what was tested and the pass criteria, in three parts:
simulation accuracy, parameter estimation accuracy, and failure handling and
domain limits.

## Simulation accuracy

### States, outputs and solver coverage

For the eight OMEX (Open Modeling EXchange) models, the reference adapters read
the Systems Biology Markup Language (SBML) equations and Simulation Experiment
Description Markup Language (SED-ML) sampling times directly. Every state and output must satisfy
`abs(native - reference) <= 1e-8 + 1e-4*abs(reference)` at the same observation
times, with no interpolation. Simultaneous scheduled assignments are split into
segments so that both the initial post-event sample and later pre-event samples
are compared. `omex-simulations.omex` contains the eight original model archives
and their 94 native requests. QSPQuest runs the 47 backward differentiation
formula (BDF) requests with their stored
tolerances and callback budgets. The 47 Rodas5P requests are kept from earlier
engine builds and are not run: the current engine uses upstream Diffsol, which
has no Rodas5P.
Each reference adapter's settings are in the scripts and per-case reports.
Normalizing the SBML time symbol changes notation only. AMICI uses Adams with
fixed-point iteration for the Yuan cases whose fractional power has a singular
analytic Jacobian at zero, and BDF/Newton for the other cases.

The 592 additional fixed presets use the same tolerance and exact time grids.
`source-presets.omex` holds each preset's native JSON model and request and one
independent reference CSV. QSPQuest is rerun against these stored references; the
original tools that produced them are not rerun. Their hashes are in
`results/source-presets.json.gz`, and source scope and attribution are in
`inputs/model-sources.json`. The models of 176 presets also declare
constant bookkeeping outputs from the October 2026 recording, such as
`validation_run_nonce` and source-acquisition flags. They are compared like any
other output but carry no scientific meaning; current model library versions do
not include them. Presets that were left out or held back are not
counted in the 592. This tests simulation only, not fitting or full-paper
reproduction.

Most presets run with BDF; 11 use TSIT45. The 28 presets from seven models that
earlier builds ran with Rodas5P now use BDF with the 1,000x or 10,000x tighter
local tolerances chosen for the model library. Their models, grids, doses and
budgets are unchanged, and they are compared with the same references and limits.

### Upstream model and condition coverage

The 59 R models come from nlmixr2lib 0.3.2. All 420 conditions are run with BDF,
using typical individual effects (eta=0), fixed parameters and
explicit dose events. Source-to-native symbol mappings are included. Comparisons
require finite values on the exact time grid, with
`abs(delta) <= 1e-7 + 1e-6*abs(reference)` for every compared output, including
the mapped source states. The original reference runs enable rxode2's
safeZero/safeLog/safePow guards; a finite value produced by a guard is not
counted as a solution of an undefined expression.

With BDF, the results are 402 matches against the unchanged reference,
one more match using an explicit administered-amount reference, one
infusion-history discrepancy with the original reference, and 16 rejections. The
match that needed a changed reference is not counted as an unchanged-reference
pass.

A second comparison used rxode2 rebuilt from the pinned source with an
infusion-history patch. BDF had 404 valid matches and 16 rejections. Both
R and QSPQuest were rerun for the recorded engine. The patch changes only how the
reference handles infusion history; native equations and request parameters are
unchanged. Original and corrected results are reported separately.

### Dosing and event behavior

Scheduled assignments and dose events are assessed at their declared sample
positions, including initial post-event and later pre-event observations. The
upstream infusion-history discrepancy is documented in `UPSTREAM_ISSUES.md`.

Nine analytic test cases checked the reference's administered-dose history across
bolus, duration/rate infusions, bioavailability, unequal repeat and overlapping
doses, compartments, subjects, and a no-dose zero-history policy. The pass limit
was absolute error <= `1e-7`. Unmodified rxode2 failed seven of nine cases; the
patched reference passed all nine, with maximum absolute error below `2e-13`.
These cases test the reference patch, not QSPQuest's dosing behavior directly.

## Parameter estimation accuracy

`individual-fitting.omex` contains every final qsp definition, observation row,
dose event, parameter role, physical bound, known error standard deviation (SD),
reference equation,
and native fitting request; only the BDF requests are run. It uses a COMBINE container with a native fitting
format, not a SED-ML estimation task. Data source hashes and the original GPL
contributor records are included.

There are 1,906 contracts from 21 datasets, each executed with BDF.
The objectives are fully normalized individual Gaussian likelihoods with no
estimated random effects. Known additive SD is 20% of maximum subject response.
The twelve ACOP examples estimate clearance or Vmax; theophylline estimates
clearance, volume and absorption rate; phenobarbital estimates clearance and
volume. Other contracts estimate the declared single response, weight,
pharmacokinetic (PK), myelosuppression, physiologically based pharmacokinetic
(PBPK), or binding parameter. Warfarin is PK-only. Neonatal
weights are simulated. These narrow contracts do not reproduce the source
population analyses or cover fitting for all 59 nlmixr2lib models.

A match requires all three:

- Every parameter difference <= `1e-6 + 0.001*abs(reference)`.
- Absolute normalized Gaussian negative log-likelihood (NLL) difference <= `1e-4`.
- Every fitted prediction difference <= `1e-6 + 0.001*abs(reference)`.

The native fit must also converge. The R reference runs nlmixr2est estimation
via first-order conditional estimation with interaction (FOCEI); scalar cases
without random effects use scalar optimization. Objective function values (OFV)
from different methods are not compared with normalized NLL. White blood cell
(WBC), PBPK and binding
references use a positive log-multiplier within the same physical bounds. The
stored final WBC requests use the best converged fit among runs from the source
start, seven log-spaced starts and a physical midpoint; the reference estimate is
never used as a native start. Replaying the final requests tests these recorded
contracts.

Each contract states its data exclusions: the initial cohort excludes
time-zero observations and linear ACOP rows at/after the time-216 reset;
phenobarbital and WBC exclude too-sparse subjects. WBC legacy infusion start/stop
rows are paired into amount/rate events. The neonatal contracts retain time zero.

All 1,906 native fits converged and matched the three criteria above.
Five datasets and their concrete limits are listed in
`REMAINING_FITTING_CONTRACTS.md`. Population fitting equivalence is outside this report.

## Failure handling and domain limits

Undefined expressions, missing dose history, poles and finite-time blow-up were
tested with the original requests and parameters unchanged. BDF
rejected the same 16 conditions in both the original and corrected comparisons. `NEGATIVE_CONDITIONS.md` lists the native domain/solver diagnostics and
actual R outcomes; `UPSTREAM_ISSUES.md` provides the algebra and history analysis.

Reference results are judged on mathematical validity, not only on a successful
exit. Finite values produced by the safeZero, safeLog and safePow guards are not
counted as solutions of an undefined expression. Non-finite or missing outputs and
solver failures are recorded. Rejected requests are not counted as successful
simulations. These checks cover the listed negative conditions, not every
possible invalid input.

## Reference methods and environments

`environment.json` records the QSPQuest compiler source, executable and DiffSol
backend hashes, and the reference tool versions. Replays record the hash of the
binary actually used; a binary for another platform has a different hash even when
built from the same source. The results apply to the recorded engine build.

The Python tools require NumPy, pandas, and python-libsbml. R references use rxode2 5.1.8,
nlmixr2lib 0.3.2, nlmixr2est 7.1.0, jsonlite, digest, and nlme. Configure
`R_LIBS_USER` if these are in a separate library. Additional engines need their
recorded environments: mrgsolve in R, OrdinaryDiffEq in Julia, or the appropriate
RoadRunner/COPASI/AMICI Python package. See `environment.json` for exact recorded
versions. Solvers in different engines need not be identical: rxode2 uses LSODA,
mrgsolve/COPASI use LSODA, RoadRunner uses CVODE BDF, and OrdinaryDiffEq uses
Rodas5P. BDF and LSODA are different algorithms.

## Supporting evidence

`results/summary.json` gives aggregate counts; CSV files give one row per
comparison; `.json.gz` files preserve full per-case metrics and identities.
Read a detailed report with Python's `gzip.open` and `json.load`. `manifest.json`
lists a checksum for every file. `scripts/verify.py` checks all checksums,
COMBINE manifests, fixture-member hashes, result counts, and engine identities.
The inputs are test fixtures, not a model catalog. Replays write predictions,
trajectories and logs only to the new output directory you choose.

## Optional rerun instructions

For a complete dependency-free setup and one selected comparison, start with
[First replay](docs/first-replay.md). The commands below run optional reference
engines or additional recorded tests.
Install the dependencies listed under reference methods and environments, then
set `QSPQUEST_EXECUTABLE` to the QSPQuest command-line interface (CLI).

```sh
python scripts/verify.py
export QSPQUEST_EXECUTABLE=/path/to/qspquest
python scripts/reproduce.py --suite omex-rxode2 \
  --cli "$QSPQUEST_EXECUTABLE" --output replay-omex
```

For independent engines, use `--suite mrgsolve`, `ordinarydiffeq`, `roadrunner`,
`copasi` or `amici`. A selected model can be run with:

```sh
python scripts/reproduce.py --suite mrgsolve \
  --model qspquest_diffdose_2026_idr_benchmark \
  --cli "$QSPQUEST_EXECUTABLE" --output replay-mrgsolve
```

The runner reruns the native BDF simulation, then generates the reference and
compares the two. Omit `--model` for all eight models. Additional source presets
use `--suite presets`; this compares new native trajectories against the retained
source references without regenerating the original source validators.

For upstream conditions, use `--suite nlmixr2-bdf`:

```sh
python scripts/reproduce.py --suite nlmixr2-bdf \
  --model PK_1cmt --case baseline \
  --cli "$QSPQUEST_EXECUTABLE" --output replay-nlmixr2
```

Omit `--model` and `--case` for the complete matrix. For the corrected-reference
comparison, follow `upstream-patches/rxode2/README.md`, put the patched library
first in `R_LIBS_USER`, and use `nlmixr2-patched-bdf`. The runner checks the analytic dose-history regressions
before the matrix comparison.

For fitting, `--model` selects the dataset and `--case` selects the subject:

```sh
python scripts/reproduce.py --suite fitting \
  --model Bolus_1CPT --case 1 \
  --cli "$QSPQUEST_EXECUTABLE" --output replay-fit
```

Omit both selectors for all 1,906 fits. Reruns write new trajectories, optimizer
output and logs to the chosen output directory.

## Engine and reference identities

`environment.json` identifies the QSPQuest build used for the native runs. Each
comparison records its input hashes and the QSPQuest binary actually run. Stored
reference trajectories are kept unchanged with their engine versions; a
comparison against a stored reference does not rerun that reference engine.
