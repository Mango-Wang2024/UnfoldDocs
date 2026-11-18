# UnfoldPenalizedSplines.jl

Additive penalized spline models for [Unfold.jl](https://github.com/unfoldtoolbox/Unfold.jl).

## Overview

UnfoldPenalizedSplines.jl is a companion package for modelling smooth, non-linear relationships between predictors and responses.

Each predictor has its own cubic spline. The package estimates how strongly each spline should be smoothed using restricted maximum likelihood (REML).

## Marginal effects: unpenalized and penalized splines

UnfoldSim generates the EEG recording from our **200 predictor pairs**. The predictors set the height of the P300 peak through our sine and cosine effects. The comparison and the demo use the same recording. Run the code from the root of the [package repository](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl).

```@raw html
<details>
<summary>Load packages and example data</summary>
```

Install the fitting and plotting dependencies once from the package repository root.

```sh
julia --project=. -e 'using Pkg; Pkg.instantiate()'
julia --project=assets -e 'using Pkg; Pkg.instantiate()'
```

Start Julia with `julia --project=.` and run the Julia blocks in order.

```julia
using Unfold, UnfoldPenalizedSplines, DataFrames, DelimitedFiles

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

Use `effects()` to predict the EEG response at 21 values of **x₁**, with **x₂ held at 0**.

```julia
grid = Dict(:x1 => collect(range(-2, 2; length=21)), :x2 => [0.0])
effects_unpenalized = effects(grid, unpenalized)
effects_penalized = effects(grid, penalized)
```

Pass those predictions to the Makie script included in the package repository. It writes `erp-effects.png` and `erp-effects.svg` to your current folder.

```julia
curves = vcat(effects_unpenalized, effects_penalized;
              source=:method => ["Unpenalized", "REML"])
columns = [:channel, :eventname, :time, :x1, :x2, :yhat, :method]
mktemp() do path, io
    writedlm(io, permutedims(string.(columns)), ',')
    writedlm(io, Matrix(curves[:, columns]), ',')
    close(io)
    run(`$(Base.julia_cmd()) --project=assets assets/plot_erp_effects.jl $path erp-effects.png`)
end
```

```@raw html
<figure style="margin: 1.5rem 0;">
  <img src="assets/erp-effects.svg?v=unfoldsim" alt="Marginal EEG responses for x1: unpenalized fitting on the left and REML-smoothed fitting on the right, with x2 held at zero." style="display: block; width: 100%; height: auto;">
</figure>
```

- **Left — unpenalized:** both smoothing strengths are zero, so the spline coefficients receive no bending penalty.
- **Right — penalized:** REML estimates one smoothing strength per predictor: **λ₁ ≈ 0.6430** and **λ₂ ≈ 69.82**. The basis size remains fixed at 8.

The penalty controls bending **across predictor values**. It does not impose a separate smoothness penalty along time. Although only x₁ varies in this plot, both predictors were used during fitting.

[Example code](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/examples/erp_effects.jl) · [Plot code](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/assets/plot_erp_effects.jl) · [Results and data](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/results/erp_effects/RESULTS.md)

## Adjust spline smoothing

Use the selector to compare four settings:

- **Unpenalized:** both λ values are zero and both sliders are locked.
- **x₁:** adjust λ₁; λ₂ stays fixed at its REML estimate, 69.82.
- **x₂:** adjust λ₂; λ₁ stays fixed at its REML estimate, 0.6430.
- **Both predictors:** adjust both smoothing strengths.

Each slider position shows a refitted model with the chosen λ values, keeping the same data and basis size. Larger λ means stronger smoothing across that predictor's values. The first panel varies x₁ with x₂ = 0; the second varies x₂ with x₁ = 0.

```@raw html
<iframe id="spline-smoothing-demo" src="assets/smoothing-demo.htm?v=unfoldsim" title="Adjust the smoothing strengths of the x1 and x2 splines" loading="lazy" style="display: block; width: 100%; height: 720px; border: 0; background: white; color-scheme: only light;"></iframe>
<script src="assets/embed-demo.js"></script>
```

Demo source on GitHub · [Refitting](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/examples/smoothing_fits.jl) · [Makie plots](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/assets/plot_smoothing_demo.jl) · [Slider controls](https://github.com/Mango-Wang2024/UnfoldDocs/blob/main/penalized-splines/assets/smoothing-demo.htm)

## Tutorial outline

### Data preparation

### Model specification and fitting

### Prediction and reporting

## Explore the package

- [Package repository](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl)
- [Package README](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/README.md)
- [Example with two predictors](https://github.com/Mango-Wang2024/UnfoldPenalizedSplines.jl/blob/main/examples/two_predictors.jl)
