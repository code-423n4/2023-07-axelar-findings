|  no    | Issue   | Instance |
|------|---------|----------|  
|[G-01]|Using a positive conditional flow to save a NOT opcode|17|
|[G-02]|Use hardcode address instead address(this)|24|
|[G-03]|Amounts should be checked for 0 before calling a transfer|9|
|[G-04]|x += y costs more gas than x = x + y for state variables|1|
|[G-05]|If statements that use && can be refactored into nested if statements|6|
|[G-06]|abi.encode() is less efficient than abi.encodePacked()|30|
|[G-07]|Use constants instead of type(uintx).max|10|
|[G-08]|Using bools for storage incurs overhead|6|
|[G-09]|Don’t Initialize Variables with Default Value|4|
|[G-10]|Use custom errors rather than revert()/require() strings to save gas|100|
|[G-11]|Using XOR (^) and OR (pipe line) bitwise equivalents|3|
|[G-12]|Make 3 event parameters indexed when possible|8|
|[G-13]|Cache state variables outside of loop to avoid reading storage on every iteration|4|
|[G-14]| With assembly, .call (bool success) transfer can be done gas-optimized|5|
|[G-15]|Use calldata instead of memory for function arguments that do not get mutated|10|
|[G-16]|Use do while loops instead of for loops|1|



## [G-01] Using a positive conditional flow to save a NOT opcode

One way to save gas is to use positive conditions instead. For example, instead of writing if (!condition), you can write if (condition == false). This may seem counterintuitive at first, but it can actually save gas because the EVM does not need to perform a separate NOT operation.

```solidity

296      if (!success) revert SetupFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L296

```solidity
363      if (!allowOperatorshipTransfer) continue;
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L363

```solidity
402      if (!success) revert TokenDeployFailed(symbol);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L402

```solidity
446      if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L446

```solidity

45     if (!signers.isSigner[msg.sender]) revert NotSigner();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L45

```solidity

55    if (!multisigApprovals[proposalHash]) revert NotApproved();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55

```solidity

19    if (!success) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L19

```solidity

51    if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L51 

```solidity

51     if (!success) revert FailedInit();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L51

```solidity

58      if (!success) revert FailedInit();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L58

```solidity
 
82    if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L82 

```solidity

49     if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) 
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L49


```solidity
57     if (!whitelistedCallers[sourceChain][interchainProposalCaller]) 
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L57

```solidity
78     if (!success) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L78

```solidity

124     if (!remoteAddressValidator.validateSender(sourceChain, sourceAddress)) revert NotRemoteService();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L124

```solidity
816     if (!success) 
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L816


```solidity
866      if (!success)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L866


## [G-02] Use hardcode address instead address(this)



The statement "Use hardcoded address instead of address(this) to save gas cost" means that using a hardcoded address instead of the address(this) expression can be more gas-efficient when sending or receiving Ether to or from the contract. This is because using the address(this) expression requires additional gas to be consumed to retrieve the address of the current contract, while using a hardcoded address does not.


```solidity

23      implementationAddress = address(this);
31      if (address(this) == implementationAddress) revert NotProxy();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L23 

```solidity

16     if (nativeValue > address(this).balance) revert InsufficientBalance();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L16 

```solidity

52      deployed = deployedAddress(address(this), salt);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol#L52 

```solidity

69      return Create3.deployedAddress(address(this), deploySalt);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L69

```solidity

73      if (msg.sender != address(this)) revert NotSelf();
374     (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));
443     abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))
449     depositHandler.destroy(address(this));
543     IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);
616     return address(uint160(uint256(keccak256(abi.encodePacked(bytes1(0xff), address(this), salt, codeHash)))));
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L73

```solidity
61     implementation_ = Create3.deployedAddress(address(this), FINAL_IMPLEMENTATION_SALT);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L61

```solidity
162      tokenManagerAddress = deployer.deployedAddress(address(this), tokenId);
193      tokenAddress = deployer.deployedAddress(address(this), tokenId);
313      _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));
696       abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
716       address(this),

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L162

```solidity
205     tokenId = ITokenManagerProxy(address(this)).tokenId();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L205

```solidity
46       uint256 balance = token.balanceOf(address(this));
48       SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);
51       return IERC20(token).balanceOf(address(this)) - balance;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L46

```solidity

19      implementationAddress = address(this);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Implementation.sol#L19

```solidity
27     if (implementationAddress == address(this)) revert NotProxy();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Implementation.sol#L27


```solidity
25     (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L25

```solidity

38     bytes memory args = abi.encode(address(this), implementationType, tokenId, params);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L38





## [G-03] Amounts should be checked for 0 before calling a transfer 



it is generally a good practice to check the amount for 0 before calling a transfer function in order to save gas. This is because if the amount is 0, there is no need to execute the transfer function, which can save gas costs.



```solidity
524      IERC20(tokenAddress).safeTransfer(account, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L524

```solidity

66      tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L66

```solidity
104     tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L104

```solidity

451     SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L451

```solidity
482     SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L482

```solidity

82     SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L82

```solidity
98     SafeTokenTransferFrom.safeTransferFrom(token, liquidityPool(), to, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L98

```solidity

48     SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L48

```solidity
64     SafeTokenTransfer.safeTransfer(token, to, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L64



## [G-04] x += y costs more gas than x = x + y for state variables


```solidity

107     totalGas += interchainCalls[i].gas;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L107



## [G-05] If statements that use && can be refactored into nested if statements


In Solidity, using nested if statements can sometimes save gas compared to using the logical AND (&&) operator within a single if statement.
When you use the AND operator, the EVM needs to evaluate both conditions before proceeding with the code block inside the if statement. This means that if the first condition is false, the second condition is still evaluated, even if it is also false. This can lead to unnecessary gas consumption.

```solidity


88      if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L88

```solidity
446     if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L446

```solidity
635     if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L635

```solidity

50     if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L50

```solidity

33     if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol#L33

```solidity

58     if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L58




## [G-06] abi.encode() is less efficient than abi.encodePacked()

abi.encode() is a more versatile function that can handle multiple arguments of different types. It is used to encode data for function calls, among other things.
abi.encodePacked() is a more specialized function that is used specifically for concatenating data into a single byte array. It is commonly used when you need to pack multiple values together into a single value.

```solidity
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
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L214
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L242
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L252
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L267
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L313
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L401
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L512
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L520
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L696
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L755
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L780
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L828

```solidity

32      slot = uint256(keccak256(abi.encode(PREFIX_EXPRESS_RECEIVE_TOKEN, tokenId, destinationAddress, amount, commandId)))

57      abi.encode(

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L32
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L57


```solidity

47      slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)));

56      slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L47

```solidity 
62        bytes memory params = abi.encode(tokenManager, distributor, name, symbol, decimals, mintAmount, mintTo);
63        bytecode = abi.encodePacked(type(StandardizedTokenProxy).creationCode, abi.encode(implementationAddress, params));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L62

```solidity


38     bytes memory args = abi.encode(address(this), implementationType, tokenId, params);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L38

```solidity

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
25       deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

47       deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

63       bytes32 newSalt = keccak256(abi.encode(sender, salt));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L25

```solidity

30    bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));
54    bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));
68    bytes32 deploySalt = keccak256(abi.encode(sender, salt));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L30

```solidity

66     emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L66

```solidity
89     bytes memory payload = abi.encode(msg.sender, interchainCall.calls);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L89


## [G-07] Use constants instead of type(uintx).max


Using constants instead of type(uintX).max can be an effective way to optimize gas costs in Solidity contracts.
type(uintX).max is a built-in constant in Solidity that represents the maximum value that can be stored in a variable of size X bits. However, this constant can be expensive in terms of gas cost, particularly for large values of X, because it requires a lot of computational steps to calculate the result.

```solidity

79    if (commandId > uint256(type(ServiceGovernanceCommand).max))

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L79

```solidity

120     if (commandId > uint256(type(GovernanceCommand).max))

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L120

```solidity


56      if (allowance_ != type(uint256).max) {
57      if (allowance_ > type(uint256).max - amount) {
58      allowance_ = type(uint256).max - amount;
88      if (_allowance != type(uint256).max) 

95      if (allowance_ != type(uint256).max) {
96      if (allowance_ > type(uint256).max - amount) {
97      allowance_ = type(uint256).max - amount;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L56
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L57
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L58
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L88
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L95
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L96
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L97



```solidity

104      if (tokenManagerImplementations.length != uint256(type(TokenManagerType).max) + 1) revert LengthMismatch();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L104

## [G‑08] Using bools for storage incurs overhead

In Solidity, using bool variables for storage can incur additional overhead, which can result in higher gas costs. This is because each bool variable in storage takes up one full 32-byte storage slot, even though it only needs one bit to represent its value.


```solidity
15      mapping(address => bool) hasVoted;
21      mapping(address => bool) isSigner;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L15
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L21

```solidity
22     mapping(bytes32 => bool) public multisigApprovals;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L22
```solidity
24      mapping(string => mapping(address => bool)) public whitelistedCallers;
27      mapping(string => mapping(address => bool)) public whitelistedSenders;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L27

```solidity
19     mapping(string => bool) public supportedByGateway;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L19


## [G-09] Don’t Initialize Variables with Default Value

In Solidity, initializing variables with their default values can result in unnecessary gas consumption. This is because the Solidity compiler automatically initializes all variables to their default values, which is zero for most types.

```solidity

74      for (uint256 i = 0; i < calls.length; i++)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74

```solidity

63     for (uint256 i = 0; i < interchainCalls.length; )

106    for (uint256 i = 0; i < interchainCalls.length; )

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63 
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106


```solidity

24     for (uint256 i = 0; i < data.length; ++i) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L24



## [G‑10] Use custom errors rather than revert()/require() strings to save gas

 using custom errors can be more gas-efficient than using strings with the revert() or require() functions. This is because strings are variable-length and can take up a significant amount of space in the contract's bytecode, leading to higher gas costs.
Custom errors, on the other hand, are defined as static variables or constants in the contract and do not take up extra space in the bytecode. Using custom errors can also make your code more readable and easier to maintain, as it allows you to define and reuse meaningful error messages throughout your contract.



```

```solidity

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

45    if (!signers.isSigner[msg.sender]) revert NotSigner();
51    if (voting.hasVoted[msg.sender]) revert AlreadyVoted();
159   if (newThreshold > length) revert InvalidSigners();
161   if (newThreshold == 0) revert InvalidSignerThreshold();
172   if (signers.isSigner[account]) revert DuplicateSigner(account);
173   if (account == address(0)) revert InvalidSigners();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L45
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L51
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L159
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L161
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L172
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L173



```solidity
55      if (!multisigApprovals[proposalHash]) revert NotApproved();
80      revert InvalidCommand();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L80


```solidity

93    revert NotGovernance();
100   if (target == address(0)) revert InvalidTarget();
121   revert InvalidCommand();
162    revert TokenNotSupported();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L93
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L100
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L121
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L162


```solidity
16      if (nativeValue > address(this).balance) revert InsufficientBalance();
20      revert ExecutionFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L16
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L20


```solidity
14       if (owner() != msg.sender) revert NotOwner();
25       if (newOwner == address(0)) revert InvalidOwner();
46       if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();
51       if (!success) revert SetupFailed();
63       if (implementation() == address(0)) revert NotProxy();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L14

```solidity
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

58     if (!success) revert FailedInit();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L58

```solidity

77     if (msg.sender != owner) revert NotOwner();

82     if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L77

```solidity
46        if (msg.sender != owner) revert NotOwner();
47        if (implementation() != address(0)) revert AlreadyInitialized();
50        if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
59         if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L46

```solidity
30     if (owner == address(0)) revert InvalidOwner();
33     if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
42     if (!success) revert SetupFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol#L30

```solidity
31      if (address(this) == implementationAddress) revert NotProxy();
57      if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
58      if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();
63      if (!success) revert SetupFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L31
```solidity
51        if (hash == 0) revert InvalidTimeLockHash();
52        if (_getTimeLockEta(hash) != 0) revert TimeLockAlreadyScheduled();
68        if (hash == 0) revert InvalidTimeLockHash();
82        if (hash == 0 || eta == 0) revert InvalidTimeLockHash();
84        if (block.timestamp < eta) revert TimeLockNotReady();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L51
```solidity
50     revert NotWhitelistedSourceAddress();
58     revert NotWhitelistedCaller();
164    revert(add(32, result), mload(result))
168    revert ProposalExecuteFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L50
```solidity
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

18     if (bytesAddress.length != 20) revert InvalidBytesLength(bytesAddress);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/libraries/AddressBytesUtils.sol#L18

```solidity

22    if (IStandardizedToken(implementationAddress).contractId() != CONTRACT_ID) revert InvalidImplementation();
25    if (!success) revert SetupFailed();if (!success) revert SetupFailed();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/StandardizedTokenProxy.sol#L22

```solidity

28      if (_interchainTokenServiceAddress == address(0)) revert ZeroAddress();
43      if (length != trustedAddresses.length) revert LengthMismatch();
84      if (bytes(chain).length == 0) revert ZeroStringLength();
85      if (bytes(addr).length == 0) revert ZeroStringLength();
96      if (bytes(chain).length == 0) revert ZeroStringLength();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L28

```solidity
28      if (interchainTokenService_ == address(0)) revert TokenLinkerZeroAddress();
36      if (msg.sender != address(interchainTokenService)) revert NotService();
44      if (msg.sender != tokenAddress()) revert NotToken();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L28

```solidity
91     if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();
133    if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L91 

```solidity
101     if (flowToAdd + flowAmount > flowToCompare + flowLimit) revert FlowLimitExceeded();

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L101

## [G-11] Using XOR (^) and OR (pipe line) bitwise equivalents


```solidity

342     if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();

446     if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L342
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L446


```solidity

92    if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L92


## [G-12] Make 3 event parameters indexed when possible



```solidity


8       event DistributorChanged(address distributor);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IDistributable.sol#L8

```solidity
 
8      event FlowLimitSet(uint256 flowLimit);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IFlowLimit.sol#L8

```solidity

8      event OperatorChanged(address operator);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IOperatable.sol#L8

```solidity

11       event PausedSet(bool paused);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IPausable.sol#L11

```solidity

14      event TrustedAddressAdded(string souceChain, string sourceAddress);
15      event TrustedAddressRemoved(string souceChain);
16      event GatewaySupportedChainAdded(string chain);
17      event GatewaySupportedChainRemoved(string chain);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol#L14
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol#L15
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol#L16
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol#L17

## [G-13] Cache state variables outside of loop to avoid reading storage on every iteration

Caching state variables outside of a loop can be an effective way to optimize gas costs in Solidity contracts.
In Solidity, reading state variables from storage can be expensive in terms of gas cost, particularly if the state variable is large or complex. This is because reading from storage requires loading data from the storage to memory, which can be slow and costly.

```solidity


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





```solidity

122          for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L122-L125



```solidity

70             for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L70-L73

## [G-14] With assembly, .call (bool success) transfer can be done gas-optimized


 the transfer() function is commonly used to transfer ether from one address to another. However, the transfer() function is known to be relatively expensive in terms of gas cost, particularly when transferring large amounts of ether.

```solidity

374     (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L374 


```solidity

18     (bool success, ) = target.call{ value: nativeValue }(callData);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L18


```solidity

50    (bool success, ) = deployedAddress_.call(init);   

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50

```solidity

57     (bool success, ) = deployedAddress_.call(init);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L57

```solidity

76     (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76





## [G-15] Use calldata instead of memory for function arguments that do not get mutated


```solidity

142   function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners 

149   function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L142

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L149

```solidity

73     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IMultisigBase.sol#L73

```solidity

42        function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42-L46

```solidity

21    function deploy(bytes memory bytecode) external 

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol#L21


```solidity

73    function _executeProposal(InterchainCalls.Call[] memory calls) internal 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L73

```solidity

15      function sendProposals(InterchainCalls.InterchainCall[] memory calls) external payable;

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L15

```solidity

559      function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L559

```solidity

18      function multicall(bytes[] calldata data) external payable returns (bytes[] memory results);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IMulticall.sol#L18

```solidity
  
22     function multicall(bytes[] calldata data) public payable returns (bytes[] memory results)

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L22

## [G-16] Use do while loops instead of for loops



```solidity

270         for (uint256 i; i < length; ++i) {
            string memory symbol = symbols[i];
            uint256 limit = limits[i];

            if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

            _setTokenMintLimit(symbol, limit);
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L270-L277


