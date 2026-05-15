https://dave356w.github.io/wheel-bias/
# wheel
Roulette wheel bias tracker — mobile PWA

## Theory

### Why arcs?

A mechanically biased wheel doesn't favour a single pocket — friction, wear, and fret deformation distribute the elevation across a physically contiguous arc of adjacent pockets. Detecting a single-pocket elevation at 38:1 against background noise would require thousands of spins. Detecting an arc of 5–9 pockets elevated together requires far fewer because the signal-to-noise ratio scales with arc size.

### Recency weighting

Raw spin counts treat a result from 300 spins ago the same as one from 5 spins ago. Mechanical wheel conditions drift. All hit-rate calculations use exponentially decaying weights:

```
weight(age) = 2^(-age / 75)
```

A spin 75 ago counts half as much as the most recent one. The effective sample size (`nEff`) is computed from the sum of weights squared over sum-squared, giving a statistically valid denominator for z-score calculations despite the non-uniform weighting.

### Circular window scan

The 38-pocket wheel is treated as a circular frequency string. The detector tests every contiguous window of size 3–9 at every starting position — 38 × 7 = **266 candidate windows** per evaluation. For each window:

1. Compute the recency-weighted observed hit rate `p_actual`
2. Compare to the fair-wheel expected rate `p_expected = size / 38`
3. Compute z-score using `nEff`:

```
SE  = sqrt(p_expected × (1 - p_expected) / nEff)
z   = (p_actual - p_expected) / SE
```

4. Score each window as `z × bias%`
5. Select the highest-scoring window as the **primary arc**

After the primary arc is found, its pockets are masked and the scan repeats on the unmasked remainder to find an independent **secondary arc**.

### False positive correction

Scanning 266 windows post-hoc inflates apparent signal — the best window out of 266 random trials will look elevated even on a perfectly fair wheel. Gates are calibrated empirically:

| Spin count | Min bias | Min z | Approx FP rate |
|-----------|---------|-------|----------------|
| < 75      | 22%     | 3.0   | ~3%            |
| 75–149    | 15%     | 3.0   | ~1%            |
| 150+      | 12%     | 3.0   | ~3.5%          |

The z ≥ 3.0 threshold is constant. A single-test z of 3.0 corresponds to p < 0.003, which stays below 5% false positives even after the scan's multiple-comparison inflation.

### Portfolio — straight arc cover

Once a qualifying arc is confirmed, the play is one straight-up chip on each arc pocket at 35:1. Every unit staked goes onto a pocket with a confirmed bias reading, undiluted by numbers outside the arc.

Pockets are ranked by recency-weighted heat and capped at 9 chips, then displayed in physical wheel order for easy placement.

The play only fires if:
- Observed weighted probability ≥ break-even hit rate × a sample-size multiplier
- Portfolio edge > 0: `E = (36 × p_arc_pockets / stake) − 1 > 0`

If neither condition is met the app reports the arc but shows no play — the signal is present but not yet strong enough to bet with positive expectation.

### Storage

Spin log persisted to `localStorage` key `wheelBias.spinLog.v9`. Each major schema version uses a new key to avoid loading stale data.
