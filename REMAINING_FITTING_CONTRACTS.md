# Example-data fitting contract coverage

Twenty-one of the 26 pinned nlmixr2data datasets now have matched individual Gaussian contracts: 1,906 contracts and 1,906 converged native BDF fits against actual nlmixr2est estimation. [Methods](METHODS.md) records equations, estimands, fixed nuisance parameters, observations, bounds and numerical limits.

The six additional completed datasets are Wang2007 (10 nonlinear response contracts), neonatal_wt (189 simulated weight contracts), warfarin (32 PK-only contracts), wbcSim (44 myelosuppression contracts), mavoglurant (120 PBPK contracts), and nimoData (12 nonlinear binding contracts). Their narrow individual fits do not reproduce the source population analyses. Neonatal weights here are simulated, not the unavailable real patient dataset.

| Dataset outside the fitting claim | Reason and required work |
| --- | --- |
| pump | Failure counts and operating durations require a count/event-rate likelihood. The native Gaussian concentration likelihood is unsuitable; a discrete-likelihood fitting capability is needed. |
| rats | The source problem uses correlated binomial/probit responses. A matched binomial/probit and correlation contract is required; Gaussian fitting would change the problem. |
| metabolite | The dataset has parent and metabolite observations, but the documentation found does not fully specify doses, initial states, conversion scale and how to fit both outputs. That specification is needed before fitting. |
| invgaussian | The source equation and parameters were found. Its inverse-Gaussian absorption input has a conditional zero-time extension, which QSPQuest's current check on time-dependent conditionals rejects. This is a QSPQuest limitation, not missing data or evidence of a flaw in the source mathematics. QSPQuest needs validated support for this conditional input before this dataset can be fitted. |
| nmtest | This is an event/simulation reference corpus, not ordinary observed clinical fitting data. It is usable for simulation semantics. A separately labelled synthetic recovery contract is possible but is outside the present fitting cohort. |

Source archive hashes and the subject-level contracts used are in inputs/individual-fitting.omex; the full source data is not included in this repository. Available source documentation or data does not mean the patient data can be redistributed, or that fitting matches the source for all 59 nlmixr2lib models. No unsuitable problem is counted as passing.
