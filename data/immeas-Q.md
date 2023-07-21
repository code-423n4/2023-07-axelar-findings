# QA Report

## Summary

### Low

| id | title |
| --- | --- |
| [L-01](#l-01-finalproxy-can-be-hijacked-by-vulnerable-implementation-contract) | `FinalProxy` can be hijacked by vulnerable implementation contract |
| [L-02](#l-02-a-users-tokens-can-be-stolen-if-they-dont-control-msgsender-address-on-all-chains) | A users tokens can be stolen if they don't control `msg.sender` address on all chains |
| [L-03](#l-03-no-event-emitted-when-a-vote-is-cast-in-multisigbase) | no event emitted when a vote is cast in `MultisigBase` |
| [L-04](#l-04-interchaintokenservice-non-remote-deploy-calls-accept-eth-but-are-not-using-it) | `InterchainTokenService` non-remote deploy calls accept `eth` but are not using it |
| [L-05](#l-05-default-values-for-deployandregisterstandardizedtoken-can-make-it-complicated-for-third-party-implementers)| default values for `deployAndRegisterStandardizedToken` can make it complicated for third party implementers |
| [L-06](#l-06-interchaintokenserviceproxy-is-finalproxy) | `InterchainTokenServiceProxy` is `FinalProxy` |
| [L-07](#l-07-low-level-call-will-always-succeed-for-non-existent-addresses) | low level `call` will always succeed for non existent addresses |
| [L-08](#l-08-interchaintokenservicegetimplementation-returns-address0-for-invalid-types) | `InterchainTokenService::getImplementation` returns `address(0)` for invalid types |
| [L-09](#l-09-consider-a-two-way-transfer-of-governance) | consider a two way transfer of governance |
| [L-10](#l-10-consider-a-two-way-transfer-of-operator-and-distributor) | consider a two way transfer of `operator` and `distributor` |

### Suggestions

| id | title |
| --- | --- |
| [S-01](#s-01-consider-adding-a-version-to-upgradable-contracts) | consider adding a version to upgradable contracts |

### Refactor

| id | title |
| --- | --- |
| [R-01](#r-01-sourceaddress-means-two-different-things-in-interchaintokenservice`) | `sourceAddress` means two different things in `InterchainTokenService` | 
| [R-02](#r-02-interchaintokenservice_validatetoken-could-have-a-more-descriptive-name) | `InterchainTokenService::_validateToken` could have a more descriptive name |
| [R-03](#r-03-axelargatewayonlymintlimiter-could-have-a-more-descriptive-name) | `AxelarGateway::onlyMintLimiter` could have a more descriptive name |

### Non critical/Informational

| id | title |
| --- | --- |
| [NC-01](#nc-01-remoteaddressvalidator_lowercase-will-not-work-for-solana-addresses) | `RemoteAddressValidator::_lowerCase` will not work for Solana addresses |
| [NC-02](#nc-02-interchaingovernance-can-be-abstract) | `InterchainGovernance` can be `abstract` |
| [NC-03](#nc-03-eta-in-proposalcancelled-event-can-be-missleading) | `eta` in `ProposalCancelled` event can be missleading |
| [NC-04](#nc-04-incomplete-natspec) | incomplete natspec |
| [NC-05](#nc-05-erroneous-comments) | erroneous comments |
| [NC-06](#nc-06-typos-and-misspellings) | typos and misspellings |

## Low

### L-01 `FinalProxy` can be hijacked by vulnerable implementation contract

`FinalProxy` is a proxy that can be upgraded but if `owner` calls `finalUpgrade` it will [deploy the final upgrade using `Create3`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L79-L83) and it can no longer be upgraded.

To determine if it has gotten the final upgrade or not the function `_finalImplementation` is done:

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L18

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L57-L64
```solidity
File: gmp-sdk/upgradable/FinalProxy.sol

18:    bytes32 internal constant FINAL_IMPLEMENTATION_SALT = keccak256('final-implementation');

...

57:    function _finalImplementation() internal view virtual returns (address implementation_) {
58:        /**
59:         * @dev Computing the address is cheaper than using storage
60:         */
61:        implementation_ = Create3.deployedAddress(address(this), FINAL_IMPLEMENTATION_SALT);
62:
63:        if (implementation_.code.length == 0) implementation_ = address(0);
64:    }
```

The issue is that if the implementation supports deploying with `Create3` a user could use the salt (`keccak256('final-implementation')`) and deploy a final implementation without calling `finalImplementation`. Since `Create3` only uses `msg.sender` and `salt` to determine the address.

`InterchainTokenService` which is the only implementation in scope using `FinalProxy` does however not appear to be vulnerable to this, since all the salts used for deploys are calculated, not supplied. But future implementations/other contracts using `FinalProxy` might be.

#### PoC
Test in `proxy.js`:
```javascript
        it('vulnerable FinalProxy implementation', async () => {
            const vulnerableDeployerFactory = await ethers.getContractFactory('VulnerableDeployer', owner);
            const maliciousContractFactory = await ethers.getContractFactory('MaliciousContract', owner);
            const vulnerableImpl = await vulnerableDeployerFactory.deploy().then((d) => d.deployed());

            const proxy = await finalProxyFactory.deploy(vulnerableImpl.address, owner.address, '0x').then((d) => d.deployed());
            expect(await proxy.isFinal()).to.be.false;

            const vulnerable = new Contract(await proxy.address, VulnerableDeployer.abi, owner);

            // steal final-implementation salt
            const salt = keccak256(toUtf8Bytes('final-implementation'));

            // someone deploys to the final-implementation address
            const bytecode = await maliciousContractFactory.getDeployTransaction().data;
            await vulnerable.vulnerableDeploy(salt,bytecode);

            // proxy is final without calling finalUpgrade
            expect(await proxy.isFinal()).to.be.true;
            const malicious = new Contract(await proxy.address, MaliciousContract.abi, owner);
            expect(await malicious.maliciousCode()).to.equal(4);
        });
```

`VulnerableDeployer.sol`:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import { Create3 } from '../deploy/Create3.sol';

contract MaliciousContract {

    function setup(bytes calldata params) external {}
    
    function maliciousCode() public pure returns(uint256) {
        return 4;
    }
}

contract VulnerableDeployer {

    function setup(bytes calldata params) external {}
    
    function vulnerableDeploy(bytes32 salt, bytes memory bytecode) public returns(address ) {
        return Create3.deploy(salt, bytecode); 
    }
}
```

#### Recommendation
Have the user deploy the final implementation and then upgrade to it without using Create3. That way a vulnerable implementation contract cannot be abused and taken over.

### L-02 A users tokens can be stolen if they don't control `msg.sender` address on all chains

When a user wants to register a token for use across chains they first call [`InterchainTokenService::deployAndRegisterStandardizedToken`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L388-L402) on the "local" chain. This will use a user provided salt together with the `msg.sender` to [create the `tokenId`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L396) which is used as the salt to create both the [`StandardizedToken`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L397) and the [`TokenManager`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L401).

They can then use this to deploy their token to any chain that Axelar supports.

Relying on `msg.sender` across chain comes with some security considerations though. If the user/protocol don't control the address used as `msg.sender` across all chains that are supported by Axelar ITS to they are susceptible to the same hack that affected [wintermute](https://rekt.news/wintermute-rekt/). Where an old gnosis wallet was used which had an address that could be stolen on another chain.

If an attacker controls the `msg.sender` address on another chain they can simply create a token and manager with the same salt that they control. This will give them the same `tokenId`. They can then send a message to the chain where the real token is and get funded real tokens. All they've done is burn/lock their fake token on their `sourceChain`. 

#### Recommendation
I recommend this is highlighted as a risk in the documentation so third party protocols building on top of Axelar are aware of this risk. 

### L-03 no event emitted when a vote is cast in `MultisigBase`

To vote in `MultisigBase` a signer submits the same data as the proposal. Then this data is hashed into a [proposal id (`topic`)](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L47-L48) which has its votes tracked, when enough votes are cast the proposal passes:

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L51-L63
```solidity
File: cgp/auth/MultisigBase.sol

51:        if (voting.hasVoted[msg.sender]) revert AlreadyVoted();
52:
53:        voting.hasVoted[msg.sender] = true;
54:
55:        // Determine the new vote count.
56:        uint256 voteCount = voting.voteCount + 1;
57:

           // @audit no event emitted to track votes

58:        // Do not proceed with operation execution if insufficient votes.
59:        if (voteCount < signers.threshold) {
60:            // Save updated vote count.
61:            voting.voteCount = voteCount;
62:            return;
63:        }
```

However there is no event emitted for when a vote is cast. This makes it difficult to track voting off chain. Which is important for transparency and for users and signers to know what `topics` are going on. `topics` can also only be tracked by their hashed value, hence emitting this will help users to query on-chain for specific votes.

#### Recommendation
Add an event for when a vote is cast, containing `signer`, `topic` and `voteCount`.


### L-04 `InterchainTokenService` non-remote deploy calls accept `eth` but are not using it

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L309

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L347

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L395

In `InterchainTokenService` the calls to deploy on the local chain are `payable` but do not use the `eth` provided. `deployCustomTokenManager` as an example:

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L343-L352
```solidity
File: its/interchain-token-service/InterchainTokenService.sol

343:    function deployCustomTokenManager(
344:        bytes32 salt,
345:        TokenManagerType tokenManagerType,
346:        bytes memory params
347:    ) public payable notPaused returns (bytes32 tokenId) { // <-- payable
348:        address deployer_ = msg.sender;
349:        tokenId = getCustomTokenId(deployer_, salt);
350:        _deployTokenManager(tokenId, tokenManagerType, params);
351:        emit CustomTokenIdClaimed(tokenId, deployer_, salt);
352:    }
```

However, none of the deploy functions use any `eth` though. A user could accidentally send `eth` here that would be stuck in the contract.

#### Recommendation
Consider removing `payable` from `registerCanonicalToken` `deployCustomTokenManager` and `deployAndRegisterStandardizedToken`. `payable` could also be removed from `TokenManagerDeployer::deployTokenManager` and `StandardizedTokenDeployer::deployStandardizedToken` as they also do not consume any `eth`.

### L-05 default values for `deployAndRegisterStandardizedToken` can make it complicated for third party implementers

When calling `InterchainTokenService` to deploy a `StandardizedToken` a user supplies [some parameters for the setup](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L388-L395).

For `mintTo` for a possible initial mint when setting up the token and `operator` for the `TokenManager` however `msg.sender` is used:

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L395-L402

```solidity
File: its/interchain-token-service/InterchainTokenService.sol

395:    ) external payable notPaused {
396:        bytes32 tokenId = getCustomTokenId(msg.sender, salt);
            //                                             here msg.sender is used for mintTo ↓↓↓↓↓↓↓↓↓↓↓
397:        _deployStandardizedToken(tokenId, distributor, name, symbol, decimals, mintAmount, msg.sender);
398:        address tokenManagerAddress = getTokenManagerAddress(tokenId);
399:        TokenManagerType tokenManagerType = distributor == tokenManagerAddress ? TokenManagerType.MINT_BURN : TokenManagerType.LOCK_UNLOCK;
400:        address tokenAddress = getStandardizedTokenAddress(tokenId);
            //                 here msg.sender is used for `operator` ↓↓↓↓↓↓↓↓↓↓
401:        _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));
402:    }
```

This can make it more complicated for third parties to develop on top of the `InterchainTokenService` as they have to keep in mind that the contract calling will be `operator` and possibly the receiver of any initial mint.

#### Recommendation
Consider adding `mintTo` and `operator` as parameters that can be passed to the call

### L-06 `InterchainTokenServiceProxy` is `FinalProxy`

`InterchainTokenServiceProxy` inherits from `FinalProxy`, this makes it possible for `owner` to accidentally upgrade `InterchainTokenService` to an un-upgradable version.

#### PoC
Test in `tokenService.js`:
```javascript
    describe('Final Upgrade Interchain Token Serivce', () => {
        it('should upgrade to new token service', async () => {
            const factory = await ethers.getContractFactory("UpgradedITS", wallet);
            const bytecode = await factory.getDeployTransaction().data;
            const finalProxy = new Contract(await service.address, FinalProxy.abi, wallet);
            await finalProxy.finalUpgrade(bytecode,'0x');

            expect(await finalProxy.isFinal()).to.equal(true);

            // contract is final
            const upgradedITS = new Contract(await service.address, UpgradedITS.abi, wallet);
            expect(await upgradedITS.foo()).to.equal(4);
        });
    });
```

`UpgradedITS.sol`:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import { Upgradable } from '../../gmp-sdk/upgradable/Upgradable.sol';

contract UpgradedITS is Upgradable {

    bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

    function contractId() external pure returns (bytes32) {
        return CONTRACT_ID;
    }

    function _setup(bytes calldata ) internal override {}

    function foo() public pure returns(uint256) {
        return 4;
    }
}
```

#### Recommendation
Consider inheriting from `Proxy` instead of `FinalProxy`.

### L-07 low level `call` will always succeed for non existent addresses

https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions:
> The low-level functions call, delegatecall and staticcall return true as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed.

Calls are done in these instances:

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L18
```solidity
File: cgp/util/Caller.sol

18:        (bool success, ) = target.call{ value: nativeValue }(callData);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76
```solidity
File: interchain-governance-executor/InterchainProposalExecutor.sol
76:            (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);
```

This is mentioned in the Automated findings report but the instances identified are wrong:
https://gist.github.com/thebrittfactor/c400e0012d0092316699c53843ecad41#low-23-contract-existence-is-not-checked-before-low-level-call

#### Recommendation
Where applicable, consider adding a check if there is code on the target.

### L-08 `InterchainTokenService::getImplementation` returns `address(0)` for invalid types

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L223-L233
```solidity
File: its/interchain-token-service/InterchainTokenService.sol

223:    function getImplementation(uint256 tokenManagerType) external view returns (address tokenManagerAddress) {
224:        // There could be a way to rewrite the following using assembly switch statements, which would be more gas efficient,
225:        // but accessing immutable variables and/or enum values seems to be tricky, and would reduce code readability.
226:        if (TokenManagerType(tokenManagerType) == TokenManagerType.LOCK_UNLOCK) {
227:            return implementationLockUnlock;
228:        } else if (TokenManagerType(tokenManagerType) == TokenManagerType.MINT_BURN) {
229:            return implementationMintBurn;
230:        } else if (TokenManagerType(tokenManagerType) == TokenManagerType.LIQUIDITY_POOL) {
231:            return implementationLiquidityPool;
232:        }
233:    }
```

Other integrations can rely on this and returning `address(0)` for the implementation contract can break their integrations.

#### Recommendation
Consider reverting with `Invalid TokenManagerType` or similar.

### L-09 consider a two way transfer of `governance`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L254-L258
```solidity
File: cgp/AxelarGateway.sol

254:    function transferGovernance(address newGovernance) external override onlyGovernance {
255:        if (newGovernance == address(0)) revert InvalidGovernance();
256:
257:        _transferGovernance(newGovernance);
257:    }
```

The `governance` address has complete control over the `AxelarGatway` since it can do upgrades. 

#### Recommendation
Consider implementing a two way (propose/accept) change procedure for it to avoid accidentally handing it over to the wrong address.

### L-10 consider a two way transfer of `operator` and `distributor`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol#L51-L53

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Operatable.sol#L51-L53

```solidity
File: its/utils/Operatable.sol

51:    function setOperator(address operator_) external onlyOperator {
52:        _setOperator(operator_);
53:    }
```
(the exact same code is in `Distributable` as well)

As these are powerful roles in for tokens/token managers.

#### Recommendation
Consider implementing a two way (propose/accept) change procedure for it to avoid accidentally handing it over to the wrong address.

## Suggestions

### S-01 Consider adding a version to upgradable contracts
That way a user can query a contract and see if it is upgraded or not.

## Refactor

### R-01 `sourceAddress` means two different things in `InterchainTokenService`

In the function `_execute` which is the entrypoint from `AxelarExecutor`, `sourceAddress` means the source for the cross chain call, i.e the `sourceChain` address of that `InterchainTokenSerivce` contract:

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L575-L579
```solidity
File: its/interchain-token-service/InterchainTokenService.sol

575:    function _execute(
576:        string calldata sourceChain,
577:        string calldata sourceAddress, // <-- `sourceAddress` is which contract initiated the cross chain instruction
578:        bytes calldata payload
579:    ) internal override onlyRemoteService(sourceChain, sourceAddress) notPaused {
```

Elsewhere in the code, in [`_processSendTokenWithDataPayload`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L622-L640), [`transmitSendToken`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L502-L509), [`expressReceiveTokenWithData`](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L467-L475) the meaning of `sourceAddress` has changed to which address the transferred tokens originate from:

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L622-L640
```solidity
File: its/interchain-token-service/InterchainTokenService.sol

622:    function _processSendTokenWithDataPayload(string calldata sourceChain, bytes calldata payload) internal {
623:        bytes32 tokenId;
624:        uint256 amount;
625:        bytes memory sourceAddress; // <-- `sourceAddress` is from which address the tokens originate

...

633:        {
634:            bytes memory destinationAddressBytes;
635:            (, tokenId, destinationAddressBytes, amount, sourceAddress, data) = abi.decode(
636:                payload,
637:                (uint256, bytes32, bytes, uint256, bytes, bytes)
638:            );
639:            destinationAddress = destinationAddressBytes.toAddress();
640:        }
```

This can cause confusion and as `sourceAddress` in the context of `_execute` is a critical for security checks to trust the call it could be risky if these are confused for eachother.

#### Recommendation
Consider changing the token transfer source to `tokenSenderAddress` for it to be clear what it means.

### R-02 `InterchainTokenService::_validateToken` could have a more descriptive name

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L726-L738
```solidity
File: its/interchain-token-service/InterchainTokenService.sol

726:    function _validateToken(address tokenAddress)
727:        internal
728:        returns (
...
732:        )
733:    {
734:        IERC20Named token = IERC20Named(tokenAddress);
735:        name = token.name();
736:        symbol = token.symbol();
737:        decimals = token.decimals();
738:    }
```

`_validateToken` doesn't do any actual validation. Could be renamed to `_getTokenData` or similar.

### R-03 `AxelarGateway::onlyMintLimiter` could have a more descriptive name

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L87-L88
```solidity
File: cgp/AxelarGateway.sol

87:    modifier onlyMintLimiter() {
88:        if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();
```

`onlyMintLimiter` could be named `onlyMintLimiterOrGov` since this is what it verifies.

## Non critical/Informational

### NC-01 `RemoteAddressValidator::_lowerCase` will not work for Solana addresses

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L54-L61
```solidity
File: its/remote-address-validator/RemoteAddressValidator.sol

54:    function _lowerCase(string memory s) internal pure returns (string memory) {
55:        uint256 length = bytes(s).length;
56:        for (uint256 i; i < length; i++) {
57:            uint8 b = uint8(bytes(s)[i]);
58:            if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));
59:        }
60:        return s;
61:    }
```

This converts an address to lowercase (65-70 -> A-F). Solana addresses however are base58 encoded versions of a 32 byte array. Thus they first have more letters and converting it to lowercase will make it another address.

#### Recommendation
Consider changing this before adding solana as a supported chain

### NC-02 `InterchainGovernance` can be `abstract`

As the contract is not complete on its own I recommend making it `abstract` to convey that it is supposed to be extended.

### NC-03 `eta` in `ProposalCancelled` event can be misleading

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L135

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L94
```solidity
File: cgp/governance/AxelarServiceGovernance.sol

94:            emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);
```

The `eta` here can be misleading as it might not be the `eta` of the event. The user can send what they want here. And if it is the correct eta that the user used to schedule it can still have been changed when scheduling the Timelock.

#### Recommendation
Consider either reading the `eta` before clearing it in `_cancelTimeLock` or don't include the `eta` in the event at all.

### NC-04 incomplete natspec

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L411-L424
```solidity
File: its/interchain-token-service/InterchainTokenService.sol

411:     * @param distributor the address that will be able to mint and burn the deployed token.
412:     * @param destinationChain the name of the destination chain to deploy to.
413:     * @param gasValue the amount of native tokens to be used to pay for gas for the remote deployment. At least the amount
414:     * specified needs to be passed to the call
415:     * @dev `gasValue` exists because this function can be part of a multicall involving multiple functions that could make remote contract calls.
416:     */
417:    function deployAndRegisterRemoteStandardizedToken(
418:        bytes32 salt,
419:        string calldata name,
420:        string calldata symbol,
421:        uint8 decimals,
422:        bytes memory distributor,
423:        bytes memory operator, // <-- operator missing from NatSpec
424:        string calldata destinationChain,
```

`@param operator` is missing from NatSpec

### NC-05 erroneous comments

#### `TokenManagerProxy.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/TokenManagerProxy.sol#L8-L13
```solidity
File: its/proxies/TokenManagerProxy.sol

 8:/**
 9: * @title TokenManagerProxy
10: * @dev This contract is a proxy for token manager contracts. It implements ITokenManagerProxy and
11: * inherits from FixedProxy from the gmp sdk repo
12: */
13:contract TokenManagerProxy is ITokenManagerProxy {
```

It does not inherit from `FixedProxy`.

#### `InterchainProposalExecutor.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L19-L22
```solidity
File: interchain-governance-executor/InterchainProposalExecutor.sol

19: *
20: * This contract is abstract and some of its functions need to be implemented in a derived contract.
21: */
22:contract InterchainProposalExecutor is IInterchainProposalExecutor, AxelarExecutable, Ownable {
```

The contract is not abstract at all.
 
#### `AxelarGateway.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L40-L41
```solidity
File: cgp/AxelarGateway.sol

40:    /// @dev Storage slot with the address of the current governance. `keccak256('mint-limiter') - 1`.
41:    bytes32 internal constant KEY_MINT_LIMITER = bytes32(0x627f0c11732837b3240a2de89c0b6343512886dd50978b99c76a68c6416a4d92);
```

Should say `address of the current mint limiter` not `governance`

#### `RemoteAddressValidator.sol`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L24-L27

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L40-L41
```solidity
File: its/remote-address-validator/RemoteAddressValidator.sol

24:     * @dev Constructs the RemoteAddressValidator contract, both array parameters must be equal in length
...
27:    constructor(address _interchainTokenServiceAddress) {

...

40:    function _setup(bytes calldata params) internal override {
41:        (string[] memory trustedChainNames, string[] memory trustedAddresses) = abi.decode(params, (string[], string[]));
```

`both array parameters must be equal in length` should be for the `_setup` call not the `constructor`

### NC-06 typos and misspellings

#### `InterchainTokenService.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L380

```solidity
File: its/interchain-token-service/InterchainTokenService.sol

380:     * can be calculated ahead of time) then a mint/burn TokenManager is used. Otherwise a lock/unlcok TokenManager is used.
                                                                                                 -> unlock
```

Also apprears at:
- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L406

#### `InterchainToken.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L28

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L40

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L76
```solidity
File: its/interchain-token/InterchainToken.sol

28:     * TokenManager specifically to do it permissionlesly.
                                          -> permissionlessly

40:     * @param amount The amount of token to be transfered.
                                               -> transferred

76:     * @param amount The amount of token to be transfered.
                                               -> transferred
```

#### `InterchainTokenService.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L159

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L496

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L500

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L559
```solidity
File: its/interchain-token-service/InterchainTokenService.sol

159:     * @return tokenManagerAddress deployement address of the TokenManager.
                                    -> deployment

496:     * @param sourceAddress the address where the token is coming from, which will also be used for reimburment of gas.
                                                                                                     -> reimbursement

500:     * @param metadata the data to be passed to the destiantion.
                                                     -> destination

559:    function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)
                                                                   -> implementations
```

#### `TokenManager.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L78

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L103

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L159
```solidity
File: its/token-manager/TokenManager.sol

 78:     * @notice Calls the service to initiate the a cross-chain transfer after taking the appropriate amount of tokens from the user.
                                              -> a cross-chain ...

103:     * @notice Calls the service to initiate the a cross-chain transfer with data after taking the appropriate amount of tokens from the user.
                                              -> a cross-chain ...

159:     * @return the amount of token actually given, which will onle be differen than `amount` in cases where the token takes some on-transfer fee.
                                                               -> only -> different
```

#### `TokenManagerProxy.sol`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/TokenManagerProxy.sol#L73
```solidity
File: its/proxies/TokenManagerProxy.sol

73:        address implementaion_ = implementation();
                -> implementation
```

#### `IInterchainTokenExecutable.sol`, `IInterchainTokenExpressExecutable.sol`


https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenExecutable.sol#L16

```solidity
File: its/interfaces/IInterchainTokenExecutable.sol

16:     * @param tokenId the tokenId of the token manager managing the token. You can access it's address by querrying the service
                                                                                                          -> querying
```
Same in https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenExpressExecutable.sol#L18