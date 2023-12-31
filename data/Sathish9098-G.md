
# GAS OPTIMIZATIONS

| GAS COUNT | Issues | Instances | Gas Saved |
|------------------|------------------|------------------|------------------|
| [G-1]    |  Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate |1|    21018  |
| [G-2]    | Using ``calldata`` instead of ``memory`` for read-only arguments in external functions saves gas|8|    2820 |
| [G-3]    | Avoiding the overhead of ``bool`` storage    | 6 | 100600    |
| [G-4]    | Avoid ``contract existence`` checks by using low level calls   | 37     | 3700 |
| [G-5]    | Use ``calldata`` pointer Saves more gas than ``memory`` pointer   | 2 | 600 GAS (Per Iteration) |
| [G-6]    | ``IF’s/require()`` statements that check input arguments should be at the ``top`` of the function| 2| 300 |
| [G-7]    | Functions guaranteed to revert when called by normal users can be marked ``payable``  | 11 | 231 |
| [G-8]    | Optimize ``names`` to save gas  |  27   | - |
| [G-9]    | Default value initialization  | 4    | 80    |
| [G-10]    | Use constants instead of ``type(uintX).max``    | 7 | 91|
| [G-11]    | Splitting ``require()/if()`` statements that use ``&&`` saves gas    | 4    | 52    |
| [G-12]    | Caching ``global variables`` is more expensive than using the actual variable (use ``msg.sender`` instead of caching it)  | 10    | -   |

##

## [G-1] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

We can combine multiple mappings below into structs. We can then pack the structs by modifying the uint type for the values. This will result in cheaper storage reads since multiple mappings are accessed in functions and those values are now occupying the same storage slot, meaning the slot will become warm after the first ``SLOAD``. In addition, when writing to and reading from the struct values we will avoid a ``Gsset (20000 gas)`` and ``Gcoldsload (2100 gas)``, since multiple struct values are now occupying the same slot.

### ``RemoteAddressValidator.sol``: struct can be used for ``remoteAddressHashes``,``remoteAddresses`` since they are using same ``string`` as key. Also both mapping always used together in a same functions like ``addTrustedAddress()``, ``removeTrustedAddress()`` So ``struct`` is more efficient than ``mapping``. As per gas tests this will save ``21018 GAS``

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L15-L16


```diff
FILE: 2023-07-axelar/contracts/its/remote-address-validator/RemoteAddressValidator.sol

- 15: mapping(string => bytes32) public remoteAddressHashes;
- 16: mapping(string => string) public remoteAddresses;

+ struct RemoteAddressData {
+    bytes32 addressHash;
+    string addressString;
+ }

+ mapping(string => RemoteAddressData) public remoteAddressData;

```

###


```diff
FILE: Breadcrumbs2023-07-axelar/contracts/its/remote-address-validator/RemoteAddressValidator.sol

15: mapping(string => bytes32) public remoteAddressHashes;
16: mapping(string => string) public remoteAddresses;

```

##

## [G-2] Using calldata instead of memory for read-only arguments in external functions saves gas

Saves ``2820 GAS``, ``8 Instances``

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

## [G-3] Avoiding the overhead of bool storage 

Saves ``120600 GAS``, ``6 Instances``

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
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L22

```solidity
FILE: 2023-07-axelar/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

24: mapping(string => mapping(address => bool)) public whitelistedCallers;
27: mapping(string => mapping(address => bool)) public whitelistedSenders;

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24

```solidity
FILE: 2023-07-axelar/contracts/its/remote-address-validator/RemoteAddressValidator.sol

19: mapping(string => bool) public supportedByGateway;

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L19

##

## [G-4] Avoid contract existence checks by using low level calls

Saves ``3700 GAS``, ``37 Instances``

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

## [G-5] Use ``Calldata`` pointer Saves more gas than ``memory`` pointer

Saves ``600 GAS``, ``per Loop Iterations``

Calling ``calldata`` instead of ``memory`` in the loop you have shown will save gas. This is because ``calldata`` is a read-only data structure, which means that it does not have to be copied into memory each time it is accessed.

## ``InterchainProposalExecutor.sol``: Use ```calldata`` instead of ``memory`` : Saves ``250-300 GAS`` Per iteration

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L75

``call`` value is not changed any where inside the loop. So ``calldata`` is more efficient than ``memory`` to save gas 

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
## ``AxelarGateway.sol``: Use ```calldata`` instead of ``memory`` : Saves ``250-300 GAS`` Per iteration

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L270-L277

``symbol`` value is not changed any where inside the loop. So ``calldata`` is more efficient than ``memory`` to save gas 


```diff
FILE: 2023-07-axelar/contracts/cgp/AxelarGateway.sol

270: for (uint256 i; i < length; ++i) {
- 271:            string memory symbol = symbols[i];
+ 271:            string calldata symbol = symbols[i];
272:            uint256 limit = limits[i];
273:
274:            if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);
275:
276:            _setTokenMintLimit(symbol, limit);
277:        }

```
##

## [G-6] IF’s/require() statements that check input arguments should be at the top of the function

Saves ``300 GAS``, ``2 Instance``

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Cheaper to check the function parameter before making check . Saves ``200- 300  GAS``

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L172-L173

```diff
FILE: 2023-07-axelar/contracts/cgp/auth/MultisigBase.sol

169:  address account = newAccounts[i];
170:
171: // Check that the account wasn't already set as a signer for this epoch.
+ 273: if (account == address(0)) revert InvalidSigners();
- 172: if (signers.isSigner[account]) revert DuplicateSigner(account);
- 273: if (account == address(0)) revert InvalidSigners();
+ 172: if (signers.isSigner[account]) revert DuplicateSigner(account);

```

### tokenAddresses(symbol) == address(0) should be checked before ``limit`` to avoid unnecessary variable creation. After variable creation if any fails its waste of gas 

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L271-L274

```diff
FILE: 2023-07-axelar/contracts/cgp/AxelarGateway.sol

271: string memory symbol = symbols[i];
+ 274: if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);
272: uint256 limit = limits[i];
273:
- 274: if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

```

##

## [G-7] Functions guaranteed to revert when called by normal users can be marked ``payable`` 

Saves ``231 GAS``, ``11 Instance``

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

## [G-8] Optimize names to save gas

Saves ``27 Instances``

public/external function names and public member variable names can be optimized to save gas. See this [link](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

```solidity

///@audit getProposalEta(),executeProposal(),
FILE: 2023-07-axelar/contracts/cgp/governance/InterchainGovernance.sol

///@audit executeMultisigProposal(),
FILE: 2023-07-axelar/contracts/cgp/governance/AxelarServiceGovernance.sol

///@audit getChainName(),getTokenManagerAddress(),getValidTokenManagerAddress(),getTokenAddress(),getStandardizedTokenAddress(),getCanonicalTokenId(),getCustomTokenId(),getImplementation(),getParamsLockUnlock(),getParamsMintBurn(),getParamsLiquidityPool(),getFlowLimit(),getFlowOutAmount(),getFlowInAmount(),registerCanonicalToken(),deployRemoteCanonicalToken(),deployCustomTokenManager(),deployRemoteCustomTokenManager(),deployAndRegisterStandardizedToken(),deployAndRegisterRemoteStandardizedToken(),expressReceiveToken(),expressReceiveTokenWithData(),transmitSendToken(),setFlowLimit(),
FILE: 2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

```


##

## [G-9] Default value initialization

Saves ``80 GAS``,``4 Instance``

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

## [G-10] Use constants instead of type(uintX).max

Saves ``91 GAS``,``7 Instance``

Using constants instead of type(uintX).max saves gas in Solidity. This is because the type(uintX).max function has to dynamically calculate the maximum value of a uint256, which can be expensive in terms of gas. Constants, on the other hand, are stored in the bytecode of your contract, so they do not have to be recalculated every time you need them. Saves ``13 GAS``

```solidity
FILE: 2023-07-axelar/contracts/its/interchain-token/InterchainToken.sol

56:  if (allowance_ != type(uint256).max) {

57:  if (allowance_ > type(uint256).max - amount) {

58:  allowance_ = type(uint256).max - amount;

88: if (_allowance != type(uint256).max) {

95: if (allowance_ != type(uint256).max) {

96: if (allowance_ > type(uint256).max - amount) {
                    
97: allowance_ = type(uint256).max - amount;

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L56-L58


##

## [G-11] Splitting require()/if() statements that use && saves gas

Saves ``52 GAS``,``4 Instance``

See this [issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 13 gas

```diff
FILE: 2023-07-axelar/contracts/its/remote-address-validator/RemoteAddressValidator.sol

58:  if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L58

```diff
FILE: 2023-07-axelar/contracts/cgp/AxelarGateway.sol

88: if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();

446: if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

635: if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L88

## 

## [G-12] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

The ``msg.sender`` variable is a special variable that always refers to the address of the sender of the current transaction. This variable is not stored in memory, so it is much cheaper to use than a cached global variable.

```diff
FILE: Breadcrumbs2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

- 348:  address deployer_ = msg.sender;
- 349:        tokenId = getCustomTokenId(deployer_, salt);
+ 349:        tokenId = getCustomTokenId(msg.sender, salt);
350:        _deployTokenManager(tokenId, tokenManagerType, params);
- 351:        emit CustomTokenIdClaimed(tokenId, deployer_, salt);
+ 351:        emit CustomTokenIdClaimed(tokenId, msg.sender, salt);


-        address deployer_ = msg.sender;
-         tokenId = getCustomTokenId(deployer_, salt);
+         tokenId = getCustomTokenId(msg.sender, salt);
        _deployRemoteTokenManager(tokenId, destinationChain, gasValue, tokenManagerType, params);
-        emit CustomTokenIdClaimed(tokenId, deployer_, salt);
+        emit CustomTokenIdClaimed(tokenId, msg.sender, salt);


-  address caller = msg.sender;
        ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
        IERC20 token = IERC20(tokenManager.tokenAddress());

-        SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
+        SafeTokenTransferFrom.safeTransferFrom(token, msg.sender, destinationAddress, amount);
-        _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, caller);
+   _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, msg.sender);


- address caller = msg.sender;
        ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
        IERC20 token = IERC20(tokenManager.tokenAddress());

-        SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
+        SafeTokenTransferFrom.safeTransferFrom(token, msg.sender, destinationAddress, amount);
        _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount);

-        _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller);
+        _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, msg.sender);

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L372-L375

```diff
FILE: Breadcrumbs2023-07-axelar/contracts/its/interchain-token/InterchainToken.sol

- address sender = msg.sender;
        ITokenManager tokenManager = getTokenManager();
        /**
         * @dev if you know the value of `tokenManagerRequiresApproval()` you can just skip the if statement and just do nothing or _approve.
         */
        if (tokenManagerRequiresApproval()) {
-            uint256 allowance_ = allowance[sender][address(tokenManager)];-            
+           uint256 allowance_ = allowance[msg.sender][address(tokenManager)];
            if (allowance_ != type(uint256).max) {
                if (allowance_ > type(uint256).max - amount) {
                    allowance_ = type(uint256).max - amount;
                }

-                _approve(sender, address(tokenManager), allowance_ + amount);
+                _approve(msg.sender, address(tokenManager), allowance_ + amount);
            }

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L49



