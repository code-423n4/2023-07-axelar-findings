# Axelar Non-Critical Findings

## [N-01] Else-block not required
One level of nesting can be removed by not having an else block since the if-block reverts.

### 1 Instance
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L161-#L169
```solidity
    if (result.length > 0) {
        // The failure data is a revert reason string.
        assembly {
            revert(add(32, result), mload(result))
        }
    } else {
        // There is no failure data, just revert with no reason.
            revert ProposalExecuteFailed();
    }
```
code could be refactored to:
```solidity
    if (result.length > 0) {
        // The failure data is a revert reason string.
        assembly {
            revert(add(32, result), mload(result))
        }
    }
    // There is no failure data, just revert with no reason.
    revert ProposalExecuteFailed();
    
```

## [N-02] Consider using delete rather than assigning zero/false to clear values
The delete keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

### 1 Instance
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L97
```solidity
    function removeTrustedAddress(string calldata chain) external onlyOwner {
        if (bytes(chain).length == 0) revert ZeroStringLength();
        remoteAddressHashes[chain] = bytes32(0);
        remoteAddresses[chain] = '';
        emit TrustedAddressRemoved(chain);
    }
```
The function could be refactored to: 
```solidity
    function removeTrustedAddress(string calldata chain) external onlyOwner {
        if (bytes(chain).length == 0) revert ZeroStringLength();
        delete remoteAddressHashes[chain];
        delete remoteAddresses[chain];
        emit TrustedAddressRemoved(chain);
    }
```

## [N-03] Event is not properly indexed
Index event fields make the field more quickly accessible to off-chain tools that parse events. This is especially useful when it comes to filtering based on an address. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Where applicable, each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three applicable fields, all of the applicable fields should be indexed. 

### 13 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IAxelarServiceGovernance.sol#L15
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IAxelarServiceGovernance.sol#L16
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IAxelarServiceGovernance.sol#L17
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IMultisigBase.sol#L22
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L7
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L10
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol#L32
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol#L33-#L40
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol#L67
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol#L68-#L75
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IDistributable.sol#L8
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IFlowLimit.sol#L8
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IOperatable.sol#L8
