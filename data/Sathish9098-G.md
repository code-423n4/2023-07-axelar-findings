
# GAS OPTIMIZATIONS

##

## [G-1] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the ``abi.decode()`` step has to use a for-loop to copy each index of the ``calldata`` to the ``memory`` index. Each iteration of this for-loop costs at least 60 gas ``(i.e. 60 * <mem_array>.length)``. Using ``calldata`` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracts to use ``calldata`` arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use ``calldata`` when the ``external`` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one.

### ``InterchainTokenService.sol``: ``distributor``,``operator`` can be ``calldata`` instead of ``memory``: Saves ``564 GAS``

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L422

```diff
FILE: 2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

417: function deployAndRegisterRemoteStandardizedToken(
418:        bytes32 salt,
419:        string calldata name,
420:        string calldata symbol,
421:        uint8 decimals,
- 422:        bytes memory distributor,
+ 422:        bytes calldata distributor,
- 423:        bytes memory operator,
+ 423:        bytes calldata operator,
424:        string calldata destinationChain,
425:        uint256 gasValue
426:    ) external payable notPaused {


```

### ``InterchainTokenService.sol``: ``sourceChain``,``sourceAddress``,``destinationAddress`` can be ``calldata`` instead of ``memory``: Saves ``846 GAS``

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L467


```diff
FILE: 2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

467:   function expressReceiveTokenWithData(
468:        bytes32 tokenId,
- 469:        string memory sourceChain,
+ 469:        string calldata sourceChain,
- 470:        bytes memory sourceAddress,
+ 470:        bytes calldata sourceAddress,
471:        address destinationAddress,
472:        uint256 amount,
473:        bytes calldata data,
474:        bytes32 commandId
475:    ) external {


- 506:   bytes memory destinationAddress,
+ 506:   bytes calldata destinationAddress,

```

### ``MultisigBase.sol``: ``newAccounts`` can be ``calldata`` instead of ``memory``: Saves ``282 GAS``

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L142-L144

```diff
FILE: 2023-07-axelar/contracts/cgp/auth/MultisigBase.sol

- 142: function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {
+ 142: function rotateSigners(address[] calldata newAccounts, uint256 newThreshold) external virtual onlySigners {
143:        _rotateSigners(newAccounts, newThreshold);
144:    }

```

### ``InterchainProposalSender.sol``: ``destinationChain``,``destinationContract`` can be ``calldata`` instead of ``memory``: Saves ``564 GAS``

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L81


```diff
FILE: 2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalSender.sol

80: function sendProposal(
- 81:        string memory destinationChain,
+ 81:        string calldata destinationChain,
- 82:        string memory destinationContract,
+ 82:        string calldata destinationContract,
83:        InterchainCalls.Call[] calldata calls
84:    ) external payable override {


```

### ``ConstAddressDeployer.sol``: ``bytecode``,``bytecode`` can be ``calldata`` instead of ``memory``: Saves ``564 GAS``

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L24

```diff
FILE: Breadcrumbs2023-07-axelar/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

- 24: function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {
+ 24: function deploy(bytes calldata bytecode, bytes32 salt) external returns (address deployedAddress_) {


42: function deployAndInit(
- 43:         bytes memory bytecode,
+ 43:         bytes calldata bytecode,
44:        bytes32 salt,
45:        bytes calldata init
46:    ) external returns (address deployedAddress_) {

```

## [G-2] Avoiding the overhead of bool storage 

```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27 Use ``uint256(1)`` and ``uint256(2)`` for true/false to avoid a ``Gwarmaccess`` ``(100 gas)`` for the extra ``SLOAD``, and to avoid ``Gsset`` ``(20000 gas)`` when changing from false to true, after having been true in the past

```solidity
FILE: 2023-07-axelar/contracts/cgp/auth/MultisigBase.sol

15: mapping(address => bool) hasVoted;
21: mapping(address => bool) isSigner;

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L15

```solidity
FILE: 2023-07-axelar/contracts/cgp/governance/AxelarServiceGovernance.sol

22: mapping(bytes32 => bool) public multisigApprovals;

```

```solidity
FILE: 2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

24: mapping(string => mapping(address => bool)) public whitelistedCallers;
27: mapping(string => mapping(address => bool)) public whitelistedSenders;

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24

##

## [G-3] Functions guaranteed to revert when called by normal users can be marked ``payable`` 

If a function modifier such as ``onlyOwner`` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are ``CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2)``, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L96-L96

```solidity
92:     function setWhitelistedProposalCaller(
93:         string calldata sourceChain,
94:         address sourceCaller,
95:         bool whitelisted
96:     ) external override onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L111-L111

```solidity

107:     function setWhitelistedProposalSender(
108:         string calldata sourceChain,
109:         address sourceSender,
110:         bool whitelisted
111:     ) external override onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L56-L56

```solidity

52:     function upgrade(
53:         address newImplementation,
54:         bytes32 newImplementationCodeHash,
55:         bytes calldata params
56:     ) external override onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L547-L547

```solidity
547:     function setPaused(bool paused) external onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83-L83

```solidity

83:     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L95-L95

```solidity

95:     function removeTrustedAddress(string calldata chain) external onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L106-L106

```solidity
106:     function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L119-L119

```solidity
119:     function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner 

```


```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L343-L345

```solidity

343:     function deployCustomTokenManager( 
344:         bytes32 salt,
345:         TokenManagerType tokenManagerType, 
346:         bytes memory params
347:     ) public payable notPaused returns (bytes32 tokenId) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L365-L368

```solidity

365:     function deployRemoteCustomTokenManager( 
366:         bytes32 salt,
367:         string calldata destinationChain,
368:         TokenManagerType tokenManagerType, 
369:         bytes calldata params,
370:         uint256 gasValue
371:     ) external payable notPaused returns (bytes32 tokenId) 

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L509-L509

```solidity
502:     function transmitSendToken(
503:         bytes32 tokenId,
504:         address sourceAddress,
505:         string calldata destinationChain,
506:         bytes memory destinationAddress,
507:         uint256 amount,
508:         bytes calldata metadata
509:     ) external payable onlyTokenManager(tokenId) notPaused  

```

##

## [G-5] Optimize names to save gas

public/external function names and public member variable names can be optimized to save gas. See this [link](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

```solidity

///@audit getProposalEta(),executeProposal(),
FILE: 2023-07-axelar/contracts/cgp/governance/InterchainGovernance.sol

///@audit executeMultisigProposal(),
FILE: 2023-07-axelar/contracts/cgp/governance/AxelarServiceGovernance.sol

///@audit getChainName(),getTokenManagerAddress(),getValidTokenManagerAddress(),getTokenAddress(),getStandardizedTokenAddress(),getCanonicalTokenId(),getCustomTokenId(),getImplementation(),getParamsLockUnlock(),getParamsMintBurn(),getParamsLiquidityPool(),getFlowLimit(),getFlowOutAmount(),getFlowInAmount(),registerCanonicalToken(),deployRemoteCanonicalToken(),deployCustomTokenManager(),deployRemoteCustomTokenManager(),deployAndRegisterStandardizedToken(),deployAndRegisterRemoteStandardizedToken(),expressReceiveToken(),expressReceiveTokenWithData(),transmitSendToken(),setFlowLimit(),
FILE: 2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

````
## [G-6] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including ``EXTCODESIZE`` (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

```solidity
FILE: Breadcrumbs2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

102: deployer = ITokenManagerDeployer(tokenManagerDeployer_).deployer();

162: tokenManagerAddress = deployer.deployedAddress(address(this), tokenId);

172: if (ITokenManagerProxy(tokenManagerAddress).tokenId() != tokenId) revert TokenManagerDoesNotExist(tokenId);

182: tokenAddress = ITokenManager(tokenManagerAddress).tokenAddress();

193: tokenAddress = deployer.deployedAddress(address(this), tokenId);

277: flowLimit = tokenManager.getFlowLimit();

287: flowOutAmount = tokenManager.getFlowOutAmount();

297: flowInAmount = tokenManager.getFlowInAmount();

311: if (gateway.tokenAddresses(tokenSymbol) == tokenAddress) revert GatewayToken();

331: tokenAddress = ITokenManager(tokenAddress).tokenAddress();

445: if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

476: if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

480: IERC20 token = IERC20(tokenManager.tokenAddress());

539: tokenManager.setFlowLimit(flowLimits[i]);

566:  if (ITokenManager(implementation).implementationType() != uint256(tokenManagerType)) revert InvalidTokenManagerImplementation();

610: amount = tokenManager.giveToken(destinationAddress, amount);

653: amount = tokenManager.giveToken(expressCaller, amount);

657: amount = tokenManager.giveToken(destinationAddress, amount);

658: IInterchainTokenExpressExecutable(destinationAddress).executeWithInterchainToken(sourceChain, sourceAddress, data, tokenId, amount);

713: string memory destinationAddress = remoteAddressValidator.getRemoteAddress(destinationChain);

723: gateway.callContract(destinationChain, destinationAddress, payload);

735:  name = token.name();

736: symbol = token.symbol();

737: decimals = token.decimals();

888: IInterchainTokenExpressExecutable(destinationAddress).expressExecuteWithInterchainToken(
            sourceChain,
            sourceAddress,
            data,
            tokenId,
            amount
        );

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L102

```solidity
FILE: Breadcrumbs2023-07-axelar/contracts/cgp/AxelarGateway.sol

287: if (AxelarGateway(newImplementation).contractId() != contractId()) revert InvalidImplementation();

317: IAxelarAuth(AUTH_MODULE).transferOperatorship(newOperatorsData);

329: bool allowOperatorshipTransfer = IAxelarAuth(AUTH_MODULE).validateProof(messageHash, proof);

449: depositHandler.destroy(address(this)

496: IAxelarAuth(AUTH_MODULE).transferOperatorship(newOperatorsData);

524: IERC20(tokenAddress).safeTransfer(account, amount);

526: IBurnableMintableCappedERC20(tokenAddress).mint(account, amount);

543: IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);

545: IERC20(tokenAddress).safeCall(abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount));

547: IERC20(tokenAddress).safeTransferFrom(sender, IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0)), amount);

```

```solidity
FILE: 2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalSender.sol

101: gateway.callContract(interchainCall.destinationChain, interchainCall.destinationContract, payload);

```

##

## [G-] Default value initialization

If a variable is not set/initialized, it is assumed to have the default value`` (0, false, 0x0 etc depending on the data type)``.
Explicitly initializing it with its default value is an anti-pattern and wastes gas.

Saves ``15-20`` GAS per instance 


```diff
FILE: 2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalSender.sol

- 63: for (uint256 i = 0; i < interchainCalls.length; ) {
+ 63: for (uint256 i; i < interchainCalls.length; ) {

- 105: uint256 totalGas = 0;
+ 105: uint256 totalGas ;

- 106: for (uint256 i = 0; i < interchainCalls.length; ) {
+ 106: for (uint256 i ; i < interchainCalls.length; ) {
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63

```diff
FILE: 2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

- 74: for (uint256 i = 0; i < calls.length; i++) {
+ 74: for (uint256 i ; i < calls.length; i++) {


```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74

##

## [G-] ``Calldata`` pointer Saves more gas than ``memory`` pointer

Calling ``calldata`` instead of ``memory`` in the loop you have shown will save gas. This is because ``calldata`` is a read-only data structure, which means that it does not have to be copied into memory each time it is accessed.

## ``InterchainProposalExecutor.sol``: Use ```calldata`` instead of ``memory`` : Saves ``224 GAS`` Per iteration

```diff
FILE: Breadcrumbs2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

74: for (uint256 i = 0; i < calls.length; i++) {
- 75:            InterchainCalls.Call memory call = calls[i];
+ 75:            InterchainCalls.Call calldata call = calls[i];
76:            (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);
77:
78:            if (!success) {
79:                _onTargetExecutionFailed(call, result);
80:            } else {
81:                _onTargetExecuted(call, result);
82:            }

```



## [G-] Use constants instead of type(uintX).max

Using constants instead of type(uintX).max saves gas in Solidity. This is because the type(uintX).max function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. Constants, on the other hand, are stored in the bytecode of your contract, so they do not have to be recalculated every time you need them.

```solidity
FILE: 2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalExecutor.sol



```

## Cheaper input valdiations should come before expensive operations

## Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.

Use calldata instead of memory in the loops . If the local variable value not changed inside the loop 

 InterchainProposalExecutor.sol  for loop 
