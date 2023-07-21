| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutable) |
| 2 | [state variables can be packed into fewer storage slots](#2-state-variables-can-be-packed-into-fewer-storage-slots) |
| 3 | [expressions for constant values such as a call to keccak256(), should use immutable rather than constant](#3-expressions-for-constant-values-such-as-a-call-to-keccak256-should-use-immutable-rather-than-constant) |
| 4 | [Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate](#4-multiple-addressid-mappings-can-be-combined-into-a-single-mapping-of-an-addressid-to-a-struct-where-appropriate) |
| 5 | [state variables should be cached in stack variables rather than re-reading them from storage](#5-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-caught) |
| 6 | [can make the variable outside the loop to save gas](#6-can-make-the-variable-outside-the-loop-to-save-gas) |
| 7 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#7-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 8 | [require() or revert() statements that check input arguments should be at the top of the function](#8-require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function) |
| 9 | [using `calldata` instead of `memory` for read-only arguments in external functions saves gas](#9-using-calldata-instead-of-memory-for-read-only-arguments-in-external-functions-saves-gas) |
| 10 | [abi.encode() is less efficient than abi.encodepacked()](#10-abiencode-is-less-efficient-than-abiencodepacked) |
| 11 | [Ternary over if ... else](#11-ternary-over-if--else) |
| 12 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#12-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 13 | [Optimize names to save gas](#13-optimize-names-to-save-gas) |
| 14 | [part of the code can be pre calculated](#14-part-of-the-code-can-be-pre-calculated) |
| 15 | [Use selfbalance() instead of address(this).balance](#15-use-selfbalance-instead-of-addressthisbalance) |
| 16 | [Use hardcoded address instead address(this)](#16-use-hardcoded-address-instead-addressthis) |
| 17 | [Empty receive functions can cause gas issues](#17-empty-receive-functions-can-cause-gas-issues) |


## 1. State variables only set in the constructor should be declared immutable.

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas). While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

- [InterchainProposalSender.sol#L39](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L39)
- [InterchainProposalSender.sol#L40](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L40)


## 2. state variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

`tokenManager` and `decimals` can be put together to reduce the slots needed by 1
- [StandardizedToken.sol#L54](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L54)


## 3. expressions for constant values such as a call to keccak256(), should use immutable rather than constant

13 instances
- [AxelarGateway.sol#L44-L57](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L44-L57)

- [TimeLock.sol#L13](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L13)

- [FinalProxy.sol#L18](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L18)

- [FlowLimit.sol#L15](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol#L15)
- [FlowLimit.sol#L16](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol#L16)

- [InterchainTokenServiceProxy.sol#L12](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol#L12)

- [RemoteAddressValidatorProxy.sol#L12](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/RemoteAddressValidatorProxy.sol#L12)

- [StandardizedTokenProxy.sol#L14](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol#L14)

- [InterchainTokenService.sol#L62](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L62)
- [InterchainTokenService.sol#L63](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L63)
- [InterchainTokenService.sol#L64](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L64)
- [InterchainTokenService.sol#L71](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L71)

- [RemoteAddressValidator.sol#L21](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L21)

- [StandardizedToken.sol#L27](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L27)


## 4. Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

`remoteAddressHashes` `remoteAddresses` `supportedByGateway` all get chain name so they can change into a mapping to struct. will not reduce slot usage but there are accesses to multiple of them in a function which we will save gas there
- [RemoteAddressValidator.sol#L15-L19](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L15-L19)


## 5. state variables should be cached in stack variables rather than re-reading them from storage (ones not caught)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`signerEpoch` is being read from storage i times but it doesnt change so it can be cached before the for loop and use the cached version for it. saves 100*i gas
- [MultisigBase.sol#L123](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L123)


## 6. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

- [MultisigBase.sol#L169](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L169)

- [AxelarGateway.sol#L271](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L271)
- [AxelarGateway.sol#L272](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L272)
- [AxelarGateway.sol#L345](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L345)
- [AxelarGateway.sol#L349](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L349)
- [AxelarGateway.sol#L350](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L350)
- [AxelarGateway.sol#L374](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L374)

- [InterchainProposalExecutor.sol#L75](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L75)
- [InterchainProposalExecutor.sol#L76](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76)

- [Multicall.sol#L25](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Multicall.sol#L25)

- [RemoteAddressValidator.sol#L57](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L57)
- [RemoteAddressValidator.sol#L109](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L109)
- [RemoteAddressValidator.sol#L122](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L122)


## 7. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [InterchainProposalSender.sol#L63](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63)
- [InterchainProposalSender.sol#L105](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L105)
- [InterchainProposalSender.sol#L106](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106)

- [InterchainProposalExecutor.sol#L74](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74)

- [TokenManager.sol#L118](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L118)


## 8. require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

swap the sorting of the checks. `signers.isSigner[account]` is much more expensive to read than the other revert
- [MultisigBase.sol#L172-L173](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L172-L173)

swap #L54 and #L55 because first line uses a function call but the second one is cheaper and possibly can save gas.
- [Create3.sol#L54-L55](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L54-L55)


## 9. using `calldata` instead of `memory` for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. 

- [MultisigBase.sol#L142](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142)


## 10. abi.encode() is less efficient than abi.encodepacked()

if it is used with keccak256 we dont change them because of safety reasons but otherwise it can be changed to save gas

- [InterchainProposalSender.sol#L89](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L89)

- [StandardizedTokenDeployer.sol#L62](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/StandardizedTokenDeployer.sol#L62)
- [StandardizedTokenDeployer.sol#L63](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/StandardizedTokenDeployer.sol#L63)

- [TokenManagerDeployer.sol#L38](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/TokenManagerDeployer.sol#L38)

- [InterchainTokenService.sol#L242](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L242)
- [InterchainTokenService.sol#L252](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L252)
- [InterchainTokenService.sol#L267](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L267)
- [InterchainTokenService.sol#L313](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L313)
- [InterchainTokenService.sol#L401](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L401)
- [InterchainTokenService.sol#L512](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L512)
- [InterchainTokenService.sol#L520](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L520)
- [InterchainTokenService.sol#L696](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L696)
- [InterchainTokenService.sol#L755](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L755)
- [InterchainTokenService.sol#L780](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L780)


## 11. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [AxelarGateway.sol#L376-L377](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L376-L377)
- [AxelarGateway.sol#L523-L526](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L523-L526)

- [InterchainProposalExecutor.sol#L78-L81](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L78-L81)

- [TokenManager.sol#L68-L71](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L68-L71)


## 12. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- [MultisigBase.sol#L142](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142)

- [AxelarGateway.sol#L254](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L254)
- [AxelarGateway.sol#L260](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L260)
- [AxelarGateway.sol#L266](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L266)
- [AxelarGateway.sol#L284](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L284)
- [AxelarGateway.sol#L385](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L385)
- [AxelarGateway.sol#L421](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L421)
- [AxelarGateway.sol#L427](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L427)
- [AxelarGateway.sol#L455](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L455)
- [AxelarGateway.sol#L469](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L469)
- [AxelarGateway.sol#L495](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L495)

- [Upgradable.sol#L52](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L52)
- [Upgradable.sol#L78](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L78)

- [Distributable.sol#L51](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol#L51)

- [Operatable.sol#L51](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Operatable.sol#L51)

- [TokenManager.sol#L61](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L61)
- [TokenManager.sol#L142](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L142)
- [TokenManager.sol#L161](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L161)
- [TokenManager.sol#L171](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L171)

- [TokenManagerLiquidityPool.sol#L67](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L67)

- [InterchainTokenService.sol#L509](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L509)
- [InterchainTokenService.sol#L534](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L534)
- [InterchainTokenService.sol#L547](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L547)
- [InterchainTokenService.sol#L579](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L579)

- [RemoteAddressValidator.sol#L83](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83)
- [RemoteAddressValidator.sol#L95](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L95)
- [RemoteAddressValidator.sol#106](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#106)
- [RemoteAddressValidator.sol#L119](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L119)

- [StandardizedToken.sol#L49](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L49)
- [StandardizedToken.sol#L76](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L76)
- [StandardizedToken.sol#L86](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L86)


## 13. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 


## 14. part of the code can be pre calculated

these parts of the code can be pre calculated and given to the contract as constants this will stop the use of extra operations
even if its for code readability consider putting comments instead.

`keccak256('axelar-gateway')` can be cached and saved as a constant this will lower gas usage because of not using keccak256() when its used
- [AxelarGateway.sol#L247](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L247)


## 15. Use selfbalance() instead of address(this).balance

Use assembly when getting a contract's balance of ETH.

You can use selfbalance() instead of address(this).balance when getting your contract's balance of ETH to save gas. Additionally, you can use balance(address) instead of address.balance() when getting an external contract's balance of ETH.

Saves 15 gas when checking internal balance, 6 for external

- [Caller.sol#L16](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L16)


## 16. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [Caller.sol#L16](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L16)
- [AxelarGateway.sol#L73](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L73)
- [AxelarGateway.sol#L374](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L374)
- [AxelarGateway.sol#L443](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L443)
- [AxelarGateway.sol#L449](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L449)
- [AxelarGateway.sol#L543](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L543)
- [AxelarGateway.sol#L616](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L616)

- [InterchainProposalSender.sol#L93](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L93)

- [ConstAddressDeployer.sol#L70](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L70)

- [Create3Deployer.sol#L69](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L69)

- [Create3.sol#L52](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L52)

- [FinalProxy.sol#L61](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L61)

- [Upgradable.sol#L23](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L23)
- [Upgradable.sol#L31](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L31)

- [Implementation.sol#L19](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Implementation.sol#L19)
- [Implementation.sol#L27](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Implementation.sol#L27)

- [Multicall.sol#L25](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Multicall.sol#L25)

- [TokenManagerDeployer.sol#L38](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/TokenManagerDeployer.sol#L38)

- [TokenManager.sol#L205](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L205)

- [TokenManagerLockUnlock.sol#L46](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L46)
- [TokenManagerLockUnlock.sol#L48](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L48)
- [TokenManagerLockUnlock.sol#L51](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51)

- [InterchainTokenService.sol#L162](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L162)
- [InterchainTokenService.sol#L193](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L193)
- [InterchainTokenService.sol#L313](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L313)
- [InterchainTokenService.sol#L696](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L696)
- [InterchainTokenService.sol#L716](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L716)


## 17. Empty receive functions can cause gas issues

In Solidity, functions that receive Ether without corresponding functionality to utilize or withdraw these funds can inadvertently lead to a permanent loss of value. This is because if someone sends Ether to the contract, they may be unable to retrieve it. To avoid this, functions receiving Ether should be accompanied by additional methods that process or allow the withdrawal of these funds.

- [FixedProxy.sol#L](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L59)
- [Multisig.sol#L](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/Multisig.sol#L41)
