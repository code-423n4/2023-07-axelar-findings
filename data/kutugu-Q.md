# Findings Summary

| ID     | Title                                                                     | Severity |
| ------ | ------------------------------------------------------------------------- | -------- |
| [L-01] | Cross chain messages have no any executing protect                        | Low      |
| [L-02] | Gas obtained by SELFDESTRUCT will be stuck in the contract                | Low      |
| [L-03] | CommandId may be repeated so that subsequent messages cannot be executed  | Low      |

# Detailed Findings

# [L-01] Cross chain messages have no any executing protect

## Description

Cross-chain message execution has no protection, can be called by anyone, and has no expiration time, which may cause the user to lose funds.
For example, users want to call only by themselves at a certain time, otherwise it will affect income.
When a message remains unexecuted for a long time, anyone can execute it and may excess the user's expectations

## Recommendations

Add expiration time protection and msg.sender execution protection

# [L-02] Gas obtained by SELFDESTRUCT will be stuck in the contract

## Description

AxelarGateway does not have any way to withdraw the gas compensation from the selfdestruct, which will be stuck in the contract and cannot be withdrawn.

## Recommendations

Add a withdrawn function 

# [L-03] CommandId may be repeated so that subsequent messages cannot be executed

## Description

```solidity
    function callContract(
        string calldata destinationChain,
        string calldata destinationContractAddress,
        bytes calldata payload
    ) external {
        emit ContractCall(msg.sender, destinationChain, destinationContractAddress, keccak256(payload), payload);
    }
```

Cross-chain message execution is conducted by invoking the `AxelarGateway.callContract` by InterchainProposalSender, and corresponding logs are generated to form commandId. From the code, logs are not include nonce. commandId may conflict if the msg.sender sends the same message multiple times.

## Recommendations

Add nonce to prevent commandId conflicts