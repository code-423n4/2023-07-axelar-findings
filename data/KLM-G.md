# Report: Gas Optimization - Prefer require over if in Solidity Functions
## Overview

This report highlights the importance of using `require` statements instead of `if` conditions in Solidity functions to optimize gas consumption. By utilizing `require` statements, the number of EVM opcodes generated can be reduced, resulting in more efficient gas usage. This finding applies to all contracts reviewed during the audit.

## Potential Benefits

- Reduced gas consumption: require statements consume only the necessary gas to validate the condition, resulting in more efficient usage.
- Simplified code: require statements provide a concise and expressive way to enforce conditions, enhancing code readability and maintainability.

## Recommendation

Based on the gas optimization benefits mentioned above, it is recommended to replace if conditions with require statements wherever appropriate within the audited contracts. By doing so, gas consumption can be minimized, leading to more efficient and cost-effective execution of the contracts.

## Contracts concerned :
- All