# UnfoldPenalizedSplines.jl

Additive penalized spline models for [Unfold.jl](https://github.com/unfoldtoolbox/Unfold.jl).

## Overview

UnfoldPenalizedSplines.jl is a companion package for modelling smooth, non-linear relationships between predictors and responses.

Each predictor has its own cubic spline. The package estimates how strongly each spline should be smoothed using restricted maximum likelihood (REML).

## Marginal effects: unpenalized and penalized splines

This example fits simulated continuous EEG from **200 events**, using **8 basis functions per predictor**. Both panels show predictions from the same data and model structure.

Each colored curve is the predicted EEG response at one value of **x₁**, from −2 to 2, while **x₂ is held at 0**. The colors identify predictor values; the curves are complete predicted responses, rather than individual spline basis functions.

```@raw html
<figure style="margin: 1.5rem 0;">
  <img src="assets/erp-effects.svg?v=aeb9b24" alt="Marginal EEG responses for x1: unpenalized fitting on the left and REML-smoothed fitting on the right, with x2 held at zero." style="display: block; width: 100%; height: auto;">
</figure>
```

- **Left — unpenalized:** both smoothing strengths are zero, so the spline coefficients receive no bending penalty.
- **Right — penalized:** REML estimates one smoothing strength per predictor: **λ₁ ≈ 0.7233** and **λ₂ ≈ 93.34**. The basis size remains fixed at 8.

The penalty controls bending **across predictor values**. It does not impose a separate smoothness penalty along time. Although only x₁ varies in this plot, both predictors were used during fitting.

[Example code](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/examples/erp_effects.jl) · [Plot code](assets/plot-erp-effects.jl) · [Results and data](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/results/erp_effects/RESULTS.md)

## Adjust spline smoothing

Use the selector to compare four settings:

- **Unpenalized:** both λ values are zero and both sliders are locked.
- **x₁:** adjust λ₁; λ₂ stays fixed at its REML estimate, 93.34.
- **x₂:** adjust λ₂; λ₁ stays fixed at its REML estimate, 0.7233.
- **Both predictors:** adjust both smoothing strengths.

Each slider position shows a refitted model with the chosen λ values, keeping the same data and basis size. Larger λ means stronger smoothing across that predictor's values. The first panel varies x₁ with x₂ = 0; the second varies x₂ with x₁ = 0.

```@raw html
<iframe id="spline-smoothing-demo" src="assets/smoothing-demo.htm" title="Adjust the smoothing strengths of the x1 and x2 splines" loading="lazy" style="display: block; width: 100%; height: 720px; border: 0; background: white; color-scheme: only light;"></iframe>
<script src="assets/embed-demo.js"></script>
```

## Tutorial outline

### Data preparation

### Model specification and fitting

### Prediction and reporting

## Explore the package

- [Package repository](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl)
- [Package README](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/README.md)
- [Example with two predictors](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/examples/two_predictors.jl)
