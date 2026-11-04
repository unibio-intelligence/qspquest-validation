# Negative conditions

This table lists all 16 rejected requests with the independent R result for each.
A successful R exit does not mean the outputs are finite. rxode2's safeZero,
safeLog and safePow guards were enabled and are recorded for each run.

| Source model | Condition | QSPQuest diagnostic | R result |
| --- | --- | --- | --- |
| Cirincione_2017_exenatide | no_dose | domain | contains nonfinite/missing outputs |
| Xie_2019_agomelatine | no_dose | domain | finite guarded outputs |
| Xie_2019_agomelatine | retained_nlmixr-v032-agomelatine-baseline-pilot-v1 | domain | finite guarded outputs |
| indirect_1cpt_inhi_kin_r0rmaxcrmax | infusion | domain | finite guarded outputs |
| indirect_1cpt_inhi_kin_r0rmaxcrmax | no_dose | domain | finite guarded outputs |
| indirect_1cpt_inhi_kin_r0rmaxcrmax | retained_nlmixr-v032-numeric-pilot-v5 | domain | finite guarded outputs |
| indirect_1cpt_inhi_kout_r0rmaxcrmax | infusion | domain | finite guarded outputs |
| indirect_1cpt_inhi_kout_r0rmaxcrmax | no_dose | domain | finite guarded outputs |
| indirect_1cpt_inhi_kout_r0rmaxcrmax | retained_nlmixr-v032-numeric-pilot-v5 | domain | finite guarded outputs |
| indirect_prec_1cpt_inhi_r0rmaxcrmax | infusion | domain | finite guarded outputs |
| indirect_prec_1cpt_inhi_r0rmaxcrmax | no_dose | domain | finite guarded outputs |
| indirect_prec_1cpt_inhi_r0rmaxcrmax | retained_nlmixr-v032-required-inputs-pilot-v1 | domain | finite guarded outputs |
| indirect_prec_1cpt_stim_r0rmaxcrmax | double_dose | solver | solver failure |
| indirect_prec_1cpt_stim_r0rmaxcrmax | repeated_dose | solver | solver failure |
| tgi_no_sat_powerLaw | retained_nlmixr-v032-numeric-pilot-v5 | solver | finite guarded outputs |
| tgi_sat_genVonBertalanffy | retained_nlmixr-v032-numeric-pilot-v5 | solver | contains nonfinite/missing outputs |

Where the solution blows up in finite time, R may return an enormous but finite
guarded value; see `UPSTREAM_ISSUES.md`. Domain rejections are neither numerical
mismatches nor passing simulations. Parameters and dose events were not changed.

Corrected or synthetic variants of these conditions are not included here and are not counted as matches.
