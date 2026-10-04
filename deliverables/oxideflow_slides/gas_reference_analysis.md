# Gas-reference analysis

Artifact reviewed: <https://claude.ai/code/artifact/4542ba2f-645b-4ac6-9062-cf133b84fef9>

Local numerical sources: `data/ClaudeArtifacts/Gas_Refs/`, the adsorption JSONL files under
`data/ClaudeArtifacts/`, and the OVFE data embedded in
`data/ClaudeArtifacts/OVFE_Benchmark/ovfe_benchmark_artifact.html`.

## Question 1: Which species disagree most across models?

For each species, the static comparison value is the median energy across the available
models. The normalized disagreement is:

`mean absolute % deviation = mean(|E_model − E_median| / |E_median|) × 100`

This quantity controls for molecular size, but it is not a physical accuracy error. Absolute
total-energy zeros differ between checkpoints and training Hamiltonians.

| Species | n | Median energy (eV) | Mean absolute % deviation | Median absolute % deviation | Worst model | Worst deviation |
|---|---:|---:|---:|---:|---|---:|
| O | 15 | −1.7515 | 63.84% | 38.97% | eSEN-30M-OAM | 451.91% |
| H | 15 | −0.9103 | 62.41% | 26.41% | eSEN-30M-OAM | 332.67% |
| O2, pilot | 5 | −9.3127 | 7.91% | — | UMA-M-oc20 | 19.19% |
| OH | 15 | −7.7780 | 3.24% | 1.72% | MACE-mh1-matpes | 13.36% |
| H2 | 15 | −6.7658 | 2.03% | 1.45% | UMA-oc22 | 9.36% |
| H2O | 15 | −14.1961 | 1.94% | 0.53% | UMA-oc22 | 9.75% |
| CO | 15 | −14.6940 | 1.77% | 1.04% | MACE-mh1-matpes | 11.73% |

Interpretation:

- Open-shell O and H are qualitatively less stable than molecular references.
- OH is the hardest complete molecular/radical reference by average normalized disagreement.
- Among the closed-shell H2, H2O, and CO set, H2 has the largest average disagreement.
- O2 currently has only five models. Its 7.91% result cannot support a 15-model ranking.

## Question 2: Which models struggle most?

The table below averages percentage deviation across H2, H2O, CO, and OH.

| Model | Mean molecular-reference deviation | Main contributors |
|---|---:|---|
| MACE-mh1-matpes | 8.53% | OH 13.36%, CO 11.73%, H2O 7.71% |
| UMA-oc22 | 6.52% | H2O 9.75%, H2 9.36%, OH 5.43% |
| UMA-oc20 | 3.49% | OH 9.73% |
| CHGNet-0.3.0 | 3.02% | OH 4.37%, H2O 4.32% |
| UMA-M-omat | 2.74% | H2 4.39%, H2O 3.63% |

Atomic-reference failures follow a different pattern:

| Model | Atomic outlier |
|---|---|
| eSEN-30M-OAM | O 451.91%; H 332.67% |
| UMA-M-omat | H 162.68% |
| UMA-oc22 | H 98.79% |
| UMA-oc20 | O 94.24% |
| MACE-mh1-matpes | O 91.78% |

eSEN provides the clearest reason to avoid isolated atomic references. Its O and H energies
are extreme outliers, while its H2, H2O, CO, and OH energies each sit within 0.2% of the
cross-model median. The water-splitting reference bypasses the unstable atomic predictions.

## Question 3: Is one static reference better?

The sensitivity test replaces each model-specific gas term with the cross-model median:

| Reference term | Static median (eV) |
|---|---:|
| O_ref, raw = E(H2O) − E(H2) | −7.4339 |
| O_ref, corrected | −4.9239 |
| OH_ref = E(H2O) − 1/2 E(H2) | −10.8167 |
| CO | −14.6940 |
| H2O | −14.1961 |
| 1/2 H2 | −3.3829 |

| Benchmark | Valid rows | Own-reference MAE | Static-reference MAE | Change | Rows improved |
|---|---:|---:|---:|---:|---:|
| O_Ads_1 | 31 | 0.877 eV | 0.864 eV | −1.5% | 13/31 |
| O_OH_Trendline | 1,822 | 0.841 eV | 0.871 eV | +3.5% | 741/1,822 |
| CO_H_Ads, literature CO rows | 28 | 0.173 eV | 0.344 eV | +99.1% | 11/28 |
| H2O_ads, primary geometry | 39 | 0.509 eV | 0.477 eV | −6.2% | 22/39 |
| OVFE_Benchmark | 30 | 1.244 eV | 1.197 eV | −3.8% | 17/30 |

The median and mean static references give the same qualitative result. A static value helps
some tasks through offset cancellation but harms others. It also redistributes error sharply:

- MACE-mh1-matpes CO MAE changes from 0.054 to 1.740 eV.
- UMA-oc22 O*/OH* MAE changes from 1.525 to 0.704 eV.
- CHGNet O*/OH* MAE changes from 0.525 to 0.958 eV.
- MACE-mh1-matpes O*/OH* MAE changes from 0.885 to 1.883 eV.

## Recommended interpretation

1. Keep model-specific gas references for production calculations.
2. Prefer reaction-based references over isolated O or H atoms.
3. Use a static median reference as an ablation test that exposes offset sensitivity.
4. Record the model checkpoint, gas geometry, relaxation settings, and correction convention
   with every cached reference.

The +2.51 eV oxygen correction changes every model by the same amount. It addresses the
water-formation convention but cannot reduce model-to-model dispersion by itself. OH has no
equivalent correction because E(H2O) − 1/2 E(H2) contains no O2 term.
