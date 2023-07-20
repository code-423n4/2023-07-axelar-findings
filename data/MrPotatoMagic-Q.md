# QA Report

## [N-01] Modifier should be utilized for checks only and not make state changes

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L44

The onlySigners modifier in the link above should be made into a function since it makes changes to storage. These are the lines where state changes occur:
```solidity
File: contracts/cgp/auth/MultisigBase.sol
53: voting.hasVoted[msg.sender] = true;
61: voting.voteCount = voteCount;
66: voting.voteCount = 0;
71: voting.hasVoted[signers.accounts[i]] = false;
```

## [N-02] Typo error reduces code readability

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L559

implementations is written incorrectly as implementaions on Line 559 (as parameter) and Line 564.
```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
559: function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)
560:         internal
561:         pure
562:         returns (address implementation)
563:     {
564:         implementation = implementaions[uint256(tokenManagerType)];
565:         if (implementation == address(0)) revert ZeroAddress();
566:         if (ITokenManager(implementation).implementationType() != uint256(tokenManagerType)) revert InvalidTokenManagerImplementation();
567:     }
```

## [N-03] Missing event emission

There are 2 instances of this:

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L613

Event is emitted in the if block but not else block.
```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
609: if (expressCaller == address(0)) {
610:             amount = tokenManager.giveToken(destinationAddress, amount);
611:             emit TokenReceived(tokenId, sourceChain, destinationAddress, amount);
612:         } else {
613:             amount = tokenManager.giveToken(expressCaller, amount); //@audit missing event emission here
614:         }
```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L653

In this instance, an early return misses event emission on Line 653.
```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
652: if (expressCaller != address(0)) {
653:                 amount = tokenManager.giveToken(expressCaller, amount); 
654:                 return; //@audit early return misses event emission
655:           }
656:         }
657:       amount = tokenManager.giveToken(destinationAddress, amount);           
658:       IInterchainTokenExpressExecutable(destinationAddress).executeWithInterchainToken(sourceChain, sourceAddress, data, tokenId, amount);
659:       emit TokenReceivedWithData(tokenId, sourceChain, destinationAddress, amount, sourceAddress, data);
660:     }
```

## [N-04] Existing function should be used maintain pattern across _processCommand() functions

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L84

The _getProposalHash() function is used to get the proposal hash in the [_processCommand() function in the InterchainGovernance.sol](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L125) contract but not in the [_processCommand() function in the AxelarServiceCommand.sol contract](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L84) (which inherits from InterchainGovernance.sol). 

We can take a look at the difference in pattern below:
```solidity
File: contracts/cgp/governance/InterchainGovernance.sol
125: bytes32 proposalHash = _getProposalHash(target, callData, nativeValue);
```
```solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol
84: bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));
```

## [L-01] Missing EOA check for Distributor and Operator roles

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/Distributable.sol#L39

Missing EOA check for distributor
```solidity
File: contracts/its/utils/Distributable.sol
39: function _setDistributor(address distributor_) internal {
40:         assembly {
41:             sstore(DISTRIBUTOR_SLOT, distributor_)
42:         }
43:         emit DistributorChanged(distributor_);
44:     }
```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/Operatable.sol#L39

Missing EOA check for Operator:
```solidity
File: contracts/its/utils/Operatable.sol
39: function _setOperator(address operator_) internal {
40:         assembly {
41:             sstore(OPERATOR_SLOT, operator_)
42:         }
43:         emit OperatorChanged(operator_);
44:     }
```

Solution for both would be to check the extcodesize(distributor/operator) > 0. If this is true, we revert with an error mentioning that the distributor/operator is not an EOA.

## [L-02] msg.value >= gasValue check mentioned in comments is not implemented in the code

There are 3 instances of this issue occurring in the following 3 functions:

1. deployRemoteCanonicalToken - https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L322

2. deployRemoteCustomTokenManager - https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L360

3. deployAndRegisterRemoteStandardizedToken - https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L413
