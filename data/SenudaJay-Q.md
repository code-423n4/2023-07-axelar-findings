
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L53
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L73

To maintain code clarity and consistency it is better to use `_getProposalHash(target, callData, nativeValue)` to retrieve the proposal hash in the above instances rather than computing them directly