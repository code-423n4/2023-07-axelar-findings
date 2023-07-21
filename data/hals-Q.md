# Findings Summary

| ID              | Title                                                                              | Severity     |
| --------------- | ---------------------------------------------------------------------------------- | ------------ |
| [L-01]   | Lack of minimum signers & minimum threshold check                                  | Low          |
| [L-02] | Signers could lose their balance of tokens if the target function uses `tx.origin` | Low          |
| [L-03] | Executed interchain proposals can be re-executed again                             | Low          |
| [L-04]   | Interchain proposals can be executed out of order                                  | Low          |
| [NC-01] | `hasSignerVoted` function returns false if the proposal has been executed          | Non Critical |
| [NC-02] | `getSignerVotesCount` function returns zero if the proposal has been executed      | Non Critical |

# Low

## [L-01] Lack of minimum signers & minimum threshold check

## Vulnerability Details

- In the current `_rotateSigners` function design; it requires at least two signers to be set; so at least one vote is only required to execute a proposal or change signers (minimu threshold).
- For risk mitigation: it is suggested to set a minimum of **2 for the `threshold`** and a **minimum of 3 signers**.
- By doing this; two different signers votes will be needed before a proposal is executed; so in case a signer account is compromised; the other two signers will be able to mitigate his actions/change signers.
- Also; if a signer is down, the other two signers will be able to vote for the proposal and get it executed.

## Impact

Mitigate the risk of executing malicious proposals/changing signers if one of the two signers accounts is compromised.

## Proof of Concept

- [MultisigBase.sol/\_rotateSigners function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L157)

```solidity
File:contracts/cgp/auth/MultisigBase.sol
Line 157: length = newAccounts.length;
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

In `_rotateSigners` function: ensure that the minimum signers number is >= 3,and minimum threshold is >= 2.

## [L-02] Signers could lose their balance of tokens if the target function uses `tx.origin`

## Vulnerability Details

- In `AxelarServiceGovernance.sol`/`executeMultisigProposal` function & in `Multisig.sol`/`execute` function:  
  proposals are executed by signers via a low level `call` opcode.
- The target function on the receiver side can contain malicious code that uses `tx.origin`.
- This can harm the signer by providing the malicious targeted function approval to withdraw/deposit the signer's balance of tokens.

## Impact

Signers could lose their tokens balance.

## Proof of Concept

- [Caller.sol/\_call function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Caller.sol#L18)

```solidity
File:contracts/cgp/util/Caller.sol
Line 18: (bool success, ) = target.call{ value: nativeValue }(callData);
```

- [Multisig.sol/execute function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/Multisig.sol#L30-L36)

```solidity
File:contracts/cgp/governance/Multisig.sol
Line 30-36:
    function execute(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable onlySigners {
        _call(target, callData, nativeValue);
    }
```

- [AxelarServiceGovernance.sol/executeMultisigProposal function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L48-L62)

```solidity
File:contracts/cgp/governance/AxelarServiceGovernance.sol
Line 48-62:
    function executeMultisigProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable onlySigners {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));


        if (!multisigApprovals[proposalHash]) revert NotApproved();


        multisigApprovals[proposalHash] = false;


        _call(target, callData, nativeValue);


        emit MultisigExecuted(proposalHash, target, callData, nativeValue);
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Signers should be advised to use an untouched wallet address so that target code interaction can't harm them.

## [L-03] Executed interchain proposals can be re-executed again

## Vulnerability Details

In `InterchainProposalExecutor` contract/`_execute` function : there's no check if the proposal that's going to be executed by the relayers has been executed before or not.

## Impact

This will lead to funds drainage if the proposal has been executed before if the proposal has a value to send to the target.

## Proof of Concept

- [InterchainProposalExecutor.sol/\_execute function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L41-L67)

```solidity
File:contracts/interchain-governance-executor/InterchainProposalExecutor.sol
Line 41-67:
    function _execute(
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) internal override {
        _beforeProposalExecuted(sourceChain, sourceAddress, payload);


        // Check that the source address is whitelisted
        if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) {
            revert NotWhitelistedSourceAddress();
        }


        // Decode the payload
        (address interchainProposalCaller, InterchainCalls.Call[] memory calls) = abi.decode(payload, (address, InterchainCalls.Call[]));


        // Check that the caller is whitelisted
        if (!whitelistedCallers[sourceChain][interchainProposalCaller]) {
            revert NotWhitelistedCaller();
        }


        // Execute the proposal with the given arguments
        _executeProposal(calls);


        _onProposalExecuted(sourceChain, sourceAddress, interchainProposalCaller, payload);


        emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Add a mechanism to track the proposals; so that the relayer can check if the proposal has been executed before or not.

## [L-04] Interchain proposals can be executed out of order

## Vulnerability Details

Interchain proposals can be executed out of order; any relayer can call the `_execute` function to execute the proposals in any order they want.

## Impact

A malicious relayer can execute the proposals in a way that is beneficial to them.

## Proof of Concept

- [InterchainProposalExecutor.sol/\_execute function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L41-L67)

```solidity
File:contracts/interchain-governance-executor/InterchainProposalExecutor.sol
Line 41-67:
    function _execute(
        string calldata sourceChain,
        string calldata sourceAddress,
        bytes calldata payload
    ) internal override {
        _beforeProposalExecuted(sourceChain, sourceAddress, payload);


        // Check that the source address is whitelisted
        if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) {
            revert NotWhitelistedSourceAddress();
        }


        // Decode the payload
        (address interchainProposalCaller, InterchainCalls.Call[] memory calls) = abi.decode(payload, (address, InterchainCalls.Call[]));


        // Check that the caller is whitelisted
        if (!whitelistedCallers[sourceChain][interchainProposalCaller]) {
            revert NotWhitelistedCaller();
        }


        // Execute the proposal with the given arguments
        _executeProposal(calls);


        _onProposalExecuted(sourceChain, sourceAddress, interchainProposalCaller, payload);


        emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Check that proposals are always executed in order; otherwise, if the risk is deemed acceptable, update the documentation to highlight that.

# Non Critical

## [NC-01] `hasSignerVoted` function returns false if the proposal has been executed

## Vulnerability Details

If the proposal has been executed then it will return false for all signers; it's only valid if the proposal hasn't been executed yet.

## Proof of Concept

- [MultisigBase.sol/hasSignerVoted function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L111-L113)

```solidity
File:contracts/cgp/auth/MultisigBase.sol
Line 111-113:
    function hasSignerVoted(address account, bytes32 topic) external view override returns (bool) {
        return votingPerTopic[signerEpoch][topic].hasVoted[account];
    }
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Add another mechanism to save the votes of the signers even after the proposal execution.

## [NC-02] `getSignerVotesCount` function returns zero if the proposal has been executed

## Vulnerability Details

`getSignerVotesCount` function is by design for the current running proposals voting, so if the proposal has been executed then it will return zero.

## Proof of Concept

- [MultisigBase.sol/getSignerVotesCount function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L119-L129)

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

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Add another mechanism to save the number of votes of a proposal even after the proposal execution.
