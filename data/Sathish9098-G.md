
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

## [G-3] Remove unused state variables to save gas

The code you provided defines two state variables, ``governanceChain`` and ``governanceAddress``. These state variables are declared as public, which means that they are accessible from outside the contract. The code also assigns the values of ``governanceChain_`` and ``governanceAddress_`` to the state variables ``governanceChain`` and ``governanceAddress``.

The gas cost of declaring a state variable is 20,000 gas. The gas cost of assigning a value to a string state variable is ``3200 GAS``.

In this case, the gas cost of declaring the state variables ``governanceChain`` and ``governanceAddress`` is ``40,000 gas``. The gas cost of assigning the values of ``governanceChain_`` and ``governanceAddress_ ``to the state variables ``governanceChain`` and ``governanceAddress`` is ``3200 gas``.

However, the state variables governanceChain and governanceAddress are never used in the contract. This means that the gas cost of declaring and assigning values to these state variables is wasted.

If you remove the state variables governanceChain and governanceAddress from the contract, the gas cost of the contract will be reduced by ``46400 gas``.

```solidity
FILE: 2023-07-axelar/contracts/cgp/governance/InterchainGovernance.sol

21: string public governanceChain;
22: string public governanceAddress;

39: governanceChain = governanceChain_;
40: governanceAddress = governanceAddress_;

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L21-L22
 
##

## [G-4] Functions guaranteed to revert when called by normal users can be marked ``payable`` 

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

don't emit state variable when stack varibale available 

Superflows event

combine emits to avoid gas 



Optimize names of public/external functions 



STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE 
Only missing instance 

The result of function calls should be cached rather than re-calling the function 3

Using storage instead of memory for structs/arrays saves gas

Only missing instances 

Multiple accesses of a mapping/array should use a local variable cache

 <x> += <y>/<x> -= <y> costs more gas than <x> = <x> + <y>/<x> = <x> - <y> for state variables  

Avoid contract existence check 

Don't initialize default values to variables 

Use constants instead of type(uint256).max

[GAS-16]	State variables used within a function more than once should be cached to save gas
[GAS-17]	Mappings used within a function more than once should be cached to save gas



 
