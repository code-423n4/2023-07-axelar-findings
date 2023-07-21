## Any comments for the judge to contextualize your findings

The findings are mainly based on the following areas of the protocol.

Lack of validation checks in the `tokenManager` address retrieval during `_processSendTokenWithDataPayload` function execution - this seemed to a misconfiguration on the developer side.
How the native tokens passed into pay the gas fees for the cross chain transaction can get locked in the contract.
How the bridge deployers can use the ethereum locked in the contract itself, to pay the gas fees of the transaction without supplying native tokens as `msg.value`.
How the `Multisig__rotateSigners` function can be misused by a malicious signer to delay a pending transaction to gain undue adavantage.
How the centralization risks (single point of failure) in the critical contracts of the protocol can be vulnerable to attacks.
There are multiple vulnerablities related to insufficient input validation checks as well.

## Approach taken in evaluating the codebase

Initially the `Interchain Governance` system was analysed. The contracts related to the `InterchainGovernance`, `InterchainProposalExecutor` and `InterchainProposalSender` were thouroghly analyzed.
Further the `Timelock` and `Multisig` contracts were audited followed by the `proxy` contracts.
Later the `Interchain token service` contract was audited together with the `tokenManager` contract and `RemoteAddressValidator` contract. Since they were the main contracts of the `Interchain Token Service`

## Architecture recommendations

Architecture can be further improved to save gas. There are multiple quality assurances that can be performed such as contract existence check when using low level `call` and `delegatecall`.

## Codebase quality analysis

Codebase was well structured. The proxy contracts were nicely coded to handle different scenarios related to the protocol. The `interchainTokenService` contract was a critical contract which had a nice structure to it.

Centralization risks

There was a huge amount of centralization risk mainly seen in the `InterchainProposalExecutor` contract since critical functions of it were only controlled by a single address which was set during the constructor.

Mechanism review

The protocol had two main components.

The interchain governance proposal system
The interchain token transfer system

Both of them were nicely structured to accomadate different chains to work together using the protocol.

Systemic risks

The system could be bit difficult to comprehend for the users who are using the `express transfer` of the `interchain token service`. SInce there is a new mechainsm introduced as the `express caller` to transfer tokens on behalf of the caller.

### Time spent:
25 hours