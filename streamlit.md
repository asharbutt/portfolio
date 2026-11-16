---
layout: page
redirect_from: /projects.html
title: Streamlit
---

Interactive dashboards built on the pricing code. The code for each one is in the [streamlitprojects](https://github.com/asharbutt/streamlitprojects) repo.

## Foundations

Pricing fundamentals, Greeks, basic numerical methods, and curve construction.

### Black-Scholes Pricer & Greeks Dashboard

Closed-form Black-Scholes implementation for European calls and puts, computing all first- and second-order Greeks. Streamlit dashboard for interactive price surfaces and Greek sensitivity visualisation.

[Code on GitHub](https://github.com/asharbutt/streamlitprojects/blob/main/utils/bs.py) | [Open dashboard](https://kmy9xfhrnggijylqib5zoi.streamlit.app/Black_Scholes)

### Monte Carlo Visualisation and Pricer

Monte Carlo visualisation dashboard with a pricer, with options for different asset SDEs and variance reduction techniques.

[Code on GitHub](https://github.com/asharbutt/streamlitprojects/blob/main/utils/vanilla_mc_sim.py) | [Open dashboard](https://kmy9xfhrnggijylqib5zoi.streamlit.app/Monte_Carlo_Simulator)

### Binomial Tree Pricer (CRR Model)

Cox-Ross-Rubinstein binomial tree supporting European and American options. Visualises tree structure, convergence to Black-Scholes as step count increases, and early exercise boundaries for American puts.

[Code on GitHub](https://github.com/asharbutt/streamlitprojects/blob/main/utils/binomial.py) | [Open dashboard](https://kmy9xfhrnggijylqib5zoi.streamlit.app/Binomial_Tree)

### Implied Volatility Solver

Root-finding engine (Newton-Raphson and Brent's method) to extract implied volatility from market prices. Plots volatility smile and surface across strikes and maturities using real or synthetic data.

## Intermediate

Monte Carlo methods, exotic payoffs, PDE solvers, and richer volatility models.

### Monte Carlo European & Exotic Option Pricer

Monte Carlo engine with variance reduction (antithetic variates, control variates). Prices Europeans, Asians (arithmetic & geometric), barriers, and lookbacks. Compares convergence rates across variance reduction techniques.

### Implied Probability Density Extractor

Applies the Breeden-Litzenberger result: numerically differentiates call prices twice with respect to strike on a fine grid. Plots the risk-neutral density and compares to lognormal, highlighting skew and kurtosis.

### Local Volatility Surface (Dupire)

Implements Dupire's formula to extract the local volatility surface from market implied vols. 3D surface visualisation and comparison of local vol prices to market prices for exotic products.

### Finite Difference PDE Solver for Options

Solves the Black-Scholes PDE via explicit, implicit, and Crank-Nicolson schemes. Handles American options with penalty and PSOR methods. Visualises grid evolution and convergence/stability trade-offs.

### Heston Stochastic Volatility Model

Semi-analytical pricing (characteristic function + FFT) and Monte Carlo for the Heston model. Calibrates to a market vol surface and visualises fit quality, implied vol dynamics, and vol-of-vol sensitivity.

### Delta Hedging Simulator

Simulates daily delta-hedging of a short option position under Black-Scholes and real-world dynamics (jumps, stochastic vol). Shows P&L distributions, hedging error vs rebalancing frequency, and model misspecification impact.

## Advanced

Calibration, multi-factor models, XVA, and production-grade computational methods.

### SABR Model Calibration & Smile Dynamics

Implements the SABR approximation (Hagan et al.) and calibrates to swaption or equity vol surfaces. Visualises backbone dynamics, smile evolution with expiry, and comparison of SABR to market quotes.

### Longstaff-Schwartz (LSM) for American/Bermudan Options

Least-squares Monte Carlo with polynomial and Laguerre basis functions. Prices Bermudans on single- and multi-asset underlyings. Analyses exercise boundary convergence and basis function sensitivity.

### Multi-Asset Basket & Rainbow Option Pricer

Monte Carlo with correlated GBM paths (Cholesky decomposition). Prices baskets, best-of, worst-of, and spread options. Visualises correlation sensitivity and comparison to single-asset hedges.

### Interest Rate Derivatives: Hull-White / LGM

Trinomial tree and Monte Carlo for the Hull-White one-factor model. Prices caps, floors, swaptions, and callable bonds. Plots short-rate distributions and calibration to yield curve and vol surface.

### XVA Engine (CVA / DVA / FVA)

Exposure simulation engine generating future portfolio paths and computing expected positive/negative exposure profiles. Calculates CVA and DVA under wrong-way risk scenarios. Dashboard shows exposure profiles, CVA term structure, and credit spread sensitivity.

### FFT Option Pricer (Carr-Madan)

Carr-Madan FFT method pricing across a full strike spectrum in one pass under Heston, Variance Gamma, and CGMY models. Benchmarks speed vs Monte Carlo and plots model-implied densities side by side.

### Adjoint Algorithmic Differentiation (AAD) for Greeks

Tape-based reverse-mode AD computing Greeks for a Monte Carlo pricer in a single backward pass. Compares speed and accuracy against bump-and-revalue across hundreds of sensitivities.

## Advisory & Visualisation Tools

Practical tools for disputes, investigations, and quantitative advisory — bridging pricing knowledge and client-facing deliverables.

### Structured Products Pricing Engine

Monte Carlo pricing framework for common structured products: memory coupon autocallables, Phoenix notes, reverse convertibles, and capital-protected notes. Models path-dependent features including memory coupon accumulation, discrete barrier monitoring, and multi-asset worst-of mechanics.

### Autocallable Valuation & Dispute Analysis

End-to-end valuation of a Phoenix autocallable under competing assumptions. Demonstrates how different volatility models, dividend forecasts, and correlation inputs lead to materially different valuations — the core of structured product disputes. Includes sensitivity waterfall charts.

### Interactive Yield Curve & Forward Rate Visualiser

Streamlit app for visualising yield curves, forward curves, and discount factors across dates and currencies. Supports overlay of historical curves, spread analysis, and curve shift scenarios.

### Volatility Surface Explorer

Interactive 3D volatility surface visualisation tool built with Streamlit and Plotly. Loads market data, fits parametric models (SABR, SVI), and lets users rotate, slice, and compare surfaces across dates.

### Swap Repricing & Curve Sensitivity Tool

Reprices a portfolio of interest rate swaps from first principles — constructing discount and forecast curves, computing present values, and running DV01 and key rate duration analysis. Demonstrates the kind of independent repricing work central to swap disputes.

### FX Derivatives Valuation Toolkit

Prices FX forwards, vanilla options (Garman-Kohlhagen), and common structured FX products (accumulators, TARFs). Includes smile construction from risk reversals and butterflies, and scenario analysis for advisory use cases.

### Energy Market Data Explorer

Streamlit dashboard for exploring energy commodity price data — spot and forward curves for gas hubs (TTF, NBP, JKM), crude benchmarks, and LNG indices. Time-series analysis, seasonal decomposition, hub spread analysis, and historical volatility computation.

### Trade Surveillance & Anomaly Detection

Statistical framework for detecting anomalous trading patterns in time-series data. Implements z-score screening, rolling volatility breakpoint detection, and clustering methods (DBSCAN) to flag unusual activity.

### Loan & DCF Valuation Model

Discounted cash flow model for loan valuation under multiple scenarios — base case, stressed, and recovery. Incorporates credit spread term structures, prepayment assumptions, and probability-weighted outcomes.
