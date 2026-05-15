https://dave356w.github.io/wheel-bias/
# wheel-bias
Roulette wheel bias tracker — mobile PWA
Here's the strategy the code implements:

## Detection: Where is the wheel running hot?

The wheel's 38 pockets are split into **4 arcs** along physical wheel order. The split point for each half of the wheel is found by a weighted-density median — whichever pocket is the tipping point where cumulative hits reach 50% of that half's total.

All hit counting is **recency-weighted** with a 75-spin half-life (`spinWeight = 2^(-age/75)`), so recent spins count more than old ones. Each arc's observed rate is compared to its fair-wheel expected rate, and a z-score is computed using the effective sample size (accounting for the exponential weighting). An arc is declared **hot** only if it clears both a minimum bias threshold (5–8% depending on sample size) and a minimum z (1.10–1.55).

## Portfolio: What to bet?

Qualifying hot arcs feed a 3-layer portfolio builder:

**Layer 1 — Inside Primaries** (up to 2 bets): Streets, six-lines, or corners whose pockets overlap heavily with the hot arc. Each candidate must pass:
- Arc density gate (≥22% of the arc's pockets sit inside this bet for streets/corners, ≥30% for six-lines)
- Precision gate (≥50% of the bet's own pockets are in the arc)
- Hit-rate gate (observed weighted probability ≥ break-even × a sample-size multiplier)
- Minimum 2-pocket overlap with the arc

The second inside primary is only added if it covers ≥2 arc pockets not already in the first.

**Layer 2 — Straight gap-fill**: Any hot-arc pockets not already covered by the inside primaries get straight-up chips (up to 9), sorted by recency-weighted heat. Only added if the group's observed probability beats break-even.

**Layer 3 — Outside volume**: A dozen, column, or half-bet whose arc-slice hit rate (weighted probability of just the arc portion landing in this outside zone) beats the sample-adjusted minimum. Requires ≥33% arc density and ≥4 pocket overlap. Only one outside bet is added — the highest scorer.

## Scoring

Each zone is scored as:

```
arcDensity × arc.bias × arc.z × (payout/35 + 0.5) + pocketHeat × 0.001
```

Higher payout bets get a multiplier bonus (the `payout/35 + 0.5` term). The `pocketHeat` tiebreaker favours zones whose specific arc pockets have been hit recently, not just the arc overall. The final score is then edge-adjusted: `score × (1 + max(edge, 0))`.

The result is a ranked, non-redundant bet ticket list — each subsequent bet tracked for how many new arc pockets it adds beyond what's already covered.
