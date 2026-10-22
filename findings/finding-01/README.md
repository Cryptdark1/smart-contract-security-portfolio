# Ghost Loan Entries from Incomplete Loan Cleanup

**Platform:** Cantina  
**Finding ID:** #919  
**Contest:** [Insert contest name]  
**Severity:** Low  
**Status:** Duplicate of finding #14  
**Category:** State management / loan lifecycle cleanup

## Summary

This finding investigates stale ownership entries left behind when loan records are deleted without corresponding cleanup of ownership tracking structures.

The protocol tracks loan ownership using `ownedTokens` and `tokenOwner`. The reported issue is that `_cleanupLoan` deletes the loan record but does not consistently remove its associated ownership references.

## Root Cause

The reported cleanup logic deletes `loans[tokenId]` but leaves the token ID in the owner's enumeration structures.

This can cause `loanCount(owner)` and `loanAtIndex(owner, index)` to expose stale entries for loans that no longer have active loan data.

## Attack Path and Affected Logic

The report investigated loan termination and replacement flows that invoke `_cleanupLoan`, including position transformation and liquidation.

The intended invariant is that every token ID listed in an owner's active loan enumeration corresponds to a valid active loan record.

## Proof of Concept

A Foundry test was prepared to investigate stale loan entries after position replacement.

**Verification status:** The exact reproduction should be independently verified before describing the PoC as a confirmed, reliable reproduction. The submitted test includes approximate storage-slot manipulation.

## Impact Assessment

The judge retained the duplicate classification, stating that the impact was limited to inflated `loanCount` and `loanAtIndex` values and did not have realistic consequences sufficient to change the outcome.

Potential downstream effects on integrations were argued during escalation but were not accepted by the judge as grounds for a severity change.

## Recommended Remediation

Ensure that loan cleanup removes obsolete ownership and enumeration references consistently. Add regression tests covering repayment, liquidation, and position replacement.

## Outcome

This report was marked as a duplicate of finding #14. A request to reconsider the severity was not accepted.

## Research Takeaways

- Tracing state changes across loan lifecycle functions
- Examining consistency between mappings and enumeration arrays
- Formulating protocol invariants
- Developing Foundry-based proof-of-concept tests
- Assessing vulnerability impact and responding to severity decisions

## References

- Cantina finding:
- Related finding #14
