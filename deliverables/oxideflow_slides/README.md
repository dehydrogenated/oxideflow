# OxideFlow — slide-ready content pack

Prepared for the existing `Final_Presentation.pdf` visual system: white field, black headline,
teal accent, concise left-hand message, and quantitative evidence on the right.

## Recommended continuation

Insert these results before the current blank **Key Takeaways** slide:

1. Results: Computational Wall Time
2. Gas References: Model Disagreement
3. Gas References: Static Reference Test
4. Results: Propane and Propene
5. Key Takeaways
6. Limitations
7. Future Outlook

---

## Slide — Results: Computational Wall Time

### Headline

**The same three adsorption relaxations span 9.7× in wall time.**

### Body copy

- Orb-v2 completed the controlled workload in **53 s**; UMA-M-oc20 required **517 s**.
- Median wall time was **176 s** across 14 models.
- Runtime reflects both cost per optimization step and the number of steps required.

### Takeaway line

**Wall time should be a first-class model-selection metric, alongside accuracy and convergence.**

### Figure

Use [`walltime_controlled.svg`](./walltime_controlled.svg) on the right-hand side.

### Footnote

Single Sockeye one-GPU job; sum of TiO2–CO*, TiO2–H*, and IrO2–CO* relaxations at
fmax = 0.02 eV Å−1. Shared bulk/slab relaxations and cached gas references are excluded.

### Exact chart data

| Model | Wall time (s) |
|---|---:|
| Orb-v2 | 53.337 |
| CHGNet-0.3.0 | 106.105 |
| MACE-mh1-omat | 123.324 |
| MACE-mh1-matpes | 131.343 |
| MACE-mh1-oc20 | 132.767 |
| UMA-omat | 157.785 |
| UMA-oc22 | 160.887 |
| SevenNet-omni-oc22 | 191.372 |
| SevenNet-omni-oc20 | 192.054 |
| SevenNet-omni-omat24 | 206.675 |
| SevenNet-omni-mpa | 216.807 |
| eSEN-30M-OAM | 339.589 |
| UMA-M-omat | 493.949 |
| UMA-M-oc20 | 517.030 |

---

## Slide — Gas References: Model Disagreement

### Headline

**Atomic O and H drive nearly all gas-reference disagreement.**

### Body copy

- Mean deviation from the 15-model median is **63.8% for O** and **62.4% for H**.
- Molecular references are much tighter: **OH 3.24%**, **H2 2.03%**, **H2O 1.94%**,
  and **CO 1.77%**.
- eSEN is the atomic outlier, but its H2, H2O, CO, and OH energies sit within **0.2%** of
  the median. The water-splitting cycle removes its unstable atomic terms.
- Among molecular references, MACE-mh1-matpes deviates most at **8.53%** on average,
  followed by UMA-oc22 at **6.52%**.

### Takeaway line

**Reaction-based references are more stable than isolated open-shell atoms.**

### Equations

O reference used for the corrected oxygen convention:

`O_ref,corr = E(H2O) − E(H2) + 2.51 eV`

OH water-splitting reference:

`OH_ref = E(H2O) − ½E(H2)`

### Figure

Use [`gas_reference_percent.svg`](./gas_reference_percent.svg).

### Footnote

Percentage = mean absolute deviation from the cross-model median divided by the absolute
median energy. It measures disagreement, not accuracy against a physical ground truth.
O2 gives 7.91% across only 5 models and remains a pilot result.

### Exact chart data

| Gas species | Models (n) | Mean absolute % deviation from median | Largest deviation |
|---|---:|---:|---|
| O | 15 | 63.84% | eSEN, 451.91% |
| H | 15 | 62.41% | eSEN, 332.67% |
| O2 — pilot | 5 | 7.91% | UMA-M-oc20, 19.19% |
| OH | 15 | 3.24% | MACE-mh1-matpes, 13.36% |
| H2 | 15 | 2.03% | UMA-oc22, 9.36% |
| H2O | 15 | 1.94% | UMA-oc22, 9.75% |
| CO | 15 | 1.77% | MACE-mh1-matpes, 11.73% |

---

## Slide — Gas References: Static Reference Test

### Headline

**One static reference does not improve the benchmarks consistently.**

### Body copy

- A neutral test replaces each model’s gas term with the **15-model median** for that species.
- Static references modestly improve H2O adsorption (**−6.2% MAE**) and OVFE (**−3.8%**).
- They slightly worsen the full O*/OH* sweep (**+3.5%**) and nearly double CO adsorption
  MAE (**+99.1%**).
- The result depends strongly on the checkpoint: the static CO value changes
  MACE-mh1-matpes from **0.054 to 1.740 eV MAE**.

### Takeaway line

**Use model-specific references for production. Use a static reference only as a sensitivity test.**

### Figure

Use [`gas_reference_sensitivity.svg`](./gas_reference_sensitivity.svg).

### Footnote

Static values are the median model references, not fitted literature values. Negative changes
mean lower MAE. Adsorption MAE remains in eV because several literature adsorption energies
approach or cross zero, which makes per-row percentage error unstable.

### Exact chart data

| Benchmark | Own reference MAE (eV) | Static median MAE (eV) | Change |
|---|---:|---:|---:|
| O_Ads_1 | 0.877 | 0.864 | −1.5% |
| O_OH_Trendline | 0.841 | 0.871 | +3.5% |
| CO_H_Ads, literature CO rows | 0.173 | 0.344 | +99.1% |
| H2O_ads, primary geometry | 0.509 | 0.477 | −6.2% |
| OVFE_Benchmark | 1.244 | 1.197 | −3.8% |

The full calculation and model-level tables are in
[`gas_reference_analysis.md`](./gas_reference_analysis.md).

---

## Slide — Results: Propane and Propene

### Headline

**The lowest nominal MAE did not converge—so there is no production winner yet.**

### Body copy

- Pilot scope: **3 models × 3 oxides × 2 adsorbates = 18 relaxations**.
- Only **5/18** met the force criterion; all five were Orb-v2 runs.
- Nominal MAE: **UMA-M-oc20 0.446 eV**, **eSEN 0.647 eV**, **Orb-v2 0.881 eV**.
- eSEN reproduced stronger Ti → Ru → Ir binding for both molecules, but none of its six
  relaxations force-converged.

### Takeaway line

**Treat this as a diagnostic pilot, not a validated production benchmark.**

### Figure

Use [`propane_propene_pilot.svg`](./propane_propene_pilot.svg).

### Footnote

One fixed orientation and one undercoordinated-metal site per oxide/molecule. DFT targets are
the closest-geometry configurations identified from the GAME-Net-Ox PBE+D3 dataset, not global
adsorption minima. Current metrics use the corrected literature values in the benchmark script;
three labels in `data/ClaudeArtifacts/propane_ads/pp.jsonl` are stale.

### Exact summary data

| Model | MAE (eV) | Mean signed error (eV) | Force-converged | Adsorbed by distance proxy | Extended |
|---|---:|---:|---:|---:|---:|
| UMA-M-oc20 | 0.446 | −0.104 | 0/6 | 2/6 | 2/6 |
| eSEN-30M-OAM | 0.647 | +0.647 | 0/6 | 3/6 | 3/6 |
| Orb-v2 | 0.881 | +0.881 | 5/6 | 1/6 | 0/6 |

Reference energies used for the current comparison:

| Oxide | Propane (eV) | Propene (eV) |
|---|---:|---:|
| TiO2(110) | −0.5013 | −0.9329 |
| RuO2(110) | −0.5394 | −1.3277 |
| IrO2(110) | −0.9043 | −2.1267 |

---

## Slide — Key Takeaways

### Headline

**Choosing an MLIP is a multi-objective decision.**

### Four short blocks

**Chemistry beats scale**  
Training chemistry and reference task matter more than parameter count.

**Metric changes the winner**  
Trend fidelity and absolute MAE answer different questions.

**No universal model**  
Nine distinct models lead at least one reported benchmark metric.

**Reliability is a gate**  
Convergence, state retention, and reference provenance come before ranking.

### Bottom line

**Select a model + settings + reference convention—not a model name alone.**

---

## Slide — Limitations

### Headline

**The benchmark is broad, but the references and state coverage are not yet uniform.**

### Reference comparability

- Literature targets mix PBE, PBE+U, RPBE, and PBE-D3.
- Magnetism and open-shell states are not represented consistently across universal MLIPs.
- Several tests are small: H has 1 target, CO has 2, and H2O covers 3 oxides.

### Workflow coverage

- The 33-member rutile family includes forced or metastable prototypes and seven
  low-convergence chemistries.
- Propane/propene currently samples one site and one orientation; only 5/18 runs converged.
- Only the CO/H timing subset is hardware-controlled; broader timing tables mix environments.

### Engineering note

The prior height/freeze/supercell sweeps retain an approximately 0.068 eV geometry offset from
the corrected slab construction and should be rerun before final settings are locked.

---

## Slide — Future Outlook

### Headline

**Turn the benchmark into a reproducible selection-and-validation loop.**

### Figure

Use [`future_outlook_roadmap.svg`](./future_outlook_roadmap.svg).

### Supporting copy

- Standardize end-to-end wall time on one GPU, including startup, bulk, slab, and gas costs.
- Tune fmax, frozen depth, vacuum, supercell, and standoff only for accuracy–speed finalists.
- Rerun propane/propene with larger budgets, multiple sites, and multiple orientations.
- Validate MLIP minima with targeted DFT single points or selective DFT re-relaxation.
- Extend from molecular adsorption to dissociation, H abstraction, and dehydrogenation pathways.

---

## Benchmark inventory — concise slide version

O_Ads_1 — Zhao & Kulik (2019)  
O_OH_Trendline — Comer et al. (2022)  
CO_H_Ads — Andriuc et al. (2025); Pope et al. (2025)  
H2O_ads — González et al. (2019)  
OVFE_Benchmark — Kowalski, Meyer & Marx (2009)  
Gas_Refs — OxideFlow internal gas-reference audit (2026); Kowalski, Meyer & Marx (2009)  
Config_Tuning — OxideFlow internal configuration-sensitivity study (2026)  
propane_ads — Van Hout et al. (2026); ioChem-BD dataset 10.19061/iochem-bd-1-396

`Gas_Refs` and `Config_Tuning` are explicitly labeled internal because they are not published
literature benchmarks. `Config_Tuning` is already populated locally with height, vacuum,
freezing, and supercell sweeps.

---

## Verified bibliography — use in speaker notes or reference appendix

1. Zhao, Q.; Kulik, H. J. “Stable Surfaces That Bind Too Tightly: Can Range-Separated Hybrids
   or DFT+U Improve Paradoxical Descriptions of Surface Chemistry?” *J. Phys. Chem. Lett.*
   **2019**, *10*(17), 5090–5098. <https://doi.org/10.1021/acs.jpclett.9b01650>

2. Comer, B. M.; Li, J.; Abild-Pedersen, F.; Bajdich, M.; Winther, K. T. “Unraveling
   Electronic Trends in O* and OH* Surface Adsorption in the MO2 Transition-Metal Oxide
   Series.” *J. Phys. Chem. C* **2022**, *126*(18), 7903–7909.
   <https://doi.org/10.1021/acs.jpcc.2c02381>

3. Andriuc, O.; Siron, M.; Persson, K. A. “Systematic Computational Study of Oxide
   Adsorption Properties for Applications in Photocatalytic CO2 Reduction.” *Surface Science*
   **2025**, *758*, 122745. <https://doi.org/10.1016/j.susc.2025.122745>

4. Pope, C.; Yun, J.; Reddy, R.; Jamir, J.; Kim, M.; Asthagiri, A.; Weaver, J. F. “CO
   Oxidation on IrO2(110) Surfaces.” *Surface Science* **2025**, *751*, 122619.
   <https://doi.org/10.1016/j.susc.2024.122619>

5. González, D.; Heras-Domingo, J.; Pantaleone, S.; Rimola, A.; Rodríguez-Santiago, L.;
   Solans-Monfort, X.; Sodupe, M. “Water Adsorption on MO2 (M = Ti, Ru, and Ir) Surfaces.
   Importance of Octahedral Distortion and Cooperative Effects.” *ACS Omega* **2019**, *4*(2),
   2989–2999. <https://doi.org/10.1021/acsomega.8b03350>

6. Kowalski, P. M.; Meyer, B.; Marx, D. “Composition, Structure, and Stability of the Rutile
   TiO2(110) Surface: Oxygen Depletion, Hydroxylation, Hydrogen Migration, and Water
   Adsorption.” *Phys. Rev. B* **2009**, *79*(11), 115410.
   <https://doi.org/10.1103/PhysRevB.79.115410>

7. Van Hout, T.; Loveday, O.; Morales-Vidal, J.; Morandi, S.; López, N. “Evaluating the
   Transfer Learning from Metals to Oxides with GAME-Net-Ox.” *Digital Discovery* **2026**,
   *5*, 407–414. <https://doi.org/10.1039/D5DD00331H>

8. Van Hout, T. et al. GAME-Net-Ox computational dataset, ioChem-BD.
   <https://doi.org/10.19061/iochem-bd-1-396>

### Citation-year notes

- The Pope DOI was registered under a 2024 manuscript identifier, but the journal issue is
  *Surface Science* volume 751 (January 2025); cite the article as 2025.
- The GAME-Net-Ox paper was first published online in December 2025 and appears in the 2026
  volume; cite the journal article as 2026.
