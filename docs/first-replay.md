# Verify the archive and replay one preset

This tutorial checks the validation files, runs one source preset with your
QSPQuest command-line interface (CLI), and compares its full trajectory against the retained reference.
It requires Python 3.11 or newer and a built QSPQuest CLI. No additional Python
packages or reference-engine installation are needed for this preset replay.

Run these commands from the validation repository root. If you need to build the
CLI, follow the [engine installation guide](https://github.com/unibio-intelligence/qspquest/blob/main/docs/installation.md)
first. The archived results identify their exact engine in
[environment.json](../environment.json). A replay with another engine build
produces a new comparison.

## Set up and verify

Create the environment and output beside the checkout so archive verification
continues to cover only the release files:

```sh
python3 -m venv ../qspquest-validation-env
. ../qspquest-validation-env/bin/activate
python --version
python -B scripts/verify.py
export QSPQUEST_EXECUTABLE="$(cd ../qspquest && pwd)/target/release/qspquest"
test -x "$QSPQUEST_EXECUTABLE"
```

The executable assignment assumes a sibling checkout named `qspquest`. If yours
is elsewhere, set `QSPQUEST_EXECUTABLE` to its absolute executable path instead.
Verification reports 1,714 archived simulation comparisons and 1,906 fitting
comparisons with consistent recorded QSPQuest identities. This verifies
the archived evidence; it does not rerun those scientific comparisons.

## Run one comparison

```sh
python -B scripts/reproduce.py --suite presets \
  --model qspquest-ortiz-2024-ravulizumab-nmosd-pkpd \
  --case week50-40-60-pk-free-c5-ode \
  --cli "$QSPQUEST_EXECUTABLE" --output ../qspquest-preset-replay
```

A successful run prints `{"matched": 1}`. Inspect its saved summary and the actual
binary identity:

```sh
python -m json.tool ../qspquest-preset-replay/replay-engine.json
python -m json.tool ../qspquest-preset-replay/summary.json
```

`summary.json` contains one case with three state/output comparison columns
across 353 observation times. `max_scaled_error <= 1` means the values satisfy
`abs(actual-reference) <= 1e-8 + 1e-4*abs(reference)` at the exact observation
times. Missing columns, nonfinite values and incomplete trajectories fail the
comparison. `replay-engine.json` identifies the executable you actually ran;
`same_binary: false` is expected for another build or platform. The complete
native result in the replay workspace records the actual compiler source hash.
Run the following command to read it.

```sh
python - <<'PY'
import json
from pathlib import Path

workspace = Path("../qspquest-preset-replay/workspace")
case = workspace / "additional-model-simulations-20261005" / "qspquest-ortiz-2024-ravulizumab-nmosd-pkpd"
result = case / "week50-40-60-pk-free-c5-ode/result.json"
print(json.loads(result.read_text())["identity"]["compiler_source_sha256"])
PY
```

Compare that hash with `engine.compiler_source_sha256` in `environment.json`.
Matching source hashes can have different executable hashes across platforms.
A different source hash identifies a replay with different engine sources.

The preset uses its archived native model and request. It checks deterministic
simulation against retained source trajectories, not physical-unit qualification,
fitting, population inference or a complete source-paper reproduction.

## Common problems and next steps

- If the output directory already exists, choose a new `--output` path. Results
  are never silently overwritten.
- If the executable cannot be found, check `QSPQUEST_EXECUTABLE` with `test -x`
  and use an absolute path.
- A checksum failure means the snapshot has changed or is incomplete. Restore
  the release files before treating its archived evidence as intact.
- A numerical failure on another engine build is a new result to investigate;
  do not change tolerances to make it pass. Keep the output directory and logs.

[Methods](../METHODS.md#optional-rerun-instructions) documents the optional R,
Julia and other independent-reference workflows and their recorded dependencies.
