## GAS-1: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* InterchainToken.sol (Line: 49)
* InterchainTokenService.sol (Line: 348, 372, 447, 478)
* TokenManager.sol (Line: 89, 115)

## GAS-2: Do not calculate constants

### Description

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

### Affected file

* AxelarGateway.sol (Line: 44, 45, 46, 47, 48, 49, 50, 52, 53, 54, 55, 56, 57)
* Create3.sol (Line: 41)
* FinalProxy.sol (Line: 18)
* FlowLimit.sol (Line: 15, 16)
* InterchainTokenService.sol (Line: 62, 63, 64, 71)
* InterchainTokenServiceProxy.sol (Line: 12)
* RemoteAddressValidator.sol (Line: 21)
* RemoteAddressValidatorProxy.sol (Line: 12)
* StandardizedToken.sol (Line: 27)
* StandardizedTokenProxy.sol (Line: 14)
* TimeLock.sol (Line: 13)

## GAS-3: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* AxelarGateway.sol (Line: 79, 88, 285, 293, 296, 394, 402, 432, 434, 446, 519, 523, 537)
* Caller.sol (Line: 19)
* ConstAddressDeployer.sol (Line: 51, 81)
* Create3.sol (Line: 55)
* Create3Deployer.sol (Line: 58)
* ExpressCallHandler.sol (Line: 91, 133, 212, 252)
* FinalProxy.sol (Line: 77, 80, 82)
* FlowLimit.sol (Line: 113, 126)
* InitProxy.sol (Line: 46, 50, 50, 59)
* InterchainProposalExecutor.sol (Line: 78)
* InterchainToken.sol (Line: 54, 56, 57, 93, 95, 96)
* InterchainTokenService.sol (Line: 445, 476, 652, 816, 866)
* Multicall.sol (Line: 27)
* Proxy.sol (Line: 33, 33, 40, 42)
* RemoteAddressValidator.sol (Line: 84, 96)
* StandardizedTokenDeployer.sol (Line: 31)
* StandardizedTokenProxy.sol (Line: 25)
* TimeLock.sol (Line: 51, 68, 82)
* TokenManagerDeployer.sol (Line: 23)
* TokenManagerProxy.sol (Line: 37)
* Upgradable.sol (Line: 58, 63)

## GAS-4: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* AxelarGateway.sol (Line: 254, 260, 266, 280, 385, 421, 427, 455, 469, 495)
* Distributable.sol (Line: 51)
* InterchainProposalExecutor.sol (Line: 92, 107)
* InterchainTokenService.sol (Line: 534, 547, 575)
* MultisigBase.sol (Line: 142)
* Operatable.sol (Line: 51)
* RemoteAddressValidator.sol (Line: 83, 95, 106, 119)
* StandardizedToken.sol (Line: 49, 76, 86)
* TokenManager.sol (Line: 61, 161, 171)
* TokenManagerLiquidityPool.sol (Line: 67)
* Upgradable.sol (Line: 52, 78)

## GAS-5: Internal functions only called once can be inlined to save gas

### Description

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

### Affected file

* AxelarGateway.sol (Line: 505, 615, 633, 644, 652, 662, 677)
* Distributable.sol (Line: 39)
* InterchainGovernance.sol (Line: 113)
* InterchainProposalExecutor.sol (Line: 73, 126, 142, 156, 178)
* InterchainTokenService.sol (Line: 599, 622, 666, 678, 748, 872, 880)
* Operatable.sol (Line: 39)
* Upgradable.sol (Line: 87)

## GAS-6: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* AxelarGateway.sol (Line: 72, 78, 87)
* Distributable.sol (Line: 20)
* Implementation.sol (Line: 26)
* InterchainTokenService.sol (Line: 123, 132)
* MultisigBase.sol (Line: 44)
* Operatable.sol (Line: 20)
* Pausable.sol (Line: 20)
* TokenManager.sol (Line: 35, 43)
* Upgradable.sol (Line: 29)

## GAS-7: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* RemoteAddressValidator.sol (Line: 30)

## GAS-8: The result of a function call should be cached rather than re-calling the function

### Description

External calls are expensive. Consider caching.

### Affected file

* InterchainTokenService.sol (Line: 226, 226, 228, 230, 513, 521)

## GAS-9: Use ```assembly``` to write address storage values

### Affected file

* AxelarGateway.sol (Line: 68, 69)
* FixedProxy.sol (Line: 24)
* Implementation.sol (Line: 19)
* InterchainGovernance.sol (Line: 39, 40, 41, 42)
* InterchainProposalSender.sol (Line: 43, 44)
* InterchainTokenService.sol (Line: 98, 99, 100, 101, 102, 106, 107, 108, 110, 111)
* MultisigBase.sol (Line: 48)
* RemoteAddressValidator.sol (Line: 29, 30)
* StandardizedToken.sol (Line: 56, 58)
* StandardizedTokenDeployer.sol (Line: 33, 34, 35)
* TimeLock.sol (Line: 22)
* TokenManager.sol (Line: 29)
* TokenManagerDeployer.sol (Line: 24)
* TokenManagerProxy.sol (Line: 31, 32, 33)
* Upgradable.sol (Line: 23)

## GAS-10: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* AxelarServiceGovernance.sol (Line: 79)
* InterchainGovernance.sol (Line: 120)
* InterchainToken.sol (Line: 56, 57, 58, 88, 95, 96, 97)
* InterchainTokenService.sol (Line: 104)

## GAS-11: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* AxelarGateway.sol (Line: 73, 374, 443, 449, 543, 616)
* Caller.sol (Line: 16)
* ConstAddressDeployer.sol (Line: 70)
* Create3.sol (Line: 52)
* Create3Deployer.sol (Line: 69)
* FinalProxy.sol (Line: 61)
* Implementation.sol (Line: 19, 27)
* InterchainProposalSender.sol (Line: 93)
* InterchainTokenService.sol (Line: 162, 193, 313, 696, 716)
* Multicall.sol (Line: 25)
* TokenManager.sol (Line: 205)
* TokenManagerDeployer.sol (Line: 38)
* TokenManagerLockUnlock.sol (Line: 46, 48, 51)
* Upgradable.sol (Line: 23, 31)

## GAS-12: Use selfbalance()

### Description

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

### Affected file

* Caller.sol (Line: 16)

## GAS-13: Using bools for storage incurs overhead

### Description

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back. This is the compiler’s defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false instead.

### Affected file

* AxelarServiceGovernance.sol (Line: 22)
* InterchainProposalExecutor.sol (Line: 24, 27)
* RemoteAddressValidator.sol (Line: 19)

## GAS-14: Using immutable on variables that are only set in the constructor and never after (2.1k gas per var)

### Description

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

### Affected file

* InterchainGovernance.sol (Line: 21, 22)
* InterchainProposalSender.sol (Line: 39, 40)

## GAS-15: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* AxelarGateway.sol (Line: 294, 374, 398)
* Caller.sol (Line: 18)
* ConstAddressDeployer.sol (Line: 50)
* Create3Deployer.sol (Line: 57)
* FinalProxy.sol (Line: 81)
* InitProxy.sol (Line: 58)
* InterchainProposalExecutor.sol (Line: 76)
* InterchainTokenService.sol (Line: 813, 853)
* Multicall.sol (Line: 25)
* Proxy.sol (Line: 41)
* StandardizedTokenProxy.sol (Line: 24)
* TokenManagerProxy.sol (Line: 36)
* Upgradable.sol (Line: 61)

## GAS-16: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* AxelarGateway.sol (Line: 562, 584, 598)
* ConstAddressDeployer.sol (Line: 25, 47, 63)
* Create3Deployer.sol (Line: 30, 54, 68)
* ExpressCallHandler.sol (Line: 32, 57)
* FlowLimit.sol (Line: 47, 56)
* InterchainProposalExecutor.sol (Line: 66)
* InterchainProposalSender.sol (Line: 89)
* InterchainTokenService.sol (Line: 203, 214, 242, 252, 267, 313, 401, 512, 520, 696, 755, 780, 828)
* StandardizedTokenDeployer.sol (Line: 62)
* TokenManagerDeployer.sol (Line: 38)