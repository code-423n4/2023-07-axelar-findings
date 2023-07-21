### Summary of the contracts in scope

The audited contracts are part of Axelar Network’s interchain general message passing, governance and token transfer services. The assets in scope implement a few new features:

- In AxelarGateway, a new upgrade mechanism (that uses interchain decentralized governance) is introduced. Additionally, a mint limiter role is added, setting limits to the amounts that can be minted for any token that supports interchain minting.
- Related to the above, a general interchain governance mechanism is introduced, that allows a combination of interchain messages and multisignature wallet transactions to execute arbitrary governance proposals across chains.
- A general interchain token service. This service allows users to permissionlessly deploy new token managers that act as bridges for arbitrary tokens, using a variety of patterns, e.g. a typical mint/burn bridge or a lock/unlock bridge.

The codebase also includes helpers that allow deploying contracts using the create3 pattern, that produces a deployment address that is solely dependent on the sender address and a chosen salt, not the bytecode.

### Methodology and time spent

I spent about 11 hours diving deep into the codebase, and then a couple more cleaning up the reports, thinking through the proofs of concept, and preparing the submissions (including this report). My approach was straightforward: visual inspection of the codebase, cross-checking with unit tests and documentation, and thinking through potential edge cases related to timing, reentrancy, ordering of transactions, etc.

I believe I did a thorough review of the governance-related changes, but a slightly less in-depth review of the interchain token service. The following summarizes my findings and thoughts from what I could gather in this time.

I focused on High/Medium severity issues (though I only found Medium ones) and this Analysis, so it’s likely that I won’t be submitting a QA or gas report. The audit bots seem to have caught some of the usual QA issues, so I will highlight some things that are not covered in bot output, and affect the code in a broader way, in “Other recommendations” below.

### New/unexpected things

- I wasn’t familiar with the CREATE3 pattern, and it looks very useful. It does introduce some risks, as it makes it easier to deploy contracts with the same address in other chains, which could have unintended consequences (this relates to one of the Medium severity issues I submitted).
- Even though it’s out of scope, the EternalStorage pattern is an interesting approach that I wasn’t familiar with either. I can see how it adds flexibility to handle storage during upgrades. However, it also increases the complexity as it requires specific getters for each variable, and it’s unclear to me if the benefits outweigh the increased attack surface and potential for errors. I’d be curious to hear how this has worked out for this team so far.
- The express token transfer mechanism is impressive; the idea that someone can send the tokens to the destination ahead of time and then be paid back when the message is received is a great way to do fast bridging in a mostly trustless way.

### Architecture feedback

Generally the codebase looks clean and the architecture seems sound. Interchain governance and generalized token bridging are massive problems to tackle and the developers have approached them in a straightforward, simple way.

I’ve raised a couple Medium issues that relate to timing and ordering of messages and transactions, on the multisig and the interchain governance contracts. This would prompt me to suggest taking a closer look at potential timing conditions with interchain messages, and the impact of messages getting delayed due to gas. This is one of the trickiest things to get right when doing cross-chain messaging, so I wouldn’t be surprised if there are a few other edge cases where this could cause issues.

A general suggestion I have for the architecture is on the amount of proxy contracts that are defined in the codebase. A lot of different proxies are available, and in some cases there is very little difference between each type. This leads to some duplication, e.g. proxy fallback functions re-implemented in several places, which increases the attack surface and risk of errors. I would suggest, on a future iteration, to attempt to combine some of these into fewer contracts to improve maintainability. For instance, proxies that differ only in the `contractId()` could have the ID be defined as an immutable and set at proxy deployment time.

### Centralization risks

Some of the contracts in scope present upgrade mechanisms but use decentralized governance with timelocks. The cases where multisigs are used are a form of centralization risk (especially if the number of signers is low). AxelarServiceGovernance requiring both an interchain approval *and* a multisig execution provides some additional security against compromised signers.

In general, token projects using the Axelar gateway token transfer service are trusting the Axelar network and its governance mechanism with minting power for their tokens - this governance is decentralized and this is part of the core trust assumption when using Axelar, but it’s worth noting that the mint limiter is a separate, more centralized role (according to the docs, to be handled through a multisig).

For the interchain token service, an important centralization risk is noted in `DESIGN.md`:
> Users using Custom Bridges need to trust the deployers as they could easily confiscate the funds of users if they wanted to, same as any `ERC20` distributor could confiscate the funds of users.

This is an important consideration and ideally it should be mentioned in user documentation or user interfaces. Luckily, not all bridges suffer this risk, just like not all ERC20s allow confiscation. In particular canonical tokens, and bridges using standardized tokens where the distributor is the token manager, are less susceptible to the deployer doing arbitrary confiscations or minting. This is something that could be surfaced in documentation or UI as well, so that users can know what trust assumptions they are working with when they bridge with each particular token and bridge combination.

In the interchain token service, Axelar governance is able to add trusted addresses that can perform arbitrary token transfers on the InterchainTokenService, so this should be noted as an important centralization risk that affects both canonical and custom bridges. (I have also raised a Medium severity issue related to this, where the deployer address retains this ability even after the ownership is transferred to a governance account).

### Systemic risks

As mentioned above, timing and gas are some things that are tricky to handle in any protocol that uses cross-chain messaging, and this additional complexity introduces some risk from the possibility of stalling/deadlocks and also simply from the fact that it’s a bigger attack sufrace.

In general, users of these services that are, for instance, extending a DAO or token with interchain governance or bridging, would be plugging their tokens or DAOs into a multiple-chain, multiple-consensus, multiple-VM super-system, so this should always be done with care. Such a system, going beyond a single chain’s consensus, can suddenly be exposed to unexpected issues like reorgs or consensus problems (e.g. forking) in a separate chain causing an impact in token supply in the token’s “native” chain.

Luckily, it seems the team has considered these risks and at first sight Axelar Network has mechanisms to mitigate them (as mentioned in section 5.2 of the [Axelar whitepaper](https://axelar.network/axelar_whitepaper.pdf)), but I’d need to do a deeper dive into the rest of the codebase to assert this with more confidence that all the potential risks stemming from this are covered.

### Other recommendations

- Comments in a lot of the contracts are sparse; spending some time adding NatSpec to all functions, storage variables and modifiers could really help with maintainability. Even just having a `@notice` on all external/public functions would be a big improvement. This is especially important for things like this: [https://github.com/code-423n4/2023-07-axelar/blob/9f642fe854eb11ad9f18fe028e5a8353c258e926/contracts/cgp/AxelarGateway.sol#L291-L292](https://github.com/code-423n4/2023-07-axelar/blob/9f642fe854eb11ad9f18fe028e5a8353c258e926/contracts/cgp/AxelarGateway.sol#L291-L292) - this audit note is important enough that including it in the NatSpec for the function would be useful. But in general, the meaning of all parameters and return values is valuable information. Similarly, it is not clear in TokenManagerDeployer and StandardizedTokenDeployer that these are meant to be used through delegatecalls - without some documentation, it looks like someone could try to deploy tokens or managers by calling these directly.
- I’ve noticed the project is not using any external dependencies (e.g. OpenZeppelin libraries) that could greatly simplify the codebase; the project could reuse token contracts or even rely on a third-party multisig implementation. I assume this is a conscious choice, and would love to hear more about the rationale for this, but it’s worth pointing out as a tradeoff where it’s unclear if the risk of bugs in third-party code outweighs the risks from lots of additional custom code.
- The use of `commandId := calldataload(4)` in some command processing functions suggests that it would be cleaner to pass the commandId to the internal `_execute` functions explicitly to avoid the need to use assembly. This assignment is a sort of “out-of-band” parameter-passing mechanism and could be prone to errors, as it obscures the fact that the params for `_execute` are not everything that the function needs from the caller. If there is a concern for backwards compatibility, a separate `_executeWithCommandId` or similar could be added to `AxelarExecutable`.
- Interchain calls add complexity to any codebase, and are hard to reason about, so I strongly suggest adding automated end-to-end tests if you don’t have them already. Ideally these tests would run on a testnet or local but real nodes for at least two different chains, and test running governance actions or token transfers between the two. I appreciate this is very time consuming but it helps mitigate a lot of the risks.

### Time spent:
13 hours