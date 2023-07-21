# Gas Optimization

# Summary

| Number | Optimization Details                                                                                                     | Context |
| :----: | :----------------------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Using `calldata` instead of `memory` for read-only arguments in external functions saves gas                             |   10    |
| [G-02] | `TokenManagerLockUnlock.sol::_takeToken` SafeTokenTransfer we should check amount for possible gas save                  |    1    |
| [G-03] | Using `storage` instead of `memory` for structs/arrays saves gas                                                         |    7    |
| [G-04] | `Keccak256()` should only need to be called on a specific string literal once                                            |    4    |
| [G-05] | Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous require() or if statement  |    4    |
| [G-06] | Functions `guaranteed` to when called by normal users can be marked Payable                                              |   16    |
| [G-07] | `Empty blocks` should be removed or emit something                                                                       |    8    |
| [G-08] | With `assembly`, .call (bool success)  transfer can be done gas-optimized                                                |    9    |
| [G-09] | Can make the variable outside the loop to save gas                                                                       |    4    |
| [G-10] | Use `assembly` to write address storage values                                                                           |    3    |
| [G-11] | Duplicated `if()` checks should be refactored to a modifier or function                                                  |    8    |
| [G-12] | Should use arguments instead of state variable                                                                           |    1    |
| [G-13] | Not using the named return variable when a function returns, wastes deployment gas                                       |    3    |
| [G-14] | Expressions for `constant` values such as a call to keccak256(), should use immutable rather than constant               |    1    |
| [G-15] | Multiple `address /ID` mappings can be combined into a `single mapping` of an address/ID to a struct , where appropriate |    1    |
| [G-16] | Don’t Initialize Variables with Default Value                                                                            |    4    |
| [G-17] | abi.encode() is less efficient than abi.encodePacked()                                                                   |   31    |
| [G-18] | Use hardcode address instead `address(this)`                                                                             |   25    |
| [G-19] | Amounts should be checked for 0 before calling a transfer                                                                |    9    |
| [G-20] | Using `bools` for storage incurs overhead                                                                                |    6    |
| [G-21] | Make `3 event` parameters indexed when possible                                                                          |    8    |
| [G-22] | Use constants instead of `type(uintx).max`                                                                               |   10    |
| [G-23] | Using a positive conditional flow to save a NOT opcode                                                                   |   18    |
| [G-24] | Using `XOR (^)` and `OR () `bitwise equivalents                                                                          |    3    |
| [G-25] | `If-statements` that use `&&` can be refactored into nested if statements                                                |    6    |
| [G-26] | `x += y` costs more gas than `x = x + y` for state variables                                                             |    1    |
| [G-27] | Use `custom errors` rather than `revert()/require()` strings to save gas                                                 |    -    |
| [G-28] | Cache `state variables` outside of loop to avoid reading storage on every iteration                                      |    4    |
| [G-29] | Use `do while` loop instead of for loop                                                                                  |    1    |

## [G-01] Using `calldata` instead of `memory` for read-only arguments in external functions saves gas

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```diff
file:  contracts/cgp/auth/MultisigBase.sol

-142   function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners
+142   function rotateSigners(address[] calldata newAccounts, uint256 newThreshold) external virtual onlySigners

-149   function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
+      function _rotateSigners(address[] calldata newAccounts, uint256 newThreshold) internal {
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

## [G-02] `TokenManagerLockUnlock.sol::_takeToken` SafeTokenTransfer we should check amount for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

48        SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L48

```diff
diff --git a/TokenManagerLockUnlock.org.sol b/TokenManagerLockUnlock.sol
index cb5b187..2888bb0 100644
--- a/TokenManagerLockUnlock.org.sol
+++ b/TokenManagerLockUnlock.sol
@@ -2,7 +2,7 @@
         IERC20 token = IERC20(tokenAddress());
         uint256 balance = token.balanceOf(address(this));

-        SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);
+        if (amount != 0) SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

         // Note: This allows support for fee-on-transfer tokens
         return IERC20(token).balanceOf(address(this)) - balance;

```

## [G-03] Using `storage` instead of `memory` for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```diff
file: /contracts/cgp/AxelarGateway.sol

-332        bytes32[] memory commandIds;
+           bytes32[] storage commandIds;

-334        string[] memory commands;
+           string[] storage commands;


-334        bytes[] memory params;
+           bytes[] storage params;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L332-L334

```solidity
file: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

34        (address interchainProposalCaller, InterchainCalls.Call[] memory calls) = abi.decode(payload, (address, InterchainCalls.Call[]));

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L54

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

## [G-04] `Keccak256()` should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

```solidity
file: /contracts/its/interchain-token-service/InterchainTokenService.sol

71     bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L71

```solidity
file: /contracts/its/proxies/InterchainTokenServiceProxy.sol

12     bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol#L12

```solidity
file: /contracts/its/proxies/StandardizedTokenProxy.sol

14     bytes32 private constant CONTRACT_ID = keccak256('standardized-token');

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol#L14

```solidity
file: /contracts/its/token-implementations/StandardizedToken.sol#L27

27     bytes32 private constant CONTRACT_ID = keccak256('standardized-token');

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L27

## [G-05] Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous require() or if statement

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

## [G-06] Functions `guaranteed` to when called by normal users can be marked Payable

The onlyOwner modifier makes a function revert if not called by the address registered as the owner

```diff
file: /contracts/its/interchain-token-service/InterchainTokenService.sol

-534    function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {
+534    function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external payable onlyOperator {


-547    function setPaused(bool paused) external onlyOwner {
+       function setPaused(bool paused) external payable onlyOwner {
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

## [G-07] `Empty blocks` should be removed or emit something

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

## [G-08] With `assembly`, .call (bool success)  transfer can be done gas-optimized

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

## [G-10] Use `assembly` to write address storage values

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

## [G-11] Duplicated `if()` checks should be refactored to a modifier or function

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

## [G-12] Should use arguments instead of state variable

state variables should not used in emit , This will save near 97 gas

```solidity
file: /contracts/cgp/AxelarGateway.sol

///@audit  the ' KEY_MINT_LIMITER ' is state variable , first should cached to stack and after that should emit.

688        emit MintLimiterTransferred(getAddress(KEY_MINT_LIMITER), newMintLimiter);

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L688

## [G-13] Not using the named return variable when a function returns, wastes deployment gas

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

## [G-14] Expressions for `constant` values such as a call to keccak256(), should use immutable rather than constant

While it doesn't save any gas because the compiler knows that developers often make this mistake, it's still best to use theright tool for the task at hand. There is a difference between constant variables and immutable variables, and they shouldeach be used in their appropriate contexts. constants should be used for literal values written into the code, and immutablevariables should be used for expressions, or values calculated in, or passed into the constructor.

```solidity
file: /contracts/gmp-sdk/deploy/Create3.sol

41     bytes32 internal constant DEPLOYER_BYTECODE_HASH = keccak256(type(CreateDeployer).creationCode);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L41

## [G-15] Multiple `address /ID` mappings can be combined into a `single mapping` of an address/ID to a struct , where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to re
the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

```solidity
file: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

24    mapping(string => mapping(address => bool)) public whitelistedCallers;


27    mapping(string => mapping(address => bool)) public whitelistedSenders;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

## [G-16] Don’t `Initialize` Variables with Default Value

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

## [G-17] `abi.encode()` is less efficient than abi.encodePacked()

use abi.encodePacked() where possible to save gas

```solidity
file:   contracts/its/interchain-token-service/InterchainTokenService.sol

203    tokenId = keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_ID, chainNameHash, tokenAddress));

214    tokenId = keccak256(abi.encode(PREFIX_CUSTOM_TOKEN_ID, sender, salt));

242    params = abi.encode(operator, tokenAddress);

252    params = abi.encode(operator, tokenAddress);

267    params = abi.encode(operator, tokenAddress, liquidityPoolAddress);

313    _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(),
tokenAddress));

401    _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));

512    payload = abi.encode(SELECTOR_SEND_TOKEN, tokenId, destinationAddress, amount);

520    payload = abi.encode(SELECTOR_SEND_TOKEN_WITH_DATA, tokenId, destinationAddress, amount, sourceAddress.toBytes(), metadata);

696    abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)

755    bytes memory payload = abi.encode(SELECTOR_DEPLOY_TOKEN_MANAGER, tokenId, tokenManagerType, params);

780    bytes memory payload = abi.encode(

828    return keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_SALT, tokenId));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L203

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

## [G-18] Use hardcode address instead `address(this)`

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

## [G-19] Amounts should be checked for 0 before calling a transfer

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

## [G‑20] Using bools for storage incurs overhead

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

## [G-21] Make `3 event` parameters indexed when possible

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

## [G-22] Use constants instead of `type(uintx).max`

type(uint256).max etc. it uses more gas in the distribution process and also for each transaction than constant usage.

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

## [G-23] Using a positive `conditional` flow to save a NOT opcode

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

## [G-24] Using `XOR (^)` and `OR (|) `bitwise equivalents

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

## [G-25] `If-statements` that use `&&` can be refactored into nested if statements

```solidity
file:    contracts/cgp/AxelarGateway.sol

88      if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();

446     if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

635     if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L88

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

## [G-26] `x += y` costs more gas than `x = x + y` for state variables

```solidity
file:    contracts/interchain-governance-executor/InterchainProposalSender.sol

107     totalGas += interchainCalls[i].gas;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L107

## [G‑27] Use `custom errors` rather than `revert()/require()` strings to save gas

Using custom errors rather than revert() or require() strings can indeed save gas in Solidity smart contracts. When you use revert() or require() with string messages, the entire string message is stored in the transaction's revert message, which consumes unnecessary gas.

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

50        if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert
InvalidImplementation();

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
file: its/interchain-token-service/InterchainTokenService.sol

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

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

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
file: contracts/its/utils/FlowLimit.sol

101     if (flowToAdd + flowAmount > flowToCompare + flowLimit) revert FlowLimitExceeded();

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L101

## [G-28] Cache `state variables` outside of loop to avoid reading storage on every iteration

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.

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

```solidity
file:      contracts/cgp/auth/MultisigBase.sol

122          for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L122-L125

```solidity
file:    contracts/cgp/auth/MultisigBase.sol

70             for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L70-L73

## [G-29] Use `do while` loop instead of for loop

```solidity
file: contracts/cgp/AxelarGateway.sol

270         for (uint256 i; i < length; ++i) {
            string memory symbol = symbols[i];
            uint256 limit = limits[i];

            if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

            _setTokenMintLimit(symbol, limit);
        }

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L270-L277

### Recommanded code

```diff
diff --git a/AxelarGateway.org.sol b/AxelarGateway.sol
index e688c08..dff73eb 100644
--- a/AxelarGateway.org.sol
+++ b/AxelarGateway.sol
@@ -1,9 +1,8 @@
-   for (uint256 i; i < length; ++i) {
+   uint256 i;
+   do{
             string memory symbol = symbols[i];
             uint256 limit = limits[i];

             if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

             _setTokenMintLimit(symbol, limit);
-            }
+        }while(i < length);

```
