# Contributing to QSPQuest validation

Contributions are welcome: reproduction reports, problems with a comparison,
new reference engines or cases, and documentation fixes.

## How to contribute

1. **Open an issue first.** For a problem, give the suite, model and case, the
   command you ran, your QSPQuest version and the output. To propose a
   comparison, describe the reference engine or dataset, its version and its
   license. Except for typo fixes, wait for a maintainer to agree on the
   approach before you open a pull request.
2. **Open a focused pull request.** Make one change per pull request, link its
   issue (`Closes #123`) and run `python3 -B scripts/verify.py` and
   `python3 -B -m unittest discover -s scripts -p 'test_*.py'` first. Never
   loosen a tolerance to make a comparison pass; a new failure is a result to
   report.
3. **Review and release.** Maintainers re-run the affected comparisons and
   regenerate the result files and `manifest.json`. Accepted changes are
   published with the next recorded evidence, and you are credited.

Validation contributions are licensed under [GPL-3.0-only](LICENSE); component
notices remain applicable. Do not submit unlicensed data or personal data.
Report security issues privately to
[helpdesk@unibiointelligence.com](mailto:helpdesk@unibiointelligence.com), not in
public issues.
