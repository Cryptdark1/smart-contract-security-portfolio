# Unbounded Merkle Proof Length in `isRatified()`

## Finding Details

- **Platform:** Cantina
- **Finding ID:** #3744
- **Severity:** Informational
- **Status:** Duplicate
- **Category:** Gas Griefing / Unbounded Input
- **Research Areas:** Merkle proof verification, gas consumption, input validation

## Summary

The `isRatified()` function in `EcrecoverRatifier` decodes `ratifierData` into several values, including a Merkle proof represented as `bytes32[] memory proof`.

The reported concern was that the function does not explicitly limit the proof length before passing the proof to `HashLib.isLeaf()`. An excessively long proof may therefore require additional computation and gas during verification.

This investigation focused on whether unbounded proof input could create unnecessary gas costs for callers that rely on the ratifier during operations such as `take()`.

## Technical Analysis

The relevant code path decodes the proof and passes it to the Merkle verification function:

```solidity
(
    Signature memory sig,
    bytes32 root,
    uint256 leafIndex,
    bytes32[] memory proof
) = abi.decode(
    ratifierData,
    (Signature, bytes32, uint256, bytes32[])
);

require(
    HashLib.isLeaf(
        root,
        HashLib.hashOffer(offer),
        leafIndex,
        proof
    ),
    InvalidProof()
);
```

The reported issue concerns the absence of an explicit proof-length bound in this code path.

If proof verification performs work proportional to the number of proof elements, larger inputs may increase gas consumption. The practical significance depends on the implementation of `HashLib.isLeaf()`, the protocol's call flow, and the limits imposed by the transaction and calling contract.

## Potential Impact

An excessively long proof could increase the computational cost of verification. Depending on how the ratifier is called, this may increase the gas required by users or automated systems attempting an operation that depends on ratification.

The report proposed gas griefing as a potential concern. However, the practical impact and whether legitimate operations can be meaningfully disrupted require validation against the full implementation and calling context.

## Recommended Mitigation

Consider enforcing a maximum proof length before performing Merkle verification.

For example, if the protocol's supported tree depth permits it:

```solidity
if (proof.length > MAX_PROOF_LENGTH) {
    revert ProofTooLarge();
}
```

The maximum should be derived from the protocol's actual Merkle-tree structure and supported proof depths. A limit of 32 should not be adopted without confirming that it preserves all legitimate use cases.

## Submission Outcome

This finding was submitted through Cantina as an **Informational** issue and marked **Duplicate**.

It is included here as an example of security research into unbounded input, proof verification, and gas-cost considerations. The duplicate outcome is disclosed rather than presenting the report as an accepted vulnerability.

## Lessons Learned

- Trace user-controlled input through decoding and downstream verification.
- Examine whether loops or repeated hashing make computation scale with input size.
- Trace the complete calling context before concluding that gas consumption can cause meaningful disruption.
- Distinguish a plausible concern from a demonstrated security impact.
- Derive input limits from protocol requirements rather than choosing arbitrary constants.

## References

- **Platform:** Cantina
- **Finding:** #3744 — Unbounded Merkle Proof Length Allows Gas Griefing in `isRatified()`
- **Contest:** Morpho Midnight
