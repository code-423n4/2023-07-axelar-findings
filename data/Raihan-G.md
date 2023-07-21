# Gas Optimization


# Summry
|      | ISSUE | INSTANCE |         
|------|-------|----------|
|[G-01]|Use calldata instead of memory|2|
|[G-02]|Avoid contract existence checks by using low level calls|25|
|[G-03]|Can Make The Variable Outside The Loop To Save Gas |8|
|[G-04]|Use assembly to write address storage values|9|
|[G-05]|A modifier used only once and not being inherited should be inlined to save gas|8|
|[G-06]|Make 3 event parameters indexed when possible|10|
|[G-07]|With assembly, .call (bool success) transfer can be done gas-optimized|2|
|[G-08]|Amounts should be checked for 0 before calling a transfer|6|
|[G-09]|abi.encode() is less efficient than abi.encodePacked()|2|
|[G-10]|Using bools for storage incurs overhead|4|
|[G-11]|Use constants instead of type(uintx).max|10|
|[G-12]|Use of emit inside a loop|3|
|[G-13]|Using this.<fn>() wastes gas|2|
|[G-14]|Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if statement|7|
|[G-15]|Functions guaranteed to revert when called by normal users can be marked payable|19|
|[G-16]|SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE|2|
|[G-17]|State variables only set in the constructor should be declared immutable|4|
|[G-18]|Use do while loops instead of for loops|1|
|[G-19]|Cache state variables outside of loop to avoid reading storage on every iteration|4|
|[G-20]|Using XOR (^) and OR (|) bitwise equivalents|3|
|[G-21]|Using a positive conditional flow to save a NOT opcode|17|
|[G-22]|Don’t Initialize Variables with Default Value|4|
|[G-23]|Use hardcode address instead address(this)|14|
|[G-24]|Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)|7|
|[G-25]|Use selfbalance() instead of address(this).balance|1|
|[G-26]|Use assembly to emit events|3|
|[G-27]|Use assembly to perform efficient back-to-back calls|2|
|[G-28]|Duplicated require()/if() checks should be refactored to a modifier or function|4|
|[G-29]|When possible, use assembly instead of unchecked{++i}|2|
|[G-30]|Use assembly for math (add, sub, mul, div)|2|


## [G-01] Use calldata instead of memory
using calldata instead of memory for function arguments in external functions can help to reduce gas costs and improve the performance of your contracts.
When a function is marked as external, its arguments are passed in the calldata section of the transaction, which is a read-only area of memory that contains the input data for the transaction. Using calldata instead of memory for function arguments can be more gas-efficient, especially for functions that take large arguments.


```solidity
File: contracts/cgp/auth/MultisigBase.sol
142  function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {        
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142


```solidity
File: contracts/cgp/interfaces/IMultisigBase.sol
73  function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol#L73

## [G‑02] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.  

```solidity
File: contracts/cgp/util/Upgradable.sol
45    if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Upgradable.sol#L45

```solidity
File: contracts/cgp/AxelarGateway.sol
287   if (AxelarGateway(newImplementation).contractId() != contractId()) revert InvalidImplementation();

496   IAxelarAuth(AUTH_MODULE).transferOperatorship(newOperatorsData);

524    IERC20(tokenAddress).safeTransfer(account, amount);

526   IBurnableMintableCappedERC20(tokenAddress).mint(account, amount);

543   IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);

545     IERC20(tokenAddress).safeCall(abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount));

547    IERC20(tokenAddress).safeTransferFrom(sender, IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0)), amount);

548    IBurnableMintableCappedERC20(tokenAddress).burn(bytes32(0));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L287

```solidity
File: contracts/gmp-sdk/upgradable/InitProxy.sol
50   if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L50


```solidity
File: contracts/gmp-sdk/upgradable/Proxy.sol
33  if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Proxy.sol#L33

```solidity
File: contracts/gmp-sdk/upgradable/Upgradable.sol
57    if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L57

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
102    deployer = ITokenManagerDeployer(tokenManagerDeployer_).deployer();

172    if (ITokenManagerProxy(tokenManagerAddress).tokenId() != tokenId) revert TokenManagerDoesNotExist(tokenId);

182   tokenAddress = ITokenManager(tokenManagerAddress).tokenAddress();

331   tokenAddress = ITokenManager(tokenAddress).tokenAddress();

566    if (ITokenManager(implementation).implementationType() != uint256(tokenManagerType)) revert InvalidTokenManagerImplementation();


888   IInterchainTokenExpressExecutable(destinationAddress).expressExecuteWithInterchainToken(
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L102


```solidity
File: contracts/its/proxies/StandardizedTokenProxy.sol
22   if (IStandardizedToken(implementationAddress).contractId() != CONTRACT_ID) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol#L22

```solidity
File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
85   return IERC20(token).balanceOf(liquidityPool_) - balance;

96   uint256 balance = IERC20(token).balanceOf(to);

100  return IERC20(token).balanceOf(to) - balance;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L85

```solidity
File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
51   return IERC20(token).balanceOf(address(this)) - balance;

62   uint256 balance = IERC20(token).balanceOf(to);

66   return IERC20(token).balanceOf(to) - balance;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51

## [G-03] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```
```solidity
File: contracts/cgp/AxelarGateway.sol
271  string memory symbol = symbols[i];

272  uint256 limit = limits[i];
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L271

```solidity
File: contracts/cgp/auth/MultisigBase.sol
169  address account = newAccounts[i];
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L169

```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
76    (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76


```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
57  uint8 b = uint8(bytes(s)[i]);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L57

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
109   string calldata chainName = chainNames[i];

122   string calldata chainName = chainNames[i];
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L109

```solidity
File: contracts/its/utils/Multicall.sol
25   (bool success, bytes memory result) = address(this).delegatecall(data[i]);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Multicall.sol#L25

## [G-04] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.
example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```
```solidity
File: contracts/cgp/AxelarGateway.sol
68  AUTH_MODULE = authModule_;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L68

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
43    gateway = IAxelarGateway(_gateway);

44    gasService = IAxelarGasService(_gasService);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L43

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
98  remoteAddressValidator = IRemoteAddressValidator(remoteAddressValidator_);

99    gasService = IAxelarGasService(gasService_);

100   tokenManagerDeployer = tokenManagerDeployer_;

101    standardizedTokenDeployer = standardizedTokenDeployer_;

102    deployer = ITokenManagerDeployer(tokenManagerDeployer_).deployer();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L98

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
29  interchainTokenServiceAddress = _interchainTokenServiceAddress;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L29

## [G-05] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times.

By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File: contracts/gmp-sdk/upgradable/Upgradable.sol
29   modifier onlyProxy() {
        // Prevent setup from being called on the implementation
        if (address(this) == implementationAddress) revert NotProxy();
        _;
    }
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L29

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
123    modifier onlyRemoteService(string calldata sourceChain, string calldata sourceAddress) {

132    modifier onlyTokenManager(bytes32 tokenId) {    
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L123

```solidity
File: contracts/its/token-manager/TokenManager.sol
35   modifier onlyService() {
        if (msg.sender != address(interchainTokenService)) revert NotService();
        _;
    }

43    modifier onlyToken() {
        if (msg.sender != tokenAddress()) revert NotToken();
        _;
    }    
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L43

```solidity
File: contracts/its/utils/Distributable.sol
20   modifier onlyDistributor() {
        if (distributor() != msg.sender) revert NotDistributor();
        _;
    }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol#L20

```solidity
File: contracts/its/utils/Implementation.sol
26   modifier onlyProxy() {
        if (implementationAddress == address(this)) revert NotProxy();
        _;
    }    
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Implementation.sol#26

```solidity
File: contracts/its/utils/Operatable.sol
20    modifier onlyOperator() {
        if (operator() != msg.sender) revert NotOperator();
        _;
    }
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Operatable.sol#L20

## [G-06] Make 3 event parameters indexed when possible
 events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows for more efficient searching of events.
Exmple:
```
• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes value);

• event UserMetadataEmitted(uint256 indexed userId, bytes32 indexed key, bytes indexed value);
```

```solidity
File: contracts/cgp/interfaces/IAxelarAuthWeighted.sol
15     event OperatorshipTransferred(address[] newOperators, uint256[] newWeights, uint256 newThreshold);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IAxelarAuthWeighted.sol#L15

```solidity
File: contracts/cgp/interfaces/IAxelarServiceGovernance.sol
15   event MultisigApproved(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
    event MultisigCancelled(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
    event MultisigExecuted(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/#L15-L17

```solidity
File: contracts/cgp/interfaces/IMultisigBase.sol
22    event SignersRotated(address[] newAccounts, uint256 newThreshold);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol#L22

```solidity
File:    contracts/its/interfaces/IRemoteAddressValidator.sol
14      event TrustedAddressAdded(string souceChain, string sourceAddress);

15      event TrustedAddressRemoved(string souceChain);

16      event GatewaySupportedChainAdded(string chain);

17      event GatewaySupportedChainRemoved(string chain);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol#L14

```solidity
File: contracts/its/interfaces/IDistributable.sol
8    event DistributorChanged(address distributor);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IDistributable.sol#L8

## [G-07] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
-   (bool success,) = dest.call{value:amount}("");
 bool success; 
 assembly {  
            success := call(gas(), dest, amount, 0, 0)
     }  
```solidity
File: contracts/cgp/util/Caller.sol
18   (bool success, ) = target.call{ value: nativeValue }(callData);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L18

```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
76    (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76

## [G-08] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.


```solidity
File: contracts/cgp/AxelarGateway.sol
524    IERC20(tokenAddress).safeTransfer(account, amount);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L524

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
451   SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L451

```solidity
File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
82   SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount);

98   SafeTokenTransferFrom.safeTransferFrom(token, liquidityPool(), to, amount);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L82

```solidity
File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
48  SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

64   SafeTokenTransfer.safeTransfer(token, to, amount);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L48


## [G-09] abi.encode() is less efficient than abi.encodePacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.
```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
401   _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));

696   abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L401


## [G-10] Using bools for storage incurs overhead
  
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

    Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from 'false' to 'true', after having been 'true' in the past


```solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol
22   mapping(bytes32 => bool) public multisigApprovals;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L22

```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
24    mapping(string => mapping(address => bool)) public whitelistedCallers;

27    mapping(string => mapping(address => bool)) public whitelistedSenders;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
19   mapping(string => bool) public supportedByGateway;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L19


## [G-11] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol
79    if (commandId > uint256(type(ServiceGovernanceCommand).max)) {    
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L79

```solidity
File: contracts/cgp/governance/InterchainGovernance.sol
120   if (commandId > uint256(type(GovernanceCommand).max)) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L120

```solidity
File: contracts/its/interchain-token/InterchainToken.sol
56  if (allowance_ != type(uint256).max) {

57  if (allowance_ > type(uint256).max - amount) {

58  allowance_ = type(uint256).max - amount;

88  if (_allowance != type(uint256).max) {

95  if (allowance_ != type(uint256).max) {

96  if (allowance_ > type(uint256).max - amount) {

97  allowance_ = type(uint256).max - amount;    
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L56

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
104    if (tokenManagerImplementations.length != uint256(type(TokenManagerType).max) + 1) revert LengthMismatch();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L104


## [G-12] Use of emit inside a loop
Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.

```solidity
File: contracts/cgp/AxelarGateway.sol
376  if (success) emit Executed(commandId);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L376

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
111    emit GatewaySupportedChainAdded(chainName);

124    emit GatewaySupportedChainRemoved(chainName);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L111

## [G-13] Using this.<fn>() wastes gas
Calling an external function internally, through the use of this wastes the gas overhead of calling an external function (100 gas). Instead, change the function from external to public, and remove the this

```solidity
File: contracts/cgp/util/Upgradable.sol
49    (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Upgradable.sol#L49

```solidity
File:  contracts/gmp-sdk/upgradable/Upgradable.sol
61    (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L61

## [G‑14] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if statement
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
This will stop the check for overflow and underflow so it will save gas

```solidity
File: contracts/its/interchain-token/InterchainToken.sol
58   allowance_ = type(uint256).max - amount;

89   _approve(sender, msg.sender, _allowance - amount);

97   allowance_ = type(uint256).max - amount;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L58

```solidity
File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
85   return IERC20(token).balanceOf(liquidityPool_) - balance;

100  return IERC20(token).balanceOf(to) - balance;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L85

```solidity
File: contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
51    return IERC20(token).balanceOf(address(this)) - balance;

66    return IERC20(token).balanceOf(to) - balance;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51


## [G-15] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File: contracts/gmp-sdk/upgradable/Upgradable.sol
52    function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner {

78    function setup(bytes calldata data) external override onlyProxy {        
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L52

```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
92   function setWhitelistedProposalCaller(
        string calldata sourceChain,
        address sourceCaller,
        bool whitelisted
    ) external override onlyOwner {

107   function setWhitelistedProposalSender(
        string calldata sourceChain,
        address sourceSender,
        bool whitelisted
    ) external override onlyOwner {                
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L92

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
534   function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {

547    function setPaused(bool paused) external onlyOwner {

575     function _execute(
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) internal override onlyRemoteService(sourceChain, sourceAddress) notPaused {        
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L534

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
83    function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

95    function removeTrustedAddress(string calldata chain) external onlyOwner {

106   function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

19    function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {        
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83

```solidity
File: contracts/its/token-implementations/StandardizedToken.sol
49   function setup(bytes calldata params) external override onlyProxy {

76   function mint(address account, uint256 amount) external onlyDistributor {

86   function burn(address account, uint256 amount) external onlyDistributor {        
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L49

```solidity
File: contracts/its/token-manager/TokenManager.sol
61   function setup(bytes calldata params) external override onlyProxy {

161   function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256) {

171    function setFlowLimit(uint256 flowLimit) external onlyOperator {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L61

```solidity
File: contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol
67   function setLiquidityPool(address newLiquidityPool) external onlyOperator {    
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L67

```solidity
File: contracts/its/utils/Distributable.sol
51   function setDistributor(address distr) external onlyDistributor {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol#L51

## [G-16] SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE
state variables should not used in emit

uint256 private globalNetDepositCap;

  function setGlobalNetDepositCap(uint256 _newGlobalNetDepositCap) external      override onlyRole(SET_GLOBAL_NET_DEPOSIT_CAP_ROLE) {    	
     globalNetDepositCap = _newGlobalNetDepositCap;
	emit GlobalNetDepositCapChange(globalNetDepositCap);
 }

```solidity
File: contracts/cgp/AxelarGateway.sol
682   emit GovernanceTransferred(getAddress(KEY_GOVERNANCE), newGovernance);

688    emit MintLimiterTransferred(getAddress(KEY_MINT_LIMITER), newMintLimiter);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L682


## [G‑17] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.


```solidity
File: contracts/cgp/governance/InterchainGovernance.sol
39   governanceChain = governanceChain_;

40   governanceAddress = governanceAddress_;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L39

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
43    gateway = IAxelarGateway(_gateway);

44    gasService = IAxelarGasService(_gasService);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L43

## [G-18] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
```solidity
File: contracts/cgp/AxelarGateway.sol
270      for (uint256 i; i < length; ++i) {
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L270-L277



## [G-19] Cache state variables outside of loop to avoid reading storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.

```solidity
File:   contracts/its/remote-address-validator/RemoteAddressValidator.sol
108     for (uint256 i; i < length; ++i) {
            string calldata chainName = chainNames[i];
            supportedByGateway[chainName] = true;
            emit GatewaySupportedChainAdded(chainName);
        }


119   function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {
        uint256 length = chainNames.length;
       for (uint256 i; i < length; ++i) {
            string calldata chainName = chainNames[i];
            supportedByGateway[chainName] = false;
            emit GatewaySupportedChainRemoved(chainName);
        }
    }
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L108-L112

```solidity
File:   contracts/cgp/auth/MultisigBase.sol
122     for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L122-L125


```solidity
File:    contracts/cgp/auth/MultisigBase.sol
70             for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L70-L73

## [G-20] Using XOR (^) and OR (|) bitwise equivalents
Estimated savings: 73 gas

```solidity
File:   contracts/cgp/AxelarGateway.sol
342     if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();

446     if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L342

```solidity
File:   contracts/cgp/governance/InterchainGovernance.sol
92    if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L92


## [G-21] Using a positive conditional flow to save a NOT opcode

```solidity
File:    contracts/cgp/AxelarGateway.sol
296      if (!success) revert SetupFailed();

363      if (!allowOperatorshipTransfer) continue;

402      if (!success) revert TokenDeployFailed(symbol);

446      if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L296

```solidity
File:    contracts/cgp/auth/MultisigBase.sol
45     if (!signers.isSigner[msg.sender]) revert NotSigner();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L45

```solidity
File:  contracts/cgp/governance/AxelarServiceGovernance.sol
55    if (!multisigApprovals[proposalHash]) revert NotApproved();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55

```solidity
File:    contracts/cgp/util/Caller.sol
19    if (!success) 
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L19

```solidity
File:  contracts/gmp-sdk/upgradable/Upgradable.sol
51    if (!success) revert SetupFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L51 

```solidity
File:   contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
51     if (!success) revert FailedInit();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L51

```solidity
File:    contracts/gmp-sdk/deploy/Create3Deployer.sol
58      if (!success) revert FailedInit();
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L58

```solidity
File:   contracts/gmp-sdk/upgradable/FinalProxy.sol
82    if (!success) revert SetupFailed();
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L82 

```solidity
File:  contracts/interchain-governance-executor/InterchainProposalExecutor.sol
49     if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) 

57     if (!whitelistedCallers[sourceChain][interchainProposalCaller]) 

78     if (!success) 
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L49

```solidity
File:    contracts/its/interchain-token-service/InterchainTokenService.sol
124     if (!remoteAddressValidator.validateSender(sourceChain, sourceAddress)) revert NotRemoteService();

816     if (!success) 

866      if (!success)
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L124

## [G-22] Initialize variables with no default value
If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

As an example:
```
for (uint256 i = 0; i < num.length; ++i) {};
```
should be replaced with
```
for (uint256 i; i < num.length; ++i) {};
```
Consider removing explicit initializations for default values.

[reference](https://gist.github.com/IllIllI000/e075d189c1b23dce256cd166e28f3397)

```solidity
File:    contracts/interchain-governance-executor/InterchainProposalExecutor.sol
74      for (uint256 i = 0; i < calls.length; i++)
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74

```solidity
File:  contracts/interchain-governance-executor/InterchainProposalSender.sol
63     for (uint256 i = 0; i < interchainCalls.length; )

106    for (uint256 i = 0; i < interchainCalls.length; )

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63 

```solidity
File:   contracts/its/utils/Multicall.sol
24     for (uint256 i = 0; i < data.length; ++i) 
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L24


## [G-23] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address) 


```solidity
File:   contracts/cgp/AxelarGateway.sol
73      if (msg.sender != address(this)) revert NotSelf();

374     (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));

443     abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))

449     depositHandler.destroy(address(this));

543     IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);

616     return address(uint160(uint256(keccak256(abi.encodePacked(bytes1(0xff), address(this), salt, codeHash)))));
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L73


```solidity
File:    contracts/its/interchain-token-service/InterchainTokenService.sol
162      tokenManagerAddress = deployer.deployedAddress(address(this), tokenId);

193      tokenAddress = deployer.deployedAddress(address(this), tokenId);

313      _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));

696       abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)

716       address(this),
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L162


```solidity
File:    contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol
46       uint256 balance = token.balanceOf(address(this));

48       SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

51       return IERC20(token).balanceOf(address(this)) - balance;
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L46

## [G-24] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
File: contracts/its/interchain-token/InterchainToken.sol
49  address sender = msg.sender;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L49

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
348  address deployer_ = msg.sender;

372  address deployer_ = msg.sender;

447  address caller = msg.sender;

478  address caller = msg.sender;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L348

```solidity
File: contracts/its/token-manager/TokenManager.sol
89   address sender = msg.sender;

115  address sender = msg.sender;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L89

## [G-25] Use selfbalance() instead of address(this).balance
it's recommended to use the selfbalance() function instead of address(this).balance. The selfbalance() function is a built-in Solidity function that returns the balance of the current contract in Wei and is considered more gas-efficient and secure.

```solidity
File: contracts/cgp/util/Caller.sol
16   if (nativeValue > address(this).balance) revert InsufficientBalance();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L16

## [G‑26] Use assembly to emit events
We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic emit event for eventSentAmountExample:
```
 // uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```
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


```solidity
File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
90   emit Deployed(keccak256(bytecode), salt, deployedAddress_);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L90

```solidity
File: contracts/gmp-sdk/deploy/Create3Deployer.sol
33    emit Deployed(keccak256(bytecode), salt, deployedAddress_);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L33

```solidity
File: contracts/its/test/utils/Pausable.sol
15   emit TestEvent();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/test/utils/Pausable.sol#L15

## [G-27] Use assembly to perform efficient back-to-back calls
If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

```solidity
File: contracts/gmp-sdk/upgradable/Upgradable.sol
57   if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L57


```solidity
File: contracts/cgp/util/Upgradable.sol
45   if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Upgradable.sol#


## [G-28] Duplicated require()/if() checks should be refactored to a modifier or function

```solidity
File: contracts/cgp/AxelarGateway.sol
394   if (tokenAddress == address(0)) {

432   if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

519     if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

537   if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
    
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L394


## [G-29] When possible, use assembly instead of unchecked{++i}
You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```
Gas: 1329
```
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```
Gas: 709

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
65  unchecked {
                ++i;
            }

108  unchecked {
                ++i;
            }            
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L65

## [G-30] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

```solidity
File: contracts/cgp/auth/MultisigBase.sol
56   uint256 voteCount = voting.voteCount + 1;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L56

```solidity
File: contracts/gmp-sdk/util/TimeLock.sol
54   uint256 minimumEta = block.timestamp + _minimumTimeLockDelay;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L54