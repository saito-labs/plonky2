# Removing ZK from Plonky2

In its current state, this repository aims to remove all of the ZK components
from the Plonky2 protocol. To further understand this task, one can consult the
[Plonky2 paper](https://docs.rs/crate/plonky2/latest/source/plonky2.pdf) for a
baseline understanding of Plonky2. 

## Summary of Changes

The original Plonky2 implementation provides a `CircuitConfig` for each circuit
that contains a boolean to control whether or not the circuit should be ran in
zero-knowledge:

```rust
pub struct CircuitConfig {
    ...
    /// A boolean to activate the zero-knowledge property. When this is set to `false`, proofs *may*
    /// leak additional information.
    pub zero_knowledge: bool,
    ...
}
```

Beyond setting this boolean to false, this repo removes all functionality that
relates to this parameter with the aim to discard any overhead between
the ZK and non-ZK functionalities.

A baseline understanding of what is removed can be gained from observing that the Plonky2 paper states that zero-knowedge is achieved by the following steps:

1) Blinding prover polynomials before padding the degree of the polynomials to a
power of two
    * Blind trace polynomails by adding rows filled with random elements to the
        trace
    * Blind the permutation polynomial by adding pairs of randomized rows to the
        trace with copy constraints between each pair of columns
2) Hiding values and commitments during the FRI protocol to make it
zero-knowledge
    * Send evaluations of prover polynomials on a coset of the evaluation domain
    * Transform Merkle trees into hiding vector commitments by wrapping each
        leaf in a has-based commitment


To achieve the removal of 1), the following functionalities were removed:
* Remove option to construct ZK circuits
* Remove all functions used to achieve blinding
    * `num_blinding_gates`
    * `blinding_counts`
    * `blind`
    * `blind_and_pad` changed to `pad` and doesn't call `blind` function


To remove 2), following functionalities were deleted:
* Remove all instances of `hiding` or `blinding` in the FRI protocol
    * Construction of `FriParams`
    * `PolynomialBatch` that represents a FRI oracle
    * Calculating low degree extension values
* Remove everything related to salting
    * `PolynomialBatch` that represents a FRI oracle
    * Fetching LDE values at the `index * step`th point
    * Calculating low degree extension values
    * Evaluating values in `FriInitialTreeProof`
    * Combining initial evals in `fri_combine_initial`
    * Validating FRI proof shape
    * Recursive verifier
* Use `lde` instead of `lde_on_coset` in the computation the quotient
    polynomials to avoid evaluation on a coset of `H`

## Resulting Benchmarks

To compare the original Plonky2 implementation and this altered fork of it, the
original benchmarks from Plonky2 were used. The benchmarks can be ran by calling
`cargo bench --benches` to evaluate core functions such as multiplicative
inverse on the Goldilocks Field and hasing performance. Upon running the
benchmarks on this implementation with removed ZK components and the original
repository with the `zero_knowledge` parameter set to false, the results showed
that this implementaiton achieves very marginal speedups from the original
implementation. The results of the benchmarks can be found in the `benches`
directory (to be added).
