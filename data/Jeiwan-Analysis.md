The audited protocol represents a cross-chain bridge that facilitates the interaction between different EVM and non-EVM chains. While the core functionality of the bridge, namely the `AxelarGateway` contract, was out of scope of the audit, the in-scope contracts can be grouped as follows:
1. Contracts in `cgp/governance/` implement a cross-chain governance solution that allows to create, deliver, and execute proposals on different chains. The core contract `AxelarServiceGovernance` allows a group of signers to create and manager proposals in a coordinated way using a multisig mechanism.
2. Contracts in `gmp-sdk/deploy` and `gmp-sdk/upgradable` are auxiliary contracts that are extensively used by the core contracts. The `Create3Deployer` contract implements the CREATE3 solution that's used to deploy all the core contracts of the protocol. The set of proxy contracts is used by the core contracts of the protocol to cut the gas cost of deploying contract for the users of the protocol.
3. Contracts in `interchain-governance-executor` are meant to send (the `InterchainProposalSender` contract) and execute (the `InterchainProposalExecutor` contract) cross-chain governance proposals. The set of contracts is tightly integrated with the governance module.
4. Contracts in `its` ("Interchain Token Service") form a big mechanism of deploying, using, and transferring cross-chain tokens. The contracts allow to deploy a token manager for any existing or a new token, which facilitates cross-chain transferring of tokens. At the core of the module is the `InterchainTokenService` contract that's responsible for deploying standardized tokens and token manager on the current and remote chains.

The four describe modules are tightly integrated with the `AxelarGateway`–they call the contract to facilitate all cross-chain operations. For a bridge protocol, it's crucial to guarantee that a cross-chain message can only created and sent by an authentic sender. The protocol achieves this by letting recipients check the source chain and the source address of all messages they receive.

Since the protocol implements a mechanism of deploying and transferring of ERC20 tokens and custom token implementations, it's also critical to guarantee the connection between one token deployed to multiple chains. The protocol achieves this by creating a unique token ID for each token: the mechanism of deploying tokens to other chains guarantees that the token on the other chains will have the same token ID.

The codebase follows the best practices, while implementing such complex techniques as CREATE3 deployments and advanced usage of proxy contracts. Specifically, the protocol ensures unique and deterministic addresses for all contracts deployed by users.

The protocol has low centralization risks. All user-deployed contracts are either managed by the user who deploys them (e.g. the operator role in a token manager and the distributor role in a standardized token are set by the deployer) or have no manager at all (e.g. token managers deployed for canonical tokens). The `InterchainTokenService` (the core contract of the Interchain Token Service) has two trusted roles:
1. the owner is able to upgrade the contract to a new implementation;
1. the operator is able to set the flow limit for the contract: the flow limit allows to limit the amount of tokens that are transferred cross-chain via the contract.

The audit of the protocol was performed using the line-by-line method, followed by a higher-level checking of the codebase against the documentation.

### Time spent:
30 hours