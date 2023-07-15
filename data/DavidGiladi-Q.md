
### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Array out of bounds accesses | [Array out of bounds accesses](#array-out-of-bounds-accesses) | 1 |
|[L-2] Missing contract-existence checks before low-level calls | [Missing contract-existence checks before low-level calls](#missing-contract-existence-checks-before-low-level-calls) | 16 |
|[L-3] Local variable shadowing | [Local variable shadowing](#local-variable-shadowing) | 12 |
|[L-4] Contracts that lock Ether | [Contracts that lock Ether](#contracts-that-lock-ether) | 1 |
|[L-5] Missing gap Storage Variable in Upgradeable Contract | [Missing gap Storage Variable in Upgradeable Contract](#missing-gap-storage-variable-in-upgradeable-contract) | 4 |
|[L-6] Unbounded gas for external call | [Unbounded gas for external call](#unbounded-gas-for-external-call) | 17 |
|[L-7] Avoid Usage of Ownable and Prefer Ownable2Step | [Avoid Usage of Ownable and Prefer Ownable2Step](#avoid-usage-of-ownable-and-prefer-ownable2step) | 3 |
|[L-8] Use 'abi.encodeCall()' Instead of 'abi.encodeSignature()'/abi.encodeSelector() | [Use 'abi.encodeCall()' Instead of 'abi.encodeSignature()'/abi.encodeSelector()](#use-abiencodecall-instead-of-abiencodesignatureabiencodeselector) | 16 |
Total: 8 issues


### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Cast to bytes or bytes32 for clearer semantic meaning | [Cast to bytes or bytes32 for clearer semantic meaning](#cast-to-bytes-or-bytes32-for-clearer-semantic-meaning) | 4 |
|[N-2] Control structures do not follow the Solidity Style Guide | [Control structures do not follow the Solidity Style Guide](#control-structures-do-not-follow-the-solidity-style-guide) | 36 |
|[N-3] Custom error has no error details | [Custom error has no error details](#custom-error-has-no-error-details) | 67 |
|[N-4] Cyclomatic complexity | [Cyclomatic complexity](#cyclomatic-complexity) | 1 |
|[N-5] Dead-code | [Dead-code](#dead-code) | 5 |
|[N-6] Token contract should have a blacklist function or modifier | [Token contract should have a blacklist function or modifier](#token-contract-should-have-a-blacklist-function-or-modifier) | 2 |
|[N-7] Missing inheritance | [Missing inheritance](#missing-inheritance) | 2 |
|[N-8] Name reused | [Name reused](#name-reused) | 2 |
|[N-9] Consider Disable Ownership Renouncement in Ownable Contracts | [Consider Disable Ownership Renouncement in Ownable Contracts](#consider-disable-ownership-renouncement-in-ownable-contracts) | 4 |
|[N-10] Event Emission Preceding External Calls: A Best Practice | [Event Emission Preceding External Calls: A Best Practice](#event-emission-preceding-external-calls-a-best-practice) | 28 |
|[N-11] Variable names too similar | [Variable names too similar](#variable-names-too-similar) | 2 |
|[N-12] Too many digits | [Too many digits](#too-many-digits) | 3 |
|[N-13] Unused Custom Error definition | [Unused Custom Error definition](#unused-custom-error-definition) | 7 |
|[N-14] Unused imports | [Unused imports](#unused-imports) | 12 |
|[N-15] Contracts containing only utility functions should be made into libraries | [Contracts containing only utility functions should be made into libraries](#contracts-containing-only-utility-functions-should-be-made-into-libraries) | 3 |

Total: 15 issues

## Array out of bounds accesses
- Severity: Low
- Confidence: Medium

### Description
If the lengths of arrays are not checked before they're accessed, user operations may not be executed properly due to the potential issue of accessing an array out of its bounds. In Solidity, accessing an array beyond its boundaries can cause a runtime error known as an out-of-bounds exception. This error arises when attempting to read or write to an element of an array that does not exist. Therefore, it is critical to avoid out-of-bounds exceptions while accessing arrays in Solidity code to prevent unexpected halts and possible loss of funds.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 559          address[] memory implementaions
```
 is accessed on File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 564          implementation = implementaions[uint256(tokenManagerType)]
```
 index that might be out-of-bounds

[contracts/its/interchain-token-service/InterchainTokenService.sol#L559](contracts/its/interchain-token-service/InterchainTokenService.sol#L559)


</details>

# 



## Missing contract-existence checks before low-level calls
- Severity: Low
- Confidence: High

### Description
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0` or use the `extcodesize` assembly operation to verify the presence of contract code at the specified address. Both these methods ensure the existence of a contract before making a low-level call.

<details>

<summary>
There are 16 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 294          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(IAxelarGateway.setup.selector, setupParams))
```

[contracts/cgp/AxelarGateway.sol#L294](contracts/cgp/AxelarGateway.sol#L294)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 374          (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId))
```

[contracts/cgp/AxelarGateway.sol#L374](contracts/cgp/AxelarGateway.sol#L374)


- File: contracts/cgp/util/Caller.sol
```
 
Line: 18          (bool success, ) = target.call{ value: nativeValue }(callData)
```

[contracts/cgp/util/Caller.sol#L18](contracts/cgp/util/Caller.sol#L18)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 49          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params))
```

[contracts/cgp/util/Upgradable.sol#L49](contracts/cgp/util/Upgradable.sol#L49)


- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 50          (bool success, ) = deployedAddress_.call(init)
```

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 57          (bool success, ) = deployedAddress_.call(init)
```

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L57](contracts/gmp-sdk/deploy/Create3Deployer.sol#L57)


- File: contracts/gmp-sdk/upgradable/FinalProxy.sol
```
 
Line: 81          (bool success, ) = finalImplementation_.delegatecall(abi.encodeWithSelector(BaseProxy.setup.selector, setupParams))
```

[contracts/gmp-sdk/upgradable/FinalProxy.sol#L81](contracts/gmp-sdk/upgradable/FinalProxy.sol#L81)


- File: contracts/gmp-sdk/upgradable/InitProxy.sol
```
 
Line: 58          (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, params))
```

[contracts/gmp-sdk/upgradable/InitProxy.sol#L58](contracts/gmp-sdk/upgradable/InitProxy.sol#L58)


- File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 41          (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, setupParams))
```

[contracts/gmp-sdk/upgradable/Proxy.sol#L41](contracts/gmp-sdk/upgradable/Proxy.sol#L41)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 61          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params))
```

[contracts/gmp-sdk/upgradable/Upgradable.sol#L61](contracts/gmp-sdk/upgradable/Upgradable.sol#L61)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 76          (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData)
```

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L813-L815](contracts/its/interchain-token-service/InterchainTokenService.sol#L813-L815)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 853          (bool success, ) = standardizedTokenDeployer.delegatecall(
            abi.encodeWithSelector(
                IStandardizedTokenDeployer.deployStandardizedToken.selector,
                salt,
                tokenManagerAddress,
                distributor,
                name,
                symbol,
                decimals,
                mintAmount,
                mintTo
            )
        )
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L853-L865](contracts/its/interchain-token-service/InterchainTokenService.sol#L853-L865)


- File: contracts/its/proxies/StandardizedTokenProxy.sol
```
 
Line: 24          (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IStandardizedToken.setup.selector, params))
```

[contracts/its/proxies/StandardizedTokenProxy.sol#L24](contracts/its/proxies/StandardizedTokenProxy.sol#L24)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 36          (bool success, ) = impl.delegatecall(abi.encodeWithSelector(TokenManagerProxy.setup.selector, params))
```

[contracts/its/proxies/TokenManagerProxy.sol#L36](contracts/its/proxies/TokenManagerProxy.sol#L36)


- File: contracts/its/utils/Multicall.sol
```
 
Line: 25          (bool success, bytes memory result) = address(this).delegatecall(data[i])
```

[contracts/its/utils/Multicall.sol#L25](contracts/its/utils/Multicall.sol#L25)


</details>

# 

## Contracts that lock Ether
- Severity: Low
- Confidence: High

### Description
Contract with a `payable` function, but without a withdrawal capacity.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- Contract locking ether found:
	Contract File: contracts/its/utils/TokenManagerDeployer.sol
```
 
Line: 15          contract TokenManagerDeployer is ITokenManagerDeployer 
```
 has payable functions:
	 - File: contracts/its/interfaces/ITokenManagerDeployer.sol
```
 
Line: 26          function deployTokenManager(
        bytes32 tokenId,
        uint256 implementationType,
        bytes calldata params
    ) external payable;
```

	 - File: contracts/its/utils/TokenManagerDeployer.sol
```
 
Line: 33          function deployTokenManager(
        bytes32 tokenId,
        uint256 implementationType,
        bytes calldata params
    ) external payable 
```

	But does not have a function to withdraw the ether

[contracts/its/utils/TokenManagerDeployer.sol#L15-L43](contracts/its/utils/TokenManagerDeployer.sol#L15-L43)


</details>

# 


## Local variable shadowing
- Severity: Low
- Confidence: High

### Description
Detection of shadowing using local variables.

<details>

<summary>
There are 12 instances of this issue:

</summary>

###
- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 35          string memory governanceChain
```
 shadows:
	- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 21          string public governanceChain
```
 (state variable)
	- File: contracts/cgp/interfaces/IInterchainGovernance.sol
```
 
Line: 26          function governanceChain() external view returns (string memory);
```
 (function)

[contracts/cgp/governance/AxelarServiceGovernance.sol#L35](contracts/cgp/governance/AxelarServiceGovernance.sol#L35)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 36          string memory governanceAddress
```
 shadows:
	- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 22          string public governanceAddress
```
 (state variable)
	- File: contracts/cgp/interfaces/IInterchainGovernance.sol
```
 
Line: 32          function governanceAddress() external view returns (string memory);
```
 (function)

[contracts/cgp/governance/AxelarServiceGovernance.sol#L36](contracts/cgp/governance/AxelarServiceGovernance.sol#L36)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 241          bytes memory operator
```
 shadows:
	- File: contracts/its/utils/Operatable.sol
```
 
Line: 29          function operator() public view returns (address operator_) 
```
 (function)
	- File: contracts/its/interfaces/IOperatable.sol
```
 
Line: 14          function operator() external view returns (address operator_);
```
 (function)

[contracts/its/interchain-token-service/InterchainTokenService.sol#L241](contracts/its/interchain-token-service/InterchainTokenService.sol#L241)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 251          bytes memory operator
```
 shadows:
	- File: contracts/its/utils/Operatable.sol
```
 
Line: 29          function operator() public view returns (address operator_) 
```
 (function)
	- File: contracts/its/interfaces/IOperatable.sol
```
 
Line: 14          function operator() external view returns (address operator_);
```
 (function)

[contracts/its/interchain-token-service/InterchainTokenService.sol#L251](contracts/its/interchain-token-service/InterchainTokenService.sol#L251)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 263          bytes memory operator
```
 shadows:
	- File: contracts/its/utils/Operatable.sol
```
 
Line: 29          function operator() public view returns (address operator_) 
```
 (function)
	- File: contracts/its/interfaces/IOperatable.sol
```
 
Line: 14          function operator() external view returns (address operator_);
```
 (function)

[contracts/its/interchain-token-service/InterchainTokenService.sol#L263](contracts/its/interchain-token-service/InterchainTokenService.sol#L263)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 423          bytes memory operator
```
 shadows:
	- File: contracts/its/utils/Operatable.sol
```
 
Line: 29          function operator() public view returns (address operator_) 
```
 (function)
	- File: contracts/its/interfaces/IOperatable.sol
```
 
Line: 14          function operator() external view returns (address operator_);
```
 (function)

[contracts/its/interchain-token-service/InterchainTokenService.sol#L423](contracts/its/interchain-token-service/InterchainTokenService.sol#L423)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 562          address implementation
```
 shadows:
	- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 39          function implementation() public view returns (address implementation_) 
```
 (function)
	- File: contracts/gmp-sdk/interfaces/IUpgradable.sol
```
 
Line: 16          function implementation() external view returns (address);
```
 (function)

[contracts/its/interchain-token-service/InterchainTokenService.sol#L562](contracts/its/interchain-token-service/InterchainTokenService.sol#L562)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 776          bytes memory operator
```
 shadows:
	- File: contracts/its/utils/Operatable.sol
```
 
Line: 29          function operator() public view returns (address operator_) 
```
 (function)
	- File: contracts/its/interfaces/IOperatable.sol
```
 
Line: 14          function operator() external view returns (address operator_);
```
 (function)

[contracts/its/interchain-token-service/InterchainTokenService.sol#L776](contracts/its/interchain-token-service/InterchainTokenService.sol#L776)


- File: contracts/its/interfaces/IDistributable.sol
```
 
Line: 14          address distributor
```
 shadows:
	- File: contracts/its/interfaces/IDistributable.sol
```
 
Line: 14          function distributor() external view returns (address distributor);
```
 (function)

[contracts/its/interfaces/IDistributable.sol#L14](contracts/its/interfaces/IDistributable.sol#L14)


- File: contracts/its/interfaces/IDistributable.sol
```
 
Line: 21          address distributor
```
 shadows:
	- File: contracts/its/interfaces/IDistributable.sol
```
 
Line: 14          function distributor() external view returns (address distributor);
```
 (function)

[contracts/its/interfaces/IDistributable.sol#L21](contracts/its/interfaces/IDistributable.sol#L21)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 34          address tokenAddress
```
 shadows:
	- File: contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol
```
 
Line: 28          function tokenAddress() public view override returns (address tokenAddress_) 
```
 (function)
	- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 53          function tokenAddress() public view virtual returns (address);
```
 (function)
	- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 26          function tokenAddress() external view returns (address);
```
 (function)

[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L34](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L34)


- File: contracts/its/token-manager/implementations/TokenManagerMintBurn.sol
```
 
Line: 35          address tokenAddress
```
 shadows:
	- File: contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol
```
 
Line: 28          function tokenAddress() public view override returns (address tokenAddress_) 
```
 (function)
	- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 53          function tokenAddress() public view virtual returns (address);
```
 (function)
	- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 26          function tokenAddress() external view returns (address);
```
 (function)

[contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L35](contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L35)


</details>

# 



## Unbounded gas for external call
- Severity: Low
- Confidence: High

### Description
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}()` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

<details>

<summary>
There are 17 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 294          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(IAxelarGateway.setup.selector, setupParams))
```

[contracts/cgp/AxelarGateway.sol#L294](contracts/cgp/AxelarGateway.sol#L294)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 374          (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId))
```

[contracts/cgp/AxelarGateway.sol#L374](contracts/cgp/AxelarGateway.sol#L374)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 398          (bool success, bytes memory data) = TOKEN_DEPLOYER_IMPLEMENTATION.delegatecall(
                abi.encodeWithSelector(ITokenDeployer.deployToken.selector, name, symbol, decimals, cap, salt)
            )
```

[contracts/cgp/AxelarGateway.sol#L398-L400](contracts/cgp/AxelarGateway.sol#L398-L400)


- File: contracts/cgp/util/Caller.sol
```
 
Line: 18          (bool success, ) = target.call{ value: nativeValue }(callData)
```

[contracts/cgp/util/Caller.sol#L18](contracts/cgp/util/Caller.sol#L18)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 49          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params))
```

[contracts/cgp/util/Upgradable.sol#L49](contracts/cgp/util/Upgradable.sol#L49)


- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 50          (bool success, ) = deployedAddress_.call(init)
```

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 57          (bool success, ) = deployedAddress_.call(init)
```

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L57](contracts/gmp-sdk/deploy/Create3Deployer.sol#L57)


- File: contracts/gmp-sdk/upgradable/FinalProxy.sol
```
 
Line: 81          (bool success, ) = finalImplementation_.delegatecall(abi.encodeWithSelector(BaseProxy.setup.selector, setupParams))
```

[contracts/gmp-sdk/upgradable/FinalProxy.sol#L81](contracts/gmp-sdk/upgradable/FinalProxy.sol#L81)


- File: contracts/gmp-sdk/upgradable/InitProxy.sol
```
 
Line: 58          (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, params))
```

[contracts/gmp-sdk/upgradable/InitProxy.sol#L58](contracts/gmp-sdk/upgradable/InitProxy.sol#L58)


- File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 41          (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, setupParams))
```

[contracts/gmp-sdk/upgradable/Proxy.sol#L41](contracts/gmp-sdk/upgradable/Proxy.sol#L41)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 61          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params))
```

[contracts/gmp-sdk/upgradable/Upgradable.sol#L61](contracts/gmp-sdk/upgradable/Upgradable.sol#L61)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 76          (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData)
```

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L813-L815](contracts/its/interchain-token-service/InterchainTokenService.sol#L813-L815)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 853          (bool success, ) = standardizedTokenDeployer.delegatecall(
            abi.encodeWithSelector(
                IStandardizedTokenDeployer.deployStandardizedToken.selector,
                salt,
                tokenManagerAddress,
                distributor,
                name,
                symbol,
                decimals,
                mintAmount,
                mintTo
            )
        )
```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L853-L865](contracts/its/interchain-token-service/InterchainTokenService.sol#L853-L865)


- File: contracts/its/proxies/StandardizedTokenProxy.sol
```
 
Line: 24          (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IStandardizedToken.setup.selector, params))
```

[contracts/its/proxies/StandardizedTokenProxy.sol#L24](contracts/its/proxies/StandardizedTokenProxy.sol#L24)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 36          (bool success, ) = impl.delegatecall(abi.encodeWithSelector(TokenManagerProxy.setup.selector, params))
```

[contracts/its/proxies/TokenManagerProxy.sol#L36](contracts/its/proxies/TokenManagerProxy.sol#L36)


- File: contracts/its/utils/Multicall.sol
```
 
Line: 25          (bool success, bytes memory result) = address(this).delegatecall(data[i])
```

[contracts/its/utils/Multicall.sol#L25](contracts/its/utils/Multicall.sol#L25)


</details>

# 


## Use 'abi.encodeCall()' Instead of 'abi.encodeSignature()'/abi.encodeSelector()
- Severity: Low
- Confidence: High

### Description
abi.encodeCall() has compiler type safety, whereas the other two functions do not.

<details>

<summary>
There are 16 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 294          abi.encodeWithSelector(IAxelarGateway.setup.selector, setupParams)
```
 use abi.encodeCall()

[contracts/cgp/AxelarGateway.sol#L294](contracts/cgp/AxelarGateway.sol#L294)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 374          abi.encodeWithSelector(commandSelector, params[i], commandId)
```
 use abi.encodeCall()

[contracts/cgp/AxelarGateway.sol#L374](contracts/cgp/AxelarGateway.sol#L374)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 399          abi.encodeWithSelector(ITokenDeployer.deployToken.selector, name, symbol, decimals, cap, salt)
```
 use abi.encodeCall()

[contracts/cgp/AxelarGateway.sol#L399](contracts/cgp/AxelarGateway.sol#L399)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 443          abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))
```
 use abi.encodeCall()

[contracts/cgp/AxelarGateway.sol#L443](contracts/cgp/AxelarGateway.sol#L443)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 545          abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount)
```
 use abi.encodeCall()

[contracts/cgp/AxelarGateway.sol#L545](contracts/cgp/AxelarGateway.sol#L545)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 49          abi.encodeWithSelector(this.setup.selector, params)
```
 use abi.encodeCall()

[contracts/cgp/util/Upgradable.sol#L49](contracts/cgp/util/Upgradable.sol#L49)


- File: contracts/gmp-sdk/upgradable/FinalProxy.sol
```
 
Line: 81          abi.encodeWithSelector(BaseProxy.setup.selector, setupParams)
```
 use abi.encodeCall()

[contracts/gmp-sdk/upgradable/FinalProxy.sol#L81](contracts/gmp-sdk/upgradable/FinalProxy.sol#L81)


- File: contracts/gmp-sdk/upgradable/InitProxy.sol
```
 
Line: 58          abi.encodeWithSelector(IUpgradable.setup.selector, params)
```
 use abi.encodeCall()

[contracts/gmp-sdk/upgradable/InitProxy.sol#L58](contracts/gmp-sdk/upgradable/InitProxy.sol#L58)


- File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 41          abi.encodeWithSelector(IUpgradable.setup.selector, setupParams)
```
 use abi.encodeCall()

[contracts/gmp-sdk/upgradable/Proxy.sol#L41](contracts/gmp-sdk/upgradable/Proxy.sol#L41)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 61          abi.encodeWithSelector(this.setup.selector, params)
```
 use abi.encodeCall()

[contracts/gmp-sdk/upgradable/Upgradable.sol#L61](contracts/gmp-sdk/upgradable/Upgradable.sol#L61)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 814          abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
```
 use abi.encodeCall()

[contracts/its/interchain-token-service/InterchainTokenService.sol#L814](contracts/its/interchain-token-service/InterchainTokenService.sol#L814)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 854          abi.encodeWithSelector(
                IStandardizedTokenDeployer.deployStandardizedToken.selector,
                salt,
                tokenManagerAddress,
                distributor,
                name,
                symbol,
                decimals,
                mintAmount,
                mintTo
            )
```
 use abi.encodeCall()

[contracts/its/interchain-token-service/InterchainTokenService.sol#L854-L864](contracts/its/interchain-token-service/InterchainTokenService.sol#L854-L864)


- File: contracts/its/proxies/StandardizedTokenProxy.sol
```
 
Line: 24          abi.encodeWithSelector(IStandardizedToken.setup.selector, params)
```
 use abi.encodeCall()

[contracts/its/proxies/StandardizedTokenProxy.sol#L24](contracts/its/proxies/StandardizedTokenProxy.sol#L24)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 36          abi.encodeWithSelector(TokenManagerProxy.setup.selector, params)
```
 use abi.encodeCall()

[contracts/its/proxies/TokenManagerProxy.sol#L36](contracts/its/proxies/TokenManagerProxy.sol#L36)


- File: contracts/its/token-manager/implementations/TokenManagerMintBurn.sol
```
 
Line: 48          abi.encodeWithSelector(IERC20BurnableMintable.burn.selector, from, amount)
```
 use abi.encodeCall()

[contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L48](contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L48)


- File: contracts/its/token-manager/implementations/TokenManagerMintBurn.sol
```
 
Line: 62          abi.encodeWithSelector(IERC20BurnableMintable.mint.selector, to, amount)
```
 use abi.encodeCall()

[contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L62](contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L62)


</details>

# 


## Missing gap Storage Variable in Upgradeable Contract
- Severity: Low
- Confidence: High

### Note 
I reported only on contracts that was missing in the wining bot 

### Description
upgradeable contracts that are missing a '__gap' storage variable. In upgradeable contracts, it is important to reserve storage slots for future versions to introduce new storage variables. The '__gap' storage variable acts as a placeholder, allowing for seamless upgrades without affecting existing storage layout.When a contract is not designed with a '__gap' storage variable, adding new storage variables in subsequent versions becomes problematic. It can lead to storage collisions or layout incompatibilities, making it difficult to upgrade the contract without requiring costly data migrations or redeployments.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 6          interface IUpgradable 
```
 missing __gap storage variable

[contracts/cgp/interfaces/IUpgradable.sol#L6-L31](contracts/cgp/interfaces/IUpgradable.sol#L6-L31)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 7          abstract contract Upgradable is IUpgradable 
```
 missing __gap storage variable

[contracts/cgp/util/Upgradable.sol#L7-L69](contracts/cgp/util/Upgradable.sol#L7-L69)


- File: contracts/gmp-sdk/interfaces/IUpgradable.sol
```
 
Line: 8          interface IUpgradable is IOwnable 
```
 missing __gap storage variable

[contracts/gmp-sdk/interfaces/IUpgradable.sol#L8-L27](contracts/gmp-sdk/interfaces/IUpgradable.sol#L8-L27)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 12          abstract contract Upgradable is Ownable, IUpgradable 
```
 missing __gap storage variable

[contracts/gmp-sdk/upgradable/Upgradable.sol#L12-L88](contracts/gmp-sdk/upgradable/Upgradable.sol#L12-L88)



</details>

# 


## Avoid Usage of Ownable and Prefer Ownable2Step
- Severity: Low
- Confidence: High


### Note 
I reported only on contracts that was missing in the wining bot 

### Description
The 'Ownable'/'OwnableUpgradeable' contracts provides a basic ownership mechanism, but it lacks an additional step for secure ownership transfer, which is crucial for certain scenarios, such as upgrading contracts or handling complex ownership transitions.

By utilizing the 'Ownable2Step'/'Ownable2StepUpgradeable' contract, which extends the functionality of 'Ownable'/'OwnableUpgradeable' with a two-step ownership transfer process, developers can enhance the security and flexibility of their contracts. The two-step process involves the current owner initiating a transfer request, which requires confirmation from the new owner before the ownership is transferred.

Using 'Ownable2Step'/'Ownable2StepUpgradeable' mitigates risks associated with accidental or unauthorized ownership transfers, ensuring that ownership transitions are intentional and verified. It provides an extra layer of protection, particularly in situations where contract upgrades or complex ownership management are involved.

By leveraging 'Ownable2Step'/'Ownable2StepUpgradeable', developers can enhance the security and integrity of their contracts, promoting trust and reducing the potential for ownership-related vulnerabilities.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 12          abstract contract Upgradable is Ownable, IUpgradable 
```

[contracts/gmp-sdk/upgradable/Upgradable.sol#L12-L88](contracts/gmp-sdk/upgradable/Upgradable.sol#L12-L88)



- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 37          contract InterchainTokenService is
    IInterchainTokenService,
    AxelarExecutable,
    Upgradable,
    Operatable,
    ExpressCallHandler,
    Pausable,
    Multicall

```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L37-L896](contracts/its/interchain-token-service/InterchainTokenService.sol#L37-L896)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 12          contract RemoteAddressValidator is IRemoteAddressValidator, Upgradable 
```

[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L12-L139](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L12-L139)


</details>

# 



## Cast to bytes or bytes32 for clearer semantic meaning
- Severity: Non-Critical
- Confidence: High

### Description
Using a cast on a single argument, rather than abi.encodePacked() makes the intended operation more clear, leading to less reviewer confusion.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 350          abi.encodePacked(commands[i])
```

[contracts/cgp/AxelarGateway.sol#L350](contracts/cgp/AxelarGateway.sol#L350)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 396          abi.encodePacked(symbol)
```

[contracts/cgp/AxelarGateway.sol#L396](contracts/cgp/AxelarGateway.sol#L396)


- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 435          abi.encodePacked(type(DepositHandler).creationCode)
```

[contracts/cgp/AxelarGateway.sol#L435](contracts/cgp/AxelarGateway.sol#L435)


- File: contracts/its/proxies/InterchainTokenServiceProxy.sol
```
 
Line: 23          abi.encodePacked(operator)
```

[contracts/its/proxies/InterchainTokenServiceProxy.sol#L23](contracts/its/proxies/InterchainTokenServiceProxy.sol#L23)


</details>

# 



## Control structures do not follow the Solidity Style Guide
- Severity: Non-Critical
- Confidence: High

### Description
Control structures should follow the One True Brace [OTB style](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures). This means: 1. Opening braces should be on the same line as the declaration. 
2. Closing braces should be on their own line, at the same indentation level as the beginning of the declaration. 
3. The opening brace should be preceded by a single space.

<details>

<summary>
There are 36 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 427          function burnToken(bytes calldata params, bytes32) external onlySelf 
```
 These control structures in the function `burnToken` should follow the One True Brace (OTB) style:

    - Line: 439:             DepositHandler depositHandler = new DepositHandler{ salt: salt }();

[contracts/cgp/AxelarGateway.sol#L427-L453](contracts/cgp/AxelarGateway.sol#L427-L453)


- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 33          constructor(
        address gateway,
        string memory governanceChain,
        string memory governanceAddress,
        uint256 minimumTimeDelay,
        address[] memory cosigners,
        uint256 threshold
    ) InterchainGovernance(gateway, governanceChain, governanceAddress, minimumTimeDelay) MultisigBase(cosigners, threshold) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 40:     ) InterchainGovernance(gateway, governanceChain, governanceAddress, minimumTimeDelay) MultisigBase(cosigners, threshold) {}

[contracts/cgp/governance/AxelarServiceGovernance.sol#L33-L40](contracts/cgp/governance/AxelarServiceGovernance.sol#L33-L40)


- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 168          receive() external payable 
```
 These control structures in the function `receive` should follow the One True Brace (OTB) style:

    - Line: 168: receive() external payable {}

[contracts/cgp/governance/InterchainGovernance.sol#L168](contracts/cgp/governance/InterchainGovernance.sol#L168)


- File: contracts/cgp/governance/Multisig.sol
```
 
Line: 20          constructor(address[] memory accounts, uint256 threshold) MultisigBase(accounts, threshold) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 20: constructor(address[] memory accounts, uint256 threshold) MultisigBase(accounts, threshold) {}

[contracts/cgp/governance/Multisig.sol#L20](contracts/cgp/governance/Multisig.sol#L20)


- File: contracts/cgp/governance/Multisig.sol
```
 
Line: 41          receive() external payable 
```
 These control structures in the function `receive` should follow the One True Brace (OTB) style:

    - Line: 41: receive() external payable {}

[contracts/cgp/governance/Multisig.sol#L41](contracts/cgp/governance/Multisig.sol#L41)


- File: contracts/cgp/util/Caller.sol
```
 
Line: 11          function _call(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) internal 
```
 These control structures in the function `_call` should follow the One True Brace (OTB) style:

    - Line: 18:         (bool success, ) = target.call{ value: nativeValue }(callData);

[contracts/cgp/util/Caller.sol#L11-L22](contracts/cgp/util/Caller.sol#L11-L22)


- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 68          function _setup(bytes calldata data) internal virtual 
```
 These control structures in the function `_setup` should follow the One True Brace (OTB) style:

    - Line: 68: function _setup(bytes calldata data) internal virtual {}

[contracts/cgp/util/Upgradable.sol#L68](contracts/cgp/util/Upgradable.sol#L68)


- File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 51          function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) 
```
 These control structures in the function `deploy` should follow the One True Brace (OTB) style:

    - Line: 58:         CreateDeployer deployer = new CreateDeployer{ salt: salt }();

[contracts/gmp-sdk/deploy/Create3.sol#L51-L63](contracts/gmp-sdk/deploy/Create3.sol#L51-L63)


- File: contracts/gmp-sdk/upgradable/BaseProxy.sol
```
 
Line: 32          function setup(bytes calldata params) external 
```
 These control structures in the function `setup` should follow the One True Brace (OTB) style:

    - Line: 32: function setup(bytes calldata params) external {}

[contracts/gmp-sdk/upgradable/BaseProxy.sol#L32](contracts/gmp-sdk/upgradable/BaseProxy.sol#L32)


- File: contracts/gmp-sdk/upgradable/BaseProxy.sol
```
 
Line: 67          receive() external payable virtual 
```
 These control structures in the function `receive` should follow the One True Brace (OTB) style:

    - Line: 67: receive() external payable virtual {}

[contracts/gmp-sdk/upgradable/BaseProxy.sol#L67](contracts/gmp-sdk/upgradable/BaseProxy.sol#L67)


- File: contracts/gmp-sdk/upgradable/FinalProxy.sol
```
 
Line: 26          constructor(
        address implementationAddress,
        address owner,
        bytes memory setupParams
    ) Proxy(implementationAddress, owner, setupParams) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 30:     ) Proxy(implementationAddress, owner, setupParams) {}

[contracts/gmp-sdk/upgradable/FinalProxy.sol#L26-L30](contracts/gmp-sdk/upgradable/FinalProxy.sol#L26-L30)


- File: contracts/gmp-sdk/upgradable/FixedProxy.sol
```
 
Line: 31          function setup(bytes calldata setupParams) external 
```
 These control structures in the function `setup` should follow the One True Brace (OTB) style:

    - Line: 31: function setup(bytes calldata setupParams) external {}

[contracts/gmp-sdk/upgradable/FixedProxy.sol#L31](contracts/gmp-sdk/upgradable/FixedProxy.sol#L31)


- File: contracts/gmp-sdk/upgradable/FixedProxy.sol
```
 
Line: 59          receive() external payable virtual 
```
 These control structures in the function `receive` should follow the One True Brace (OTB) style:

    - Line: 59: receive() external payable virtual {}

[contracts/gmp-sdk/upgradable/FixedProxy.sol#L59](contracts/gmp-sdk/upgradable/FixedProxy.sol#L59)


- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 87          function _setup(bytes calldata data) internal virtual 
```
 These control structures in the function `_setup` should follow the One True Brace (OTB) style:

    - Line: 87: function _setup(bytes calldata data) internal virtual {}

[contracts/gmp-sdk/upgradable/Upgradable.sol#L87](contracts/gmp-sdk/upgradable/Upgradable.sol#L87)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 73          function _executeProposal(InterchainCalls.Call[] memory calls) internal 
```
 These control structures in the function `_executeProposal` should follow the One True Brace (OTB) style:

    - Line: 76:             (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L73-L84](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L73-L84)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 156          function _onTargetExecutionFailed(
        InterchainCalls.Call memory, /* call */
        bytes memory result
    ) internal virtual 
```
 These control structures in the function `_onTargetExecutionFailed` should follow the One True Brace (OTB) style:

    - Line: 165:             }

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L156-L170](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L156-L170)


- File: contracts/interchain-governance-executor/InterchainProposalSender.sol
```
 
Line: 88          function _sendProposal(InterchainCalls.InterchainCall memory interchainCall) internal 
```
 These control structures in the function `_sendProposal` should follow the One True Brace (OTB) style:

    - Line: 92:             gasService.payNativeGasForContractCall{ value: interchainCall.gas }(

[contracts/interchain-governance-executor/InterchainProposalSender.sol#L88-L102](contracts/interchain-governance-executor/InterchainProposalSender.sol#L88-L102)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 559          function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)
        internal
        pure
        returns (address implementation)
    
```
 These control structures in the function `_sanitizeTokenManagerImplementation` should follow the One True Brace (OTB) style:

    - Line: 563:     {

[contracts/its/interchain-token-service/InterchainTokenService.sol#L559-L567](contracts/its/interchain-token-service/InterchainTokenService.sol#L559-L567)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 622          function _processSendTokenWithDataPayload(string calldata sourceChain, bytes calldata payload) internal 
```
 These control structures in the function `_processSendTokenWithDataPayload` should follow the One True Brace (OTB) style:

    - Line: 633:         {
    - Line: 642:         {

[contracts/its/interchain-token-service/InterchainTokenService.sol#L622-L660](contracts/its/interchain-token-service/InterchainTokenService.sol#L622-L660)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 707          function _callContract(
        string calldata destinationChain,
        bytes memory payload,
        uint256 gasValue,
        address refundTo
    ) internal 
```
 These control structures in the function `_callContract` should follow the One True Brace (OTB) style:

    - Line: 715:             gasService.payNativeGasForContractCall{ value: gasValue }(

[contracts/its/interchain-token-service/InterchainTokenService.sol#L707-L724](contracts/its/interchain-token-service/InterchainTokenService.sol#L707-L724)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 726          function _validateToken(address tokenAddress)
        internal
        returns (
            string memory name,
            string memory symbol,
            uint8 decimals
        )
    
```
 These control structures in the function `_validateToken` should follow the One True Brace (OTB) style:

    - Line: 733:     {

[contracts/its/interchain-token-service/InterchainTokenService.sol#L726-L738](contracts/its/interchain-token-service/InterchainTokenService.sol#L726-L738)


- File: contracts/its/interchain-token/InterchainToken.sol
```
 
Line: 43          function interchainTransfer(
        string calldata destinationChain,
        bytes calldata recipient,
        uint256 amount,
        bytes calldata metadata
    ) external payable 
```
 These control structures in the function `interchainTransfer` should follow the One True Brace (OTB) style:

    - Line: 66:         tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

[contracts/its/interchain-token/InterchainToken.sol#L43-L67](contracts/its/interchain-token/InterchainToken.sol#L43-L67)


- File: contracts/its/interchain-token/InterchainToken.sol
```
 
Line: 79          function interchainTransferFrom(
        address sender,
        string calldata destinationChain,
        bytes calldata recipient,
        uint256 amount,
        bytes calldata metadata
    ) external payable 
```
 These control structures in the function `interchainTransferFrom` should follow the One True Brace (OTB) style:

    - Line: 104:         tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

[contracts/its/interchain-token/InterchainToken.sol#L79-L105](contracts/its/interchain-token/InterchainToken.sol#L79-L105)


- File: contracts/its/proxies/InterchainTokenServiceProxy.sol
```
 
Line: 19          constructor(
        address implementationAddress,
        address owner,
        address operator
    ) FinalProxy(implementationAddress, owner, abi.encodePacked(operator)) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 23:     ) FinalProxy(implementationAddress, owner, abi.encodePacked(operator)) {}

[contracts/its/proxies/InterchainTokenServiceProxy.sol#L19-L23](contracts/its/proxies/InterchainTokenServiceProxy.sol#L19-L23)


- File: contracts/its/proxies/RemoteAddressValidatorProxy.sol
```
 
Line: 20          constructor(
        address implementationAddress,
        address owner,
        bytes memory params
    ) Proxy(implementationAddress, owner, params) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 24:     ) Proxy(implementationAddress, owner, params) {}

[contracts/its/proxies/RemoteAddressValidatorProxy.sol#L20-L24](contracts/its/proxies/RemoteAddressValidatorProxy.sol#L20-L24)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 54          function _getImplementation(IInterchainTokenService interchainTokenServiceAddress_, uint256 implementationType_)
        internal
        view
        returns (address impl)
    
```
 These control structures in the function `_getImplementation` should follow the One True Brace (OTB) style:

    - Line: 58:     {

[contracts/its/proxies/TokenManagerProxy.sol#L54-L60](contracts/its/proxies/TokenManagerProxy.sol#L54-L60)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 66          function setup(bytes calldata setupParams) external 
```
 These control structures in the function `setup` should follow the One True Brace (OTB) style:

    - Line: 66: function setup(bytes calldata setupParams) external {}

[contracts/its/proxies/TokenManagerProxy.sol#L66](contracts/its/proxies/TokenManagerProxy.sol#L66)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 94          receive() external payable virtual 
```
 These control structures in the function `receive` should follow the One True Brace (OTB) style:

    - Line: 94: receive() external payable virtual {}

[contracts/its/proxies/TokenManagerProxy.sol#L94](contracts/its/proxies/TokenManagerProxy.sol#L94)


- File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 49          function setup(bytes calldata params) external override onlyProxy 
```
 These control structures in the function `setup` should follow the One True Brace (OTB) style:

    - Line: 50:         {
    - Line: 60:         {

[contracts/its/token-implementations/StandardizedToken.sol#L49-L68](contracts/its/token-implementations/StandardizedToken.sol#L49-L68)


- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 83          function sendToken(
        string calldata destinationChain,
        bytes calldata destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) external payable virtual 
```
 These control structures in the function `sendToken` should follow the One True Brace (OTB) style:

    - Line: 92:         interchainTokenService.transmitSendToken{ value: msg.value }(

[contracts/its/token-manager/TokenManager.sol#L83-L100](contracts/its/token-manager/TokenManager.sol#L83-L100)


- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 109          function callContractWithInterchainToken(
        string calldata destinationChain,
        bytes calldata destinationAddress,
        uint256 amount,
        bytes calldata data
    ) external payable virtual 
```
 These control structures in the function `callContractWithInterchainToken` should follow the One True Brace (OTB) style:

    - Line: 119:         interchainTokenService.transmitSendToken{ value: msg.value }(

[contracts/its/token-manager/TokenManager.sol#L109-L127](contracts/its/token-manager/TokenManager.sol#L109-L127)


- File: contracts/its/token-manager/TokenManager.sol
```
 
Line: 136          function transmitInterchainTransfer(
        address sender,
        string calldata destinationChain,
        bytes calldata destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) external payable virtual onlyToken 
```
 These control structures in the function `transmitInterchainTransfer` should follow the One True Brace (OTB) style:

    - Line: 145:         interchainTokenService.transmitSendToken{ value: msg.value }(

[contracts/its/token-manager/TokenManager.sol#L136-L153](contracts/its/token-manager/TokenManager.sol#L136-L153)


- File: contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol
```
 
Line: 19          constructor(address interchainTokenService_) TokenManager(interchainTokenService_) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 19: constructor(address interchainTokenService_) TokenManager(interchainTokenService_) {}

[contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol#L19](contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol#L19)


- File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 26          constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 26: constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}

[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L26](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L26)


- File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 22          constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 22: constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}

[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L22](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L22)


- File: contracts/its/token-manager/implementations/TokenManagerMintBurn.sol
```
 
Line: 23          constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) 
```
 These control structures in the function `constructor` should follow the One True Brace (OTB) style:

    - Line: 23: constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}

[contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L23](contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L23)


</details>

# 



## Custom error has no error details
- Severity: Non-Critical
- Confidence: High

### Description
Consider adding parameters to the error to indicate which user or values caused the failure.

<details>

<summary>
There are 67 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 22          error InvalidImplementation();
```

[contracts/cgp/AxelarGateway.sol#L22](contracts/cgp/AxelarGateway.sol#L22)


- File: contracts/cgp/interfaces/IAxelarServiceGovernance.sol
```
 
Line: 13          error NotApproved();
```

[contracts/cgp/interfaces/IAxelarServiceGovernance.sol#L13](contracts/cgp/interfaces/IAxelarServiceGovernance.sol#L13)


- File: contracts/cgp/interfaces/ICaller.sol
```
 
Line: 6          error InsufficientBalance();
```

[contracts/cgp/interfaces/ICaller.sol#L6](contracts/cgp/interfaces/ICaller.sol#L6)


- File: contracts/cgp/interfaces/ICaller.sol
```
 
Line: 7          error ExecutionFailed();
```

[contracts/cgp/interfaces/ICaller.sol#L7](contracts/cgp/interfaces/ICaller.sol#L7)


- File: contracts/cgp/interfaces/IInterchainGovernance.sol
```
 
Line: 13          error NotGovernance();
```

[contracts/cgp/interfaces/IInterchainGovernance.sol#L13](contracts/cgp/interfaces/IInterchainGovernance.sol#L13)


- File: contracts/cgp/interfaces/IInterchainGovernance.sol
```
 
Line: 14          error InvalidCommand();
```

[contracts/cgp/interfaces/IInterchainGovernance.sol#L14](contracts/cgp/interfaces/IInterchainGovernance.sol#L14)


- File: contracts/cgp/interfaces/IInterchainGovernance.sol
```
 
Line: 15          error InvalidTarget();
```

[contracts/cgp/interfaces/IInterchainGovernance.sol#L15](contracts/cgp/interfaces/IInterchainGovernance.sol#L15)


- File: contracts/cgp/interfaces/IInterchainGovernance.sol
```
 
Line: 16          error TokenNotSupported();
```

[contracts/cgp/interfaces/IInterchainGovernance.sol#L16](contracts/cgp/interfaces/IInterchainGovernance.sol#L16)


- File: contracts/cgp/interfaces/IMultisigBase.sol
```
 
Line: 10          error NotSigner();
```

[contracts/cgp/interfaces/IMultisigBase.sol#L10](contracts/cgp/interfaces/IMultisigBase.sol#L10)


- File: contracts/cgp/interfaces/IMultisigBase.sol
```
 
Line: 11          error AlreadyVoted();
```

[contracts/cgp/interfaces/IMultisigBase.sol#L11](contracts/cgp/interfaces/IMultisigBase.sol#L11)


- File: contracts/cgp/interfaces/IMultisigBase.sol
```
 
Line: 12          error InvalidSigners();
```

[contracts/cgp/interfaces/IMultisigBase.sol#L12](contracts/cgp/interfaces/IMultisigBase.sol#L12)


- File: contracts/cgp/interfaces/IMultisigBase.sol
```
 
Line: 13          error InvalidSignerThreshold();
```

[contracts/cgp/interfaces/IMultisigBase.sol#L13](contracts/cgp/interfaces/IMultisigBase.sol#L13)


- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 7          error NotOwner();
```

[contracts/cgp/interfaces/IUpgradable.sol#L7](contracts/cgp/interfaces/IUpgradable.sol#L7)


- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 8          error InvalidOwner();
```

[contracts/cgp/interfaces/IUpgradable.sol#L8](contracts/cgp/interfaces/IUpgradable.sol#L8)


- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 9          error InvalidCodeHash();
```

[contracts/cgp/interfaces/IUpgradable.sol#L9](contracts/cgp/interfaces/IUpgradable.sol#L9)


- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 10          error InvalidImplementation();
```

[contracts/cgp/interfaces/IUpgradable.sol#L10](contracts/cgp/interfaces/IUpgradable.sol#L10)


- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 11          error SetupFailed();
```

[contracts/cgp/interfaces/IUpgradable.sol#L11](contracts/cgp/interfaces/IUpgradable.sol#L11)


- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 12          error NotProxy();
```

[contracts/cgp/interfaces/IUpgradable.sol#L12](contracts/cgp/interfaces/IUpgradable.sol#L12)


- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 6          error EmptyBytecode();
```

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L6](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L6)


- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 7          error FailedDeploy();
```

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L7](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L7)


- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 8          error FailedInit();
```

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L8](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L8)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 13          error FailedInit();
```

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L13](contracts/gmp-sdk/deploy/Create3Deployer.sol#L13)


- File: contracts/gmp-sdk/interfaces/IProxy.sol
```
 
Line: 7          error InvalidOwner();
```

[contracts/gmp-sdk/interfaces/IProxy.sol#L7](contracts/gmp-sdk/interfaces/IProxy.sol#L7)


- File: contracts/gmp-sdk/interfaces/IProxy.sol
```
 
Line: 8          error InvalidImplementation();
```

[contracts/gmp-sdk/interfaces/IProxy.sol#L8](contracts/gmp-sdk/interfaces/IProxy.sol#L8)


- File: contracts/gmp-sdk/interfaces/IProxy.sol
```
 
Line: 9          error SetupFailed();
```

[contracts/gmp-sdk/interfaces/IProxy.sol#L9](contracts/gmp-sdk/interfaces/IProxy.sol#L9)


- File: contracts/gmp-sdk/interfaces/IProxy.sol
```
 
Line: 10          error NotOwner();
```

[contracts/gmp-sdk/interfaces/IProxy.sol#L10](contracts/gmp-sdk/interfaces/IProxy.sol#L10)


- File: contracts/gmp-sdk/interfaces/IProxy.sol
```
 
Line: 11          error AlreadyInitialized();
```

[contracts/gmp-sdk/interfaces/IProxy.sol#L11](contracts/gmp-sdk/interfaces/IProxy.sol#L11)


- File: contracts/gmp-sdk/interfaces/ITimeLock.sol
```
 
Line: 10          error InvalidTimeLockHash();
```

[contracts/gmp-sdk/interfaces/ITimeLock.sol#L10](contracts/gmp-sdk/interfaces/ITimeLock.sol#L10)


- File: contracts/gmp-sdk/interfaces/ITimeLock.sol
```
 
Line: 11          error TimeLockAlreadyScheduled();
```

[contracts/gmp-sdk/interfaces/ITimeLock.sol#L11](contracts/gmp-sdk/interfaces/ITimeLock.sol#L11)


- File: contracts/gmp-sdk/interfaces/ITimeLock.sol
```
 
Line: 12          error TimeLockNotReady();
```

[contracts/gmp-sdk/interfaces/ITimeLock.sol#L12](contracts/gmp-sdk/interfaces/ITimeLock.sol#L12)


- File: contracts/gmp-sdk/interfaces/IUpgradable.sol
```
 
Line: 9          error InvalidCodeHash();
```

[contracts/gmp-sdk/interfaces/IUpgradable.sol#L9](contracts/gmp-sdk/interfaces/IUpgradable.sol#L9)


- File: contracts/gmp-sdk/interfaces/IUpgradable.sol
```
 
Line: 10          error InvalidImplementation();
```

[contracts/gmp-sdk/interfaces/IUpgradable.sol#L10](contracts/gmp-sdk/interfaces/IUpgradable.sol#L10)


- File: contracts/gmp-sdk/interfaces/IUpgradable.sol
```
 
Line: 11          error SetupFailed();
```

[contracts/gmp-sdk/interfaces/IUpgradable.sol#L11](contracts/gmp-sdk/interfaces/IUpgradable.sol#L11)


- File: contracts/gmp-sdk/interfaces/IUpgradable.sol
```
 
Line: 12          error NotProxy();
```

[contracts/gmp-sdk/interfaces/IUpgradable.sol#L12](contracts/gmp-sdk/interfaces/IUpgradable.sol#L12)


- File: contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol
```
 
Line: 15          error ProposalExecuteFailed();
```

[contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L15](contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L15)


- File: contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol
```
 
Line: 18          error NotWhitelistedCaller();
```

[contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L18](contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L18)


- File: contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol
```
 
Line: 21          error NotWhitelistedSourceAddress();
```

[contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L21](contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L21)


- File: contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol
```
 
Line: 9          error InvalidFee();
```

[contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L9](contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L9)


- File: contracts/its/interfaces/IDistributable.sol
```
 
Line: 6          error NotDistributor();
```

[contracts/its/interfaces/IDistributable.sol#L6](contracts/its/interfaces/IDistributable.sol#L6)


- File: contracts/its/interfaces/IExpressCallHandler.sol
```
 
Line: 6          error AlreadyExpressCalled();
```

[contracts/its/interfaces/IExpressCallHandler.sol#L6](contracts/its/interfaces/IExpressCallHandler.sol#L6)


- File: contracts/its/interfaces/IExpressCallHandler.sol
```
 
Line: 7          error SameDestinationAsCaller();
```

[contracts/its/interfaces/IExpressCallHandler.sol#L7](contracts/its/interfaces/IExpressCallHandler.sol#L7)


- File: contracts/its/interfaces/IFlowLimit.sol
```
 
Line: 6          error FlowLimitExceeded();
```

[contracts/its/interfaces/IFlowLimit.sol#L6](contracts/its/interfaces/IFlowLimit.sol#L6)


- File: contracts/its/interfaces/IImplementation.sol
```
 
Line: 6          error NotProxy();
```

[contracts/its/interfaces/IImplementation.sol#L6](contracts/its/interfaces/IImplementation.sol#L6)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 16          error ZeroAddress();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L16](contracts/its/interfaces/IInterchainTokenService.sol#L16)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 17          error LengthMismatch();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L17](contracts/its/interfaces/IInterchainTokenService.sol#L17)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 18          error InvalidTokenManagerImplementation();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L18](contracts/its/interfaces/IInterchainTokenService.sol#L18)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 19          error NotRemoteService();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L19](contracts/its/interfaces/IInterchainTokenService.sol#L19)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 21          error NotTokenManager();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L21](contracts/its/interfaces/IInterchainTokenService.sol#L21)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 23          error NotCanonicalTokenManager();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L23](contracts/its/interfaces/IInterchainTokenService.sol#L23)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 24          error GatewayToken();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L24](contracts/its/interfaces/IInterchainTokenService.sol#L24)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 25          error TokenManagerDeploymentFailed();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L25](contracts/its/interfaces/IInterchainTokenService.sol#L25)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 26          error StandardizedTokenDeploymentFailed();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L26](contracts/its/interfaces/IInterchainTokenService.sol#L26)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 28          error SelectorUnknown();
```

[contracts/its/interfaces/IInterchainTokenService.sol#L28](contracts/its/interfaces/IInterchainTokenService.sol#L28)


- File: contracts/its/interfaces/IOperatable.sol
```
 
Line: 6          error NotOperator();
```

[contracts/its/interfaces/IOperatable.sol#L6](contracts/its/interfaces/IOperatable.sol#L6)


- File: contracts/its/interfaces/IPausable.sol
```
 
Line: 13          error Paused();
```

[contracts/its/interfaces/IPausable.sol#L13](contracts/its/interfaces/IPausable.sol#L13)


- File: contracts/its/interfaces/IRemoteAddressValidator.sol
```
 
Line: 10          error ZeroAddress();
```

[contracts/its/interfaces/IRemoteAddressValidator.sol#L10](contracts/its/interfaces/IRemoteAddressValidator.sol#L10)


- File: contracts/its/interfaces/IRemoteAddressValidator.sol
```
 
Line: 11          error LengthMismatch();
```

[contracts/its/interfaces/IRemoteAddressValidator.sol#L11](contracts/its/interfaces/IRemoteAddressValidator.sol#L11)


- File: contracts/its/interfaces/IRemoteAddressValidator.sol
```
 
Line: 12          error ZeroStringLength();
```

[contracts/its/interfaces/IRemoteAddressValidator.sol#L12](contracts/its/interfaces/IRemoteAddressValidator.sol#L12)


- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 15          error TokenLinkerZeroAddress();
```

[contracts/its/interfaces/ITokenManager.sol#L15](contracts/its/interfaces/ITokenManager.sol#L15)


- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 16          error NotService();
```

[contracts/its/interfaces/ITokenManager.sol#L16](contracts/its/interfaces/ITokenManager.sol#L16)


- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 17          error TakeTokenFailed();
```

[contracts/its/interfaces/ITokenManager.sol#L17](contracts/its/interfaces/ITokenManager.sol#L17)


- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 18          error GiveTokenFailed();
```

[contracts/its/interfaces/ITokenManager.sol#L18](contracts/its/interfaces/ITokenManager.sol#L18)


- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 19          error NotToken();
```

[contracts/its/interfaces/ITokenManager.sol#L19](contracts/its/interfaces/ITokenManager.sol#L19)


- File: contracts/its/interfaces/ITokenManagerDeployer.sol
```
 
Line: 12          error AddressZero();
```

[contracts/its/interfaces/ITokenManagerDeployer.sol#L12](contracts/its/interfaces/ITokenManagerDeployer.sol#L12)


- File: contracts/its/interfaces/ITokenManagerDeployer.sol
```
 
Line: 13          error TokenManagerDeploymentFailed();
```

[contracts/its/interfaces/ITokenManagerDeployer.sol#L13](contracts/its/interfaces/ITokenManagerDeployer.sol#L13)


- File: contracts/its/interfaces/ITokenManagerProxy.sol
```
 
Line: 11          error ImplementationLookupFailed();
```

[contracts/its/interfaces/ITokenManagerProxy.sol#L11](contracts/its/interfaces/ITokenManagerProxy.sol#L11)


- File: contracts/its/interfaces/ITokenManagerProxy.sol
```
 
Line: 12          error SetupFailed();
```

[contracts/its/interfaces/ITokenManagerProxy.sol#L12](contracts/its/interfaces/ITokenManagerProxy.sol#L12)


</details>

# 



## Dead-code
- Severity: Non-Critical
- Confidence: Medium

### Description
Functions that are not sued.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 126          function _beforeProposalExecuted(
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) internal virtual 
```
 is never used and should be removed

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L126-L132](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L126-L132)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 178          function _onTargetExecuted(InterchainCalls.Call memory call, bytes memory result) internal virtual 
```
 is never used and should be removed

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L178-L180](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L178-L180)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 156          function _onTargetExecutionFailed(
        InterchainCalls.Call memory, /* call */
        bytes memory result
    ) internal virtual 
```
 is never used and should be removed

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L156-L170](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L156-L170)


- File: contracts/its/proxies/InterchainTokenServiceProxy.sol
```
 
Line: 29          function contractId() internal pure override returns (bytes32) 
```
 is never used and should be removed

[contracts/its/proxies/InterchainTokenServiceProxy.sol#L29-L31](contracts/its/proxies/InterchainTokenServiceProxy.sol#L29-L31)


- File: contracts/its/proxies/RemoteAddressValidatorProxy.sol
```
 
Line: 30          function contractId() internal pure override returns (bytes32) 
```
 is never used and should be removed

[contracts/its/proxies/RemoteAddressValidatorProxy.sol#L30-L32](contracts/its/proxies/RemoteAddressValidatorProxy.sol#L30-L32)


</details>

# 


## Name reused
- Severity: Non-Critical
- Confidence: High

### Description
If a codebase has two contracts the similar names, the compilation artifacts
will not contain one of the contracts with the duplicate name.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- IUpgradable is re-used:
	- File: contracts/cgp/interfaces/IUpgradable.sol
```
 
Line: 6          interface IUpgradable 
```

	- File: contracts/gmp-sdk/interfaces/IUpgradable.sol
```
 
Line: 8          interface IUpgradable is IOwnable 
```


[contracts/cgp/interfaces/IUpgradable.sol#L6-L31](contracts/cgp/interfaces/IUpgradable.sol#L6-L31)


- Upgradable is re-used:
	- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 7          abstract contract Upgradable is IUpgradable 
```

	- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 12          abstract contract Upgradable is Ownable, IUpgradable 
```


[contracts/cgp/util/Upgradable.sol#L7-L69](contracts/cgp/util/Upgradable.sol#L7-L69)


</details>

# 


## Consider Disable Ownership Renouncement in Ownable Contracts
- Severity: Non-Critical
- Confidence: High

### Description
Ownership renouncement is a feature provided by the Ownable contract in various smart contract frameworks, such as OpenZeppelin. It allows the contract owner to transfer ownership to another address, relinquishing their control over the contract. However, in certain cases, it may be undesirable or unnecessary to allow ownership renouncement. 

It is important to carefully consider the implications of allowing ownership renouncement. Disabling renouncement can provide additional security and prevent potential risks associated with unintended ownership transfers.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 12          abstract contract Upgradable is Ownable, IUpgradable 
```

[contracts/gmp-sdk/upgradable/Upgradable.sol#L12-L88](contracts/gmp-sdk/upgradable/Upgradable.sol#L12-L88)


- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 22          contract InterchainProposalExecutor is IInterchainProposalExecutor, AxelarExecutable, Ownable 
```

[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L22-L181](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L22-L181)


- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 37          contract InterchainTokenService is
    IInterchainTokenService,
    AxelarExecutable,
    Upgradable,
    Operatable,
    ExpressCallHandler,
    Pausable,
    Multicall

```

[contracts/its/interchain-token-service/InterchainTokenService.sol#L37-L896](contracts/its/interchain-token-service/InterchainTokenService.sol#L37-L896)


- File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
```
 
Line: 12          contract RemoteAddressValidator is IRemoteAddressValidator, Upgradable 
```

[contracts/its/remote-address-validator/RemoteAddressValidator.sol#L12-L139](contracts/its/remote-address-validator/RemoteAddressValidator.sol#L12-L139)


</details>

# 


## Event Emission Preceding External Calls: A Best Practice
- Severity: Non-Critical
- Confidence: Medium

### Description

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls


<details>

<summary>
There are 28 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/cgp/AxelarGateway.sol
```
 
Line: 385          function deployToken(bytes calldata params, bytes32) external onlySelf 
```
:
	External calls:
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 398          (bool success, bytes memory data) = TOKEN_DEPLOYER_IMPLEMENTATION.delegatecall(
                abi.encodeWithSelector(ITokenDeployer.deployToken.selector, name, symbol, decimals, cap, salt)
            )
```

	Event emitted after the call(s):
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 418          emit TokenDeployed(symbol, tokenAddress)
```

	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 630          emit TokenMintLimitUpdated(symbol, limit)
```

		- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 416          _setTokenMintLimit(symbol, mintLimit)
```


[contracts/cgp/AxelarGateway.sol#L385-L419](contracts/cgp/AxelarGateway.sol#L385-L419)


- Reentrancy in File: contracts/cgp/AxelarGateway.sol
```
 
Line: 323          function execute(bytes calldata input) external override 
```
:
	External calls:
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 329          bool allowOperatorshipTransfer = IAxelarAuth(AUTH_MODULE).validateProof(messageHash, proof)
```

	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 374          (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId))
```

	Event emitted after the call(s):
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 376          emit Executed(commandId)
```


[contracts/cgp/AxelarGateway.sol#L323-L379](contracts/cgp/AxelarGateway.sol#L323-L379)


- Reentrancy in File: contracts/cgp/AxelarGateway.sol
```
 
Line: 307          function setup(bytes calldata params) external override 
```
:
	External calls:
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 317          IAxelarAuth(AUTH_MODULE).transferOperatorship(newOperatorsData)
```

	Event emitted after the call(s):
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 319          emit OperatorshipTransferred(newOperatorsData)
```


[contracts/cgp/AxelarGateway.sol#L307-L321](contracts/cgp/AxelarGateway.sol#L307-L321)


- Reentrancy in File: contracts/cgp/AxelarGateway.sol
```
 
Line: 495          function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf 
```
:
	External calls:
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 496          IAxelarAuth(AUTH_MODULE).transferOperatorship(newOperatorsData)
```

	Event emitted after the call(s):
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 498          emit OperatorshipTransferred(newOperatorsData)
```


[contracts/cgp/AxelarGateway.sol#L495-L499](contracts/cgp/AxelarGateway.sol#L495-L499)


- Reentrancy in File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 48          function executeMultisigProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable onlySigners 
```
:
	External calls:
	- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 59          _call(target, callData, nativeValue)
```

		- File: contracts/cgp/util/Caller.sol
```
 
Line: 18          (bool success, ) = target.call{ value: nativeValue }(callData)
```

	Event emitted after the call(s):
	- File: contracts/cgp/governance/AxelarServiceGovernance.sol
```
 
Line: 61          emit MultisigExecuted(proposalHash, target, callData, nativeValue)
```


[contracts/cgp/governance/AxelarServiceGovernance.sol#L48-L62](contracts/cgp/governance/AxelarServiceGovernance.sol#L48-L62)


- Reentrancy in File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 68          function executeProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable 
```
:
	External calls:
	- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 76          _call(target, callData, nativeValue)
```

		- File: contracts/cgp/util/Caller.sol
```
 
Line: 18          (bool success, ) = target.call{ value: nativeValue }(callData)
```

	Event emitted after the call(s):
	- File: contracts/cgp/governance/InterchainGovernance.sol
```
 
Line: 78          emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp)
```


[contracts/cgp/governance/InterchainGovernance.sol#L68-L79](contracts/cgp/governance/InterchainGovernance.sol#L68-L79)


- Reentrancy in File: contracts/cgp/util/Upgradable.sol
```
 
Line: 40          function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner 
```
:
	External calls:
	- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 49          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params))
```

	Event emitted after the call(s):
	- File: contracts/cgp/util/Upgradable.sol
```
 
Line: 54          emit Upgraded(newImplementation)
```


[contracts/cgp/util/Upgradable.sol#L40-L59](contracts/cgp/util/Upgradable.sol#L40-L59)


- Reentrancy in File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 29          function deploy(bytes calldata bytecode, bytes32 salt) external returns (address deployedAddress_) 
```
:
	External calls:
	- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 31          deployedAddress_ = Create3.deploy(deploySalt, bytecode)
```

	Event emitted after the call(s):
	- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 33          emit Deployed(keccak256(bytecode), salt, deployedAddress_)
```


[contracts/gmp-sdk/deploy/Create3Deployer.sol#L29-L34](contracts/gmp-sdk/deploy/Create3Deployer.sol#L29-L34)


- Reentrancy in File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 49          function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) 
```
:
	External calls:
	- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 55          deployedAddress_ = Create3.deploy(deploySalt, bytecode)
```

	- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 57          (bool success, ) = deployedAddress_.call(init)
```

	Event emitted after the call(s):
	- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 60          emit Deployed(keccak256(bytecode), salt, deployedAddress_)
```


[contracts/gmp-sdk/deploy/Create3Deployer.sol#L49-L61](contracts/gmp-sdk/deploy/Create3Deployer.sol#L49-L61)


- Reentrancy in File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 52          function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner 
```
:
	External calls:
	- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 61          (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params))
```

	Event emitted after the call(s):
	- File: contracts/gmp-sdk/upgradable/Upgradable.sol
```
 
Line: 66          emit Upgraded(newImplementation)
```


[contracts/gmp-sdk/upgradable/Upgradable.sol#L52-L71](contracts/gmp-sdk/upgradable/Upgradable.sol#L52-L71)


- Reentrancy in File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 41          function _execute(
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) internal override 
```
:
	External calls:
	- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 62          _executeProposal(calls)
```

		- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 76          (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData)
```

	Event emitted after the call(s):
	- File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
```
 
Line: 66          emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)))
```


[contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L41-L67](contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L41-L67)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 770          function _deployRemoteStandardizedToken(
        bytes32 tokenId,
        string memory name,
        string memory symbol,
        uint8 decimals,
        bytes memory distributor,
        bytes memory operator,
        string calldata destinationChain,
        uint256 gasValue
    ) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 789          _callContract(destinationChain, payload, gasValue, msg.sender)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 723          gateway.callContract(destinationChain, destinationAddress, payload)
```

	External calls sending eth:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 789          _callContract(destinationChain, payload, gasValue, msg.sender)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

	Event emitted after the call(s):
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


[contracts/its/interchain-token-service/InterchainTokenService.sol#L770-L800](contracts/its/interchain-token-service/InterchainTokenService.sol#L770-L800)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 748          function _deployRemoteTokenManager(
        bytes32 tokenId,
        string calldata destinationChain,
        uint256 gasValue,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 756          _callContract(destinationChain, payload, gasValue, msg.sender)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 723          gateway.callContract(destinationChain, destinationAddress, payload)
```

	External calls sending eth:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 756          _callContract(destinationChain, payload, gasValue, msg.sender)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 757          emit RemoteTokenManagerDeploymentInitialized(tokenId, destinationChain, gasValue, tokenManagerType, params)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L748-L758](contracts/its/interchain-token-service/InterchainTokenService.sol#L748-L758)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 841          function _deployStandardizedToken(
        bytes32 tokenId,
        address distributor,
        string memory name,
        string memory symbol,
        uint8 decimals,
        uint256 mintAmount,
        address mintTo
    ) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 853          (bool success, ) = standardizedTokenDeployer.delegatecall(
            abi.encodeWithSelector(
                IStandardizedTokenDeployer.deployStandardizedToken.selector,
                salt,
                tokenManagerAddress,
                distributor,
                name,
                symbol,
                decimals,
                mintAmount,
                mintTo
            )
        )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 869          emit StandardizedTokenDeployed(tokenId, name, symbol, decimals, mintAmount, mintTo)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L841-L870](contracts/its/interchain-token-service/InterchainTokenService.sol#L841-L870)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 808          function _deployTokenManager(
        bytes32 tokenId,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 819          emit TokenManagerDeployed(tokenId, tokenManagerType, params)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L808-L820](contracts/its/interchain-token-service/InterchainTokenService.sol#L808-L820)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 678          function _processDeployStandardizedTokenAndManagerPayload(bytes calldata payload) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 691          _deployStandardizedToken(tokenId, distributor, name, symbol, decimals, 0, distributor)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 853          (bool success, ) = standardizedTokenDeployer.delegatecall(
            abi.encodeWithSelector(
                IStandardizedTokenDeployer.deployStandardizedToken.selector,
                salt,
                tokenManagerAddress,
                distributor,
                name,
                symbol,
                decimals,
                mintAmount,
                mintTo
            )
        )
```

	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 693          _deployTokenManager(
            tokenId,
            tokenManagerType,
            abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
        )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 819          emit TokenManagerDeployed(tokenId, tokenManagerType, params)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 693          _deployTokenManager(
            tokenId,
            tokenManagerType,
            abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
        )
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L678-L698](contracts/its/interchain-token-service/InterchainTokenService.sol#L678-L698)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 678          function _processDeployStandardizedTokenAndManagerPayload(bytes calldata payload) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 691          _deployStandardizedToken(tokenId, distributor, name, symbol, decimals, 0, distributor)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 853          (bool success, ) = standardizedTokenDeployer.delegatecall(
            abi.encodeWithSelector(
                IStandardizedTokenDeployer.deployStandardizedToken.selector,
                salt,
                tokenManagerAddress,
                distributor,
                name,
                symbol,
                decimals,
                mintAmount,
                mintTo
            )
        )
```

	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 693          _deployTokenManager(
            tokenId,
            tokenManagerType,
            abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
        )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 819          emit TokenManagerDeployed(tokenId, tokenManagerType, params)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 693          _deployTokenManager(
            tokenId,
            tokenManagerType,
            abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
        )
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L678-L698](contracts/its/interchain-token-service/InterchainTokenService.sol#L678-L698)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 599          function _processSendTokenPayload(string calldata sourceChain, bytes calldata payload) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 610          amount = tokenManager.giveToken(destinationAddress, amount)
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 611          emit TokenReceived(tokenId, sourceChain, destinationAddress, amount)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L599-L615](contracts/its/interchain-token-service/InterchainTokenService.sol#L599-L615)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 622          function _processSendTokenWithDataPayload(string calldata sourceChain, bytes calldata payload) internal 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 657          amount = tokenManager.giveToken(destinationAddress, amount)
```

	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 658          IInterchainTokenExpressExecutable(destinationAddress).executeWithInterchainToken(sourceChain, sourceAddress, data, tokenId, amount)
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 659          emit TokenReceivedWithData(tokenId, sourceChain, destinationAddress, amount, sourceAddress, data)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L622-L660](contracts/its/interchain-token-service/InterchainTokenService.sol#L622-L660)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 388          function deployAndRegisterStandardizedToken(
        bytes32 salt,
        string calldata name,
        string calldata symbol,
        uint8 decimals,
        uint256 mintAmount,
        address distributor
    ) external payable notPaused 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 397          _deployStandardizedToken(tokenId, distributor, name, symbol, decimals, mintAmount, msg.sender)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 853          (bool success, ) = standardizedTokenDeployer.delegatecall(
            abi.encodeWithSelector(
                IStandardizedTokenDeployer.deployStandardizedToken.selector,
                salt,
                tokenManagerAddress,
                distributor,
                name,
                symbol,
                decimals,
                mintAmount,
                mintTo
            )
        )
```

	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 401          _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress))
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 819          emit TokenManagerDeployed(tokenId, tokenManagerType, params)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 401          _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress))
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L388-L402](contracts/its/interchain-token-service/InterchainTokenService.sol#L388-L402)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 343          function deployCustomTokenManager(
        bytes32 salt,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) public payable notPaused returns (bytes32 tokenId) 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 350          _deployTokenManager(tokenId, tokenManagerType, params)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 351          emit CustomTokenIdClaimed(tokenId, deployer_, salt)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L343-L352](contracts/its/interchain-token-service/InterchainTokenService.sol#L343-L352)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 325          function deployRemoteCanonicalToken(
        bytes32 tokenId,
        string calldata destinationChain,
        uint256 gasValue
    ) public payable notPaused 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 333          (string memory tokenName, string memory tokenSymbol, uint8 tokenDecimals) = _validateToken(tokenAddress)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 735          name = token.name()
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 736          symbol = token.symbol()
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 737          decimals = token.decimals()
```

	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 334          _deployRemoteStandardizedToken(tokenId, tokenName, tokenSymbol, tokenDecimals, '', '', destinationChain, gasValue)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 723          gateway.callContract(destinationChain, destinationAddress, payload)
```

	External calls sending eth:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 334          _deployRemoteStandardizedToken(tokenId, tokenName, tokenSymbol, tokenDecimals, '', '', destinationChain, gasValue)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

	Event emitted after the call(s):
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

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 334          _deployRemoteStandardizedToken(tokenId, tokenName, tokenSymbol, tokenDecimals, '', '', destinationChain, gasValue)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L325-L335](contracts/its/interchain-token-service/InterchainTokenService.sol#L325-L335)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 365          function deployRemoteCustomTokenManager(
        bytes32 salt,
        string calldata destinationChain,
        TokenManagerType tokenManagerType,
        bytes calldata params,
        uint256 gasValue
    ) external payable notPaused returns (bytes32 tokenId) 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 374          _deployRemoteTokenManager(tokenId, destinationChain, gasValue, tokenManagerType, params)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 723          gateway.callContract(destinationChain, destinationAddress, payload)
```

	External calls sending eth:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 374          _deployRemoteTokenManager(tokenId, destinationChain, gasValue, tokenManagerType, params)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 375          emit CustomTokenIdClaimed(tokenId, deployer_, salt)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L365-L376](contracts/its/interchain-token-service/InterchainTokenService.sol#L365-L376)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 439          function expressReceiveToken(
        bytes32 tokenId,
        address destinationAddress,
        uint256 amount,
        bytes32 commandId
    ) external 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 451          SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount)
```

	Event emitted after the call(s):
	- File: contracts/its/utils/ExpressCallHandler.sol
```
 
Line: 95          emit ExpressReceive(tokenId, destinationAddress, amount, commandId, expressCaller)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 453          _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, caller)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L439-L454](contracts/its/interchain-token-service/InterchainTokenService.sol#L439-L454)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 467          function expressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId
    ) external 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 482          SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount)
```

	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 484          _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 888          IInterchainTokenExpressExecutable(destinationAddress).expressExecuteWithInterchainToken(
            sourceChain,
            sourceAddress,
            data,
            tokenId,
            amount
        )
```

	Event emitted after the call(s):
	- File: contracts/its/utils/ExpressCallHandler.sol
```
 
Line: 137          emit ExpressReceiveWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, expressCaller)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 486          _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L467-L487](contracts/its/interchain-token-service/InterchainTokenService.sol#L467-L487)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 309          function registerCanonicalToken(address tokenAddress) external payable notPaused returns (bytes32 tokenId) 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 310          (, string memory tokenSymbol, ) = _validateToken(tokenAddress)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 735          name = token.name()
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 736          symbol = token.symbol()
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 737          decimals = token.decimals()
```

	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 313          _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress))
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 813          (bool success, ) = tokenManagerDeployer.delegatecall(
            abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)
        )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 819          emit TokenManagerDeployed(tokenId, tokenManagerType, params)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 313          _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress))
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L309-L314](contracts/its/interchain-token-service/InterchainTokenService.sol#L309-L314)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 502          function transmitSendToken(
        bytes32 tokenId,
        address sourceAddress,
        string calldata destinationChain,
        bytes memory destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) external payable onlyTokenManager(tokenId) notPaused 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 513          _callContract(destinationChain, payload, msg.value, sourceAddress)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 723          gateway.callContract(destinationChain, destinationAddress, payload)
```

	External calls sending eth:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 513          _callContract(destinationChain, payload, msg.value, sourceAddress)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 514          emit TokenSent(tokenId, destinationChain, destinationAddress, amount)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L502-L523](contracts/its/interchain-token-service/InterchainTokenService.sol#L502-L523)


- Reentrancy in File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 502          function transmitSendToken(
        bytes32 tokenId,
        address sourceAddress,
        string calldata destinationChain,
        bytes memory destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) external payable onlyTokenManager(tokenId) notPaused 
```
:
	External calls:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 521          _callContract(destinationChain, payload, msg.value, sourceAddress)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 723          gateway.callContract(destinationChain, destinationAddress, payload)
```

	External calls sending eth:
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 521          _callContract(destinationChain, payload, msg.value, sourceAddress)
```

		- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 715          gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            )
```

	Event emitted after the call(s):
	- File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 522          emit TokenSentWithData(tokenId, destinationChain, destinationAddress, amount, sourceAddress, metadata)
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L502-L523](contracts/its/interchain-token-service/InterchainTokenService.sol#L502-L523)


</details>

# 


## Variable names too similar
- Severity: Non-Critical
- Confidence: Medium

### Description
Detect variables with names that are too similar.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- Variable File: contracts/cgp/AxelarGateway.sol
```
 
Line: 60          address internal immutable AUTH_MODULE
```
 is too similar to File: contracts/cgp/AxelarGateway.sol
```
 
Line: 64          address authModule_
```


[contracts/cgp/AxelarGateway.sol#L60](contracts/cgp/AxelarGateway.sol#L60)


- Variable File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 559          address[] memory implementaions
```
 is too similar to File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 562          address implementation
```


[contracts/its/interchain-token-service/InterchainTokenService.sol#L559](contracts/its/interchain-token-service/InterchainTokenService.sol#L559)


</details>

# 


## Unused imports
- Severity: Non-Critical
- Confidence: High

### Description
Identify unused imports in source files that can be safely removed. Please note that this detector does not support files with cyclic imports or files that use 'import {...} from' directives.

<details>

<summary>
There are 12 instances of this issue:

</summary>

###
- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/cgp/AxelarGateway.sol.
Consider removing the following imports:

    - File: contracts/cgp/AxelarGateway.sol
```
 
Line: 15          import { AdminMultisigBase } from './AdminMultisigBase.sol';
```
contracts/gmp-sdk/interfaces/IERC20.sol 
contracts/gmp-sdk/interfaces/IERC20.sol

[contracts/cgp/AxelarGateway.sol#L15](contracts/cgp/AxelarGateway.sol#L15)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/gmp-sdk/upgradable/FinalProxy.sol.
Consider removing the following imports:

    - File: contracts/gmp-sdk/upgradable/FinalProxy.sol
```
 
Line: 9          import { Proxy } from './Proxy.sol';
```
contracts/gmp-sdk/interfaces/IProxy.sol 
    - File: contracts/gmp-sdk/upgradable/FinalProxy.sol
```
 
Line: 9          import { Proxy } from './Proxy.sol';
```
contracts/gmp-sdk/upgradable/BaseProxy.sol 
contracts/gmp-sdk/upgradable/BaseProxy.sol

[contracts/gmp-sdk/upgradable/FinalProxy.sol#L9](contracts/gmp-sdk/upgradable/FinalProxy.sol#L9)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/gmp-sdk/upgradable/Proxy.sol.
Consider removing the following imports:

    - File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 7          import { BaseProxy } from './BaseProxy.sol';
```
contracts/gmp-sdk/interfaces/IUpgradable.sol 
    - File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 7          import { BaseProxy } from './BaseProxy.sol';
```
contracts/gmp-sdk/interfaces/IProxy.sol 
    - File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 7          import { BaseProxy } from './BaseProxy.sol';
```
contracts/gmp-sdk/upgradable/BaseProxy.sol 
and adding the following:

    - File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 7          import { BaseProxy } from './BaseProxy.sol';
```
/Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/proxies/InterchainTokenServiceProxy.sol 
    - File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 7          import { BaseProxy } from './BaseProxy.sol';
```
/Users/noam/Documents/Code4rena/2023-07-axelar/contracts/gmp-sdk/test/ProxyTest.sol 
    - File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 7          import { BaseProxy } from './BaseProxy.sol';
```
/Users/noam/Documents/Code4rena/2023-07-axelar/contracts/gmp-sdk/test/proxy/TestProxy.sol 
    - File: contracts/gmp-sdk/upgradable/Proxy.sol
```
 
Line: 7          import { BaseProxy } from './BaseProxy.sol';
```
/Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/proxies/RemoteAddressValidatorProxy.sol 
/Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/proxies/RemoteAddressValidatorProxy.sol

[contracts/gmp-sdk/upgradable/Proxy.sol#L7](contracts/gmp-sdk/upgradable/Proxy.sol#L7)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalSender.sol.
Consider removing the following imports:

    - File: contracts/interchain-governance-executor/InterchainProposalSender.sol
```
 
Line: 8          import { InterchainCalls } from './lib/InterchainCalls.sol';
```
contracts/interchain-governance-executor/lib/InterchainCalls.sol 
contracts/interchain-governance-executor/lib/InterchainCalls.sol

[contracts/interchain-governance-executor/InterchainProposalSender.sol#L8](contracts/interchain-governance-executor/InterchainProposalSender.sol#L8)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol.
Consider removing the following imports:

    - File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 29          import { Multicall } from '../utils/Multicall.sol';
```
contracts/gmp-sdk/interfaces/IAxelarGateway.sol 
    - File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 29          import { Multicall } from '../utils/Multicall.sol';
```
contracts/its/interfaces/ITokenManagerDeployer.sol 
    - File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 29          import { Multicall } from '../utils/Multicall.sol';
```
contracts/gmp-sdk/deploy/Create3Deployer.sol 
    - File: contracts/its/interchain-token-service/InterchainTokenService.sol
```
 
Line: 29          import { Multicall } from '../utils/Multicall.sol';
```
contracts/gmp-sdk/interfaces/IERC20.sol 
contracts/gmp-sdk/interfaces/IERC20.sol

[contracts/its/interchain-token-service/InterchainTokenService.sol#L29](contracts/its/interchain-token-service/InterchainTokenService.sol#L29)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/interfaces/IInterchainTokenService.sol.
Consider removing the following imports:

    - File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 12          import { IMulticall } from './IMulticall.sol';
```
contracts/gmp-sdk/interfaces/IAxelarGateway.sol 
    - File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 12          import { IMulticall } from './IMulticall.sol';
```
contracts/its/interfaces/ITokenManagerDeployer.sol 
contracts/its/interfaces/ITokenManagerDeployer.sol

[contracts/its/interfaces/IInterchainTokenService.sol#L12](contracts/its/interfaces/IInterchainTokenService.sol#L12)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/token-implementations/StandardizedToken.sol.
Consider removing the following imports:

    - File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 12          import { Distributable } from '../utils/Distributable.sol';
```
contracts/its/interfaces/ITokenManager.sol 
    - File: contracts/its/token-implementations/StandardizedToken.sol
```
 
Line: 12          import { Distributable } from '../utils/Distributable.sol';
```
contracts/its/interfaces/IERC20BurnableMintable.sol 
contracts/its/interfaces/IERC20BurnableMintable.sol

[contracts/its/token-implementations/StandardizedToken.sol#L12](contracts/its/token-implementations/StandardizedToken.sol#L12)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol.
Consider removing the following imports:

    - File: contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol
```
 
Line: 7          import { IAxelarGateway } from '../../../gmp-sdk/interfaces/IAxelarGateway.sol';
```
contracts/gmp-sdk/interfaces/IAxelarGateway.sol 
    - File: contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol
```
 
Line: 7          import { IAxelarGateway } from '../../../gmp-sdk/interfaces/IAxelarGateway.sol';
```
contracts/gmp-sdk/interfaces/IERC20.sol 
contracts/gmp-sdk/interfaces/IERC20.sol

[contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol#L7](contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol#L7)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol.
Consider removing the following imports:

    - File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
```
 
Line: 8          import { SafeTokenTransferFrom } from '../../../gmp-sdk/util/SafeTransfer.sol';
```
contracts/gmp-sdk/interfaces/IERC20.sol 
contracts/gmp-sdk/interfaces/IERC20.sol

[contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L8](contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L8)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol.
Consider removing the following imports:

    - File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
```
 
Line: 8          import { SafeTokenTransferFrom, SafeTokenTransfer } from '../../../gmp-sdk/util/SafeTransfer.sol';
```
contracts/gmp-sdk/interfaces/IERC20.sol 
contracts/gmp-sdk/interfaces/IERC20.sol

[contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L8](contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L8)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/token-manager/implementations/TokenManagerMintBurn.sol.
Consider removing the following imports:

    - File: contracts/its/token-manager/implementations/TokenManagerMintBurn.sol
```
 
Line: 9          import { SafeTokenCall } from '../../../gmp-sdk/util/SafeTransfer.sol';
```
contracts/gmp-sdk/interfaces/IERC20.sol 
contracts/gmp-sdk/interfaces/IERC20.sol

[contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L9](contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L9)


- Unused imports found in /Users/noam/Documents/Code4rena/2023-07-axelar/contracts/its/utils/TokenManagerDeployer.sol.
Consider removing the following imports:

    - File: contracts/its/utils/TokenManagerDeployer.sol
```
 
Line: 9          import { TokenManagerProxy } from '../proxies/TokenManagerProxy.sol';
```
contracts/gmp-sdk/deploy/Create3Deployer.sol 
    - File: contracts/its/utils/TokenManagerDeployer.sol
```
 
Line: 9          import { TokenManagerProxy } from '../proxies/TokenManagerProxy.sol';
```
contracts/its/interfaces/ITokenManagerDeployer.sol 
contracts/its/interfaces/ITokenManagerDeployer.sol

[contracts/its/utils/TokenManagerDeployer.sol#L9](contracts/its/utils/TokenManagerDeployer.sol#L9)


</details>

# 

## Unused Custom Error definition
- Severity: Non-Critical
- Confidence: High

### Description
There may be cases where a custom error superficially appears to be used, but this is only because there are multiple definitions of the custom error in different files. In such cases, the custom error definition should be moved into a separate file. 

<details>

<summary>
There are 7 instances of this issue:

</summary>

###
- File: contracts/its/interfaces/IExpressCallHandler.sol
```
 
Line: 7          error SameDestinationAsCaller();
```
 The custom error `SameDestinationAsCaller` is defined but never used:

[contracts/its/interfaces/IExpressCallHandler.sol#L7](contracts/its/interfaces/IExpressCallHandler.sol#L7)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 22          error ExecuteWithInterchainTokenFailed(address contractAddress);
```
 The custom error `ExecuteWithInterchainTokenFailed` is defined but never used:

[contracts/its/interfaces/IInterchainTokenService.sol#L22](contracts/its/interfaces/IInterchainTokenService.sol#L22)


- File: contracts/its/interfaces/IInterchainTokenService.sol
```
 
Line: 27          error DoesNotAcceptExpressExecute(address contractAddress);
```
 The custom error `DoesNotAcceptExpressExecute` is defined but never used:

[contracts/its/interfaces/IInterchainTokenService.sol#L27](contracts/its/interfaces/IInterchainTokenService.sol#L27)


- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 17          error TakeTokenFailed();
```
 The custom error `TakeTokenFailed` is defined but never used:

[contracts/its/interfaces/ITokenManager.sol#L17](contracts/its/interfaces/ITokenManager.sol#L17)


- File: contracts/its/interfaces/ITokenManager.sol
```
 
Line: 18          error GiveTokenFailed();
```
 The custom error `GiveTokenFailed` is defined but never used:

[contracts/its/interfaces/ITokenManager.sol#L18](contracts/its/interfaces/ITokenManager.sol#L18)


- File: contracts/its/interfaces/ITokenManagerProxy.sol
```
 
Line: 11          error ImplementationLookupFailed();
```
 The custom error `ImplementationLookupFailed` is defined but never used:

[contracts/its/interfaces/ITokenManagerProxy.sol#L11](contracts/its/interfaces/ITokenManagerProxy.sol#L11)


- File: contracts/its/utils/Multicall.sol
```
 
Line: 13          error MulticallFailed(bytes err);
```
 The custom error `MulticallFailed` is defined but never used:

[contracts/its/utils/Multicall.sol#L13](contracts/its/utils/Multicall.sol#L13)


</details>

# 

## Cyclomatic complexity
- Severity: Non-Critical
- Confidence: High

### Description
Detects functions with high (> 11) cyclomatic complexity.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 323          function execute(bytes calldata input) external override 
```
 has a high cyclomatic complexity (13).

[contracts/cgp/AxelarGateway.sol#L323-L379](contracts/cgp/AxelarGateway.sol#L323-L379)


</details>

# 


## Token contract should have a blacklist function or modifier
- Severity: Non-Critical
- Confidence: Medium

### Description
NFT thefts have increased recently, so with the addition of stolen NFTs to the platform, NFTs can be converted into liquidity. To prevent this, we recommend adding a blacklist function.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/its/token-implementations/StandardizedTokenLockUnlock.sol
```
 
Line: 7          contract StandardizedTokenLockUnlock is StandardizedToken 
```
Token contract does not contain a blacklist. Consider adding one for increased security.
[contracts/its/token-implementations/StandardizedTokenLockUnlock.sol#L7-L11](contracts/its/token-implementations/StandardizedTokenLockUnlock.sol#L7-L11)


- File: contracts/its/token-implementations/StandardizedTokenMintBurn.sol
```
 
Line: 7          contract StandardizedTokenMintBurn is StandardizedToken 
```
Token contract does not contain a blacklist. Consider adding one for increased security.
[contracts/its/token-implementations/StandardizedTokenMintBurn.sol#L7-L11](contracts/its/token-implementations/StandardizedTokenMintBurn.sol#L7-L11)


</details>

# 


## Missing inheritance
- Severity: Non-Critical
- Confidence: High

### Description
This detector identifies instances where a contract defines functions that match an interface's signature, but does not explicitly inherit from the interface. Explicitly declaring inheritance relationships can improve code readability and maintainability, and ensures that the contract adheres to the defined interface.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 17          contract AxelarGateway is IAxelarGateway, IGovernable, AdminMultisigBase 
```
 should inherit from File: contracts/its/interfaces/IStandardizedTokenProxy.sol
```
 
Line: 9          interface IStandardizedTokenProxy 
```


[contracts/cgp/AxelarGateway.sol#L17-L692](contracts/cgp/AxelarGateway.sol#L17-L692)


- File: contracts/its/proxies/TokenManagerProxy.sol
```
 
Line: 13          contract TokenManagerProxy is ITokenManagerProxy 
```
 should inherit from File: contracts/gmp-sdk/interfaces/IProxy.sol
```
 
Line: 6          interface IProxy 
```


[contracts/its/proxies/TokenManagerProxy.sol#L13-L95](contracts/its/proxies/TokenManagerProxy.sol#L13-L95)


</details>

# 


## Too many digits
- Severity: Non-Critical
- Confidence: Medium

### Description

Literals with many digits are difficult to read and review.


<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 427          function burnToken(bytes calldata params, bytes32) external onlySelf 
```
 uses literals with too many digits:
	- File: contracts/cgp/AxelarGateway.sol
```
 
Line: 435          address depositHandlerAddress = _getCreate2Address(salt, keccak256(abi.encodePacked(type(DepositHandler).creationCode)))
```


[contracts/cgp/AxelarGateway.sol#L427-L453](contracts/cgp/AxelarGateway.sol#L427-L453)


- File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 38          library Create3 
```
 uses literals with too many digits:
	- File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 41          bytes32 internal constant DEPLOYER_BYTECODE_HASH = keccak256(type(CreateDeployer).creationCode)
```


[contracts/gmp-sdk/deploy/Create3.sol#L38-L76](contracts/gmp-sdk/deploy/Create3.sol#L38-L76)


- File: contracts/its/utils/TokenManagerDeployer.sol
```
 
Line: 33          function deployTokenManager(
        bytes32 tokenId,
        uint256 implementationType,
        bytes calldata params
    ) external payable 
```
 uses literals with too many digits:
	- File: contracts/its/utils/TokenManagerDeployer.sol
```
 
Line: 39          bytes memory bytecode = abi.encodePacked(type(TokenManagerProxy).creationCode, args)
```


[contracts/its/utils/TokenManagerDeployer.sol#L33-L42](contracts/its/utils/TokenManagerDeployer.sol#L33-L42)


</details>

# 



## Contracts containing only utility functions should be made into libraries
- Severity: Non-Critical
- Confidence: High

### Description
Contracts that only contain utility functions can be converted into libraries. This enhances modularity and reusability of the code.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
```
 
Line: 5          contract ConstAddressDeployer 
```

[contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L5-L92](contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L5-L92)


- File: contracts/gmp-sdk/deploy/Create3.sol
```
 
Line: 16          contract CreateDeployer 
```

[contracts/gmp-sdk/deploy/Create3.sol#L16-L31](contracts/gmp-sdk/deploy/Create3.sol#L16-L31)


- File: contracts/gmp-sdk/deploy/Create3Deployer.sol
```
 
Line: 12          contract Create3Deployer 
```

[contracts/gmp-sdk/deploy/Create3Deployer.sol#L12-L71](contracts/gmp-sdk/deploy/Create3Deployer.sol#L12-L71)


</details>

# 

