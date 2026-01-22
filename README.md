
# Bayesian Inference for an Inverse Heat Problem

This repository contains a Python notebook that shows how Bayesian inference can be used to solve an inverse boundary value problem for the steady‑state heat equation (Laplace equation). The goal is to recover an unknown temperature boundary condition from noisy interior temperature measurements.

The project combines:

* Finite‑difference discretization of the Laplace equation
* A reduced‑order boundary parameterization using Gaussian radial basis functions (RBFs)
* A Bayesian statistical model (Gaussian prior and Gaussian likelihood)
* Markov Chain Monte Carlo (MCMC) sampling via Metropolis–Hastings

---

## Problem Description

Consider a 2D unit square domain (\Omega = [0,1] \times [0,1]), governed by the Laplace equation:

$$
\nabla^2 u = 0 \quad \text{in } \Omega
$$

Boundary conditions:

* The left boundary temperature (u(0,y) = f(y)) is *unknown*
* The remaining boundaries are fixed (implicitly zero in the implementation)

Observe noisy temperature measurements at a finite set of interior points:

$$
\nu_i = u(x_i, y_i) + \eta_i, \quad \eta_i \sim \mathcal{N}(0, \sigma^2)
$$

The inverse problem is to infer the unknown boundary function (f(y)) from these interior observations.

---

## Bayesian Formulation

### Parameterization

The unknown boundary condition is represented as a finite expansion of Gaussian RBFs:

$$
f(y) = \sum_{k=1}^K \alpha_k , \exp!\left( -\frac{(y - y_k)^2}{2\ell^2} \right)
$$

where:

* (\alpha_k) are unknown coefficients
* (y_k) are fixed RBF centers along the boundary
* (\ell) is the RBF width

### Prior

A zero‑mean Gaussian prior is placed on the coefficients:

$$
\alpha \sim \mathcal{N}(0, \tau^2 I)
$$

### Likelihood

Given predicted measurements (G(\alpha)), the likelihood is:

$$
\nu \mid \alpha \sim \mathcal{N}(G(\alpha), \sigma^2 I)
$$

### Posterior

The log‑posterior (up to a constant) is:

$$
\log \pi(\alpha \mid \nu) = -\frac{|G(\alpha) - \nu|^2}{2\sigma^2}

* \frac{|\alpha|^2}{2\tau^2}
  $$

---

## Forward Model

The forward map consists of:

1. Solving the Laplace equation on an (N \times N) interior grid using finite differences
2. Applying the RBF‑defined boundary condition on the left boundary
3. Solving the resulting sparse linear system
4. Evaluating the solution at measurement points using Gaussian mollification, which mimics spatially averaged sensor readings

This entire process defines a nonlinear map:

$$
\alpha ;\longmapsto; G(\alpha)
$$

---

## MCMC Sampling

Posterior sampling is performed using a Metropolis–Hastings algorithm:

* Proposal: Gaussian random walk
* Initial state: (\alpha = 0)
* Burn‑in: first third of samples discarded

The sampler produces an ensemble of boundary realizations, from which posterior statistics are computed.

---

## Outputs and Diagnostics

The notebook produces:

### Boundary reconstruction

* Posterior mean of the inferred boundary condition
* 95% credible interval
* Reference boundary samples (for validation)

### MCMC diagnostics

* Trace plots for selected coefficients
* Autocorrelation functions
* Acceptance rate (with reference optimal value ≈ 0.234)
* Log‑posterior trace

These diagnostics help assess convergence and sampling efficiency.

---

## 📦 Dependencies

The code relies on standard scientific Python libraries:

* `numpy`
* `matplotlib`
* `scipy` (sparse matrices and linear solvers)

All computations are CPU‑based and suitable for execution in Jupyter or Google Colab.

---

## Data

Measurement data are loaded from a NumPy `.npz` file containing:

* Interior measurement locations
* Noisy temperature observations
* Noise level
* Reference boundary samples (for comparison and visualization)

