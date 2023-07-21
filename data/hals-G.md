# Findings Summary

| ID            | Title                                                                                  | Severity         |
| ------------- | -------------------------------------------------------------------------------------- | ---------------- |
| [G-01] | Using `for` loop to get `voteCount` instead of using the Voting struct `voteCount` key | Gas Optimization |

# Gas Optimizations

## [G-01] Using `for` loop to get `voteCount` instead of using the Voting struct `voteCount` key 

## Details

In `hasSignerVoted` function: the number of votes on a specific proposal is counted by looping over the `signers.accounts` array and counting the number of votes based on whether the signer has voted or not; while this can be avoided by directly getting the number of votes from `votingPerTopic[signerEpoch][topic].votingCount`

## Proof of Concept

Instances: 1

- [MultisigBase.sol/hasSignerVoted function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L119-L129)

```solidity
File:contracts/cgp/auth/MultisigBase.sol
Line 119-129:
     function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
        uint256 length = signers.accounts.length;
        uint256 voteCount;
        for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }

        return voteCount;
    }
```
