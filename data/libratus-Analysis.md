
# Overview
This analysis is a part of an audit for Axelar, a cross-chain communication protocol. Axelar uses Proof-of-Stake network of nodes to relay messages between different blockchains. The audit is focused on two new features: *Interchain Governance* and *Interchain Token Service*. Both features are built on top of existing infrastructure and mechanisms of Axelar protocol.

**Interchain Governance** allows DAOs to create and execute governance proposals across multiple blockchains. The implemetation includes features like:

- Creating and executing proposals.

- Timelock mechanism.

- Multisig execution.

**Interchain Token Service** is a set of tools for building interchain ERC20 tokens with two main use-cases:

- Creating bridged version of existing assets like USDC or UNI.

- Developing interchain-native ERC20 tokens where developers have control over the token contracts on multiple chains.

Both of these types of assets leverage Axelar infrastructure to enable secure cross-chain bridging.

# Architecture

## Proposal Execution
Proposals in the Interchain Governance component can execute any smart contract method on any of the supported blockchains. There are two types of safeguards that can be used before executing a proposal: a timelock and a multisig. It is vital to ensure that these two safeguards are working as intended.

It is good that the timelock has a minimum delay property. Otherwise it would be possible to create proposals with a very low delay.

Multisig may or may not be a valid safeguard depending on how it is configured. If threshold value is too low, then not only can the minority execute proposals but they can also completely seize access of a multisig because signers rotation is also guarded by the same threshold.

One downside of the governance component is that it is not possible to use both safeguards at the same time and for each proposal the creator has to choose one or the other.
  
## Sending proposals cross-chain

Cross-chain communication is achieved with the help of two contracts, `InterchainProposalSender` (deployed on source chain) and `InterchainProposalExecutor` (deployed on destination chain). 

It is important to have proper access control in this pipeline to not allow attackers to execute random code on the destination chain. This is currently done by an owner of the Executor contract that maintains a whitelist of callers from the source chain. While it is a simple and robust solution, it is also a centralized one. If the goal is to reduce centralization, then it is better to only allow changing the whitelist via proposals themselves.

## Interchain Tokens

InterchainToken is a very lightweight extension of ERC20. It makes it easy for developers to implement a standard and leaves little room for errors.

The biggest potential issue is related to bridging limits in TokenManager. While it is generally a good practice to have such limits, the operator managing them can be any account, including EOA. This introduces centralization risk since it allows token developers to stop bridging at a whim by lowering the limits. There should be an interchain token development guidance where best practices regarding deploying and operating such tokens are shared. Also, users should be educated that depending on the token implementation bridging can be stopped.

## Interchain Token Service

InterchainTokenService is an Upgradable and Pausable smart contract. It is understandable that upgradability can be an important feature for project development, but it brings additional risks. With a malicious upgrade and adversary can mint any number of tokens, change flow limits and more. If possible, upgradability logic should be implemented under a timelock.  

# Codebase

Codebase is very clean with minimal code duplication. Docummentation attached to the repo outlines the high-level design, while the abundance of in-code comments help understand the implementation details. Tests are mainly focused on critical parts of the protocol. The coverage of some peripherals can be increased.
  
### InterchainGovernance/AxelarInterchainGovernance
There is some code duplication in AxelarInterchainGovernance. For example hash calculation can be done by calling parent's `_getProposalHash`  instead of doing the calculation itself.  

### InterchainToken
Token Manager follows Proxy-Implementation pattern. Since the proxy isn't upgradable it is unclear what are the benefits versus using implementation contract directly. Whatever the benefits are, it should be evaluated whether they worth the risk of increasing the complexity.

### Axelar Gateway

Axelar Gateway seems to be copied from another repository just for the audit. It is not documented and uses old solidity version.

### Multisig

Multisig contract exists but is not used and the purpose of it is unclear. Other contracts inherit directly from MultisigBase.

Including multisig logic in `onlySigners` modifier is confusing. It would be much clearer to separate the process of voting and the process of action execution. For example, `onlySigners` is applied to payable methods and so the last signer would always be the one responsible to pay for the execution. On the opposite, if someone has already voted, they are excluded from being able to execute the action, even though they may have the native token to pay for it.  

### InterchainTokenService

The project uses custom upgradability implementation. It is simple and lightweight. However, using custom implementations is always risky and reduces security properties of the system. Open source implementations can be considered. 
 
# Summary

The audited features are well designed and the code is clean and easy to understand. There are important safeguards implemented. The biggest concern is probably centralization risk. Some contracts are upgradable and some allow for arbitrary EOA accounts to perform vital management functions.
The use of custom upgradability implementation increases smart contract risks and using proxy-implementation patterns brings some complication to the code that could be avoided.

### Time spent:
36 hours