# Missing address 0 check

Address 0 check should be added to the `target`

```
File: cgp/governance/AxelarServiceGovernance.sol

59:                _call(target, callData, nativeValue);
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L59

Similar to the check added here: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L100: `if (target == address(0)) revert InvalidTarget();`

---

# Reuse existing `_getProposalHash()` method to calculate proposal hash

The existing `_getProposalHash()` method should be used here, instead of rewriting this logic.

```
File: cgp/governance/InterchainGovernance.sol

73:    bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L73

---

# External call recipient may consume all transaction gas

There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. Use `addr.call{gas: <amount>}("")` or [this](https://github.com/nomad-xyz/ExcessivelySafeCall) library instead.

```
File: cgp/util/Caller.sol

18:    (bool success, ) = target.call{ value: nativeValue }(callData);
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L18

---

# Iteration is not required, as `voteCount` is already maintained on `Voting` struct

```
File: cgp/auth/MultisigBase.sol

119:    function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
120:        uint256 length = signers.accounts.length;
121:        uint256 voteCount;
122:        for (uint256 i; i < length; ++i) {
123:            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
124:                voteCount++;
125:            }
126:        }
127:
128:        return voteCount;
129:    }
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L119-L129

Fix: Update the method to be:

```
function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
    return votingPerTopic[signerEpoch][topic].voteCount;
}
```

---

# Unnecesary return statements

The return statements are redundant.

```
File: cgp/governance/InterchainGovernance.sol

131:    return;
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L131

```
File: cgp/governance/InterchainGovernance.sol

136:    return;
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L136

```
File: cgp/governance/AxelarServiceGovernance.sol

90:    return;
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L90

```
File: cgp/governance/AxelarServiceGovernance.sol

95:    return;
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L95

```
File: cgp/governance/AxelarServiceGovernance.sol

100:    return;
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L100

```
File: cgp/governance/AxelarServiceGovernance.sol

105:    return;
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L105

---
