# BooleanCanalization

An R library for computing canalization measures — specifically **effective connectivity (K_eff)** and **unified canalization (K_u)** — for Boolean functions and Boolean networks.

## Overview

Canalization in Boolean networks refers to the phenomenon where certain input configurations dominate the output, effectively reducing the number of inputs that matter for a given function. This library quantifies canalization through two complementary measures:

### Effective Connectivity (K_eff)

K_eff measures the average number of inputs that *actually* influence a Boolean function's output, using **schema-based decomposition**. Schemata (prime implicants) are minimal input patterns — containing fixed positions (0, 1) and wildcards (#) — that uniquely determine the output. K_eff counts how many positions are fixed (non-wildcard) on average across all input states.

### Unified Canalization (K_u) via Input Symmetry

K_u extends K_eff by exploiting **permutation symmetry** among inputs. Many Boolean functions are partially symmetric: for a subset of inputs, the output depends only on *how many* are 1, not *which ones* are 1. K_u captures this through **two-symbol schemata**, a generalization of ordinary schemata:

- **Ordinary schemata** use three symbols: `0`, `1`, and `#` (wildcard — either value)
- **Two-symbol schemata** add primed symbols: `0'`, `1'`, and `#'` (permuting wildcard)

Primed positions are interchangeable — the function's output is invariant to permutations among them. For example, the schema `(0'1'#')` covers all inputs where the three permuting positions contain 1–2 ones, regardless of which specific positions hold them. The number of covered inputs follows the binomial coefficient: `|(0'1'#')| = C(3,1) + C(3,2) = 6`.

**Composite schemata** combine both types: fixed wildcards `#` (position doesn't matter independently) and permuting wildcards `#'` (positions are interchangeable as a group). For example, `(#0'1'#')` has one ordinary wildcard and three symmetric positions.

K_u is computed by finding the largest two-symbol schema covering each input vector, averaging the covering dimensions, and subtracting from k: `K_u = k - k_r*`. Since two-symbol schemata can cover more input states than ordinary schemata, K_u ≤ K_eff — it reveals additional redundancy that K_eff misses.

### Integrated K_eff

Extends effective connectivity to temporal Boolean network dynamics, tracking how K_eff evolves as the network is iterated through its update rules over time.

## Repository Structure

```
├── ComputeKu.R                        # Compute K_u (unified canalization via symmetry)
├── ComputeKeff.R                      # Compute K_eff for a single Boolean function
├── ComputeKeffFromSchemata.R          # Compute K_eff from pre-computed schemata
├── ComputeIntegratedKeff.R            # K_eff over iterated Boolean network dynamics
├── ComputeIntegrateBoolNet.R          # Iterate a Boolean network and track schemata
├── ComputePrimeImplicants.R           # Prime implicant computation
├── ComputeDetectCubes.R               # Cube detection for schema decomposition
├── ComputeDecomposeSchemataNoOverlaps.R  # Non-overlapping schema decomposition
├── ComputeSchemaSetOperations.R       # Set operations on schema representations
├── Documentation.pdf                  # Detailed documentation and examples
└── LICENSE
```

## Getting Started

### Requirements

- R (≥ 3.0)
- `R.utils` package (for `ComputeKu.R`)
- `parallel` package (for `ComputeKu.R`, optional multi-core support)
- `BoolNet` package (for `ComputeIntegrateBoolNet.R`)

### Computing K_eff for a Boolean Function

```r
source('ComputeKeff.R')

# Define a 3-input Boolean function as its output column
# Example: a nearly-canalizing function (7 of 8 outputs are 1)
k <- 3
func <- c(1, 1, 1, 1, 0, 1, 1, 1)

result <- ComputeKeff(func, k, ReturnSchemata = TRUE)
print(result$Keff)
# [1] 1.25

print(result$EssentialSchemata)
# Shows the minimal input patterns determining each output
```

### Computing K_u with Symmetry-based Schemata

```r
source('ComputeKu.R')

k <- 3
func <- c(1, 1, 1, 1, 0, 1, 1, 1)

result <- ComputeKu(func, k, ComputeKeff = TRUE, ReturnSymmSchs = TRUE)
print(result$Ku)
# [1] 0.8112781

print(result$Keff)
# [1] 1.25

print(result$EssentialSchemata)
# Two-symbol schemata with encoding: 0->0, 1->1, #->2, 0'->3, 1'->4, #'->5
```

### Computing Integrated K_eff for a Boolean Network

```r
source('ComputeIntegrateBoolNet.R')
source('ComputeIntegratedKeff.R')
require(BoolNet)

# Load a Boolean network
Net <- loadNetwork("network.bn", symbolic = FALSE)

# Iterate and track schema transitions
SchemaCausationChain <- IntegrateBoolNet(Net, NumIters = 5)

# Compute K_eff at each time step
IntegratedKeff <- ComputeIntegratedKeff(SchemaCausationChain)
print(IntegratedKeff)
```

## Documentation

See [Documentation.pdf](Documentation.pdf) for detailed mathematical background, algorithm descriptions, and additional usage examples. Appendix A walks through the K_u computation with a worked example.