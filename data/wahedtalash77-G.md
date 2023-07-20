## [G-01]Remove the unnecessary `return` statement to save gas

90 : `return;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L90

95: `return;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L95

100 : `return;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L100

105 : `return;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L105
62: `return;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L62

654: `return;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L654

## [G-02] Caching global variables is expensive than using the variable itself
Don't cache msg.sender 
49: `address sender = msg.sender;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L49

348: `address deployer_ = msg.sender;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L348

447: `address caller = msg.sender;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L447

## [G-03] Use calldata instead of memory for function arguments that do not get mutated

```
 function _getProposalHash(
        address target,
        bytes memory callData,
        uint256 nativeValue
    ) 
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L143C3-L147C6
```
function _processCommand(
        uint256 commandId,
        address target,
        bytes memory callData,
        uint256 nativeValue,
        uint256 eta
    ) internal override
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L72C5-L78C24



## [G-04] Optimize names to save gas
>Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

>See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
 

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol

**Recommended Mitigation Steps**
>Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas.

## [G-05] `variable == false ` instead of   `!variable`.
==a bit cheaper when you replace:==
19: `if (!success) {`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L19

78: ` if (!success) {`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L78

55: ` if (!multisigApprovals[proposalHash]) revert NotApproved();`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55

45: `if (!signers.isSigner[msg.sender]) revert NotSigner();`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L45

## [G-06]  Avoiding Initialization of the variables to their default value.
105: `uint256 totalGas = 0;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L105

106: `for (uint256 i = 0; i < interchainCalls.length; ) {`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106

## [G-07]  x += y costs more gas than x = x + y for state variables
107: `totalGas += interchainCalls[i].gas;`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L107

## [G-08] You can save storage by reordering Holding struct fields.
```
 struct Call {
        address target;
        uint256 value;
        bytes callData;
    }
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/lib/InterchainCalls.sol#L26C4-L30C6