[L-01] `FinalProxy.finalUpgrade` doesn't verify the contract ID of the implementation
## Targets
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/FinalProxy.sol#L79
## Impact
`InterchainTokenService` can be upgraded to an incorrect implementation irrevocably, breaking the core contract of the protocol.
## Proof of Concept
The [FinalProxy.finalUpgrade](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/FinalProxy.sol#L72) sets the final implementation of the proxy, without allowing to change it later. The proxy contract is meant to work with the contracts of the protocol, and each such contract implements the `contractId` function (e.g. [InterchainTokenService.contractId](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L144)) that returns the unique ID of the contract. The [Upgradable](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Upgradable.sol#L7) contract (that's extensively used by the proxy contracts of the protocol) [checks the `contractId` of a new implementation](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Upgradable.sol#L45), however `FinalProxy.finalUpgrade` doesn't do that.

In the protocol, [InterchainTokenServiceProxy](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/proxies/InterchainTokenServiceProxy.sol#L11) is the only proxy that inherits from the `FinalProxy` contract. This means that if `InterchainTokenService` is upgraded to an incorrect implementation, it'll break the entire cross-chain token management functionality of the protocol.
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider checking the contract ID of the final implementation in the `FinalProxy.finalUpgrade` function, as it's done in `Upgradable.upgrade`.