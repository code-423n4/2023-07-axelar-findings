## Summary

### Gas Optimization

|no|Issue|Instances||
|-|:-|:-:|:-:|
| [G-01] | Use assembly to write address storage values | 3 | - |
| [G-02] | Duplicated require()/if() checks should be refactored to a modifier or function | 3 | - |
| [G-03] | With assembly, .call (bool success)  transfer can be done gas-optimized | 9 | - |
| [G-04] | Using storage instead of memory for structs/arrays saves gas | 7 | - |
| [G-05] | Empty blocks should be removed or emit something | 8 | - |
| [G-06] | Before transfer of  some functions, we should check some variables for possible gas save | 1 | - |
| [G-07] | Should use arguments instead of state variable  | 1 | - |
| [G-08] | Using calldata instead of memory for read-only arguments in external functions saves gas | 5 | - |
| [G-09] | Can make the variable outside the loop to save gas | 4 | - |
| [G-10] | Not using the named return variable when a function returns, wastes deployment gas | 3 | - |
| [G-11] | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct , where appropriate | 1 | - |
| [G-12] | Expressions for constant values such as a call to keccak256(), should use immutable rather than constant | 1 | - |
| [G-13] | Functions guaranteed to when called by normal users can be marked Payable | 17 | - |
| [G-14] | Keccak256() should only need to be called on a specific string literal once | 4 | - |
| [G-15] | Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if statement | 4 | - |



## Gas Optimizations  


## [G-01] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```solidity
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
file: /contracts/its/to6en-implementations/StandardizedToken.sol

56            tokenManager = tokenManager_;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L56

```solidity
file: /contracts/its/utils/StandardizedTokenDeployer.sol

34        implementationLockUnlockAddress = implementationLockUnlockAddress_;

35        implementationMintBurnAddress = implementationMintBurnAddress_;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/StandardizedTokenDeployer.sol#L34-L35



## [G-02] Duplicated require()/if() checks should be refactored to a modifier or function   

 to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.


```solidity
file: /contracts/gmp-sdk/util/TimeLock.sol

51        if (hash == 0) revert InvalidTimeLockHash();

68        if (hash == 0) revert InvalidTimeLockHash();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L51

```solidity
file: /contracts/cgp/AxelarGateway.sol

432        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

519        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

537        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L432

```solidity
file: /contracts/its/utils/ExpressCallHandler.sol

91        if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

133       if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L91





## [G-03] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0),this storage disappears and gas optimization is provided.


```solidity
file: /contracts/cgp/util/Caller.sol

18        (bool success, ) = target.call{ value: nativeValue }(callData);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L18

```solidity
file: /contracts/cgp/AxelarGateway.sol

374            (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));

398            (bool success, bytes memory data) = TOKEN_DEPLOYER_IMPLEMENTATION.delegatecall(

441            (bool success, bytes memory returnData) = depositHandler.execute(

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L374

```solidity
file: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

76            (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76

```solidity
file: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

50        (bool success, ) = deployedAddress_.call(init);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50

```solidity
file: /contracts/gmp-sdk/deploy/Create3Deployer.sol

57        (bool success, ) = deployedAddress_.call(init);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L57

```solidity 
file: /contracts/gmp-sdk/upgradable/FinalProxy.sol

81            (bool success, ) = finalImplementation_.delegatecall(abi.encodeWithSelector(BaseProxy.setup.selector, setupParams));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L81

```solidity
file: /contracts/its/utils/Multicall.sol

25            (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Multicall.sol#L25





## [G-04] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


```solidity
file: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

41        (string[] memory trustedChainNames, string[] memory trustedAddresses) = abi.decode(params, (string[], string[]));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L41

```solidity
file: /contracts/cgp/governance/AxelarServiceGovernance.sol

38        address[] memory cosigners,

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L38


```solidity
file: /contracts/cgp/auth/MultisigBase.sol

35    constructor(address[] memory accounts, uint256 threshold) {

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L35

```solidity
file: /contracts/cgp/AxelarGateway.sol

332        bytes32[] memory commandIds;
334        string[] memory commands;
334        bytes[] memory params;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L332-L334


```solidity
file: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

34        (address interchainProposalCaller, InterchainCalls.Call[] memory calls) = abi.decode(payload, (address, InterchainCalls.Call[]));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L54




## [G-05] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

```solidity
file: /contracts/gmp-sdk/upgradable/BaseProxy.sol

32    function setup(bytes calldata params) external {}

67    receive() external payable virtual {}

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/BaseProxy.sol#L32

```solidity
file: /contracts/its/proxies/TokenManagerProxy.sol

66    function setup(bytes calldata setupParams) external {}

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/TokenManagerProxy.sol#L66


```solidity
file: /contracts/cgp/governance/InterchainGovernance.sol

168    receive() external payable {}

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L168

```solidity
file: /contracts/cgp/governance/AxelarServiceGovernance.sol

40    ) InterchainGovernance(gateway, governanceChain, governanceAddress, minimumTimeDelay) MultisigBase(cosigners, threshold) {}

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L40

```solidity
file: /contracts/cgp/governance/Multisig.sol

20    constructor(address[] memory accounts, uint256 threshold) MultisigBase(accounts, threshold) {}

41    receive() external payable {}

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/Multisig.sol#L20

```solidity
file: /contracts/gmp-sdk/upgradable/Upgradable.sol

87    function _setup(bytes calldata data) internal virtual {}

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L87



## [G-06] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

48        SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L48



## [G-07] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas  

```solidity
file: /contracts/cgp/AxelarGateway.sol

///@audit  the ' KEY_MINT_LIMITER ' is state variable , first should cached to stack and after that should emit.

688        emit MintLimiterTransferred(getAddress(KEY_MINT_LIMITER), newMintLimiter);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L688



## [G-08] Using calldata instead of memory for read-only arguments in external functions saves gas 

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

```solidity
file: /contracts/cgp/interfaces/IMultisigBase.sol

73    function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol#L73

```solidity
file: /contracts/interchain-governance-executor/InterchainProposalSender.sol

81        string memory destinationChain,

82        string memory destinationContract,

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L81

```solidity
file: /contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

15    function sendProposals(InterchainCalls.InterchainCall[] memory calls) external payable;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L15

```solidity
file: /contracts/gmp-sdk/deploy/Create3.sol

21    function deploy(bytes memory bytecode) external {

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L21




## [G-09] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/cgp/auth/MultisigBase.sol

169            address account = newAccounts[i];

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L169

```solidity 
file: /contracts/cgp/AxelarGateway.sol

271            string memory symbol = symbols[i];

272            uint256 limit = limits[i];

345            bytes32 commandId = commandIds[i];

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L271


## [G-10] Not using the named return variable when a function returns, wastes deployment gas


```solidity
file: /contracts/gmp-sdk/deploy/Create3.sol

74        deployed = address(uint160(uint256(keccak256(abi.encodePacked(hex'd6_94', deployer, hex'01')))));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L74


```solidity
file: /contracts/its/utils/FlowLimit.sol

47        slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)));

56        slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol#L47



## [G-11] Not using the named return variable when a function returns, wastes deployment gas


```solidity
file: /contracts/gmp-sdk/deploy/Create3.sol

74        deployed = address(uint160(uint256(keccak256(abi.encodePacked(hex'd6_94', deployer, hex'01')))));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L74


```solidity
file: /contracts/its/utils/FlowLimit.sol

47        slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)));

56        slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol#L47



## [G-12] Expressions for constant values such as a call to keccak256(), should use immutable rather than constant

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor.

```solidity
file: /contracts/gmp-sdk/deploy/Create3.sol

41     bytes32 internal constant DEPLOYER_BYTECODE_HASH = keccak256(type(CreateDeployer).creationCode);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L41




## [G-13] Functions guaranteed to when called by normal users can be marked Payable

The onlyOwner modifier makes a function revert if not called by the address registered as the owner

```solidity
file: /contracts/its/interchain-token-service/InterchainTokenService.sol

534    function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {

547    function setPaused(bool paused) external onlyOwner {

579    ) internal override onlyRemoteService(sourceChain, sourceAddress) notPaused {        

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L534


```solidity
file: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

67    function setLiquidityPool(address newLiquidityPool) external onlyOperator {

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L67


```solidity
file: /contracts/its/utils/Distributable.sol

51    function setDistributor(address distr) external onlyDistributor {

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol#L51

```solidity
file: /contracts/its/token-implementations/StandardizedToken.sol

49    function setup(bytes calldata params) external override onlyProxy {

76    function mint(address account, uint256 amount) external onlyDistributor {

86    function burn(address account, uint256 amount) external onlyDistributor {        

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L49

```solidity
file: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

83     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

95     function removeTrustedAddress(string calldata chain) external onlyOwner {
 
106    function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {   

119    function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {         

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83


```solidity
file: /contracts/cgp/auth/MultisigBase.sol

142     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142


```solidity
file: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

96     ) external override onlyOwner {

111    ) external override onlyOwner {    

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L96

```solidity
file: /contracts/gmp-sdk/upgradable/Upgradable.sol

56    ) external override onlyOwner {

78    function setup(bytes calldata data) external override onlyProxy {    

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L56



## [G-14] Duplicated require()/if() checks should be refactored to a modifier or function   

 to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.


```solidity
file: /contracts/gmp-sdk/util/TimeLock.sol

51        if (hash == 0) revert InvalidTimeLockHash();

68        if (hash == 0) revert InvalidTimeLockHash();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L51

```solidity
file: /contracts/cgp/AxelarGateway.sol

432        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

519        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

537        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L432

```solidity
file: /contracts/its/utils/ExpressCallHandler.sol

91        if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

133       if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L91




## [G-15] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if statement  

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
This will stop the check for overflow and underflow so it will save gas


```solidity
file: /contracts/its/interchain-token/InterchainToken.sol

57                if (allowance_ > type(uint256).max - amount) {

58                allowance_ = type(uint256).max - amount;

89            _approve(sender, msg.sender, _allowance - amount);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L57


```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

66        return IERC20(token).balanceOf(to) - balance;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L66

