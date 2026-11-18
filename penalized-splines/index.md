# UnfoldPenalizedSplines.jl

Additive penalized spline models for [Unfold.jl](https://github.com/unfoldtoolbox/Unfold.jl).

## Overview

UnfoldPenalizedSplines.jl is a companion package for modelling smooth, non-linear relationships between predictors and responses.

Each predictor has its own cubic spline. The package estimates how strongly each spline should be smoothed using restricted maximum likelihood (REML).

## Marginal effects: unpenalized and penalized splines

We use a simulated EEG recording with **200 events**. Run the example from the root of the [package repository](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl).

```@raw html
<details>
<summary>Load packages and example data</summary>
```

```julia
using Unfold, UnfoldPenalizedSplines, DataFrames, DelimitedFiles
using CairoMakie, UnfoldMakie

folder = joinpath("results", "erp_effects")
event_rows, _ = readdlm(joinpath(folder, "events.csv"), ',', Float64; header=true)
recording, _ = readdlm(joinpath(folder, "recording.csv"), ',', Float64; header=true)
events = DataFrame(x1=event_rows[:, 1], x2=event_rows[:, 2],
                   latency=Int.(event_rows[:, 3]))
data = reshape(recording[:, 3], 1, :)  # one EEG channel
```

```@raw html
</details>
```

Fit the same spline model twice, first without a penalty and then with REML smoothing.

```julia
formula = @formula(0 ~ 1 + pspl(x1, 8) + pspl(x2, 8))
basis = firbasis(τ=(-0.1, 0.5), sfreq=100; interpolate=false)

unpenalized = fit(UnfoldModel, formula, events, data, basis;
                  solver=(X, y) -> Unfold.solver_default(X.X, y))
penalized = fit(UnfoldModel, formula, events, data, basis)
```

Use `effects()` to predict the EEG response at 21 values of **x₁**, with **x₂ held at 0**, and plot one panel per fit.

```julia
grid = Dict(:x1 => -2:0.2:2, :x2 => [0.0])
curves = vcat(effects(grid, unpenalized), effects(grid, penalized);
              source=:model => ["(a) Unpenalized spline model", "(b) Penalized spline model"])

plot_erp(curves; mapping=(; color=:x1, group=:x1 => nonnumeric, col=:model))
```

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

Each slider position shows a refitted model with the chosen λ values, keeping the same data and basis size. Larger λ means stronger smoothing across that predictor's values. The first panel varies x₁ with x₂ = 0; the second varies x₂ with x₁ = 0. The demo switches between figures rendered in advance with [CairoMakie](assets/plot-smoothing-demo.jl).

```@raw html
<iframe id="spline-smoothing-demo" src="assets/smoothing-demo.htm?v=makie" title="Adjust the smoothing strengths of the x1 and x2 splines" loading="lazy" style="display: block; width: 100%; height: 720px; border: 0; background: white; color-scheme: only light;"></iframe>
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
