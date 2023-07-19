
Gas Optimizations Report
========================

Table of contents
=================

* [Gas Findings](#gas-findings)
	* [[Gas-1] Do not cache msg.sender](#gas-1-do-not-cache-msgsender)
	* [[Gas-2] Use != 0 instead of > 0](#gas-2-use--0-instead-of--0)
	* [[Gas-3] Rearrange state variables](#gas-3-rearrange-state-variables)
    * [[Gas-4] State variables that could be set immutable](#gas-4-state-variables-that-could-be-set-immutable)
	* [[Gas-5] Unnecessary index init](#gas-5-unnecessary-index-init)
    * [[Gas-6] Change owner with two steps verification process](#gas-6-change-owner-with-two-steps-verification-process)
    * [[Gas-7] Unnecessary equals boolean](#gas-7-unnecessary-equals-boolean)

# Gas Findings

## [Gas-1] Do not cache msg.sender
We recommend not to cache msg.sender since calling it is 2 gas while reading a variable is more.

        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/Ownable.sol#L11
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/test/DepositReceiver.sol#L23
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/test/ReceiverImplementation.sol#L32
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/test/ReceiverImplementation.sol#L59
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/test/ReceiverImplementation.sol#L86


## [Gas-2] Use != 0 instead of > 0
Using != 0 is slightly cheaper than > 0. We recommend to replace > with != in the following places:

        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L159
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L635
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L79
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L16
        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/express/AxelarValuedExpressExecutable.sol#L175


## [Gas-3] Rearrange state variables
You can change the order of the storage variables to decrease memory uses. This is because the new suggested new orders will use less storage slots.

        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol: 
old number of slots: 5 
new number of slots: 4 
the new order of types is
        1. string
        2. string
        3. bytes32
        4. address
        5. uint8

## [Gas-4] State variables that could be set immutable
You can set the following state variables to immutable and save gas:

- [MultisigBase.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol): signerEpoch
- [DepositHandler.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/DepositHandler.sol): _lockedStatus
- [DestinationChainTokenSwapper.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/test/gmp/DestinationChainTokenSwapper.sol): tokenA
- [DestinationChainTokenSwapper.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/test/gmp/DestinationChainTokenSwapper.sol): tokenB
- [SourceChainSwapCaller.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/test/gmp/SourceChainSwapCaller.sol): gateway


## [Gas-5] Unnecessary index init
In for loops you initialize the index to start from 0, but it already initialized to 0 in default and this assignment cost gas. It is more clear and gas efficient to declare without assigning 0 and will have the same meaning:

        https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L24


## [Gas-6] Change owner with two steps verification process
Consider having two steps verification to change owner to avoid human errors. The following contracts use direct transfer.

- [AxelarGateway.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol)
- [Upgradable.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol)


## [Gas-7] Unnecessary equals boolean
Boolean variables can be checked within conditionals directly without the use of equality operators to true/false.

- [AxelarAuthWeighted.sol#L124](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/AxelarAuthWeighted.sol#L124)

