# UnfoldPenalizedSplines.jl

Additive penalized spline models for [Unfold.jl](https://github.com/unfoldtoolbox/Unfold.jl).

## Overview

UnfoldPenalizedSplines.jl is a companion package for modelling smooth, non-linear relationships between predictors and responses.

Each predictor has its own cubic spline. The package estimates how strongly each spline should be smoothed using restricted maximum likelihood (REML).

## Marginal effects: unpenalized and penalized splines

### Load the example data

Run this example from the root of the [package repository](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl), with CairoMakie and UnfoldMakie available in your Julia environment. It uses the saved simulated EEG recording and its table of **200 events**.

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

### Fit both models

Both fits use the same formula and **8 basis functions per predictor**. The first fits the spline coefficients without a penalty. The second estimates one smoothing strength per predictor using REML.

```julia
formula = @formula(0 ~ 1 + pspl(x1, 8) + pspl(x2, 8))
basis = firbasis(τ=(-0.1, 0.5), sfreq=100; interpolate=false)

unpenalized = fit(UnfoldModel, formula, events, data, basis;
                  solver=(X, y) -> Unfold.solver_default(X.X, y))
penalized = fit(UnfoldModel, formula, events, data, basis)
```

### Predict and plot

`effects()` predicts the EEG response over time at each of 21 values of **x₁**, holding **x₂ at 0**. Each color corresponds to one value of x₁.

```julia
grid = Dict(:x1 => collect(range(-2, 2; length=21)), :x2 => [0.0])
left = effects(grid, unpenalized)
right = effects(grid, penalized)

fig = Figure(size=(900, 390))
mapping = (; x=:time, y=:yhat, color=:x1, group=:x1 => nonnumeric)
for (panel, curves, title) in [(1, left, "Unpenalized spline model"),
                               (2, right, "Penalized spline model")]
    plot_erp!(fig[1, panel], curves; mapping,
        layout=(; show_legend=false), visual=(; colormap=:roma),
        axis=(; title, xlabel="Time [s]", ylabel="Voltage [µV]",
                xticks=[0.0, 0.2, 0.4], yticks=[-3.0, 0.0, 3.0, 6.0, 9.0],
                limits=(-0.1, 0.5, -6.5, 10.5)))
end
Colorbar(fig[1, 3]; colormap=:roma, limits=(-2, 2), ticks=[-2, 0, 2],
         width=12, topspinecolor=:transparent, ticksvisible=false)
Label(fig[1, 3, Top()], "x₁")
fig
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
