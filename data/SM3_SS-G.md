

#  Summary for GAS findings

| NO | Details | Instances |
|----|---------|-----------|
|[G-01]|   Use calldata instead of memory for function arguments that do not get mutated | 10 | 
|[G-02]|   Use do while loops instead of for loops | 1 | 
|[G-03]|   Cache state variables outside of loop to avoid reading storage on every iteration | 4 | 
|[G-04]|   With assembly, .call (bool success) transfer can be done gas-optimized | 5 | 
|[G-05]|   x += y costs more gas than x = x + y for state variables | 1 | 
|[G-06]|   If statements that use && can be refactored into nested if statements | 6 | 
|[G-07]|   Using XOR (^) and OR  bitwise equivalents | 3 | 
|[G-08]|   Using a positive conditional flow to save a NOT opcode | 17 | 
|[G-09]|   Make 3 event parameters indexed when possible | 8 | 
|[G-10]|   Use constants instead of type(uintx).max | 10 | 
|[G-11]|   Using bools for storage incurs overhead | 6 | 
|[G-12]|   Amounts should be checked for 0 before calling a transfer | 9 | 
|[G-13]|   Don’t Initialize Variables with Default Value | 4 | 
|[G-14]|   abi.encode() is less efficient than abi.encodePacked() | 31 | 
|[G-15]|   Use hardcode address instead address(this) | 25 | 
|[G-16]|   A modifier used only once and not being inherited should be inlined to save gas | 8 | 
|[G-17]|   Use assembly for loops | 10 | 
|[G-18]|   Failure to check the zero address in the constructor causes the contract to be deployed again | 4 | 
|[G-19]|   Functions guaranteed to revert when called by normal users can be marked payable | 14 | 
|[G-20]|   Use assembly for math (add, sub, mul, div) | 4 | 
|[G-21]|   Use assembly when getting a contract’s balance of ETH | 1 | 
|[G-22]|   use Fixed-size Arrays are cheaper than dynamic one if it is possiable  to save gas  | 5 | 
|[G-23]|   Use assembly to perform efficient back-to-back calls | 2 | 
|[G-24]|   Use Modifiers Instead of Functions To Save Gas | 3 | 
|[G-25]|   Use custom errors rather than revert()/require() strings to save gas | 140 | 
|[G-26]|   Enable the Solidity Compiler Optimizer to save gas  |  | 



## [G-01] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
file:  contracts/cgp/auth/MultisigBase.sol

142   function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners 

149   function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L142

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L149

```solidity
file:   contracts/cgp/interfaces/IMultisigBase.sol

73     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IMultisigBase.sol#L73

```solidity
file:   contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

42        function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42-L46

```solidity
file:   contracts/gmp-sdk/deploy/Create3.sol

21    function deploy(bytes memory bytecode) external 

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol#L21


```solidity
file:   contracts/interchain-governance-executor/InterchainProposalExecutor.sol

73    function _executeProposal(InterchainCalls.Call[] memory calls) internal 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L73

```solidity
file:   contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

15      function sendProposals(InterchainCalls.InterchainCall[] memory calls) external payable;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L15

```solidity
file:     contracts/its/interchain-token-service/InterchainTokenService.sol

559      function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L559

```solidity
file:    contracts/its/interfaces/IMulticall.sol

18      function multicall(bytes[] calldata data) external payable returns (bytes[] memory results);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IMulticall.sol#L18

```solidity
file:   contracts/its/utils/Multicall.sol
  
22     function multicall(bytes[] calldata data) public payable returns (bytes[] memory results)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L22



## [G-02] Use do while loops instead of for loops

#### in this for loop useing 463605 amount of gas 
#### but when use do while loop 462611 amount of gas
```solidity
file:

270         for (uint256 i; i < length; ++i) {
            string memory symbol = symbols[i];
            uint256 limit = limits[i];

            if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

            _setTokenMintLimit(symbol, limit);
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L270-L277


### Recommanded code 

```solidity
function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {
    uint256 length = symbols.length;
    if (length != limits.length) revert InvalidSetMintLimitsParams();

    uint256 i = 0;
    do {
        string memory symbol = symbols[i];
        uint256 limit = limits[i];

        if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

        _setTokenMintLimit(symbol, limit);

        i++;
    } while (i < length);
}

```

## [G-03] Cache state variables outside of loop to avoid reading storage on every iteration

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.

### Cache supportedByGateway and supportedByGateway to avoid storage reads on each loop iteration

```solidity
file:     contracts/its/remote-address-validator/RemoteAddressValidator.sol


108             for (uint256 i; i < length; ++i) {
            string calldata chainName = chainNames[i];
            supportedByGateway[chainName] = true;
            emit GatewaySupportedChainAdded(chainName);
        }


119        function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {
        uint256 length = chainNames.length;
        for (uint256 i; i < length; ++i) {
            string calldata chainName = chainNames[i];
            supportedByGateway[chainName] = false;
            emit GatewaySupportedChainRemoved(chainName);
        }
    }
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L108-L112

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L119-L126



### Cache votingPerTopic to avoid storage reads on each loop iteration


```solidity
file:      contracts/cgp/auth/MultisigBase.sol

122          for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L122-L125


### use they  already exist count variable instead of [signers.accounts[i]] 

```solidity
file:    contracts/cgp/auth/MultisigBase.sol

70             for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L70-L73

## [G-04] With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
file:   contracts/cgp/AxelarGateway.sol

374     (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L374 


```solidity
file:   contracts/cgp/util/Caller.sol

18     (bool success, ) = target.call{ value: nativeValue }(callData);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L18


```solidity
file:  contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

50    (bool success, ) = deployedAddress_.call(init);   

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50

```solidity
file:   contracts/gmp-sdk/deploy/Create3Deployer.sol

57     (bool success, ) = deployedAddress_.call(init);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L57

```solidity
file:   contracts/interchain-governance-executor/InterchainProposalExecutor.sol

76     (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76


## [G-05] x += y costs more gas than x = x + y for state variables


```solidity
file:    contracts/interchain-governance-executor/InterchainProposalSender.sol

107     totalGas += interchainCalls[i].gas;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L107



## [G-06] If statements that use && can be refactored into nested if statements


```solidity

file:    contracts/cgp/AxelarGateway.sol

88      if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();

446     if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

635     if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L88

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L446

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L635

```solidity
file:   contracts/gmp-sdk/upgradable/InitProxy.sol

50     if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L50

```solidity
file:   contracts/gmp-sdk/upgradable/Proxy.sol

33     if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol#L33

```solidity
file:   contracts/its/remote-address-validator/RemoteAddressValidator.sol

58     if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L58


## [G-07] Using XOR (^) and OR (|) bitwise equivalents

Estimated savings: 73 gas

```solidity
file:   contracts/cgp/AxelarGateway.sol

342     if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();

446     if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L342

```solidity
file:   contracts/cgp/governance/InterchainGovernance.sol

92    if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L92


## [G-08] Using a positive conditional flow to save a NOT opcode

```solidity
file:    contracts/cgp/AxelarGateway.sol

296      if (!success) revert SetupFailed();

363      if (!allowOperatorshipTransfer) continue;

402      if (!success) revert TokenDeployFailed(symbol);

446      if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L296

```solidity
file:    contracts/cgp/auth/MultisigBase.sol

45     if (!signers.isSigner[msg.sender]) revert NotSigner();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L45

```solidity
file:  contracts/cgp/governance/AxelarServiceGovernance.sol

55    if (!multisigApprovals[proposalHash]) revert NotApproved();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55

```solidity
file:    contracts/cgp/util/Caller.sol

19    if (!success) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L19

```solidity
file:  contracts/gmp-sdk/upgradable/Upgradable.sol

51    if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L51 

```solidity
file:   contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

51     if (!success) revert FailedInit();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L51

```solidity
file:    contracts/gmp-sdk/deploy/Create3Deployer.sol

58      if (!success) revert FailedInit();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L58

```solidity
file:   contracts/gmp-sdk/upgradable/FinalProxy.sol
 
82    if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L82 

```solidity
file:  contracts/interchain-governance-executor/InterchainProposalExecutor.sol

49     if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) 

57     if (!whitelistedCallers[sourceChain][interchainProposalCaller]) 

78     if (!success) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L49

```solidity
file:    contracts/its/interchain-token-service/InterchainTokenService.sol

124     if (!remoteAddressValidator.validateSender(sourceChain, sourceAddress)) revert NotRemoteService();

816     if (!success) 

866      if (!success)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L124



## [G-09] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed

```solidity

file:   contracts/its/interfaces/IDistributable.sol

8       event DistributorChanged(address distributor);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IDistributable.sol#L8

```solidity
file:  contracts/its/interfaces/IFlowLimit.sol
 
8      event FlowLimitSet(uint256 flowLimit);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IFlowLimit.sol#L8

```solidity
file:   contracts/its/interfaces/IOperatable.sol

8      event OperatorChanged(address operator);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IOperatable.sol#L8

```solidity
file:   contracts/its/interfaces/IPausable.sol

11       event PausedSet(bool paused);


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IPausable.sol#L11

```solidity

file:    contracts/its/interfaces/IRemoteAddressValidator.sol

14      event TrustedAddressAdded(string souceChain, string sourceAddress);
15      event TrustedAddressRemoved(string souceChain);
16      event GatewaySupportedChainAdded(string chain);
17      event GatewaySupportedChainRemoved(string chain);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol#L14


## [G-10] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file:  contracts/cgp/governance/AxelarServiceGovernance.sol

79    if (commandId > uint256(type(ServiceGovernanceCommand).max))

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L79

```solidity
file:

120     if (commandId > uint256(type(GovernanceCommand).max))

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L120

```solidity

file:   contracts/its/interchain-token/InterchainToken.sol

56      if (allowance_ != type(uint256).max) {
57      if (allowance_ > type(uint256).max - amount) {
58      allowance_ = type(uint256).max - amount;
88      if (_allowance != type(uint256).max) 

95      if (allowance_ != type(uint256).max) {
96      if (allowance_ > type(uint256).max - amount) {
97      allowance_ = type(uint256).max - amount;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L56

```solidity
file:     contracts/its/interchain-token-service/InterchainTokenService.sol

104      if (tokenManagerImplementations.length != uint256(type(TokenManagerType).max) + 1) revert LengthMismatch();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L104

## [G‑11] Using bools for storage incurs overhead

  // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

```solidity
file:   contracts/cgp/auth/MultisigBase.sol
15      mapping(address => bool) hasVoted;
21      mapping(address => bool) isSigner;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L15

```solidity
file:  contracts/cgp/governance/AxelarServiceGovernance.sol
22     mapping(bytes32 => bool) public multisigApprovals;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L22
```solidity
file:   contracts/interchain-governance-executor/InterchainProposalExecutor.sol
24      mapping(string => mapping(address => bool)) public whitelistedCallers;
27      mapping(string => mapping(address => bool)) public whitelistedSenders;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24

```solidity
file:  contracts/its/remote-address-validator/RemoteAddressValidator.sol
19     mapping(string => bool) public supportedByGateway;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L19


## [G-12] Amounts should be checked for 0 before calling a transfer 

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:
```solidity
file:    contracts/cgp/AxelarGateway.sol
524      IERC20(tokenAddress).safeTransfer(account, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L524

```solidity
file:   contracts/its/interchain-token/InterchainToken.sol

66      tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

104     tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L66

```solidity
file:   contracts/its/interchain-token-service/InterchainTokenService.sol

451     SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

482     SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L451

```solidity
file:  contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

82     SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount);
98     SafeTokenTransferFrom.safeTransferFrom(token, liquidityPool(), to, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L82

```solidity
file:   contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

48     SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);
64     SafeTokenTransfer.safeTransfer(token, to, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L48

## [G-13] Don’t Initialize Variables with Default Value
Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it’s default value costs unnecesary gas.

```solidity
file:    contracts/interchain-governance-executor/InterchainProposalExecutor.sol

74      for (uint256 i = 0; i < calls.length; i++)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74

```solidity
file:  contracts/interchain-governance-executor/InterchainProposalSender.sol

63     for (uint256 i = 0; i < interchainCalls.length; )

106    for (uint256 i = 0; i < interchainCalls.length; )

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63 

```solidity
file:   contracts/its/utils/Multicall.sol

24     for (uint256 i = 0; i < data.length; ++i) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L24


## [G-14] abi.encode() is less efficient than abi.encodePacked()

use abi.encodePacked() where possible to save gas

```solidity
file:   contracts/its/interchain-token-service/InterchainTokenService.sol
203    tokenId = keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_ID, chainNameHash, tokenAddress));
214    tokenId = keccak256(abi.encode(PREFIX_CUSTOM_TOKEN_ID, sender, salt));
242    params = abi.encode(operator, tokenAddress);
252    params = abi.encode(operator, tokenAddress);
267    params = abi.encode(operator, tokenAddress, liquidityPoolAddress);
313    _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));
401    _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));
512    payload = abi.encode(SELECTOR_SEND_TOKEN, tokenId, destinationAddress, amount);
520    payload = abi.encode(SELECTOR_SEND_TOKEN_WITH_DATA, tokenId, destinationAddress, amount, sourceAddress.toBytes(), metadata);
696    abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
755    bytes memory payload = abi.encode(SELECTOR_DEPLOY_TOKEN_MANAGER, tokenId, tokenManagerType, params);
780    bytes memory payload = abi.encode(
828    return keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_SALT, tokenId));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L203

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L828

```solidity
file:  contracts/its/utils/ExpressCallHandler.sol

32      slot = uint256(keccak256(abi.encode(PREFIX_EXPRESS_RECEIVE_TOKEN, tokenId, destinationAddress, amount, commandId)))

57      abi.encode(

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L32

```solidity
file:   contracts/its/utils/FlowLimit.sol

47      slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)));

56      slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L47

```solidity 
file:     contracts/its/utils/StandardizedTokenDeployer.sol
62        bytes memory params = abi.encode(tokenManager, distributor, name, symbol, decimals, mintAmount, mintTo);
63        bytecode = abi.encodePacked(type(StandardizedTokenProxy).creationCode, abi.encode(implementationAddress, params));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L62

```solidity

file:   contracts/its/utils/TokenManagerDeployer.sol

38     bytes memory args = abi.encode(address(this), implementationType, tokenId, params);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L38

```solidity
file:   contracts/cgp/AxelarGateway.sol

562     return keccak256(abi.encode(PREFIX_TOKEN_MINT_AMOUNT, symbol, day));

584     return keccak256(abi.encode(PREFIX_CONTRACT_CALL_APPROVED, commandId, sourceChain, sourceAddress, contractAddress, payloadHash));

597               keccak256(
                abi.encode(
                    PREFIX_CONTRACT_CALL_APPROVED_WITH_MINT,
                    commandId,
                    sourceChain,
                    sourceAddress,
                    contractAddress,
                    payloadHash,
                    symbol,
                    amount
                )
            );


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L562

```solidity
file:    contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
25       deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

47       deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

63       bytes32 newSalt = keccak256(abi.encode(sender, salt));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L25

```solidity
file: contracts/gmp-sdk/deploy/Create3Deployer.sol

30    bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));
54    bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));
68    bytes32 deploySalt = keccak256(abi.encode(sender, salt));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L30

```solidity
file:  contracts/interchain-governance-executor/InterchainProposalExecutor.sol

66     emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L66

```solidity
file:  contracts/interchain-governance-executor/InterchainProposalSender.sol
89     bytes memory payload = abi.encode(msg.sender, interchainCall.calls);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L89

## [G-15] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this. In Solidity, the address(this) expression returns the address of the current contract instance. This expression is commonly used to send or receive Ether to or from the contract.

The statement "Use hardcoded address instead of address(this) to save gas cost" means that using a hardcoded address instead of the address(this) expression can be more gas-efficient when sending or receiving Ether to or from the contract. This is because using the address(this) expression requires additional gas to be consumed to retrieve the address of the current contract, while using a hardcoded address does not.

For example, consider the following code:
```solidity
contract MyContract {
  address public myAddress = 0x1234567890123456789012345678901234567890;
  
  function receiveEther() public payable {
    // Receive Ether using address(this)
    // msg.value is the value of the Ether being sent
    address(this).transfer(msg.value);
  }
  
  function receiveEtherHardcoded() public payable {
    // Receive Ether using hardcoded address
    // msg.value is the value of the Ether being sent
    myAddress.transfer(msg.value);
  }
}

```
In this code, the receiveEther() function receives Ether using address(this), while the receiveEtherHardcoded() function receives Ether using a hardcoded address. Although both functions achieve the same result, the receiveEtherHardcoded() function is more gas-efficient, as it does not require additional gas to retrieve the address of the current contract.

Therefore, using a hardcoded address instead of the address(this) expression can be a gas-efficient way to send or receive Ether to or from a smart contract, as it reduces the gas cost of the contract. However, it is important to ensure that the hardcoded address is correct and does not change during the contract's lifetime.

```solidity
file:   contracts/gmp-sdk/upgradable/Upgradable.sol

23      implementationAddress = address(this);
31      if (address(this) == implementationAddress) revert NotProxy();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L23 

```solidity
file:  contracts/cgp/util/Caller.sol

16     if (nativeValue > address(this).balance) revert InsufficientBalance();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L16 

```solidity
file:   contracts/gmp-sdk/deploy/Create3.sol

52      deployed = deployedAddress(address(this), salt);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol#L52 

```solidity
file:   contracts/gmp-sdk/deploy/Create3Deployer.sol

69      return Create3.deployedAddress(address(this), deploySalt);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L69

```solidity
file:   contracts/cgp/AxelarGateway.sol

73      if (msg.sender != address(this)) revert NotSelf();
374     (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));
443     abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))
449     depositHandler.destroy(address(this));
543     IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);
616     return address(uint160(uint256(keccak256(abi.encodePacked(bytes1(0xff), address(this), salt, codeHash)))));
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L73

```solidity
file:  contracts/gmp-sdk/upgradable/FinalProxy.sol
61     implementation_ = Create3.deployedAddress(address(this), FINAL_IMPLEMENTATION_SALT);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L61

```solidity
file:    contracts/its/interchain-token-service/InterchainTokenService.sol
162      tokenManagerAddress = deployer.deployedAddress(address(this), tokenId);
193      tokenAddress = deployer.deployedAddress(address(this), tokenId);
313      _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));
696       abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
716       address(this),

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L162
```solidity
file:   contracts/its/token-manager/TokenManager.sol
205     tokenId = ITokenManagerProxy(address(this)).tokenId();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L205

```solidity
file:    contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
46       uint256 balance = token.balanceOf(address(this));
48       SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);
51       return IERC20(token).balanceOf(address(this)) - balance;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L46

```solidity
file:   contracts/its/utils/Implementation.sol

19      implementationAddress = address(this);

27     if (implementationAddress == address(this)) revert NotProxy();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Implementation.sol#L19


```solidity
file:   contracts/its/utils/Multicall.sol
25     (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L25

```solidity
file:  contracts/its/utils/TokenManagerDeployer.sol

38     bytes memory args = abi.encode(address(this), implementationType, tokenId, params);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L38



## [G-16] A modifier used only once and not being inherited should be inlined to save gas


```solidity
file:    contracts/cgp/auth/MultisigBase.sol

142     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners 


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L142

```solidity
file:  contracts/gmp-sdk/upgradable/Upgradable.sol

78    function setup(bytes calldata data) external override onlyProxy 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L78

```solidity
file:    contracts/its/interchain-token-service/InterchainTokenService.sol

502         function transmitSendToken(
        bytes32 tokenId,
        address sourceAddress,
        string calldata destinationChain,
        bytes memory destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) external payable onlyTokenManager(tokenId) notPaused 

575         function _execute(
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) internal override onlyRemoteService(sourceChain, sourceAddress) notPaused 


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L502-L509

```solidity
file:     contracts/its/token-manager/TokenManager.sol

136        function transmitInterchainTransfer(
        address sender,
        string calldata destinationChain,
        bytes calldata destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) external payable virtual onlyToken 

161    function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L136-L142 

```solidity
file:   contracts/its/utils/Distributable.sol
  
51     function setDistributor(address distr) external onlyDistributor 

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Distributable.sol#L51

```solidity
file:

51     function setOperator(address operator_) external onlyOperator 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Operatable.sol#L51

## [G-17] Use assembly for loops

Using assembly for loops can indeed help save gas in certain cases. Assembly is a low-level language that allows you to directly control the Ethereum Virtual Machine (EVM) operations, giving you fine-grained control over gas consumption. In some situations, using assembly loops can be more gas-efficient than using high-level loops like for or while loops.


```solidity
file:    contracts/its/remote-address-validator/RemoteAddressValidator.sol

44            for (uint256 i; i < length; ++i) {
            addTrustedAddress(trustedChainNames[i], trustedAddresses[i]);
        }
56             for (uint256 i; i < length; i++) {
            uint8 b = uint8(bytes(s)[i]);
            if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));
        }
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L44-L46

```solidity
file:      contracts/its/interchain-token-service/InterchainTokenService.sol

537              for (uint256 i; i < length; ++i) {
            ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenIds[i]));
            tokenManager.setFlowLimit(flowLimits[i]);
        }
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L537-L540

```solidity
file:     contracts/interchain-governance-executor/InterchainProposalSender.sol

63            for (uint256 i = 0; i < interchainCalls.length; ) {
            _sendProposal(interchainCalls[i]);
            unchecked {
                ++i;
            }
        }

106             for (uint256 i = 0; i < interchainCalls.length; ) {
            totalGas += interchainCalls[i].gas;
            unchecked {
                ++i;
            }
        }        


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63-L68

```solidity
file:    contracts/cgp/AxelarGateway.sol

270            for (uint256 i; i < length; ++i) {
            string memory symbol = symbols[i];
            uint256 limit = limits[i];

            if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

            _setTokenMintLimit(symbol, limit);
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L270-L276

```solidity
file:    contracts/cgp/auth/MultisigBase.sol

70            for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

122         for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }        
168             for (uint256 i; i < length; ++i) {
            address account = newAccounts[i];

            // Check that the account wasn't already set as a signer for this epoch.
            if (signers.isSigner[account]) revert DuplicateSigner(account);
            if (account == address(0)) revert InvalidSigners();

            signers.isSigner[account] = true;
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L70-L72

```solidity
file:     contracts/interchain-governance-executor/InterchainProposalExecutor.sol

74             for (uint256 i = 0; i < calls.length; i++) {
            InterchainCalls.Call memory call = calls[i];
            (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

            if (!success) {
                _onTargetExecutionFailed(call, result);
            } else {
                _onTargetExecuted(call, result);
            }
        }

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74-L83 


## [G-18] Failure to check the zero address in the constructor causes the contract to be deployed again

Zero address control is not performed in the constructor in all 3 contracts within the scope of the audit. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high.

```solidity

file:   contracts/its/utils/Implementation.sol

18         constructor() {
        implementationAddress = address(this);
    }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Implementation.sol#L18


```solidity
file:

22        constructor(address deployer_) {
        if (deployer_ == address(0)) revert AddressZero();
        deployer = Create3Deployer(deployer_);
    }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L22
```solidity
file:   contracts/its/remote-address-validator/RemoteAddressValidator.sol

27         constructor(address _interchainTokenServiceAddress) {
        if (_interchainTokenServiceAddress == address(0)) revert ZeroAddress();
        interchainTokenServiceAddress = _interchainTokenServiceAddress;
        interchainTokenServiceAddressHash = keccak256(bytes(_lowerCase(interchainTokenServiceAddress.toString())));
    }


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L27

```solidity
file:   contracts/its/token-manager/TokenManager.sol

27        constructor(address interchainTokenService_) {
        if (interchainTokenService_ == address(0)) revert TokenLinkerZeroAddress();
        interchainTokenService = IInterchainTokenService(interchainTokenService_);
    }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L27

## [G-19] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.


```solidity
file:   contracts/its/remote-address-validator/RemoteAddressValidator.sol

83    function addTrustedAddress(string memory chain, string memory addr) public onlyOwner 

95    function removeTrustedAddress(string calldata chain) external onlyOwner

106   function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner

119   function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83

```solidity
file:

92        function setWhitelistedProposalCaller(
        string calldata sourceChain,
        address sourceCaller,
        bool whitelisted
    ) external override onlyOwner

107        function setWhitelistedProposalSender(
        string calldata sourceChain,
        address sourceSender,
        bool whitelisted
    ) external override onlyOwner


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L92-L96

```solidity
fiel:

24    function transferOwnership(address newOwner) external virtual onlyOwner 

40       function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L24 

```solidity
file:   contracts/cgp/AxelarGateway.sol

421    function mintToken(bytes calldata params, bytes32) external onlySelf 

427    function burnToken(bytes calldata params, bytes32) external onlySelf

455    function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf 

469    function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf

495    function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L421 


```solidity
file:    contracts/gmp-sdk/upgradable/Upgradable.sol

52        function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L52-L56


## [G-20]| Use assembly for math (add, sub, mul, div)

When we say "use assembly for math operations," we mean that when performing basic math operations in a smart contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in math functions. This is because Solidity's built-in math functions require additional gas consumption to perform the operation.

```solidity
file:   contracts/cgp/auth/MultisigBase.sol

56      uint256 voteCount = voting.voteCount + 1;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L56

```solidity
file:  contracts/gmp-sdk/util/TimeLock.sol

54     uint256 minimumEta = block.timestamp + _minimumTimeLockDelay;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L54

```solidity
file:    contracts/its/interchain-token/InterchainToken.sol

61     _approve(sender, address(tokenManager), allowance_ + amount);
100    _approve(sender, address(tokenManager), allowance_ + amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L61

## [G-21] Use assembly when getting a contract’s balance of ETH

When we say "use assembly to get a contract's balance of Ether," we mean that when retrieving the balance of Ether held by a contract, it's more gas-efficient to use assembly code to perform the operation instead of Solidity's built-in address.balance property. This is because Solidity's built-in address.balance property requires additional gas consumption to perform the operation.


```solidity
file:    contracts/cgp/util/Caller.sol

16      if (nativeValue > address(this).balance) revert InsufficientBalance();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L16

## [22] use Fixed-size Arrays are cheaper than dynamic one if it is possiable  to save gas 

```solidity
file:      contracts/cgp/AxelarGateway.sol

332        bytes32[] memory commandIds;
333        string[] memory commands;
334        bytes[] memory params;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L332

```solidity
file:  contracts/cgp/auth/MultisigBase.sol

19     address[] accounts;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L19

```solidity
file:   contracts/interchain-governance-executor/lib/InterchainCalls.sol
 
17      Call[] calls;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/lib/InterchainCalls.sol#L17


## [G-23] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

### use assembly for TokenManagerType to save gas 

```solidity  
file:     contracts/its/interchain-token-service/InterchainTokenService.sol

226            if (TokenManagerType(tokenManagerType) == TokenManagerType.LOCK_UNLOCK) {
            return implementationLockUnlock;
        } else if (TokenManagerType(tokenManagerType) == TokenManagerType.MINT_BURN) {
            return implementationMintBurn;
        } else if (TokenManagerType(tokenManagerType) == TokenManagerType.LIQUIDITY_POOL) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L226-L230

### use assembly for getTokenManagerAddress to save gas
```solidity
file:   contracts/its/interchain-token-service/InterchainTokenService.sol

398      address tokenManagerAddress = getTokenManagerAddress(tokenId);
         TokenManagerType tokenManagerType = distributor == tokenManagerAddress ? TokenManagerType.MINT_BURN : TokenManagerType.LOCK_UNLOCK;
        address tokenAddress = getStandardizedTokenAddress(tokenId);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L398-L400


## [G-24] Use Modifiers Instead of Functions To Save Gas

```solidity
file:    contracts/cgp/AxelarGateway.sol

505         function _hasCode(address addr) internal view returns (bool) {
        bytes32 codehash = addr.codehash;

        // https://eips.ethereum.org/EIPS/eip-1052
        return codehash != bytes32(0) && codehash != 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;
    }

619       function _getTokenType(string memory symbol) internal view returns (TokenType) {
        return TokenType(getUint(_getTokenTypeKey(symbol)));
    }
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L505-L530

```solidity
file:     contracts/its/token-manager/TokenManager.sol

204         function _getTokenId() internal view returns (bytes32 tokenId) {
        tokenId = ITokenManagerProxy(address(this)).tokenId();
    }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L204-L206




## [G‑25] Use custom errors rather than revert()/require() strings to save gas

Using custom errors rather than revert() or require() strings can indeed save gas in Solidity smart contracts. When you use revert() or require() with string messages, the entire string message is stored in the transaction's revert message, which consumes unnecessary gas.
Enable the Solidity Compiler Optimizer
The Solidity compiler optimizer works to simplify complex expressions, reducing code size and execution costs through inline operations, deployment costs, and function call costs.

The Solidity optimizer is best suited for inline operations. Even though inlining functions can result in significantly larger code, it is commonly used because it allows for additional simplifications.

The compiler optimizer also impacts the deployment and function call costs of your smart contracts.

For example, deployment costs decrease as the number of “runs” decreases—the frequency with which each opcode is executed over the course of a contract. The impact on function call costs, on the other hand, grows with the number of runs. This is because code optimized for more runs costs more to deploy and
Custom errors, on the other hand, use predefined error codes (usually enum) to represent specific error conditions. These error codes are stored more efficiently in the transaction's revert message, resulting in reduced gas consumption.

Let's see an example to illustrate this gas-saving technique:

```solidity
// Using custom errors
contract CustomErrorsExample {
    error InvalidSetMintLimitsParams();
    error TokenDoesNotExist(string symbol);

    function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external {
        uint256 length = symbols.length;
        if (length != limits.length) revert InvalidSetMintLimitsParams();

        for (uint256 i = 0; i < length; ++i) {
            string memory symbol = symbols[i];
            uint256 limit = limits[i];

            if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

            _setTokenMintLimit(symbol, limit);
        }
    }

    // Other contract functions...
}


```

```solidity
file:    contracts/cgp/AxelarGateway.sol

65       if (authModule_.code.length == 0) revert InvalidAuthModule();
66       if (tokenDeployerImplementation_.code.length == 0) revert InvalidTokenDeployer();
73       if (msg.sender != address(this)) revert NotSelf();
79       if (msg.sender != getAddress(KEY_GOVERNANCE)) revert NotGovernance();
255      if (newGovernance == address(0)) revert InvalidGovernance();
261      if (newMintLimiter == address(0)) revert InvalidMintLimiter();
268      if (length != limits.length) revert InvalidSetMintLimitsParams();
274      if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);
285      if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();
296      if (!success) revert SetupFailed();
309      if (implementation() == address(0)) revert NotProxy();
338      if (chainId != block.chainid) revert InvalidChainId();
342      if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();
392      if (tokenAddresses(symbol) != address(0)) revert TokenAlreadyExists(symbol);
402      if (!success) revert TokenDeployFailed(symbol);
409      if (tokenAddress.code.length == uint256(0)) revert TokenContractDoesNotExist(tokenAddress);
432      if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
446      if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);
519      if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
537      if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
538      if (amount == 0) revert InvalidAmount();
635      if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L65

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L635



```solidity
file:  contracts/cgp/auth/MultisigBase.sol

45    if (!signers.isSigner[msg.sender]) revert NotSigner();
51    if (voting.hasVoted[msg.sender]) revert AlreadyVoted();
159   if (newThreshold > length) revert InvalidSigners();
161   if (newThreshold == 0) revert InvalidSignerThreshold();
172   if (signers.isSigner[account]) revert DuplicateSigner(account);
173   if (account == address(0)) revert InvalidSigners();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L45

```solidity
file:    contracts/cgp/governance/AxelarServiceGovernance.sol
55      if (!multisigApprovals[proposalHash]) revert NotApproved();
80      revert InvalidCommand();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55

```solidity
file:  contracts/cgp/governance/InterchainGovernance.sol

93    revert NotGovernance();
100   if (target == address(0)) revert InvalidTarget();
121   revert InvalidCommand();
162    revert TokenNotSupported();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L93

```solidity
file:   contracts/cgp/util/Caller.sol
16      if (nativeValue > address(this).balance) revert InsufficientBalance();
20      revert ExecutionFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L16

```solidity
file:    contracts/gmp-sdk/upgradable/Upgradable.sol
14       if (owner() != msg.sender) revert NotOwner();
25       if (newOwner == address(0)) revert InvalidOwner();
46       if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();
51       if (!success) revert SetupFailed();
63       if (implementation() == address(0)) revert NotProxy();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L14

```solidity
file:   contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
51      if (!success) revert FailedInit();
81      if (bytecode.length == 0) revert EmptyBytecode();
88      if (deployedAddress_ == address(0)) revert FailedDeploy();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L51

```solidity
file:    contracts/gmp-sdk/deploy/Create3.sol
 
54       if (deployed.isContract()) revert AlreadyDeployed();
55       if (bytecode.length == 0) revert EmptyBytecode();
60       if (address(deployer) == address(0)) revert DeployFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol#L54

```solidity
file:   contracts/gmp-sdk/deploy/Create3Deployer.sol

58     if (!success) revert FailedInit();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L58

```solidity
file:   contracts/gmp-sdk/upgradable/FinalProxy.sol

77     if (msg.sender != owner) revert NotOwner();

82     if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L77

```solidity
file:     contracts/gmp-sdk/upgradable/InitProxy.sol
46        if (msg.sender != owner) revert NotOwner();
47        if (implementation() != address(0)) revert AlreadyInitialized();
50        if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
59         if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L46

```solidity
file:  contracts/gmp-sdk/upgradable/Proxy.sol
30     if (owner == address(0)) revert InvalidOwner();
33     if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
42     if (!success) revert SetupFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol#L30

```solidity
file:   contracts/gmp-sdk/upgradable/Upgradable.sol
31      if (address(this) == implementationAddress) revert NotProxy();
57      if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
58      if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();
63      if (!success) revert SetupFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L31
```solidity
file:     contracts/gmp-sdk/util/TimeLock.sol
51        if (hash == 0) revert InvalidTimeLockHash();
52        if (_getTimeLockEta(hash) != 0) revert TimeLockAlreadyScheduled();
68        if (hash == 0) revert InvalidTimeLockHash();
82        if (hash == 0 || eta == 0) revert InvalidTimeLockHash();
84        if (block.timestamp < eta) revert TimeLockNotReady();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L51
```solidity
file:  contracts/interchain-governance-executor/InterchainProposalExecutor.sol
50     revert NotWhitelistedSourceAddress();
58     revert NotWhitelistedCaller();
164    revert(add(32, result), mload(result))
168    revert ProposalExecuteFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L50
```solidity
file:
97      revert ZeroAddress();
104     revert LengthMismatch();
124     revert NotRemoteService();
133     revert NotTokenManager();
172     revert TokenManagerDoesNotExist(tokenId);
311     if (gateway.tokenAddresses(tokenSymbol) == tokenAddress) revert GatewayToken();
332     if (getCanonicalTokenId(tokenAddress) != tokenId) revert NotCanonicalTokenManager();
445     if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);
476     if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);
519     if (version > 0) revert InvalidMetadataVersion(version);
536     if (length != flowLimits.length) revert LengthMismatch();
565     if (implementation == address(0)) revert ZeroAddress();
566        if (ITokenManager(implementation).implementationType() != uint256(tokenManagerType)) revert InvalidTokenManagerImplementation();
590    revert SelectorUnknown();
817    revert TokenManagerDeploymentFailed();
867    revert StandardizedTokenDeploymentFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L97
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L867

```solidity
file:  contracts/its/libraries/AddressBytesUtils.sol

18     if (bytesAddress.length != 20) revert InvalidBytesLength(bytesAddress);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/libraries/AddressBytesUtils.sol#L18

```solidity
file:  contracts/its/proxies/StandardizedTokenProxy.sol

22    if (IStandardizedToken(implementationAddress).contractId() != CONTRACT_ID) revert InvalidImplementation();
25    if (!success) revert SetupFailed();if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/StandardizedTokenProxy.sol#L22

```solidity
file:   contracts/its/remote-address-validator/RemoteAddressValidator.sol

28      if (_interchainTokenServiceAddress == address(0)) revert ZeroAddress();
43      if (length != trustedAddresses.length) revert LengthMismatch();
84      if (bytes(chain).length == 0) revert ZeroStringLength();
85      if (bytes(addr).length == 0) revert ZeroStringLength();
96      if (bytes(chain).length == 0) revert ZeroStringLength();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L28

```solidity
file:   contracts/its/token-manager/TokenManager.sol
28      if (interchainTokenService_ == address(0)) revert TokenLinkerZeroAddress();
36      if (msg.sender != address(interchainTokenService)) revert NotService();
44      if (msg.sender != tokenAddress()) revert NotToken();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L28

```solidity
file:   contracts/its/utils/ExpressCallHandler.sol
91     if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();
133    if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L91 

```solidity
file:
101     if (flowToAdd + flowAmount > flowToCompare + flowLimit) revert FlowLimitExceeded();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L101


## [26] Enable the Solidity Compiler Optimizer to save gas 
The Solidity compiler optimizer works to simplify complex expressions, reducing code size and execution costs through inline operations, deployment costs, and function call costs.

The Solidity optimizer is best suited for inline operations. Even though inlining functions can result in significantly larger code, it is commonly used because it allows for additional simplifications.

The compiler optimizer also impacts the deployment and function call costs of your smart contracts.

For example, deployment costs decrease as the number of “runs” decreases—the frequency with which each opcode is executed over the course of a contract. The impact on function call costs, on the other hand, grows with the number of runs. This is because code optimized for more runs costs more to deploy and

