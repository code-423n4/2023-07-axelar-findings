# Analysis - Axelar Network
# Summary

| List |Head |Details|
|:--|:----------------|:------|
|1 | Overview of Axelar Network Protocol| overview of the key components and features of Axelarb Network  |
|2 | My Thoughts | My own thoughts about future of the this protocol |
|3 |Audit approach | Process and steps i followed  |
|4 |Learnings | Learnings from this protocol|
|5 |Possible Systemic Risks | The possible systemic risks based on my analysis |
|6 |Code Commentary | Suggestions for existing code base |
|7 | Centralization risks | Concerns associated with centralized systems |
|8 |Gas Optimizations | Details about my gas optimizations findings and gas savings  |
|9 |Risks | Possible risks |
|10 |Non-functional aspects | general suggestions |
|11 |Time spent on analysis  | The Over all time spend for this reports |

## Overview

Axelar is a decentralized interoperability network connecting all blockchains, assets and apps through a universal set of protocols and APIs.

- It is built on top off the Cosmos SDK
- Use Axelar network to send tokens between any Cosmos and EVM chains.They can also send arbitrary messages between chains
- The relayer (can be anyone due to being permissionless) triggers the execute method automatically once the message has been confirmed by the Axelar network.

### My Thoughts
Axelar could change the future in following areas

- ``Decentralized finance (DeFi)``: Axelar could make it easier for users to access DeFi applications on different chains. This could help to increase the liquidity of DeFi markets and could also make DeFi applications more accessible to users.

- ``NFTs``: Axelar could make it easier for users to trade NFTs between different chains. This could help to increase the liquidity of NFT markets and could also make NFTs more accessible to users.

- ``Gaming``: Axelar could make it easier for users to play games across different chains. This could help to increase the reach of blockchain games and could also make games more immersive for users.

## Audit approach

I followed below steps while analyzing and auditing the code base.

1. Read the contest Readme.md and took the required notes.

  - Axelar Network protocol uses 
    - Uses Inheritance
    - Follows the ERC20 Standards
    - Uses TimeLocks 
    - Multi-chain

2. Analyzed the over all codebase one iterations very fast

3. Study of documentation to understand each contract purpose, its functionality, how it is connected with other contracts, etc.

4. Then i read old audits and already known findings. Then go through the bot races findings 

5. Then setup my testing environment things. Run the tests to checks all test passed.

6. Finally, I started with the auditing the code base in depth way I started understanding line by line code and took the necessary notes to ask some questions to sponsors.

## Stages of audit

- ``The first stage of the audit``

During the initial stage of the audit for Axelar Network, the primary focus was on analyzing gas usage and quality assurance (QA) aspects. This phase of the audit aimed to ensure the efficiency of gas consumption and verify the robustness of the platform.

- ``The second stage of the audit``

In the second stage of the audit for Axelar Network, the focus shifted towards understanding the protocol usage in more detail. This involved identifying and analyzing the important contracts and functions within the system. By examining these key components, the audit aimed to gain a comprehensive understanding of the protocol's functionality and potential risks. 

- ``The third stage of the audit``

During the third stage of the audit for Axelar Network, the focus was on thoroughly examining and marking any doubtful or vulnerable areas within the protocol. This stage involved conducting comprehensive vulnerability assessments and identifying potential weaknesses in the system. Found ``60-70`` ``vulnerable`` and ``weakness`` code parts all marked with ``@Audit tags``.

- ``The fourth stage of the audit``

During the fourth stage of the audit for Axelar Network, a comprehensive analysis and testing of the previously identified doubtful and vulnerable areas were conducted. This stage involved diving deeper into these areas, performing in-depth examinations, and subjecting them to rigorous testing, including fuzzing with various inputs. Finally concluded findings after all research's and tests. Then i reported C4 with proper formats 


## Learnings

- ``Axelar's architecture``: The Axelar network is composed of three main components: the relayers, the validators, and the gateway smart contracts. The relayers are responsible for transferring messages between chains, the validators are responsible for confirming events and commands on different chains, and the gateway smart contracts are responsible for managing the transfer of tokens and data between chains.

- ``Axelar's security model``: Axelar's security model is based on a combination of decentralized validators and secure smart contracts. The decentralized validators are responsible for confirming events and commands on different chains, and the secure smart contracts are responsible for managing the transfer of tokens and data between chains.


- ``Axelar's roadmap``: The Axelar team has a roadmap for the development of the network. The roadmap includes plans to add support for more chains, to improve the security of the network, and to make the network more user-friendly.

## Possible Systemic Risks

- ``Relayer attacks``: A relayer could potentially attack the Axelar network by submitting malicious commands to the gateway smart contracts. This could result in the theft of tokens or the loss of data.

- ``Double-spend attacks``: A user could potentially double-spend tokens by sending them to two different chains. This could be done by sending the tokens to one chain and then submitting a malicious command to the gateway smart contracts to cancel the transaction on the other chain.

- ``Sybil attack``: A malicious actor could potentially create a large number of fake validators in order to gain control of the Axelar network. This could allow the malicious actor to approve malicious commands or to prevent legitimate commands from being processed.

- ``Buggy smart contracts``: The Axelar gateway smart contracts could contain bugs that could be exploited by attackers. This could result in the theft of tokens or the loss of data.


## Code Commentary

- ``Events`` is missing ``indexed`` fields
    Index event fields make the field more quickly accessible to off-chain.Each event should use three indexed fields if     there are three or more fields.

- Shadowing variables can be confusing in contracts, as it can be difficult to track which variable is being referenced at any given time. This can lead to errors and unexpected behavior. 

  -  ``operator`` shadowed varibale used in ``InterchainTokenService.sol``. The same varibale name used in ``Operator.sol``
  -  ``gateway`` shadowed varibale used in ``InterchainGovernance.sol``. The same varibale name used in ``AxelarExecutable .sol`` as ``IAxelarGateway`` instance
  -  ``gateway, governanceChain, governanceAddress`` used in ``AxelarServiceGovernance.sol``

- Use latest version of solidity. Use atleast 0.8.17 

- Don't use bool . Use ``uint(1)`` and ``uint(2)`` for true,false

- Use ``abi.encode`` instead of ``abi.encodePacked`` when ever possible to avoid hash collisions 

- Use ``notPaused`` Modifier in important functions 

- Consider using a struct to represent trusted addresses instead of separate arrays for chain names and addresses. This can enhance code readability and make it easier to manage trusted addresses

- Explicitly specify the visibility modifiers for functions and state variables, such as public, external, internal, or private, based on their intended use

- Consider using bytes32 for governanceChain and governanceAddress if they are fixed-size strings to save gas costs

- Add informative error messages in require statements to provide clear feedback on revert reasons

- Add safeguards to reentrnacy attacks

- Use enum values directly instead of converting commandId to an IAxelarGateway.Command enum in the execute and executeWithToken functions, use the enum values directly to simplify the code.

- Add validations in ``MultisigBase.sol`` the constructor to ensure that the provided list of signers is not empty and that the threshold is not zero

-  Move the voting logic out of the onlySigners modifier and into a separate function, which can be called explicitly when a signer wants to vote on a specific topic. This separation improves code readability and allows for better control over when a vote is cast

- Some important state changes might not be adequately captured by events. Consider emitting events for significant state changes to improve transparency and allow external systems to track contract activity

- Add detailed comments explaining the purpose and functionality of each function. This can significantly improve code readability and ease maintenance

- Ensure consistent naming conventions throughout the codebase to enhance clarity and reduce confusion

- Consider breaking the contract into smaller, modular components. This can make the codebase more manageable and easier to maintain



## Centralization risks

A single point of failure is not acceptable for this project Centrality risk is high in the project, the role of ``onlyOwner``detailed below has very critical and important powers

Project and funds may be compromised by a malicious or stolen private key ``onlyOwner`` ``msg.sender``

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L96-L96
```solidity
92:     function setWhitelistedProposalCaller(
93:         string calldata sourceChain,
94:         address sourceCaller,
95:         bool whitelisted
96:     ) external override onlyOwner  
```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L111-L111

```solidity
107:     function setWhitelistedProposalSender(
108:         string calldata sourceChain,
109:         address sourceSender,
110:         bool whitelisted
111:     ) external override onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L56-L56
```solidity
52:     function upgrade(
53:         address newImplementation,
54:         bytes32 newImplementationCodeHash,
55:         bytes calldata params
56:     ) external override onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L547-L547

```solidity

547:     function setPaused(bool paused) external onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83-L83

```solidity
83:     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L95-L95

```solidity

95:     function removeTrustedAddress(string calldata chain) external onlyOwner

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L106-L106

```solidity

106:     function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner  

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L119-L119

```solidity

119:     function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner 

```
## Gas Optimizations

In the context of optimizing gas consumption, several improvements have been identified within the codebase of the ``Axelar protocol``. One of the main optimizations involves combining multiple address mappings into a single mapping of an address to a ``struct`` where appropriate. By packing the structs and modifying the uint type for the values, ``storage`` reads become more ``cost-efficient``, resulting in ``substantial gas savings``. Additionally, using the ``calldata`` keyword instead of ``memory`` for read-only arguments in external functions reduces gas usage significantly, as it avoids unnecessary ``data copying``.

Another gas optimization relates to avoiding the ``overhead`` of using ``bool storage``, which incurs extra gas costs due to read and write operations. By replacing bool variables with appropriate uint values (e.g., 1 and 2) to represent ``true`` and ``false``, the contract can avoid additional ``SLOAD`` operations, leading to substantial gas savings across multiple instances.

Furthermore, using ``low-level calls`` instead of external function calls can also save gas by eliminating ``contract existence checks``. This is particularly useful for functions that are expected to revert when called by regular users since it avoids ``unnecessary opcode execution``.

To further reduce gas consumption, the codebase can be updated to use ``calldata`` pointers instead of ``memory`` pointers, which offer cost savings in loop iterations. Additionally, moving ``IF`` and ``require()`` statements that check input arguments to the top of functions can avoid ``unnecessary computation`` and ``potential gas waste``.

Marking functions guaranteed to revert when called by normal users as ``payable`` is another optimization to reduce gas costs, as it avoids the extra gas spent on checking whether a payment was provided.

To enhance the readability and efficiency of the code, ``optimizing names`` for functions and variables can lead to gas savings by reducing method IDs and method call costs.

Finally, by eliminating unnecessary ``default value initialization``, such as explicitly setting variables to their default values, gas savings can be achieved by allowing variables to use their default values automatically.

Overall, implementing these gas optimizations can significantly enhance the efficiency and cost-effectiveness of the ``Axelar protocol``, making it more robust and scalable for its users

## Risks

- The contract uses strings for ``governanceChain`` and ``governanceAddress``. While it's possible to use strings for these purposes, it's generally better to use bytes32 for fixed-size strings to save gas costs and avoid potential security vulnerabilities. If the governance chain and address are known to have fixed lengths, consider using bytes32 instead.

- contract emits events such as ``ProposalExecuted``, ``ProposalScheduled``, and ``ProposalCancelled``, there are no comments or documentation explaining what these events signify and the expected format of the emitted data.

- Lack of access control in ``executeProposal()`` function

- The ``target`` address could be an invalid or malicious contract address, leading to unexpected behavior or potential vulnerabilities. Ensure that you add appropriate input validation to check the validity of the ``target`` address and other inputs.

- The rotateSigners function is used to both rotate the signers and set the initial signers. It might be confusing to use the same function for both purposes. Consider splitting the functionality into two separate functions, one for the initial setup and another for rotating the signers

- The getSignerVotesCount function iterates through all signers to count the votes for a topic. This operation has a linear complexity (O(n)), where n is the number of signers.

- The onlySigners modifier modifies contract state by updating the vote count and setting the hasVoted mapping. There could be potential reentrancy vulnerabilities if any function called within the modifier allows external contract calls before the state modifications are completed

- The contract uses keccak256(msg.data) to generate the vote topic hash. Using msg.data directly can lead to inconsistent hash generation due to differences in ABI encoding

- The contract only provides a single signer threshold for executing operations. In more complex multisignature contracts, it might be beneficial to have different levels of signers with different threshold requirements for certain types of transactions

- The contract does not perform any signature verification to ensure that the transactions are signed correctly by the required number of signers. Without proper signature validation, the security of the multisignature contract may be compromised

- Some functions are marked as onlySelf, which means they can only be called by the contract itself. Make sure that these functions are intended for internal use only and cannot be called by unauthorized parties

- Limit the use of delegatecall to only trusted and thoroughly audited contracts. If possible, avoid using delegatecall altogether as it can introduce complex security risks

- ``return codehash != bytes32(0) && codehash != 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470`` avoid the hardcoded value check. This will cause problems in in future.

- Be cautious with loops and array processing. If the number of ``interchainCalls`` in the ``sendProposals`` function is significant, it could cause the transaction to exceed the block gas limit, leading to a revert. Ensure that the gas limit is well within the block limit.

- ``InterchainProposalSender.sol`` contract doesn't have any nonce handling mechanism to prevent replay attacks. Consider adding a nonce or some other form of unique identifier to each proposal to prevent replay attacks

- The contract inherits from AxelarExecutable, which likely implements a fallback function. Make sure to review the fallback function's implementation in the AxelarExecutable contract to ensure it behaves as expected

-  The contract ``ConstAddressDeployer.sol`` uses create2 to deploy contracts. While this is generally safe and useful for deterministic contract deployments, be aware that contract deployment might consume more gas compared to standard create calls. Additionally, the size of the deployment bytecode should be kept in check to avoid potential out-of-gas issues

- The contract uses assembly blocks to read and write from/to storage slots. While this can be an efficient way to handle storage operations, it increases the complexity of the code. Consider carefully evaluating the need for assembly and use it only when necessary to optimize gas costs

- The ``_popExpressReceiveToken`` and ``_popExpressReceiveTokenWithData`` functions return the address expressCaller, but it is not used within the functions. If the return value is not needed, you can remove the assignment and just return early when the express caller is not found

- When accessing storage slots with sload, be cautious about reading from slots that have not been written before. For example, you can initialize storage slots to a default value (e.g., 0) before using them

- The _getExpressReceiveTokenSlot and _getExpressReceiveTokenWithDataSlot functions use keccak256 to calculate slots based on various parameters. Ensure that the combination of parameters used to calculate slots results in unique slots for each express call to avoid potential storage slot collisions

- The contract inherits from Upgradable, which might imply that this contract is designed to be upgradable. Upgradable contracts can be complex and require careful consideration to maintain security and compatibility across versions

## Non-functional aspects

- Aim for high test coverage to validate the contract's behavior and catch potential bugs or vulnerabilities
- The protocol execution flow is not explained in efficient way. 
- Its best to explain over all protocol with architecture is easy to understandings 
- Consider designing the contract to be upgradable or allow for versioning. This can help address issues, introduce new features, or adapt to evolving requirements without disrupting the entire system
- Consider using standardized security libraries like OpenZeppelin to minimize vulnerabilities

## Time spent on analysis 

``15 Hours``

















































### Time spent:
14 hours