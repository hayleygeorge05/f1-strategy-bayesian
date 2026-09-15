# Was Piastri's 2026 Dutch GP Strategy Actually a Mistake?

**Question:** Was McLaren's Oscar Piastri's four-stop strategy at the 2026 Dutch GP really a strategic mistake - or, considering how their tyres degraded during that race, could it have been almost optimal?

**Answer:** His stop count was very likely correct (a simpler 2-stop option would have been better in only ~4% of simulated outcomes, a finding that remained robust over a wide range of pit-loss assumptions). The more interesting point is the reason: Piastri's hard-tyre degradation rate during that race was ~55% greater than that of his own teammate, all in the same car — this being a genuine, driver-specific anomaly rather than one caused by the car or the track as a whole. A further analysis indicates that an additional adjustment of the compound mix (involving running more soft and less hard tyres over the same four stops) might have made a difference, although this result is less statistically significant. 

---

## Method

1. A hierarchical Bayesian model (using PyMC) was constructed for tyre degradation, with the model being fitted          separately for each type of tyre (hard, soft, and medium), using real lap-by-lap data from the 2026 Dutch GP taken    from the entire field. The lap times were modelled in terms of an individual driver intercept and a degradation       slope, with these values being partially pooled towards the overall average for the population - this ensures that    drivers with limited data on a compound recieve a reasonable estimate that reflects the appropriate level of          uncertainty rather than one that is overconfident. The analysis made use of a non-centred parametrisation in order    to avoid the divergent-sampling problems that are typical of this kind of model structure.
2. The risk associated with the safety car - a Bayesian Poisson model based on the actual number of incidents            (involving safety car, virtual safety car, or red flag) over the 11 races prior to the Dutch GP - deliberately         omits the later races, the model thus only incorporating information that would have been available at the time       the strategy decision was made.
3. Instead of simulating the timing during a safety car period, the actual laps from the real 2026 Dutch GP - which      had already taken place - were used directly. This lets the analysis credit (or penalise) each proposed strategy      fairly, according to what actually happened, not on the basis of some general probability. 
4. A comparison based on Monte Carlo methods, involving 5,000 simulated outcomes in each case, with degredation rates    from each model's posterior distribution, in order to compare Piastri's real strategy against realistic               alternatives.

## Data

Real 2026 F1 timing data via [FastF1](https://github.com/theOehrly/Fast-F1): full-field race data from the Dutch GP (for degradation models) and race control/track status data from the 11 preceding 2026 races (for the safety
car rate model).

## Results

**Tyre degradation rates (seconds per lap of tyre age):**

| | Population average | Piastri | Norris |
|---|---|---|---|
| Hard | 0.0266 [0.012, 0.043] | **0.0438** [0.012, 0.08] | 0.0286 [0.0007, 0.06] |
| Soft | -0.0053 [-0.038, 0.03] (flat) | -0.005 [-0.068, 0.061] (flat) | — |
| Medium | 0.058 [0.033, 0.085] | — (only 1 lap run) | — |

Piastri's hard-tyre degradation sits significantly above both the field
average and his own teammate's, in identical machinery — the central
finding this project is built around.

For the 2026 season, before the Dutch GP, the safety car rate was 1.83 incidents per race
[1.3, 2.5], which corresponds to a probability of ~0.025 of an incident occurring on each lap during a 72-lap race.

Real Dutch GP incident laps: 2 (mandatory red flag), 55-57 (safety
car/VSC), 70 (brief VSC).

**Strategy comparison — actual (4 stops) vs a simpler 2-stop alternative:**

| | Actual (4 stops) | Simpler 2-stop |
|---|---|---|
| Mean simulated cost | **37.80s** | 55.31s |
| P(alternative better) | — | 4.26% (3.7–4.5% across pit-loss 15-25s) |

![Actual vs simpler 2-stop alternative](strategy_comparison.png)

**Compound-mix comparison — actual (2 hard + 2 soft) vs a soft-heavy
alternative (same 4 stops, mostly soft):**

| | Actual | Soft-heavy |
|---|---|---|
| Mean simulated cost | 37.57s | **31.80s** |
| P(soft-heavy better) | 63.92% |

![Actual vs soft-heavy compound mix](compound_mix_comparison.png)

## Verdict

The idea of the 'strange strategy' framing falls apart under analysis. In view of how Piatri's particular tyres performed in that race, dividing the hard tyre use into shorter periods - which is exactly what he did - was almost ideal, and a traditional two-stop approach very likely would have cost him more time rather than less. The actual, most interesting point is to be found not in the strategy decision itself but in the race conditions: Piastri's hard tyres were degrading much more quickly than those of his teammate's, and it must be understood that no change in strategy can cure a tyre problem - it can only make the situation better or worse. There is also a secondary and less certain indication that using the soft tyres more heavily during those four stops could have provided some additional benefit. 

## Limitations

- There is causal ambiguity; although the model detects a genuine and statistically significant difference in the       hard tyre degradation between Piastri and Norris, it is unable to tell which of the possible explanations - driving   style, tyre management, track position, or dirty air caused by driving behind another car - is the correct one. 
- There is very little data available regarding Piastri's performance with medium tyres - in fact, he only completed    one lap on them (having been forced to stop under a red flag), so his individual rate of medium tyre degradation is   effectively unknown; in this case the population estimate was used, and each instance of this was explicitly noted.
- The linear degradation model without a wear ceiling was designed so that the candidate strategies were within the     stint lengths actually achieved in that race, in order to prevent the model from extrapolating beyond what would be   a physically unrealistic tyre life.
- This race did not show any credible sign of soft tyre degradation when looking at the entire population, a result     which is rather unexpected; possible partial explanations might be the fuel burn-off and the limited amount of soft   tyre data, although these are not confirmed. 
- The pit-loss time was estimated (~15-18s, on the basis of the specific pit-lane speed limit at Zandvoort), not measured directly; the stop-count result was checked to see how robust it was over a 15-25s range and proved to be valid throughout - the compound-mix finding was not retested against this same range. 

## Repository contents

- `strategy_sim.ipynb` — full notebook: data pipeline, three hierarchical degradation models (hard, soft, medium),      safety car rate model, and all Monte Carlo comparisons.
- `strategy_comparison.png` — actual vs simpler-alternative outcome distributions.
- `compound_mix_comparison.png` — actual vs soft-heavy compound mix outcome distributions.

## Possible extensions

- Extend the driver-level comparison to the full field, not just Piastri vs Norris, to see how unusual his              degradation gap really was.
- Investigate potential causes of the degradation gap directly (following distance/dirty air from telemetry, fuel-      corrected lap times).
- Re-run the compound-mix comparison across the same pit-loss sensitivity range used for the stop-count finding.
