# rxode2 administered-amount history patch

A UniBio Intelligence fix for administered-amount history in rxode2 5.1.8,
source commit `2f421fb1341a2b8954a7f86cf6aceb3938a28bef`.

`handleTlastInline` reconstructs the raw administered amount for a fixed
infusion. Two later handlers overwrite it with the internal rate. Removing
those assignments preserves the raw dose while leaving infusion input rates,
bioavailability and duration calculations intact.

The unmodified library fails seven of nine analytic history cases. Rebuilding
the pinned source with the patch passes all nine, with maximum absolute error
below 2e-13. All 404 valid conditions in the 59-model corpus match QSPQuest
with that rebuilt reference. Original source equations, doses and parameters
are unchanged; undefined source requests remain rejected.

```sh
git clone https://github.com/nlmixr2/rxode2.git rxode2-source
cd rxode2-source
git checkout 2f421fb1341a2b8954a7f86cf6aceb3938a28bef
patch -p1 < ../upstream-patches/rxode2/infusion-history.patch
mkdir -p ../patched-R-library
R CMD INSTALL -l ../patched-R-library .
cd ..
R_LIBS_USER="$PWD/patched-R-library" Rscript \
  upstream-patches/rxode2/validate_administered_history.R history-replay
```

Install rxode2's R dependencies first. In `R_LIBS_USER`, list the patched
directory first, followed by your normal R libraries.
`original/` and `patched/` hold summaries of the analytic tests; the script regenerates their trajectories. The full 420-condition results are in results/.
