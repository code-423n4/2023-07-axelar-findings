# Findings Summary

| ID     | Title                                                       | Severity |
| ------ | ----------------------------------------------------------- | -------- |
| [L-01] | Cross chain messages have no any executing protect          | Low      |
| [L-02] | EIP-4758: Deactivate SELFDESTRUCT                           | Low      |
| [L-03] | Gas obtained by SELFDESTRUCT will be stuck in the contract  | Low      |

# Detailed Findings

# [L-01] Cross chain messages have no any executing protect

## Description

Cross-chain message execution has no protection, can be called by anyone, and has no expiration time, which may cause the user to lose funds.
For example, users want to call only by themselves at a certain time, otherwise it will affect income.
When a message remains unexecuted for a long time, anyone can execute it and may excess the user's expectations

## Recommendations

Add expiration time protection and msg.sender execution protection

# [L-02] EIP-4758: Deactivate SELFDESTRUCT

## Description

The SELFDESTRUCT opcode may be removed through a hard fork in the future, which will result in applications will be broken

## Recommendations

There seems to be no need to selfdestruct the contract

# [L-03] Gas obtained by SELFDESTRUCT will be stuck in the contract

## Description

AxelarGateway does not have any way to withdraw the gas compensation from the selfdestruct, which will be stuck in the contract and cannot be withdrawn.

## Recommendations

Add a withdrawn function
