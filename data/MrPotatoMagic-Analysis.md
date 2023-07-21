# Analysis Report

## Approach taken in evaluating the codebase
To start evaluating this project, I identified the high level functionalities and its corresponding contracts involved in the scope (more on this in the Mechanism Review section below). The evaluation of the contracts (for each of a,b,c) was done in a base-to-derived contract manner. The functionalities are as follows:
a. Interchain Governance
b. Interchain Token/Message transfer
c. Helper utility contracts involved in facilitating a and b as well as to upgrade certain contracts (I will not be diving deeper into this separately but will refer to the contracts wherever necessary).

Additionally, I approached this codebase from an attacker's mindset. Through this type of mentality, I could recognise three points where failure/potential bugs could exist: 
1. The endpoints that customers interact with 
2. The internal endpoints that validators interact with
3. The internal workings and function flow that connects 1 to 2.

This diagram serves as a great reference to visualize these endpoints: https://cdn.discordapp.com/attachments/1126884239330263072/1130039751173484604/gmp-diagram.webp

## Architecture recommendations
The architecture of the protocol is quite simple and easy to understand. The developer team has done an amazing job in modularising the code into smaller chunks and storing it in separate files. The only recommendation I would have is to divide the InterchainTokenService.sol contract into two files. The pattern of this contract is as follows:
```
=> represents "calling"
Public/External functions => Internal functions (within the same contract)
```
Having the public/external functions and the internal functions in two separate files would greatly reduce the size of the file from 533 nSLOC to half of it, making the code more readable, modular and less concentrated (just like the rest of the contracts).

## Codebase quality analysis
Overall, the codebase is structured in an easy to understand manner. The documentation and Natspec comments provided helps to recognize the endpoint (both customer and validator side) and intermediary contracts involved in facilitating interchain transfers. There are minor edge case issues that can lead to big exploits. Some of these issues (that I have included in my findings) are invalid implicit type conversion, improper validation of create2 return value, strict equality check, freezing governance functions when malicious signer votes when vote count is one less than threshold etc. Additionally, there are numerous instances across the codebase where a caller's ETH can get locked. Other than this, the developer team has done a good job with handling storage variables (For example, having storage structs, enums and mappings in their own separate files, updating boolean and uint256 Voting struct members in the onlySigners() modifier correctly etc). 

## Centralization risks
There seem to be several centralization points in the protocol, which I believe decrease the decentralisation in an interoperable network. They are here as follows:
1. Owner - Whitelisting addresses indirectly adds centralization, which can compromise the whole protocol if owner's private key gets leaked. 
2. Distributor - Being allowed to mint and burn tokens
3. Operator - Allowed to set flow limit and liquidity pool

If any of these centralized entities have leaked private keys, it can destabilize and halt the protocol since they have control over important settings of the protocol like flow in and flow out limits, the address of the liquidity pool, minting and burning tokens etc. A multi-sig should be used for configuration operations like these in order to stay one step away from losing control over the protocol.

## Mechanism Review
To understand the mechanism of this protocol, we need to look at the functionalities in scope:
a. Interchain Governance
b. Interchain Token Service
c. Helper utility contracts involved in facilitating a and b as well as to upgrade certain contracts (I will not be diving deeper into this separately but will refer to the contracts wherever necessary).

Here is a mindmap to get an extremely high level overview of this protocol with the above mentioned functionalities in scope: https://drive.google.com/file/d/1YEMRzE9MChjO1_6l-ru1RUhUzhuJqxyn/view?usp=sharing

### 1. Interchain Governance

Here is a mindmap that explains the inner workings and important function calls of the Interchain Governance mechanism. I'll be referring to this while explaining: https://drive.google.com/file/d/1ocbESssCLWZi4Xtll-GrM41xWzdrCng_/view?usp=sharing

The diagram provides a high-level overview of how the main contracts are connected. The heaviest contracts (functionality-wise) involved are InterchainGovernance.sol, MultisigBase.sol, InterchainProposalSender.sol and InterchainProposalExecutor.sol. The rest of the blue box contracts (refer to the legend in diagram for more info) are either helper contracts or contracts inheriting from these heavier contracts to extend functionality. To simply explain the whole process:

1. InterchainGovernance.sol contract can create, cancel and execute proposals. 
a. Creating and cancelling of proposals happens through specific commandIds, which are basically their index in the enum they are stored in. To create and cancel, you can pass in the respective commandId (0 for create, 1 for cancel) as parameter value to the _execute() function. 
b. Execution of the proposal happens through the executeProposal() function, which finalizes the timelock (i.e. sets the hash of the proposal to 0) and executes the proposal using the low-level call in the Caller.sol contract. Note, the proposals have a minimum delay before they are ready to execute. This minimum delay is added by the Timelock.sol contract while scheduling the proposal.

2. AxelarServiceGovernance.sol - Does everything InterchainGovernance.sol can do, the only addition being execution of multisig proposals, which are handled by the signers (rotated every epoch) set in the the MultisigBase.sol contract.

3. Multisig.sol - Inherits from MultisigBase.sol and allows signers to execute any function in any target contract.

4. InterchainProposalSender.sol and InterchainProposalExecutor.sol play hand-in-hand in the sending and execution of proposals. The proposal sender is a contract on the source chain, which communicates with the proposal executor on the destination chain through the AxelarGateway.sol contract. Note in order for execution to occur on the destination chain, the source caller and source address need to be whitelisted by the owner.

5. InterchainCalls.sol - This is a contract that stores the struct definition for interchain calls, which includes destination chain specific info and calls to be executed. It is used in the proposal sender and executor contracts.

### 2. Interchain Token Service

Here is a mindmap that explains the inner workings and important function calls of the Interchain Token Service. I'll be referring to this while explaining: https://drive.google.com/file/d/1AQI1QwuYzXASMtsqL6VfIGoQza0Fygv_/view?usp=sharing

The diagram provides a high-level overview of how the main functions communicate. Refer to the legend to understand what each color represents. The way to read this mindmap would be to start from the top left corner. 

1. The first contract we come across is the InterchainTokenService.sol contract. This is by far the heaviest contract (both functionality-wise and nSLOC-wise). The first level in the hierarchy (side-ways) are the callable functions in the contract. Each function either directly or indirectly (through internal functions - level 2 in side-ways hierarchy) deploys contracts on both source and destination chains. What differentiates the contracts deployed on the source and destination chains is the "remote" keyword used in the function names. The function names and comments provided by the team are self-explanatory so I will not be delving much deeper into explaining what each function does. 

2. InterchainToken.sol and TokenManager.sol are both the originators of interchain transfer calls (since the functions in both contracts are external, thus call can originate from either). The InterchainToken.sol calls the TokenManager.sol, which further calls the InterchainTokenService.sol contract. Here is how the flow looks:
```
InterchainToken.sol => TokenManager.sol => InterchainTokenService.sol (function transmitSendToken()) => AxelarGateway.sol (function callContract())
```
They both call the transmitSendToken() function to directly/indirectly (check disgram) to emit an event in the callContract function in the AxelarGateway.sol contract. The execution on the destination chain follows as mentioned by this example provided in the third paragraph of the Overview section in the README.md file: https://github.com/code-423n4/2023-07-axelar#overview 

3. RemoteAddressValidator.sol - The main function of this contract is to add/remove trusted interchain token service addresses and add/remove Axelar supported gateway chains. This contract also provides a function, which can be used to validate if the sender is a valid interchain token service address.

4. StnadardizedToken.sol - Inherits from InterchainToken.sol to allow for interchain transfer (although this can be considered as originator of interchain transfer as well, I have not marked it in the diagram). This contract provides a setup function (Callable by onlyProxy) that sets up certain parameters like the distributor, tokenManager etc. Additionally, the distributor can use the token minting and burning functions.

5. TokenAddressStorage.sol - This can be considered as originator of interchain transfer since it inherits from the TokenManager.sol contract (not marked in diagram though). It has a setter function to set the token address as well. Although this contract is not important by itself, it is used by its child contracts - TokenManagerLockUnlock.sol, TokenManagerMintBurn.sol and TokenManagerLiqudiityPool.sol. 

6. TokenManagerLockUnlock.sol, TokenManagerMintBurn.sol and TokenManagerLiqudiityPool.sol - These are different implementation types of the token manager and are differentiated by their implementation number (i.e. their index in the TokenManagerType enum located in the ITokenManagerTypes interface). Additionally, each contract has their own giveToken() and takeToken() functions that interact with the FlowLimit.sol contract. 

7. FlowLimit.sol - This contract makes sure that while giving and taking tokens, the flow limit set by the operator is adhered to in order to prevent exploit attacks.


### Time spent:
90 hours