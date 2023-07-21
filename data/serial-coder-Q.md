# [L-01] Low-level call to an address with no contract

## Proof of Concept

The `Caller._call()` and `InterchainProposalExecutor._executeProposal()` functions below execute a low-level call to a `target` address without verifying that the `target` must not be an address with no contract.

Suppose the `target` is an address with no contract. In that case, **the low-level call's execution status will always return `true` (`success == true`), resulting in misinterpreting that a specified function call on the `target` address is successfully executed.**

**Moreover, if a native value (Ethers) is passed along with the low-level call, the Ethers will always be stuck in the `target` address.**

```solidity
    function _call(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) internal {
        if (nativeValue > address(this).balance) revert InsufficientBalance();

@>      (bool success, ) = target.call{ value: nativeValue }(callData);
        if (!success) {
            revert ExecutionFailed();
        }
    }
```
Caller._call(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Caller.sol#L18

```solidity
    function _executeProposal(InterchainCalls.Call[] memory calls) internal {
        for (uint256 i = 0; i < calls.length; i++) {
            InterchainCalls.Call memory call = calls[i];
@>          (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

            if (!success) {
                _onTargetExecutionFailed(call, result);
            } else {
                _onTargetExecuted(call, result);
            }
        }
    }
```
InterchainProposalExecutor._executeProposal(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76

The following lists functions that execute the `Caller._call()`
- [AxelarServiceGovernance.executeMultisigProposal()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L59)
- [InterchainGovernance.executeProposal()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L76)
- [Multisig.execute()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/Multisig.sol#L35)

The following lists functions that execute the `InterchainProposalExecutor._executeProposal()`
- [InterchainProposalExecutor._execute()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L62), which would be eventually called by the [AxelarExecutable.execute()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/executable/AxelarExecutable.sol#L27)

## Impact

Suppose the `target` is an address with no contract. In that case, **the low-level call's execution status will always return `true` (`success == true`), resulting in misinterpreting that a specified function call on the `target` address is successfully executed.**

**Moreover, if a native value (Ethers) is passed along with the low-level call, the Ethers will always be stuck in the `target` address.**

## Tools Used

Manual Review

## Recommended Mitigation Steps

I recommend verifying that the `target` address has a contract.

---

# [L-02] Incorrect token approvals for a token manager

The token approvals for a token manager in `InterchainToken.interchainTransfer()` and `InterchainToken.interchainTransferFrom()` are incorrect, resulting in unexpected unlimited approvals.

## Proof of Concept

The below shows the functions `InterchainToken.interchainTransfer()` and `InterchainToken.interchainTransferFrom()`.

```solidity
    function interchainTransfer(
        string calldata destinationChain,
        bytes calldata recipient,
        uint256 amount,
        bytes calldata metadata
    ) external payable {
        address sender = msg.sender;
        ITokenManager tokenManager = getTokenManager();
        /**
         * @dev if you know the value of `tokenManagerRequiresApproval()` you can just skip the if statement and just do nothing or _approve.
         */
        if (tokenManagerRequiresApproval()) {
@>          uint256 allowance_ = allowance[sender][address(tokenManager)];
@>          if (allowance_ != type(uint256).max) {
@>              if (allowance_ > type(uint256).max - amount) {
@>                  allowance_ = type(uint256).max - amount;
@>              }
@>
@>              _approve(sender, address(tokenManager), allowance_ + amount);
@>          }
        }

        // Metadata semantics are defined by the token service and thus should be passed as-is.
        tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);
    }
```
InterchainToken.interchainTransfer(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L55-L62

```solidity
    function interchainTransferFrom(
        address sender,
        string calldata destinationChain,
        bytes calldata recipient,
        uint256 amount,
        bytes calldata metadata
    ) external payable {
        uint256 _allowance = allowance[sender][msg.sender];

        if (_allowance != type(uint256).max) {
            _approve(sender, msg.sender, _allowance - amount);
        }

        ITokenManager tokenManager = getTokenManager();
        if (tokenManagerRequiresApproval()) {
@>          uint256 allowance_ = allowance[sender][address(tokenManager)];
@>          if (allowance_ != type(uint256).max) {
@>              if (allowance_ > type(uint256).max - amount) {
@>                  allowance_ = type(uint256).max - amount;
@>              }
@>
@>              _approve(sender, address(tokenManager), allowance_ + amount);
@>          }
        }

        tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);
    }
```
InterchainToken.interchainTransferFrom(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L94-L101

If the [`if (allowance_ > type(uint256).max - amount) { ... }`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L57) statement is `true`, the [`allowance_ = type(uint256).max - amount;`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L58) statement will be executed. 

This will cause the token approval to be set to `unlimited` after executing the [`_approve(sender, address(tokenManager), allowance_ + amount);`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L61).

## Impact

The user(sender)'s token transfer allowance for a token manager will be set to `unlimited` unexpectedly.

## Tools Used

Manual Review

## Recommended Mitigation Steps

Two possible solutions
1. Enforce a user (sender) to approve token transfer allowance for a token manager explicitly. The token manager's allowance will be decreased per its actual spending amount.
2. Overwrite the token manager's allowance to only an amount required to transfer in the `InterchainToken.interchainTransfer()` and `InterchainToken.interchainTransferFrom()`. After executing the above functions, this solution will reset the token manager's allowance to 0.

---

# [L-03] Potentially misinterpreting the flow limit

## Impact

If the `flowLimit`'s value is set to `0`,  it will be interpreted as an `unlimited token transfer allowance` (confirmed by the sponsor). 

However, using the value `0` in this case can lead to a misunderstanding, resulting in operational mistakes. For example, an operator might interpret that the value `0` means enforcing no in/out transfer allowance, but it's not (it actually means unlimited transfer allowance).

## Proof of Concept

```solidity
    function _addFlowOut(uint256 flowOutAmount) internal {
        uint256 flowLimit = getFlowLimit();
@>      if (flowLimit == 0) return;
        uint256 epoch = block.timestamp / EPOCH_TIME;
        uint256 slotToAdd = _getFlowOutSlot(epoch);
        uint256 slotToCompare = _getFlowInSlot(epoch);
        _addFlow(flowLimit, slotToAdd, slotToCompare, flowOutAmount);
    }

    // ... SNIPPED ...
    
    function _addFlowIn(uint256 flowInAmount) internal {
        uint256 flowLimit = getFlowLimit();
@>      if (flowLimit == 0) return;
        uint256 epoch = block.timestamp / EPOCH_TIME;
        uint256 slotToAdd = _getFlowInSlot(epoch);
        uint256 slotToCompare = _getFlowOutSlot(epoch);
        _addFlow(flowLimit, slotToAdd, slotToCompare, flowInAmount);
    }
```

- _addFlowOut(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/FlowLimit.sol#L113

- _addFlowIn(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/FlowLimit.sol#L126

## Tools Used

Manual Review

## Recommended Mitigation Steps

I recommend using `type(uint256).max` for representing an unlimited transfer allowance instead. For the value `0`, it can be used to describe no transfer allowance.

---

# [L-04] Lock of Ethers in proxy contracts

The `receive()` of the proxy contracts always accepts Ethers (native coin) without forwarding the control flow to the implementation contract.

## Proof of Concept

BaseProxy.receive(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/BaseProxy.sol#L67

FixedProxy.receive(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/FixedProxy.sol#L59

TokenManagerProxy.receive(): https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/proxies/TokenManagerProxy.sol#L94

## Impact

The implementation contracts might not be designed to accept Ethers, resulting in forever locked Ethers in the proxy contracts.

## Tools Used

Manual Review

## Recommended Mitigation Steps

The `receive()` of proxy contracts should forward the control flow to the implementation contract's `receive()` and let the implementation contract decide whether to accept Ethers or what should be done next.