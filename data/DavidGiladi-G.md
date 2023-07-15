
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Inefficient use of abi.encode() | [Inefficient use of abi.encode()](#inefficient-use-of-abiencode) | 14 | 1400 |
|[G-2] Use assembly to emit events | [Use assembly to emit events](#use-assembly-to-emit-events) | 43 | 1634 |
|[G-3] Potential Optimization by Combining Multiple Mappings into a Struct | [Potential Optimization by Combining Multiple Mappings into a Struct](#potential-optimization-by-combining-multiple-mappings-into-a-struct) | 2 | 1000 |
|[G-4] Avoid duplicate keccak256 calls | [Avoid duplicate keccak256 calls](#avoid-duplicate-keccak256-calls) | 3 | 126 |
|[G-5] Cache External Calls in Loops | [Cache External Calls in Loops](#cache-external-calls-in-loops) | 1 | 100 |
|[G-6] Functions Guaranteed to Revert When Called by Normal Users Should Be Marked Payable | [Functions Guaranteed to Revert When Called by Normal Users Should Be Marked Payable](#functions-guaranteed-to-revert-when-called-by-normal-users-should-be-marked-payable) | 33 | 693 |
|[G-7] Identical Deployments Should be Replaced with Clone | [Identical Deployments Should be Replaced with Clone](#identical-deployments-should-be-replaced-with-clone) | 1 | - |
|[G-8] Inefficient Parameter Storage | [Inefficient Parameter Storage](#inefficient-parameter-storage) | 10 | 500 |
|[G-9] Optimal State Variable Order | [Optimal State Variable Order](#optimal-state-variable-order) | 1 | 5000 |
|[G-10] Superfluous event fields | [Superfluous event fields](#superfluous-event-fields) | 1 | - |
|[G-11] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 11 | - |
|[G-12] External calls that could skip contract existence check | [External calls that could skip contract existence check](#external-calls-that-could-skip-contract-existence-check) | 47 | - |
|[G-13] Unused Named Return Variables | [Unused Named Return Variables](#unused-named-return-variables) | 2 | - |
|[G-14] Usage of Custom Errors for Gas Efficiency | [Usage of Custom Errors for Gas Efficiency](#usage-of-custom-errors-for-gas-efficiency) | 1 | 276 |

Total: 14 issues

## Inefficient use of abi.encode()
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 1400

### Description
The `abi.encode()` function is less gas efficient than `abi.encodePacked()`, especially when encoding only static types. This detector identifies instances where `abi.encode()` is used and all arguments are static, suggesting that `abi.encodePacked()` could potentially be used instead for better gas efficiency. Note: `abi.encodePacked()` should not be used if any of the arguments are dynamic types, as it could lead to hash collisions.

<details>

<summary>
There are 14 instances of this issue:

</summary>

###
- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 25          deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)))
```
 use  abi.encodepacked() instead.

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L25](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L25)


- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 47          deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)))
```
 use  abi.encodepacked() instead.

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L47](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L47)


- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 63          bytes32 newSalt = keccak256(abi.encode(sender, salt))
```
 use  abi.encodepacked() instead.

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L63](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L63)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 30          bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt))
```
 use  abi.encodepacked() instead.

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L30](contracts/gmp-sdk/deploy/Create3Deployer.sol#L30)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 54          bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt))
```
 use  abi.encodepacked() instead.

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L54](contracts/gmp-sdk/deploy/Create3Deployer.sol#L54)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 68          bytes32 deploySalt = keccak256(abi.encode(sender, salt))
```
 use  abi.encodepacked() instead.

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L68](contracts/gmp-sdk/deploy/Create3Deployer.sol#L68)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 203          tokenId = keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_ID, chainNameHash, tokenAddress))
```
 use  abi.encodepacked() instead.

[contracts/its/interchain-token-service/InterchainTokenService.sol#L203](contracts/its/interchain-token-service/InterchainTokenService.sol#L203)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 214          tokenId = keccak256(abi.encode(PREFIX_CUSTOM_TOKEN_ID, sender, salt))
```
 use  abi.encodepacked() instead.

[contracts/its/interchain-token-service/InterchainTokenService.sol#L214](contracts/its/interchain-token-service/InterchainTokenService.sol#L214)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 313          _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress))
```
 use  abi.encodepacked() instead.

[contracts/its/interchain-token-service/InterchainTokenService.sol#L313](contracts/its/interchain-token-service/InterchainTokenService.sol#L313)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 693          _deployTokenManager(
            tokenId,
            tokenManagerType,
            abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
        )
```
 use  abi.encodepacked() instead.

[contracts/its/interchain-token-service/InterchainTokenService.sol#L693-L697](contracts/its/interchain-token-service/InterchainTokenService.sol#L693-L697)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 828          return keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_SALT, tokenId))
```
 use  abi.encodepacked() instead.

[contracts/its/interchain-token-service/InterchainTokenService.sol#L828](contracts/its/interchain-token-service/InterchainTokenService.sol#L828)


- File: contracts/its/utils/ExpressCallHandler.sol
```
 
Line: 32          slot = uint256(keccak256(abi.encode(PREFIX_EXPRESS_RECEIVE_TOKEN, tokenId, destinationAddress, amount, commandId)))
```
 use  abi.encodepacked() instead.

[contracts/its/utils/ExpressCallHandler.sol#L32](contracts/its/utils/ExpressCallHandler.sol#L32)


- File: contracts/its/utils/FlowLimit.sol
```
 
Line: 47          slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)))
```
 use  abi.encodepacked() instead.

[contracts/its/utils/FlowLimit.sol#L47](contracts/its/utils/FlowLimit.sol#L47)


- File: contracts/its/utils/FlowLimit.sol
```
 
Line: 56          slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)))
```
 use  abi.encodepacked() instead.

[contracts/its/utils/FlowLimit.sol#L56](contracts/its/utils/FlowLimit.sol#L56)


</details>

# 


## Use assembly to emit events
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 1634

### Description
This detector checks for instances where a contract uses assembly code to emit events. While it's technically possible to emit events using inline assembly in Solidity, it's generally discouraged due to readability concerns and potential for errors. Events should usually be emitted using higher-level Solidity constructs.

<details>

<summary>
There are 43 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 104          emit TokenSent(msg.sender, destinationChain, destinationAddress, symbol, amount)
```

[contracts/cgp/AxelarGateway.sol#L104](contracts/cgp/AxelarGateway.sol#L104)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 112          emit ContractCall(msg.sender, destinationChain, destinationContractAddress, keccak256(payload), payload)
```

[contracts/cgp/AxelarGateway.sol#L112](contracts/cgp/AxelarGateway.sol#L112)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 123          emit ContractCallWithToken(msg.sender, destinationChain, destinationContractAddress, keccak256(payload), payload, symbol, amount)
```

[contracts/cgp/AxelarGateway.sol#L123](contracts/cgp/AxelarGateway.sol#L123)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 682          emit GovernanceTransferred(getAddress(KEY_GOVERNANCE), newGovernance)
```

[contracts/cgp/AxelarGateway.sol#L682](contracts/cgp/AxelarGateway.sol#L682)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 688          emit MintLimiterTransferred(getAddress(KEY_MINT_LIMITER), newMintLimiter)
```

[contracts/cgp/AxelarGateway.sol#L688](contracts/cgp/AxelarGateway.sol#L688)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 630          emit TokenMintLimitUpdated(symbol, limit)
```

[contracts/cgp/AxelarGateway.sol#L630](contracts/cgp/AxelarGateway.sol#L630)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 289          emit Upgraded(newImplementation)
```

[contracts/cgp/AxelarGateway.sol#L289](contracts/cgp/AxelarGateway.sol#L289)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 319          emit OperatorshipTransferred(newOperatorsData)
```

[contracts/cgp/AxelarGateway.sol#L319](contracts/cgp/AxelarGateway.sol#L319)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 376          emit Executed(commandId)
```

[contracts/cgp/AxelarGateway.sol#L376](contracts/cgp/AxelarGateway.sol#L376)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 418          emit TokenDeployed(symbol, tokenAddress)
```

[contracts/cgp/AxelarGateway.sol#L418](contracts/cgp/AxelarGateway.sol#L418)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 466          emit ContractCallApproved(commandId, sourceChain, sourceAddress, contractAddress, payloadHash, sourceTxHash, sourceEventIndex)
```

[contracts/cgp/AxelarGateway.sol#L466](contracts/cgp/AxelarGateway.sol#L466)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 482          emit ContractCallApprovedWithMint(
            commandId,
            sourceChain,
            sourceAddress,
            contractAddress,
            payloadHash,
            symbol,
            amount,
            sourceTxHash,
            sourceEventIndex
        )
```

[contracts/cgp/AxelarGateway.sol#L482-L492](contracts/cgp/AxelarGateway.sol#L482-L492)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 498          emit OperatorshipTransferred(newOperatorsData)
```

[contracts/cgp/AxelarGateway.sol#L498](contracts/cgp/AxelarGateway.sol#L498)


- File: contracts/cgp/auth/MultisigBase.sol
```
 
Line: 178          emit SignersRotated(newAccounts, newThreshold)
```

[contracts/cgp/auth/MultisigBase.sol#L178](contracts/cgp/auth/MultisigBase.sol#L178)


- File: contracts/cgp/auth/MultisigBase.sol
```
 
Line: 74          emit MultisigOperationExecuted(topic)
```

[contracts/cgp/auth/MultisigBase.sol#L74](contracts/cgp/auth/MultisigBase.sol#L74)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 61          emit MultisigExecuted(proposalHash, target, callData, nativeValue)
```

[contracts/cgp/governance/AxelarServiceGovernance.sol#L61](contracts/cgp/governance/AxelarServiceGovernance.sol#L61)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 99          emit MultisigApproved(proposalHash, target, callData, nativeValue)
```

[contracts/cgp/governance/AxelarServiceGovernance.sol#L99](contracts/cgp/governance/AxelarServiceGovernance.sol#L99)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 89          emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta)
```

[contracts/cgp/governance/AxelarServiceGovernance.sol#L89](contracts/cgp/governance/AxelarServiceGovernance.sol#L89)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 104          emit MultisigCancelled(proposalHash, target, callData, nativeValue)
```

[contracts/cgp/governance/AxelarServiceGovernance.sol#L104](contracts/cgp/governance/AxelarServiceGovernance.sol#L104)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 94          emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta)
```

[contracts/cgp/governance/AxelarServiceGovernance.sol#L94](contracts/cgp/governance/AxelarServiceGovernance.sol#L94)


- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 78          emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp)
```

[contracts/cgp/governance/InterchainGovernance.sol#L78](contracts/cgp/governance/InterchainGovernance.sol#L78)


- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 135          emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta)
```

[contracts/cgp/governance/InterchainGovernance.sol#L135](contracts/cgp/governance/InterchainGovernance.sol#L135)


- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 130          emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta)
```

[contracts/cgp/governance/InterchainGovernance.sol#L130](contracts/cgp/governance/InterchainGovernance.sol#L130)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 27          emit OwnershipTransferred(newOwner)
```

[contracts/cgp/util/Upgradable.sol#L27](contracts/cgp/util/Upgradable.sol#L27)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 54          emit Upgraded(newImplementation)
```

[contracts/cgp/util/Upgradable.sol#L54](contracts/cgp/util/Upgradable.sol#L54)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 33          emit Deployed(keccak256(bytecode), salt, deployedAddress_)
```

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L33](contracts/gmp-sdk/deploy/Create3Deployer.sol#L33)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 60          emit Deployed(keccak256(bytecode), salt, deployedAddress_)
```

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L60](contracts/gmp-sdk/deploy/Create3Deployer.sol#L60)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 66          emit Upgraded(newImplementation)
```

[contracts/gmp-sdk/upgradable/Upgradable.sol#L66](contracts/gmp-sdk/upgradable/Upgradable.sol#L66)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 66          emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)))
```

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L66](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L66)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 98          emit WhitelistedProposalCallerSet(sourceChain, sourceCaller, whitelisted)
```

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L98](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L98)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 113          emit WhitelistedProposalSenderSet(sourceChain, sourceSender, whitelisted)
```

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L113](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L113)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 819          emit TokenManagerDeployed(tokenId, tokenManagerType, params)
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L819](contracts/its/interchain-token-service/InterchainTokenService.sol#L819)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 790          emit RemoteStandardizedTokenAndManagerDeploymentInitialized(
            tokenId,
            name,
            symbol,
            decimals,
            distributor,
            operator,
            destinationChain,
            gasValue
        )
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L790-L799](contracts/its/interchain-token-service/InterchainTokenService.sol#L790-L799)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 351          emit CustomTokenIdClaimed(tokenId, deployer_, salt)
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L351](contracts/its/interchain-token-service/InterchainTokenService.sol#L351)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 375          emit CustomTokenIdClaimed(tokenId, deployer_, salt)
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L375](contracts/its/interchain-token-service/InterchainTokenService.sol#L375)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 757          emit RemoteTokenManagerDeploymentInitialized(tokenId, destinationChain, gasValue, tokenManagerType, params)
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L757](contracts/its/interchain-token-service/InterchainTokenService.sol#L757)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 869          emit StandardizedTokenDeployed(tokenId, name, symbol, decimals, mintAmount, mintTo)
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L869](contracts/its/interchain-token-service/InterchainTokenService.sol#L869)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 514          emit TokenSent(tokenId, destinationChain, destinationAddress, amount)
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L514](contracts/its/interchain-token-service/InterchainTokenService.sol#L514)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 522          emit TokenSentWithData(tokenId, destinationChain, destinationAddress, amount, sourceAddress, metadata)
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L522](contracts/its/interchain-token-service/InterchainTokenService.sol#L522)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 88          emit TrustedAddressAdded(chain, addr)
```

[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L88](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L88)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 99          emit TrustedAddressRemoved(chain)
```

[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L99](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L99)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 111          emit GatewaySupportedChainAdded(chainName)
```

[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L111](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L111)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 124          emit GatewaySupportedChainRemoved(chainName)
```

[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L124](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L124)


</details>

# 

## Potential Optimization by Combining Multiple Mappings into a Struct
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 1000

### Description
This detector identifies instances where multiple mappings of an address or ID could be combined into a single mapping to a struct. This optimisation not only saves a storage slot for each mapping that is combined, but also makes the contract more efficient in terms of gas usage. Depending on the circumstances and sizes of types, it can avoid a 20000 gas cost (Gsset) per combined mapping. Also, it can make reads and subsequent writes cheaper when a function requires both values and they both fit in the same storage slot. Lastly, if both fields are accessed in the same function, it can save approximately 42 gas per access due to not having to recalculate the key's keccak256 hash and its associated stack operations.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 24          mapping(string => mapping(address => bool)) public whitelistedCallers
```
Consider to combine with <br>-    ```Line: 27 mapping(string => mapping(address => bool)) public whitelistedSenders```<br><br><br>
[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 15          mapping(string => bytes32) public remoteAddressHashes
```
Consider to combine with <br>-    ```Line: 16 mapping(string => string) public remoteAddresses```<br>-    ```Line: 19 mapping(string => bool) public supportedByGateway```<br><br><br>
[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L15](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L15)


</details>

# 


## Identical Deployments Should be Replaced with Clone
- Severity: Gas Optimization
- Confidence: High

### Description
Detects instances where the same contract is deployed multiple times. Such cases might benefit from the clone factory pattern (EIP-1167), which can significantly reduce deployment gas costs.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
-     - CreateDeployer is deployed in multiple functions:
File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 58          CreateDeployer deployer = new CreateDeployer{ salt: salt }()
```
File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 58          CreateDeployer deployer = new CreateDeployer{ salt: salt }()
```
File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 58          CreateDeployer deployer = new CreateDeployer{ salt: salt }()
```
File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 58          CreateDeployer deployer = new CreateDeployer{ salt: salt }()
```

[contracts/gmp-sdk/deploy/Create3.sol#L58](contracts/gmp-sdk/deploy/Create3.sol#L58)


</details>

# 


## Functions Guaranteed to Revert When Called by Normal Users Should Be Marked Payable
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 693

### Description
This detector identifies functions that are guaranteed to revert when called by normal users. Such functions should be marked as payable, given that they will never succeed in processing Ether transactions.

<details>

<summary>
There are 33 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 254          function transferGovernance(address newGovernance) external override onlyGovernance 
```
 `transferGovernance` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L254-L258](contracts/cgp/AxelarGateway.sol#L254-L258)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 260          function transferMintLimiter(address newMintLimiter) external override onlyMintLimiter 
```
 `transferMintLimiter` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L260-L264](contracts/cgp/AxelarGateway.sol#L260-L264)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 266          function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter 
```
 `setTokenMintLimits` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L266-L278](contracts/cgp/AxelarGateway.sol#L266-L278)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 280          function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata setupParams
    ) external override onlyGovernance 
```
 `upgrade` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L280-L300](contracts/cgp/AxelarGateway.sol#L280-L300)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 385          function deployToken(bytes calldata params, bytes32) external onlySelf 
```
 `deployToken` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L385-L419](contracts/cgp/AxelarGateway.sol#L385-L419)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 421          function mintToken(bytes calldata params, bytes32) external onlySelf 
```
 `mintToken` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L421-L425](contracts/cgp/AxelarGateway.sol#L421-L425)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 427          function burnToken(bytes calldata params, bytes32) external onlySelf 
```
 `burnToken` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L427-L453](contracts/cgp/AxelarGateway.sol#L427-L453)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 455          function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf 
```
 `approveContractCall` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L455-L467](contracts/cgp/AxelarGateway.sol#L455-L467)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 469          function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf 
```
 `approveContractCallWithMint` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L469-L493](contracts/cgp/AxelarGateway.sol#L469-L493)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 495          function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf 
```
 `transferOperatorship` is guaranteed to revert and should be marked payable
[contracts/cgp/AxelarGateway.sol#L495-L499](contracts/cgp/AxelarGateway.sol#L495-L499)


- File: contracts/cgp/auth/MultisigBase.sol
```
 
Line: 142          function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners 
```
 `rotateSigners` is guaranteed to revert and should be marked payable
[contracts/cgp/auth/MultisigBase.sol#L142-L144](contracts/cgp/auth/MultisigBase.sol#L142-L144)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 24          function transferOwnership(address newOwner) external virtual onlyOwner 
```
 `transferOwnership` is guaranteed to revert and should be marked payable
[contracts/cgp/util/Upgradable.sol#L24-L32](contracts/cgp/util/Upgradable.sol#L24-L32)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 40          function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner 
```
 `upgrade` is guaranteed to revert and should be marked payable
[contracts/cgp/util/Upgradable.sol#L40-L59](contracts/cgp/util/Upgradable.sol#L40-L59)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 52          function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner 
```
 `upgrade` is guaranteed to revert and should be marked payable
[contracts/gmp-sdk/upgradable/Upgradable.sol#L52-L71](contracts/gmp-sdk/upgradable/Upgradable.sol#L52-L71)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 78          function setup(bytes calldata data) external override onlyProxy 
```
 `setup` is guaranteed to revert and should be marked payable
[contracts/gmp-sdk/upgradable/Upgradable.sol#L78-L80](contracts/gmp-sdk/upgradable/Upgradable.sol#L78-L80)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 92          function setWhitelistedProposalCaller(
        string calldata sourceChain,
        address sourceCaller,
        bool whitelisted
    ) external override onlyOwner 
```
 `setWhitelistedProposalCaller` is guaranteed to revert and should be marked payable
[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L92-L99](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L92-L99)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 107          function setWhitelistedProposalSender(
        string calldata sourceChain,
        address sourceSender,
        bool whitelisted
    ) external override onlyOwner 
```
 `setWhitelistedProposalSender` is guaranteed to revert and should be marked payable
[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L107-L114](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L107-L114)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 534          function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator 
```
 `setFlowLimit` is guaranteed to revert and should be marked payable
[contracts/its/interchain-token-service/InterchainTokenService.sol#L534-L541](contracts/its/interchain-token-service/InterchainTokenService.sol#L534-L541)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 547          function setPaused(bool paused) external onlyOwner 
```
 `setPaused` is guaranteed to revert and should be marked payable
[contracts/its/interchain-token-service/InterchainTokenService.sol#L547-L549](contracts/its/interchain-token-service/InterchainTokenService.sol#L547-L549)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 575          function _execute(
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) internal override onlyRemoteService(sourceChain, sourceAddress) notPaused 
```
 `_execute` is guaranteed to revert and should be marked payable
[contracts/its/interchain-token-service/InterchainTokenService.sol#L575-L592](contracts/its/interchain-token-service/InterchainTokenService.sol#L575-L592)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 83          function addTrustedAddress(string memory chain, string memory addr) public onlyOwner 
```
 `addTrustedAddress` is guaranteed to revert and should be marked payable
[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83-L89](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83-L89)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 95          function removeTrustedAddress(string calldata chain) external onlyOwner 
```
 `removeTrustedAddress` is guaranteed to revert and should be marked payable
[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L95-L100](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L95-L100)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 106          function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner 
```
 `addGatewaySupportedChains` is guaranteed to revert and should be marked payable
[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L106-L113](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L106-L113)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 119          function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner 
```
 `removeGatewaySupportedChains` is guaranteed to revert and should be marked payable
[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L119-L126](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L119-L126)


- File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 49          function setup(bytes calldata params) external override onlyProxy 
```
 `setup` is guaranteed to revert and should be marked payable
[contracts/its/token-implementations/StandardizedToken.sol#L49-L68](contracts/its/token-implementations/StandardizedToken.sol#L49-L68)


- File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 76          function mint(address account, uint256 amount) external onlyDistributor 
```
 `mint` is guaranteed to revert and should be marked payable
[contracts/its/token-implementations/StandardizedToken.sol#L76-L78](contracts/its/token-implementations/StandardizedToken.sol#L76-L78)


- File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 86          function burn(address account, uint256 amount) external onlyDistributor 
```
 `burn` is guaranteed to revert and should be marked payable
[contracts/its/token-implementations/StandardizedToken.sol#L86-L88](contracts/its/token-implementations/StandardizedToken.sol#L86-L88)


- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 61          function setup(bytes calldata params) external override onlyProxy 
```
 `setup` is guaranteed to revert and should be marked payable
[contracts/its/token-manager/TokenManager.sol#L61-L75](contracts/its/token-manager/TokenManager.sol#L61-L75)


- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 161          function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256) 
```
 `giveToken` is guaranteed to revert and should be marked payable
[contracts/its/token-manager/TokenManager.sol#L161-L165](contracts/its/token-manager/TokenManager.sol#L161-L165)


- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 171          function setFlowLimit(uint256 flowLimit) external onlyOperator 
```
 `setFlowLimit` is guaranteed to revert and should be marked payable
[contracts/its/token-manager/TokenManager.sol#L171-L173](contracts/its/token-manager/TokenManager.sol#L171-L173)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 67          function setLiquidityPool(address newLiquidityPool) external onlyOperator 
```
 `setLiquidityPool` is guaranteed to revert and should be marked payable
[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L67-L69](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L67-L69)


- File: contracts/its/utils/Distributable.sol
```
 
Line: 51          function setDistributor(address distr) external onlyDistributor 
```
 `setDistributor` is guaranteed to revert and should be marked payable
[contracts/its/utils/Distributable.sol#L51-L53](contracts/its/utils/Distributable.sol#L51-L53)


- File: contracts/its/utils/Operatable.sol
```
 
Line: 51          function setOperator(address operator_) external onlyOperator 
```
 `setOperator` is guaranteed to revert and should be marked payable
[contracts/its/utils/Operatable.sol#L51-L53](contracts/its/utils/Operatable.sol#L51-L53)


</details>

# 



## Superfluous event fields
- Severity: Gas Optimization
- Confidence: High

### Description
Block.timestamp and block.number are added to event information by default so adding them manually wastes gas.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/cgp/interfaces/IInterchainGovernance.sol
```
 
Line: 20          event ProposalExecuted(bytes32 indexed proposalHash, address indexed target, bytes callData, uint256 value, uint256 indexed timestamp);
```
The event `ProposalExecuted` in the contract `IInterchainGovernance` includes the field `timestamp` which is superfluous:
[contracts/cgp/interfaces/IInterchainGovernance.sol#L20](contracts/cgp/interfaces/IInterchainGovernance.sol#L20)


</details>

# 


## Optimal State Variable Order
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 5000

### Description
Detects optimal variable order in contract storage layouts to decrease the number of storage slots used. Each storage slot used can cost at least 5,000 gas for each write operation, and potentially up to 20,000 gas if you're turning a zero value into a non-zero value. Hence, optimizing storage usage can result in significant gas savings. The real-world savings could vary depending on your contract's specific logic and the state of the Ethereum network.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- Optimization opportunity in storage variable layout of contract File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 19          abstract contract StandardizedToken is InterchainToken, ERC20Permit, Implementation, Distributable 
```

- original variable order (count: 5 slots)
    - address tokenManager
    - string name
    - string symbol
    - uint8 decimals
    - bytes32 CONTRACT_ID


 - optimized variable order (count: 4 slots)
    - address tokenManager
    - uint8 decimals
    - string name
    - string symbol
    - bytes32 CONTRACT_ID



Recomended optimizations will use 2 less slots.

[contracts/its/token-implementations/StandardizedToken.sol#L19-L89](contracts/its/token-implementations/StandardizedToken.sol#L19-L89)


</details>

# 


## Inefficient Parameter Storage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 500

### Description
When passing function parameters, using the `calldata` area instead of `memory` can improve gas efficiency. Calldata is a read-only area where function arguments and external function calls' parameters are stored.

By using `calldata` for function parameters, you avoid unnecessary gas costs associated with copying data from `calldata` to memory. This is particularly beneficial when the parameter is read-only and doesn't require modification within the contract.

Using `calldata` for function parameters can help optimize gas usage, especially when making external function calls or when the parameter values are provided externally and don't need to be stored persistently within the contract.

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
- File: contracts/cgp/auth/MultisigBase.sol
```
 
Line: 35          address[] memory accounts
```
 should be declared as `calldata` instead 

[contracts/cgp/auth/MultisigBase.sol#L35](contracts/cgp/auth/MultisigBase.sol#L35)


- File: contracts/cgp/auth/MultisigBase.sol
```
 
Line: 142          address[] memory newAccounts
```
 should be declared as `calldata` instead 

[contracts/cgp/auth/MultisigBase.sol#L142](contracts/cgp/auth/MultisigBase.sol#L142)


- File: contracts/cgp/auth/MultisigBase.sol
```
 
Line: 149          address[] memory newAccounts
```
 should be declared as `calldata` instead 

[contracts/cgp/auth/MultisigBase.sol#L149](contracts/cgp/auth/MultisigBase.sol#L149)


- File: contracts/cgp/interfaces/IMultisigBase.sol
```
 
Line: 73          address[] memory newAccounts
```
 should be declared as `calldata` instead 

[contracts/cgp/interfaces/IMultisigBase.sol#L73](contracts/cgp/interfaces/IMultisigBase.sol#L73)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 73          InterchainCalls.Call[] memory calls
```
 should be declared as `calldata` instead 

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L73](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L73)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 157          InterchainCalls.Call memory
```
 should be declared as `calldata` instead 

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L157](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L157)


- File: contracts/interchain-governance-executor/InterchainProposalSender.sol
```
 
Line: 88          InterchainCalls.InterchainCall memory interchainCall
```
 should be declared as `calldata` instead 

[contracts/interchain-governance-executor/InterchainProposalSender.sol#L88](contracts/interchain-governance-executor/InterchainProposalSender.sol#L88)


- File: contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol
```
 
Line: 15          InterchainCalls.InterchainCall[] memory calls
```
 should be declared as `calldata` instead 

[contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L15](contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L15)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 89          address[] memory tokenManagerImplementations
```
 should be declared as `calldata` instead 

[contracts/its/interchain-token-service/InterchainTokenService.sol#L89](contracts/its/interchain-token-service/InterchainTokenService.sol#L89)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 559          address[] memory implementaions
```
 should be declared as `calldata` instead 

[contracts/its/interchain-token-service/InterchainTokenService.sol#L559](contracts/its/interchain-token-service/InterchainTokenService.sol#L559)


</details>

# 


## Usage of Custom Errors for Gas Efficiency
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 276

### Description
This detector flags functions that use revert()/require() strings, which are less gas efficient than custom errors. Custom errors, available from Solidity version 0.8.4, save approximately 50 gas each time they're used by avoiding the need to allocate and store the revert string.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/its/utils/Multicall.sol
```
 
Line: 28          revert(string(result))
```
 use custom error instead 

[contracts/its/utils/Multicall.sol#L28](contracts/its/utils/Multicall.sol#L28)


</details>

# 

## Unused Named Return Variables
- Severity: Gas Optimization
- Confidence: High

### Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 223          function getImplementation(uint256 tokenManagerType) external view returns (address tokenManagerAddress) 
```
there is not use of this variables:
@ **tokenManagerAddress**

[contracts/its/interchain-token-service/InterchainTokenService.sol#L223-L233](contracts/its/interchain-token-service/InterchainTokenService.sol#L223-L233)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 827          function _getStandardizedTokenSalt(bytes32 tokenId) internal pure returns (bytes32 salt) 
```
there is not use of this variables:
@ **salt**

[contracts/its/interchain-token-service/InterchainTokenService.sol#L827-L829](contracts/its/interchain-token-service/InterchainTokenService.sol#L827-L829)


</details>

# 

## Avoid duplicate keccak256 calls
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 126

### Description
Detects instances where keccak256 is being called on the same string literal multiple times. It should be called once and the result should be saved to an immutable variable. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- interchain-token-service has a multiple call to keccak256File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 71          bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service')
```
File: contracts/its/proxies/InterchainTokenServiceProxy.sol
```
 
Line: 12          bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service')
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L71](contracts/its/interchain-token-service/InterchainTokenService.sol#L71)


- remote-address-validator has a multiple call to keccak256File: contracts/its/proxies/RemoteAddressValidatorProxy.sol
```
 
Line: 12          bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator')
```
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 21          bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator')
```

[contracts/its/proxies/RemoteAddressValidatorProxy.sol#L12](contracts/its/proxies/RemoteAddressValidatorProxy.sol#L12)


- standardized-token has a multiple call to keccak256File: contracts/its/proxies/StandardizedTokenProxy.sol
```
 
Line: 14          bytes32 private constant CONTRACT_ID = keccak256('standardized-token')
```
File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 27          bytes32 private constant CONTRACT_ID = keccak256('standardized-token')
```

[contracts/its/proxies/StandardizedTokenProxy.sol#L14](contracts/its/proxies/StandardizedTokenProxy.sol#L14)


</details>

# 


## Cache External Calls in Loops
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 100

### Description
This detector identifies instances of repeated external calls within loops. By moving the first external call outside of the loop and using low-level calls inside the loop, it is possible to save gas by avoiding repeated EXTCODESIZE opcodes, as there is no need to check again if the contract exists. 

Note: This detector only triggers if the function call does not return any value.

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 539          tokenManager.setFlowLimit(flowLimits[i])
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L539](contracts/its/interchain-token-service/InterchainTokenService.sol#L539)


</details>

# 


## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 11 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 409          tokenAddress.code.length == uint256(0)
```
Unnecessary cast: `uint256(0)` it cast to the same type.<br>
[contracts/cgp/AxelarGateway.sol#L409](contracts/cgp/AxelarGateway.sol#L409)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 446          !success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))
```
Unnecessary cast: `uint256(0)` it cast to the same type.<br>
[contracts/cgp/AxelarGateway.sol#L446](contracts/cgp/AxelarGateway.sol#L446)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 79          commandId > uint256(type(ServiceGovernanceCommand).max)
```
Unnecessary cast: `uint256(type(ServiceGovernanceCommand).max)` it cast to the same type.<br>
[contracts/cgp/governance/AxelarServiceGovernance.sol#L79](contracts/cgp/governance/AxelarServiceGovernance.sol#L79)


- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 120          commandId > uint256(type(GovernanceCommand).max)
```
Unnecessary cast: `uint256(type(GovernanceCommand).max)` it cast to the same type.<br>
[contracts/cgp/governance/InterchainGovernance.sol#L120](contracts/cgp/governance/InterchainGovernance.sol#L120)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 104          tokenManagerImplementations.length != uint256(type(TokenManagerType).max) + 1
```
Unnecessary cast: `uint256(type(TokenManagerType).max)` it cast to the same type.<br>
[contracts/its/interchain-token-service/InterchainTokenService.sol#L104](contracts/its/interchain-token-service/InterchainTokenService.sol#L104)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 85          return IERC20(token).balanceOf(liquidityPool_) - balance
```
Unnecessary cast: `IERC20(token)` it cast to the same type.<br>
[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L85](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L85)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 96          uint256 balance = IERC20(token).balanceOf(to)
```
Unnecessary cast: `IERC20(token)` it cast to the same type.<br>
[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L96](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L96)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 100          return IERC20(token).balanceOf(to) - balance
```
Unnecessary cast: `IERC20(token)` it cast to the same type.<br>
[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L100](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L100)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 51          return IERC20(token).balanceOf(address(this)) - balance
```
Unnecessary cast: `IERC20(token)` it cast to the same type.<br>
[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 62          uint256 balance = IERC20(token).balanceOf(to)
```
Unnecessary cast: `IERC20(token)` it cast to the same type.<br>
[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L62](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L62)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 66          return IERC20(token).balanceOf(to) - balance
```
Unnecessary cast: `IERC20(token)` it cast to the same type.<br>
[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L66](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L66)


</details>

# 



## External calls that could skip contract existence check
- Severity: Gas Optimization
- Confidence: High

### Description
External calls that return data should be replaced with low-level calls, to skip the unnecessary contract existence check and save gas. Saves at least 103 gas by skipping the `EXTCODESIZE` and `ISZERO` opcodes.

<details>

<summary>
There are 47 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 547          IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0))
```
 should use a low-level call

[contracts/cgp/AxelarGateway.sol#L547](contracts/cgp/AxelarGateway.sol#L547)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 287          AxelarGateway(newImplementation).contractId()
```
 should use a low-level call

[contracts/cgp/AxelarGateway.sol#L287](contracts/cgp/AxelarGateway.sol#L287)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 329          IAxelarAuth(AUTH_MODULE).validateProof(messageHash, proof)
```
 should use a low-level call

[contracts/cgp/AxelarGateway.sol#L329](contracts/cgp/AxelarGateway.sol#L329)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 443          IERC20(tokenAddress).balanceOf(address(depositHandler))
```
 should use a low-level call

[contracts/cgp/AxelarGateway.sol#L443](contracts/cgp/AxelarGateway.sol#L443)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 441          depositHandler.execute(
                tokenAddress,
                abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))
            )
```
 should use a low-level call

[contracts/cgp/AxelarGateway.sol#L441-L444](contracts/cgp/AxelarGateway.sol#L441-L444)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 45          IUpgradable(newImplementation).contractId()
```
 should use a low-level call

[contracts/cgp/util/Upgradable.sol#L45](contracts/cgp/util/Upgradable.sol#L45)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 45          IUpgradable(this).contractId()
```
 should use a low-level call

[contracts/cgp/util/Upgradable.sol#L45](contracts/cgp/util/Upgradable.sol#L45)


- File: contracts/gmp-sdk/upgradable/InitProxy.sol
```
 
Line: 50          IUpgradable(implementationAddress).contractId()
```
 should use a low-level call

[contracts/gmp-sdk/upgradable/InitProxy.sol#L50](contracts/gmp-sdk/upgradable/InitProxy.sol#L50)


- File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 33          IUpgradable(implementationAddress).contractId()
```
 should use a low-level call

[contracts/gmp-sdk/upgradable/Proxy.sol#L33](contracts/gmp-sdk/upgradable/Proxy.sol#L33)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 57          IUpgradable(this).contractId()
```
 should use a low-level call

[contracts/gmp-sdk/upgradable/Upgradable.sol#L57](contracts/gmp-sdk/upgradable/Upgradable.sol#L57)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 57          IUpgradable(newImplementation).contractId()
```
 should use a low-level call

[contracts/gmp-sdk/upgradable/Upgradable.sol#L57](contracts/gmp-sdk/upgradable/Upgradable.sol#L57)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 566          ITokenManager(implementation).implementationType()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L566](contracts/its/interchain-token-service/InterchainTokenService.sol#L566)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 102          ITokenManagerDeployer(tokenManagerDeployer_).deployer()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L102](contracts/its/interchain-token-service/InterchainTokenService.sol#L102)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 162          deployer.deployedAddress(address(this), tokenId)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L162](contracts/its/interchain-token-service/InterchainTokenService.sol#L162)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 172          ITokenManagerProxy(tokenManagerAddress).tokenId()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L172](contracts/its/interchain-token-service/InterchainTokenService.sol#L172)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 182          ITokenManager(tokenManagerAddress).tokenAddress()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L182](contracts/its/interchain-token-service/InterchainTokenService.sol#L182)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 193          deployer.deployedAddress(address(this), tokenId)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L193](contracts/its/interchain-token-service/InterchainTokenService.sol#L193)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 277          tokenManager.getFlowLimit()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L277](contracts/its/interchain-token-service/InterchainTokenService.sol#L277)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 287          tokenManager.getFlowOutAmount()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L287](contracts/its/interchain-token-service/InterchainTokenService.sol#L287)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 297          tokenManager.getFlowInAmount()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L297](contracts/its/interchain-token-service/InterchainTokenService.sol#L297)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 736          token.symbol()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L736](contracts/its/interchain-token-service/InterchainTokenService.sol#L736)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 311          gateway.tokenAddresses(tokenSymbol)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L311](contracts/its/interchain-token-service/InterchainTokenService.sol#L311)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 737          token.decimals()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L737](contracts/its/interchain-token-service/InterchainTokenService.sol#L737)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 735          token.name()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L735](contracts/its/interchain-token-service/InterchainTokenService.sol#L735)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 713          remoteAddressValidator.getRemoteAddress(destinationChain)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L713](contracts/its/interchain-token-service/InterchainTokenService.sol#L713)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 331          ITokenManager(tokenAddress).tokenAddress()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L331](contracts/its/interchain-token-service/InterchainTokenService.sol#L331)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 445          gateway.isCommandExecuted(commandId)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L445](contracts/its/interchain-token-service/InterchainTokenService.sol#L445)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 449          tokenManager.tokenAddress()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L449](contracts/its/interchain-token-service/InterchainTokenService.sol#L449)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 480          tokenManager.tokenAddress()
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L480](contracts/its/interchain-token-service/InterchainTokenService.sol#L480)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 476          gateway.isCommandExecuted(commandId)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L476](contracts/its/interchain-token-service/InterchainTokenService.sol#L476)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 610          tokenManager.giveToken(destinationAddress, amount)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L610](contracts/its/interchain-token-service/InterchainTokenService.sol#L610)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 124          remoteAddressValidator.validateSender(sourceChain, sourceAddress)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L124](contracts/its/interchain-token-service/InterchainTokenService.sol#L124)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 653          tokenManager.giveToken(expressCaller, amount)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L653](contracts/its/interchain-token-service/InterchainTokenService.sol#L653)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 613          tokenManager.giveToken(expressCaller, amount)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L613](contracts/its/interchain-token-service/InterchainTokenService.sol#L613)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 657          tokenManager.giveToken(destinationAddress, amount)
```
 should use a low-level call

[contracts/its/interchain-token-service/InterchainTokenService.sol#L657](contracts/its/interchain-token-service/InterchainTokenService.sol#L657)


- File: contracts/its/proxies/StandardizedTokenProxy.sol
```
 
Line: 22          IStandardizedToken(implementationAddress).contractId()
```
 should use a low-level call

[contracts/its/proxies/StandardizedTokenProxy.sol#L22](contracts/its/proxies/StandardizedTokenProxy.sol#L22)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 59          interchainTokenServiceAddress_.getImplementation(implementationType_)
```
 should use a low-level call

[contracts/its/proxies/TokenManagerProxy.sol#L59](contracts/its/proxies/TokenManagerProxy.sol#L59)


- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 205          ITokenManagerProxy(address(this)).tokenId()
```
 should use a low-level call

[contracts/its/token-manager/TokenManager.sol#L205](contracts/its/token-manager/TokenManager.sol#L205)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 80          token.balanceOf(liquidityPool_)
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L80](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L80)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 85          IERC20(token).balanceOf(liquidityPool_)
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L85](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L85)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 96          IERC20(token).balanceOf(to)
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L96](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L96)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 100          IERC20(token).balanceOf(to)
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L100](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L100)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 46          token.balanceOf(address(this))
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L46](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L46)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 51          IERC20(token).balanceOf(address(this))
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 62          IERC20(token).balanceOf(to)
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L62](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L62)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 66          IERC20(token).balanceOf(to)
```
 should use a low-level call

[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L66](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L66)


- File: contracts/its/utils/TokenManagerDeployer.sol
```
 
Line: 40          deployer.deploy(bytecode, tokenId)
```
 should use a low-level call

[contracts/its/utils/TokenManagerDeployer.sol#L40](contracts/its/utils/TokenManagerDeployer.sol#L40)


</details>

# 