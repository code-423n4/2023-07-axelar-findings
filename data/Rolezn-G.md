## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#gas1-abiencode-is-less-efficient-than-abiencodepacked) | `abi.encode()` is less efficient than `abi.encodepacked()` | 28 | 1484 |
| [GAS&#x2011;2](#gas2-use-assembly-in-place-of-abidecode-to-extract-calldata-values-more-efficiently) | Use assembly in place of `abi.decode` to extract calldata values more efficiently | 2 | 224 |
| [GAS&#x2011;3](#gas3-use-assembly-to-emit-events) | Use assembly to emit events | 49 | 1862 |
| [GAS&#x2011;4](#gas4-counting-down-in-for-statements-is-more-gas-efficient) | Counting down in `for` statements is more gas efficient | 13 | 3341 |
| [GAS&#x2011;5](#gas5-use-assembly-to-write-address-storage-values) | Use `assembly` to write address storage values | 1 | 74 |
| [GAS&#x2011;6](#gas6-multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache) | Multiple accesses of a mapping/array should use a local variable cache | 6 | 480 |
| [GAS&#x2011;7](#gas7-the-result-of-a-function-call-should-be-cached-rather-than-recalling-the-function) | The result of a function call should be cached rather than re-calling the function | 26 | 1300 |
| [GAS&#x2011;8](#gas8-superfluous-event-fields) | Superfluous event fields | 2 | 68 |
| [GAS&#x2011;9](#gas9-use-nested-if-and-avoid-multiple-check-combinations) | Use nested `if` and avoid multiple check combinations | 4 | 24 |
| [GAS&#x2011;10](#gas10-using-xor-^-and-and-&-bitwise-equivalents) | Using XOR (^) and AND (&) bitwise equivalents | 57 | 741 |
| [GAS&#x2011;11](#gas11-using-thisfn-wastes-gas) | Using `this.<fn>()` wastes gas | 2 | 200 |

Total: 196 contexts over 11 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: AxelarGateway.sol

562: return keccak256(abi.encode(PREFIX_TOKEN_MINT_AMOUNT, symbol, day));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L562

```solidity
File: AxelarGateway.sol

584: return keccak256(abi.encode(PREFIX_CONTRACT_CALL_APPROVED, commandId, sourceChain, sourceAddress, contractAddress, payloadHash));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L584

```solidity
File: AxelarGateway.sol

598: abi.encode(
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L598

```solidity
File: ConstAddressDeployer.sol

25: deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L25

```solidity
File: ConstAddressDeployer.sol

47: deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L47

```solidity
File: ConstAddressDeployer.sol

63: bytes32 newSalt = keccak256(abi.encode(sender, salt));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L63

```solidity
File: InterchainProposalExecutor.sol

66: emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L66

```solidity
File: InterchainProposalSender.sol

89: bytes memory payload = abi.encode(msg.sender, interchainCall.calls);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L89

```solidity
File: InterchainTokenService.sol

203: tokenId = keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_ID, chainNameHash, tokenAddress));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L203

```solidity
File: InterchainTokenService.sol

214: tokenId = keccak256(abi.encode(PREFIX_CUSTOM_TOKEN_ID, sender, salt));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L214

```solidity
File: InterchainTokenService.sol

242: params = abi.encode(operator, tokenAddress);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L242

```solidity
File: InterchainTokenService.sol

252: params = abi.encode(operator, tokenAddress);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L252

```solidity
File: InterchainTokenService.sol

267: params = abi.encode(operator, tokenAddress, liquidityPoolAddress);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L267

```solidity
File: InterchainTokenService.sol

313: _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L313

```solidity
File: InterchainTokenService.sol

401: _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L401

```solidity
File: InterchainTokenService.sol

512: payload = abi.encode(SELECTOR_SEND_TOKEN, tokenId, destinationAddress, amount);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L512

```solidity
File: InterchainTokenService.sol

520: payload = abi.encode(SELECTOR_SEND_TOKEN_WITH_DATA, tokenId, destinationAddress, amount, sourceAddress.toBytes(), metadata);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L520

```solidity
File: InterchainTokenService.sol

696: abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L696

```solidity
File: InterchainTokenService.sol

755: bytes memory payload = abi.encode(SELECTOR_DEPLOY_TOKEN_MANAGER, tokenId, tokenManagerType, params);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L755

```solidity
File: InterchainTokenService.sol

780: bytes memory payload = abi.encode(
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L780

```solidity
File: InterchainTokenService.sol

828: return keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_SALT, tokenId));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L828

```solidity
File: ExpressCallHandler.sol

32: slot = uint256(keccak256(abi.encode(PREFIX_EXPRESS_RECEIVE_TOKEN, tokenId, destinationAddress, amount, commandId)));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L32

```solidity
File: ExpressCallHandler.sol

57: abi.encode(
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L57

```solidity
File: FlowLimit.sol

47: slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L47

```solidity
File: FlowLimit.sol

56: slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L56

```solidity
File: StandardizedTokenDeployer.sol

62: bytes memory params = abi.encode(tokenManager, distributor, name, symbol, decimals, mintAmount, mintTo);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L62

```solidity
File: StandardizedTokenDeployer.sol

63: bytecode = abi.encodePacked(type(StandardizedTokenProxy).creationCode, abi.encode(implementationAddress, params));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L63

```solidity
File: TokenManagerDeployer.sol

38: bytes memory args = abi.encode(address(this), implementationType, tokenId, params);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L38



</details>

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |






### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Use assembly in place of `abi.decode` to extract calldata values more efficiently

Instead of using `abi.decode`, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

For example, for a generic `abi.decode` call:
```solidity
(bytes32 varA, bytes32 varB, uint256 varC, uint256 varD) = abi.decode(metadata, (bytes32, bytes32, uint256, uint256));
```

We can use the following assembly call to extract the relevant metadata:

```solidity
        bytes32 varA;
        bytes32 varB;
        uint256 varC;
        uint256 varD;
        { // used to discard `data` variable and avoid extra stack manipulation
            bytes calldata data = metadata;
            assembly {
                varA := calldataload(add(data.offset, 0x00))
                varB := calldataload(add(data.offset, 0x20))
                varC := calldataload(add(data.offset, 0x40))
                varD := calldataload(add(data.offset, 0x60))
            }
        }
```

#### <ins>Proof Of Concept</ins>

```solidity
File: AxelarGateway.sol

422: (string memory symbol, address account, uint256 amount) = abi.decode(params, (string, address, uint256));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L422

```solidity
File: InterchainTokenService.sol

600: (, bytes32 tokenId, bytes memory destinationAddressBytes, uint256 amount) = abi.decode(payload, (uint256, bytes32, bytes, uint256));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L600






### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic `emit` event for `eventSentAmountExample`:
```solidity
// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```solidity
        assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: AxelarGateway.sol

104: emit TokenSent(msg.sender, destinationChain, destinationAddress, symbol, amount);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L104

```solidity
File: AxelarGateway.sol

112: emit ContractCall(msg.sender, destinationChain, destinationContractAddress, keccak256(payload), payload);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L112

```solidity
File: AxelarGateway.sol

123: emit ContractCallWithToken(msg.sender, destinationChain, destinationContractAddress, keccak256(payload), payload, symbol, amount);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L123

```solidity
File: AxelarGateway.sol

289: emit Upgraded(newImplementation);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L289

```solidity
File: AxelarGateway.sol

319: emit OperatorshipTransferred(newOperatorsData);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L319

```solidity
File: AxelarGateway.sol

376: if (success) emit Executed(commandId);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L376

```solidity
File: AxelarGateway.sol

418: emit TokenDeployed(symbol, tokenAddress);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L418

```solidity
File: AxelarGateway.sol

466: emit ContractCallApproved(commandId, sourceChain, sourceAddress, contractAddress, payloadHash, sourceTxHash, sourceEventIndex);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L466

```solidity
File: AxelarGateway.sol

498: emit OperatorshipTransferred(newOperatorsData);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L498

```solidity
File: AxelarGateway.sol

630: emit TokenMintLimitUpdated(symbol, limit);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L630

```solidity
File: AxelarGateway.sol

682: emit GovernanceTransferred(getAddress(KEY_GOVERNANCE), newGovernance);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L682

```solidity
File: AxelarGateway.sol

688: emit MintLimiterTransferred(getAddress(KEY_MINT_LIMITER), newMintLimiter);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L688

```solidity
File: MultisigBase.sol

178: emit SignersRotated(newAccounts, newThreshold);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L178

```solidity
File: AxelarServiceGovernance.sol

61: emit MultisigExecuted(proposalHash, target, callData, nativeValue);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L61

```solidity
File: AxelarServiceGovernance.sol

89: emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);
94: emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);
99: emit MultisigApproved(proposalHash, target, callData, nativeValue);
104: emit MultisigCancelled(proposalHash, target, callData, nativeValue);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L89

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L94

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L99

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L104



```solidity
File: InterchainGovernance.sol

78: emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L78

```solidity
File: InterchainGovernance.sol

130: emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);
135: emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L130

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L135



```solidity
File: Upgradable.sol

27: emit OwnershipTransferred(newOwner);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol#L27

```solidity
File: Upgradable.sol

54: emit Upgraded(newImplementation);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol#L54

```solidity
File: ConstAddressDeployer.sol

90: emit Deployed(keccak256(bytecode), salt, deployedAddress_);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L90

```solidity
File: Upgradable.sol

66: emit Upgraded(newImplementation);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L66

```solidity
File: InterchainProposalExecutor.sol

66: emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L66

```solidity
File: InterchainProposalExecutor.sol

98: emit WhitelistedProposalCallerSet(sourceChain, sourceCaller, whitelisted);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L98

```solidity
File: InterchainProposalExecutor.sol

113: emit WhitelistedProposalSenderSet(sourceChain, sourceSender, whitelisted);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L113

```solidity
File: InterchainTokenService.sol

351: emit CustomTokenIdClaimed(tokenId, deployer_, salt);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L351

```solidity
File: InterchainTokenService.sol

375: emit CustomTokenIdClaimed(tokenId, deployer_, salt);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L375

```solidity
File: InterchainTokenService.sol

514: emit TokenSent(tokenId, destinationChain, destinationAddress, amount);
522: emit TokenSentWithData(tokenId, destinationChain, destinationAddress, amount, sourceAddress, metadata);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L514

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L522



```solidity
File: InterchainTokenService.sol

611: emit TokenReceived(tokenId, sourceChain, destinationAddress, amount);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L611

```solidity
File: InterchainTokenService.sol

659: emit TokenReceivedWithData(tokenId, sourceChain, destinationAddress, amount, sourceAddress, data);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L659

```solidity
File: InterchainTokenService.sol

757: emit RemoteTokenManagerDeploymentInitialized(tokenId, destinationChain, gasValue, tokenManagerType, params);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L757

```solidity
File: InterchainTokenService.sol

819: emit TokenManagerDeployed(tokenId, tokenManagerType, params);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L819

```solidity
File: InterchainTokenService.sol

869: emit StandardizedTokenDeployed(tokenId, name, symbol, decimals, mintAmount, mintTo);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L869

```solidity
File: RemoteAddressValidator.sol

88: emit TrustedAddressAdded(chain, addr);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L88

```solidity
File: RemoteAddressValidator.sol

99: emit TrustedAddressRemoved(chain);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L99

```solidity
File: RemoteAddressValidator.sol

111: emit GatewaySupportedChainAdded(chainName);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L111

```solidity
File: RemoteAddressValidator.sol

124: emit GatewaySupportedChainRemoved(chainName);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L124

```solidity
File: Pausable.sol

15: emit TestEvent();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/test/utils/Pausable.sol#L15

```solidity
File: Distributable.sol

43: emit DistributorChanged(distributor_);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Distributable.sol#L43

```solidity
File: ExpressCallHandler.sol

95: emit ExpressReceive(tokenId, destinationAddress, amount, commandId, expressCaller);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L95

```solidity
File: ExpressCallHandler.sol

137: emit ExpressReceiveWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, expressCaller);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L137

```solidity
File: ExpressCallHandler.sol

216: emit ExpressExecutionFulfilled(tokenId, destinationAddress, amount, commandId, expressCaller);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L216

```solidity
File: FlowLimit.sol

38: emit FlowLimitSet(flowLimit);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L38

```solidity
File: Operatable.sol

43: emit OperatorChanged(operator_);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Operatable.sol#L43

```solidity
File: Pausable.sol

46: emit PausedSet(paused);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Pausable.sol#L46



</details>



### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: AxelarGateway.sol

270: for (uint256 i; i < length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L270

```solidity
File: AxelarGateway.sol

344: for (uint256 i; i < commandsLength; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L344

```solidity
File: MultisigBase.sol

122: for (uint256 i; i < length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L122

```solidity
File: MultisigBase.sol

153: for (uint256 i; i < length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L153



```solidity
File: InterchainProposalExecutor.sol

74: for (uint256 i = 0; i < calls.length; i++) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74

```solidity
File: InterchainProposalSender.sol

63: for (uint256 i = 0; i < interchainCalls.length; ) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63

```solidity
File: InterchainProposalSender.sol

106: for (uint256 i = 0; i < interchainCalls.length; ) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106

```solidity
File: InterchainTokenService.sol

537: for (uint256 i; i < length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L537

```solidity
File: RemoteAddressValidator.sol

44: for (uint256 i; i < length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L44

```solidity
File: RemoteAddressValidator.sol

56: for (uint256 i; i < length; i++) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L56

```solidity
File: RemoteAddressValidator.sol

108: for (uint256 i; i < length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L108

```solidity
File: RemoteAddressValidator.sol

121: for (uint256 i; i < length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L121

```solidity
File: Multicall.sol

24: for (uint256 i = 0; i < data.length; ++i) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L24



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |



### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Use `assembly` to write address storage values

#### <ins>Proof Of Concept</ins>

```solidity
File: TimeLock.sol

22: _minimumTimeLockDelay = minimumTimeDelay;
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L22







### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


```solidity
File: MultisigBase.sol

172: if (signers.isSigner[account]) revert DuplicateSigner(account);
175: signers.isSigner[account] = true;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L172

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L175



```solidity
File: AxelarServiceGovernance.sol

55: if (!multisigApprovals[proposalHash]) revert NotApproved();
57: multisigApprovals[proposalHash] = false;
102: multisigApprovals[proposalHash] = false;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L57

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L102



```solidity
File: AxelarServiceGovernance.sol

97: multisigApprovals[proposalHash] = true;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L97






### <a href="#gas-summary">[GAS&#x2011;16]</a><a name="GAS&#x2011;16"> Multiple Address Mappings Can Be Combined Into A Single Mapping Of An Address To A Struct, Where Appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot.

#### <ins>Proof Of Concept</ins>


```solidity
File: MultisigBase.sol

15: mapping(address => bool) hasVoted;
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L15

```solidity
File: MultisigBase.sol

21: mapping(address => bool) isSigner;
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L21

```solidity
File: InterchainProposalExecutor.sol

24: mapping(string => mapping(address => bool)) public whitelistedCallers;
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24

```solidity
File: InterchainProposalExecutor.sol

27: mapping(string => mapping(address => bool)) public whitelistedSenders;
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L27






### <a href="#gas-summary">[GAS&#x2011;17]</a><a name="GAS&#x2011;17"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#gas-summary">[GAS&#x2011;18]</a><a name="GAS&#x2011;18"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
File: InterchainProposalSender.sol

107: totalGas += interchainCalls[i].gas;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L107





### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: AxelarGateway.sol

287: if (AxelarGateway(newImplementation).contractId() != contractId()) revert InvalidImplementation();
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L287



```solidity
File: Upgradable.sol

45: if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol#L45


```solidity
File: BaseProxy.sol

49: calldatacopy(0, 0, calldatasize())
51: let result := delegatecall(gas(), implementation_, 0, calldatasize(), 0, 0)
52: returndatacopy(0, 0, returndatasize())
56: revert(0, returndatasize())
59: return(0, returndatasize())

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/BaseProxy.sol#L49

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/BaseProxy.sol#L51

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/BaseProxy.sol#L52

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/BaseProxy.sol#L56

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/BaseProxy.sol#L59



```solidity
File: FixedProxy.sol

41: calldatacopy(0, 0, calldatasize())
43: let result := delegatecall(gas(), implementation_, 0, calldatasize(), 0, 0)
44: returndatacopy(0, 0, returndatasize())
48: revert(0, returndatasize())
51: return(0, returndatasize())

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L41

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L43

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L44

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L48

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L51



```solidity
File: InitProxy.sol

49: bytes32 id = contractId();
50: if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L49-L50

```solidity
File: Upgradable.sol

57: if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L57



```solidity
File: InterchainToken.sol

21: function getTokenManager() public view virtual returns (ITokenManager tokenManager);
32: function tokenManagerRequiresApproval() public view virtual returns (bool);
50: ITokenManager tokenManager = getTokenManager();
92: ITokenManager tokenManager = getTokenManager();
54: if (tokenManagerRequiresApproval()) {
93: if (tokenManagerRequiresApproval()) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L21

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L32

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L50

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L92

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L54

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L93



```solidity
File: TokenManagerProxy.sol

76: calldatacopy(0, 0, calldatasize())
78: let result := delegatecall(gas(), implementaion_, 0, calldatasize(), 0, 0)
79: returndatacopy(0, 0, returndatasize())
83: revert(0, returndatasize())
86: return(0, returndatasize())

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L76

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L78

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L79

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L83

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L86





</details>




### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Superfluous event fields

`block.number` and `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

#### <ins>Proof Of Concept</ins>


```solidity
File: IInterchainGovernance.sol

20: event ProposalExecuted(bytes32 indexed proposalHash, address indexed target, bytes callData, uint256 value, uint256 indexed timestamp);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IInterchainGovernance.sol#L20

```solidity
File: IMultisigBase.sol

22: event SignersRotated(address[] newAccounts, uint256 newThreshold);
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IMultisigBase.sol#L22




### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Use nested `if` and avoid multiple check combinations

Using nested `if`, is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>Proof Of Concept</ins>

```solidity
File: AxelarGateway.sol

446: if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L446

```solidity
File: AxelarGateway.sol

635: if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L635

```solidity
File: InitProxy.sol

50: if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L50

```solidity
File: RemoteAddressValidator.sol

58: if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L58




#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }
    
    contract Contract0 {
    
        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }
    
    }
    
    contract Contract1 {
    
        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |




### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: AxelarGateway.sol

255: if (newGovernance == address(0)) revert InvalidGovernance();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L255

```solidity
File: AxelarGateway.sol

261: if (newMintLimiter == address(0)) revert InvalidMintLimiter();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L261

```solidity
File: AxelarGateway.sol

342: if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();
352: if (commandHash == SELECTOR_DEPLOY_TOKEN) {
354: } else if (commandHash == SELECTOR_MINT_TOKEN) {
356: } else if (commandHash == SELECTOR_APPROVE_CONTRACT_CALL) {
358: } else if (commandHash == SELECTOR_APPROVE_CONTRACT_CALL_WITH_MINT) {
360: } else if (commandHash == SELECTOR_BURN_TOKEN) {
362: } else if (commandHash == SELECTOR_TRANSFER_OPERATORSHIP) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L342

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L352

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L354

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L356

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L358

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L360

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L362



```solidity
File: AxelarGateway.sol

394: if (tokenAddress == address(0)) {
409: if (tokenAddress.code.length == uint256(0)) revert TokenContractDoesNotExist(tokenAddress);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L394

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L409



```solidity
File: AxelarGateway.sol

432: if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L432

```solidity
File: AxelarGateway.sol

519: if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L519

```solidity
File: AxelarGateway.sol

537: if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
538: if (amount == 0) revert InvalidAmount();
542: if (tokenType == TokenType.External) {
544: } else if (tokenType == TokenType.InternalBurnableFrom) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L537

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L538

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L542

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L544



```solidity
File: MultisigBase.sol

161: if (newThreshold == 0) revert InvalidSignerThreshold();
173: if (account == address(0)) revert InvalidSigners();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L161

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L173



```solidity
File: AxelarServiceGovernance.sol

86: if (command == ServiceGovernanceCommand.ScheduleTimeLockProposal) {
91: } else if (command == ServiceGovernanceCommand.CancelTimeLockProposal) {
96: } else if (command == ServiceGovernanceCommand.ApproveMultisigProposal) {
101: } else if (command == ServiceGovernanceCommand.CancelMultisigApproval) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L86

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L91

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L96

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L101



```solidity
File: InterchainGovernance.sol

92: if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)
100: if (target == address(0)) revert InvalidTarget();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L92

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L100



```solidity
File: InterchainGovernance.sol

127: if (command == GovernanceCommand.ScheduleTimeLockProposal) {
132: } else if (command == GovernanceCommand.CancelTimeLockProposal) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L127

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L132



```solidity
File: Upgradable.sol

25: if (newOwner == address(0)) revert InvalidOwner();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol#L25

```solidity
File: ConstAddressDeployer.sol

81: if (bytecode.length == 0) revert EmptyBytecode();
88: if (deployedAddress_ == address(0)) revert FailedDeploy();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L81

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L88



```solidity
File: FinalProxy.sol

39: if (implementation_ == address(0)) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L39

```solidity
File: FinalProxy.sol

63: if (implementation_.code.length == 0) implementation_ = address(0);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L63

```solidity
File: Bytes32String.sol

13: if (stringBytes.length == 0 || stringBytes.length > 31) revert InvalidStringLength();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/Bytes32String.sol#L13

```solidity
File: TimeLock.sol

51: if (hash == 0) revert InvalidTimeLockHash();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L51

```solidity
File: TimeLock.sol

68: if (hash == 0) revert InvalidTimeLockHash();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L68

```solidity
File: TimeLock.sol

82: if (hash == 0 || eta == 0) revert InvalidTimeLockHash();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L82

```solidity
File: InterchainTokenService.sol

399: TokenManagerType tokenManagerType = distributor == tokenManagerAddress ? TokenManagerType.MINT_BURN : TokenManagerType.LOCK_UNLOCK;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L399

```solidity
File: InterchainTokenService.sol

565: if (implementation == address(0)) revert ZeroAddress();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L565

```solidity
File: InterchainTokenService.sol

581: if (selector == SELECTOR_SEND_TOKEN) {
583: } else if (selector == SELECTOR_SEND_TOKEN_WITH_DATA) {
585: } else if (selector == SELECTOR_DEPLOY_TOKEN_MANAGER) {
587: } else if (selector == SELECTOR_DEPLOY_AND_REGISTER_STANDARDIZED_TOKEN) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L581

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L583

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L585

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L587



```solidity
File: InterchainTokenService.sol

609: if (expressCaller == address(0)) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L609

```solidity
File: InterchainTokenService.sol

692: TokenManagerType tokenManagerType = distributor == tokenManagerAddress ? TokenManagerType.MINT_BURN : TokenManagerType.LOCK_UNLOCK;
696: abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L692

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L696



```solidity
File: RemoteAddressValidator.sol

72: if (sourceAddressHash == interchainTokenServiceAddressHash) {
75: return sourceAddressHash == remoteAddressHashes[sourceChain];

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L72

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L75



```solidity
File: RemoteAddressValidator.sol

84: if (bytes(chain).length == 0) revert ZeroStringLength();
85: if (bytes(addr).length == 0) revert ZeroStringLength();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L84-L85

```solidity
File: RemoteAddressValidator.sol

96: if (bytes(chain).length == 0) revert ZeroStringLength();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L96

```solidity
File: RemoteAddressValidator.sol

135: if (bytes(remoteAddress).length == 0) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L135

```solidity
File: TokenManager.sol

68: if (operatorBytes.length == 0) {

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L68

```solidity
File: FlowLimit.sol

113: if (flowLimit == 0) return;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L113

```solidity
File: FlowLimit.sol

126: if (flowLimit == 0) return;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L126

```solidity
File: StandardizedTokenDeployer.sol

60: address implementationAddress = distributor == tokenManager ? implementationMintBurnAddress : implementationLockUnlockAddress;
66: if (tokenAddress.code.length == 0) revert TokenDeploymentFailed();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L60

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L66



```solidity
File: TokenManagerDeployer.sol

41: if (tokenManagerAddress.code.length == 0) revert TokenManagerDeploymentFailed();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L41



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |




### <a href="#gas-summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Using `this.<fn>()` wastes gas

Calling an external function internally, through the use of `this` wastes the gas overhead of calling an external function (100 gas).

#### <ins>Proof Of Concept</ins>

```solidity
File: Upgradable.sol

49: (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol#L49

```solidity
File: Upgradable.sol

61: (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L61






