# Axelar QA report
## **Table of Contents**


| Identifier | Issue                                                                                          |
| ---------- | ---------------------------------------------------------------------------------------------- |
| QA-01 | The check `if (v != 27 && v != 28)` is not actually needed |
| QA-02 | A concept to include ETA's for cancelling proposals too should exist |
| QA-03 | Modifiers should only be used for checks |
| QA-04 | Current implementation of `InterchainCall` needs to include explicit documentations to let users know to provide enough amount of gas could lead to proposals being stuck due to underpayment of gas |
| QA-05 | Avoid using abi.encodePacked() with dynamic types when passing the result to a hash function |
| QA-06 | Important governance addresses should always include the `0x0` check |
| QA-07 | The `SELFDESTRUCT` opcode post EIP-6780 behaves a bit different |
| QA-08 | Dependency on `block.timestamp` for TimeLock is somewhat risky |
| QA-09 | There should be a clear fund retrieval method in all contracts with the payable `receive()` fallback function |
| QA-10 | `deployAndRegisterRemoteStandardizedToken()` and sister functions could be made more effective |
| QA-11 | `Timelock.sol` should include a few more sanity checks |
| QA-12 | Typos that should be fixed |
| QA-13 | Pragma version should not be floating |

## QA-01 - The check `if (v != 27 && v != 28)` is not actually needed

### Vulnerability Details

grep the "@audit" tag in code block & see [_this_](https://twitter.com/alexberegszaszi/status/1534461421454606336?s=20&t=H0Dv3ZT2bicx00hLWJk7Fg) about why this is no longer needed.

```solidity
    /**
     * @dev Returns the address that signed a hashed message (`hash`) with
     * `signature`. This address can then be used for verification purposes.
     *
     * The `ecrecover` EVM opcode allows for malleable (non-unique) signatures:
     * this function rejects them by requiring the `s` value to be in the lower
     * half order, and the `v` value to be either 27 or 28.
     *
     * IMPORTANT: `hash` _must_ be the result of a hash operation for the
     * verification to be secure: it is possible to craft signatures that
     * recover to arbitrary addresses for non-hashed data. A safe way to ensure
     * this is by receiving a hash of the original message (which may otherwise
     * be too long), and then calling {toEthSignedMessageHash} on it.
     */
    function recover(bytes32 hash, bytes memory signature) internal pure returns (address signer) {
        // Check the signature length
        if (signature.length != 65) revert InvalidSignatureLength();

        // Divide the signature in r, s and v variables
        bytes32 r;
        bytes32 s;
        uint8 v;

        // ecrecover takes the signature parameters, and the only way to get them
        // currently is to use assembly.
        // solhint-disable-next-line no-inline-assembly
        assembly {
            r := mload(add(signature, 0x20))
            s := mload(add(signature, 0x40))
            v := byte(0, mload(add(signature, 0x60)))
        }

        // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
        // unique. Appendix F in the Ethereum Yellow paper (https://ethereum.github.io/yellowpaper/paper.pdf), defines
        // the valid range for s in (281): 0 < s < secp256k1n ÷ 2 + 1, and for v in (282): v ∈ {27, 28}. Most
        // signatures from current libraries generate a unique signature with an s-value in the lower half order.
        //
        // If your library generates malleable signatures, such as s-values in the upper range, calculate a new s-value
        // with 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 - s1 and flip v from 27 to 28 or
        // vice versa. If your library also generates signatures with 0/1 for v instead 27/28, add 27 to v to accept
        // these malleable signatures as well.
        if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) revert InvalidS();
      //@audit QA there is no need for this check, see
        if (v != 27 && v != 28) revert InvalidV();

        // If the signature is valid (and not malleable), return the signer address
        if ((signer = ecrecover(hash, v, r, s)) == address(0)) revert InvalidSignature();
    }
```

### Recommendation

Remove the check
## QA-02 - A concept to include ETA's for cancelling proposals too should exist

### Vulnerability Details

The [`InterchainGovernance.sol `](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol) contract provides functionality for creating, executing, and cancelling proposals for cross-chain governance actions. This issue is a suggestion related to the cancellation of these proposals.

In the current implementation of the Interchain Governance contract, only scheduled proposals have an estimated time of arrival (ETA), which is the timestamp after which the proposal can be executed.

Take a look at the internal _internal \_processCommand() function_ , seen [here](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L113)

```solidity
    /**
     * @notice Internal function to process a governance command
     * @param commandId The id of the command, 0 for proposal creation and 1 for proposal cancellation
     * @param target The target address the proposal will call
     * @param callData The data the encodes the function and arguments to call on the target contract
     * @param nativeValue The nativeValue of native token to be sent to the target contract
     * @param eta The time after which the proposal can be executed
     */
    function _processCommand(
        uint256 commandId,
        address target,
        bytes memory callData,
        uint256 nativeValue,
        uint256 eta
    ) internal virtual {
        if (commandId > uint256(type(GovernanceCommand).max)) {
            revert InvalidCommand();
        }

        GovernanceCommand command = GovernanceCommand(commandId);
        bytes32 proposalHash = _getProposalHash(target, callData, nativeValue);

        if (command == GovernanceCommand.ScheduleTimeLockProposal) {
            eta = _scheduleTimeLock(proposalHash, eta);

            emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);
            return;
            //@audit just more of a QA, but a concept to include eta's for cancelling proposals too should exist, so as this
        } else if (command == GovernanceCommand.CancelTimeLockProposal) {
            _cancelTimeLock(proposalHash);

            emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);
            return;
        }
    }
```

However, there is no similar mechanism for cancelled proposals. I would advise that including an ETA for cancelled proposals could increase transparency in the governance process.

### Recommendation

The recommended solution is to consider extending the `ProposalCancelled` event to include an ETA. The ETA would represent the timestamp at which the proposal was cancelled. This would likely involve only a minor code change, but might require additional off-chain tooling or indexing to be effective in improving transparency.
## QA-03 - Modifiers should only be used for checks

### Vulnerability Details

From multisig.sol, the [modifier below](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L44-L63) exists

```solidity
    /**
     * @notice Modifier to ensure the caller is a signer
     * @dev Keeps track of votes for each operation and resets the vote count if the operation is executed.
     * @dev Given the early void return, this modifier should be used with care on functions that return data.
     @audit
     */
    modifier onlySigners() {
        if (!signers.isSigner[msg.sender]) revert NotSigner();

        bytes32 topic = keccak256(msg.data);
        Voting storage voting = votingPerTopic[signerEpoch][topic];

        // Check that signer has not voted, then record that they have voted.
        if (voting.hasVoted[msg.sender]) revert AlreadyVoted();

        voting.hasVoted[msg.sender] = true;

        // Determine the new vote count.
        uint256 voteCount = voting.voteCount + 1;

        // Do not proceed with operation execution if insufficient votes.
        if (voteCount < signers.threshold) {
            // Save updated vote count.
            voting.voteCount = voteCount;
            return;
        }

        // Clear vote count and voted booleans.
        voting.voteCount = 0;

        uint256 count = signers.accounts.length;

        for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

        emit MultisigOperationExecuted(topic);

        _;
    }
```

See [this](https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/), but in short it's known that the code inside a modifier is usually executed before the function body, so any state changes or external calls will easily violate the CEI pattern.

### Recommendation

Try to use modifiers only for checks


## QA-04 - Current implementation of `InterchainCall` needs to include explicit documentations to let users know to provide enough amount of gas could lead to proposals being stuck due to underpayment of gas

### Vulnerability Details

The execution of [sendProposals()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L59) could fail if the gas provided is insufficient.

The `InterchainCall` struct is used to specify interchain calls that will be executed at the destination chain. It includes a `gas` field, which is the amount of native token transferred to the target contract as gas payment for the interchain call.

```solidity
struct InterchainCall {
    string destinationChain;
    string destinationContract;
    uint256 gas;
    Call[] calls;
}
```

And a `Call` struct is used to specify the details of a call to be executed at the destination chain.

```solidity
struct Call {
    address target;
    uint256 value;
    bytes callData;
}
```

Do note that this is being used extensively in the `InterchainProposalSender.sol `contract, an example is the [sendProposals()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L59) function, increasing the possibility of this happening.

```solidity
    /**
     * @dev Broadcast the proposal to be executed at multiple destination chains
     * @param interchainCalls An array of `InterchainCalls.InterchainCall` to be executed at the destination chains. Where each `InterchainCalls.InterchainCall` contains the following:
     * - destinationChain: destination chain
     * - destinationContract: destination contract
     * - gas: gas to be paid for the interchain transaction
     * - calls: An array of `InterchainCalls.Call` to be executed at the destination chain. Where each `InterchainCalls.Call` contains the following:
     *   - target: target contract
     *   - value: amount of tokens to send
     *   - callData: encoded function arguments
     * Note that the destination chain must be unique in the destinationChains array.
     */
    function sendProposals(InterchainCalls.InterchainCall[] calldata interchainCalls) external payable override {
        // revert if the sum of given fees are not equal to the msg.value
        revertIfInvalidFee(interchainCalls);

        for (uint256 i = 0; i < interchainCalls.length; ) {
            _sendProposal(interchainCalls[i]);
            unchecked {
                ++i;
            }
        }
    }
```

### Recommendation

Ensure that users are made aware through documentation or UI prompts that they should provide adequate amounts of gas to avoid potential issues with token transfers.


## QA-05 - Avoid using abi.encodePacked() with dynamic types when passing the result to a hash function

### Vulnerability Details

Multiple instances but take a look at [TimeLock.sol#L89-L98]](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/TimeLock.sol#L89-L98)

```solidity
    /**
     * @dev Returns the timestamp after which the timelock with the given hash can be executed.
     */
    function _getTimeLockEta(bytes32 hash) internal view returns (uint256 eta) {
        bytes32 key = keccak256(abi.encodePacked(PREFIX_TIME_LOCK, hash));


        assembly {
            eta := sload(key)
        }
    }
```

For this instance and all other instances in scope it's key to note that using `abi.encodePacked()` with dynamic types when passing the result to a hash function can lead to potential hash collisions. The `abi.encode()` function pads items to 32 bytes, ensuring better collision prevention. It is important to note that if there is only one parameter, it is possible to cast it to `bytes()` or `bytes32()` instead of using `abi.encodePacked()`. When dealing with multiple parameters that are strings or bytes, it is recommended to use `bytes.concat()`.
Key to also note that `abi.encodePacked()` is deprecated in newest solidity versions and the fact the protocol uses a floating pragma could easily ecarcebate this issue

### Recommendation

To prevent hash collisions, it is recommended to avoid using `abi.encodePacked()` with dynamic types when passing the result to a hash function. Instead, use `abi.encode()`.
If there is only one parameter, consider casting it to `bytes()` or `bytes32()` in place of `abi.encodePacked()`. Additionally, when dealing with multiple parameters that are strings or bytes, use `bytes.concat()` to concatenate them before passing them to the hash function. By following these recommendations, you can enhance the integrity and security of the hash-based operations in your contract.

## QA-06 - Important governance addresses should always include the `0x0` check

### Vulnerability Details

Below is the constructor from InterchainGocernance.sol

```solidity
    /**
     * @notice Initializes the contract
     * @param gateway The address of the Axelar gateway contract
     * @param governanceChain_ The name of the governance chain
     * @param governanceAddress_ The address of the governance contract
     * @param minimumTimeDelay The minimum time delay for timelock operations
     */
     // @audit very important functions, atleast the 0x0 check should be added, since the addresses get hashed
    constructor(
        address gateway,
        string memory governanceChain_,
        string memory governanceAddress_,
        uint256 minimumTimeDelay
    ) AxelarExecutable(gateway) TimeLock(minimumTimeDelay) {
        governanceChain = governanceChain_;
        governanceAddress = governanceAddress_;
        governanceChainHash = keccak256(bytes(governanceChain_));
        governanceAddressHash = keccak256(bytes(governanceAddress_));
    }

```

### Recommendation

Add the 0x0 checks to ensure that the addresses don't get hashed as the zero address.

## QA-07 - The `SELFDESTRUCT` opcode post EIP-6780 behaves a bit different

### Vulnerability Details

Take a look at the [upgrade()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/interfaces/IAxelarGateway.sol#L177) function of AxelarGateway.sol

```solidity
    function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata setupParams
    ) external override onlyGovernance {
        if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();

        if (AxelarGateway(newImplementation).contractId() != contractId()) revert InvalidImplementation();

        emit Upgraded(newImplementation);
        // @audit the original behaviour of `SELFDESTRUCT` has been changed and will now only work within the same transaction . Since code deletion happens at the end of the transaction this should be taken into account for the comment below
        // AUDIT: If `newImplementation.setup` performs `selfdestruct`, it will result in the loss of _this_ implementation (thereby losing the gateway)
        //        if `upgrade` is entered within the context of _this_ implementation itself.
        if (setupParams.length != 0) {
            (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(IAxelarGateway.setup.selector, setupParams));

            if (!success) revert SetupFailed();
        }

        _setImplementation(newImplementation);
    }
```

There could easily be a misunderstanding of thee effects of the `SELFDESTRUCT` opcode in the event of an upgrade as mentioned in the inline team written _AUDIT_ tag.

### Recommendation

Update the inline audit comments to reflect the changes made in EIP-6780


## QA-08 - Dependency on `block.timestamp` for TimeLock is somewhat risky

### Vulnerability Details

The `block.timestamp` is widely used in various applications including generating random numbers, timing related functionalities, and conditional statements that are time-dependent. However, it's crucial to note that miners have some level of control over block timestamps, which might lead to potential manipulations if used incorrectly in smart contracts.

In the given TimeLock contract, the `block.timestamp` has been used to determine the minimum time delay for scheduling a new time lock and for checking if a time lock is ready to be finalized.

A couple instances include the [below](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/TimeLock.sol#L42-L87)

```solidity
TimeLock.sol:54:        uint256 minimumEta = block.timestamp + _minimumTimeLockDelay;
TimeLock.sol:84:        if (block.timestamp < eta) revert TimeLockNotReady();
```

While the usage of `block.timestamp` in this context might not lead to severe issues, it's still worth considering potential miner manipulations. For instance, a miner could potentially influence the timestamp of a block within a certain range, which could affect the scheduling and finalizing of time locks, especially in scenarios where precise timing is essential.

### Recommendation

While the usage of `block.timestamp` is generally safe for longer durations (like hours or days), if the time sensitivity of the lock system is in the scale of minutes or seconds, consider the following options:

1. **Medianized Oracle:** Use a medianized timestamp from a decentralized oracle network to have a more reliable timestamp.

2. **Block Number:** Another alternative could be using the `block.number` instead of `block.timestamp`. The number of blocks produced in Ethereum is fairly regular, about one every 12 seconds. This could provide a more manipulation-resistant timing mechanism, but keep in mind this also isn't the same on different chains

## QA-09 - There should be a clear fund retrieval method in all contracts with the payable `receive()` fallback function

### Vulnerability Details

Throughout the contracts under review, a recurring pattern has been identified where a payable fallback function is used without a clear method for retrieving those funds.
Where as for some contracts there could be a complex method to retrieve these funds by calling other functions, some do not include this.

Note that it can be said that this pattern has been observed in multiple contracts including but not limited to the InterChainGovernance.sol contract of the Axelar project.

The payable fallback function enables the contract to receive ETH:

```solidity
receive() external payable {}
```

Without a mechanism to withdraw the funds, they could be permanently locked within the contract, causing financial loss. As this is easily giving attackers incentives to look for a way to steal the funds in the contract.

### Recommendation

1mplement a secure withdrawal function that only the contract owner or a trusted account can call. This would allow the safe withdrawal of any funds accidentally sent to the contract. Here's an example:

Please note that if OpenZeppelin's 'onlyOwner' is going to be used to sort this then an explicit call should be made to set the owner of the contract since axelar uses openzeppelin >=4.9.0 and from OZ's 4.9.0 version there is a need to explicitly transfer the ownership or it would be defaulted to the 0x0 address, unlike previously where it defaults to the msg.sender
## QA-10 - `deployAndRegisterRemoteStandardizedToken()` and sister functions could be made more effective

### Vulnerability Details

In the `InterchainTokenService.sol` contract, the [deployAndRegisterRemoteStandardizedToken()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interfaces/IInterchainTokenService.sol#L245) function and potentially other functions that use `gasValue` do not perform an explicit check to ensure that the `msg.value` (the amount of Ether sent with the function call) is equal to or greater than the `gasValue`.

While the sponsors argue that the `gasService.payNativeGasForContractCall` function would revert if there isn't enough value in the contract to cover the `gasValue`, the provided implementation of this function only checks if `msg.value` is zero, not if it's less than the `gasValue`.

This means that a user could potentially call these functions with a `msg.value` that's less than `gasValue`, and the function call would not fail until it reaches the `gasService.payNativeGasForContractCall` function. This could be confusing to users and might potentially lead to unexpected function failures.

```solidity
function deployAndRegisterRemoteStandardizedToken(
    bytes32 salt,
    string calldata name,
    string calldata symbol,
    uint8 decimals,
    bytes memory distributor,
    bytes memory operator,
    string calldata destinationChain,
    uint256 gasValue
) external payable notPaused {
    bytes32 tokenId = getCustomTokenId(msg.sender, salt);
    _deployRemoteStandardizedToken(tokenId, name, symbol, decimals, distributor, operator, destinationChain, gasValue);
}
```

In the above code snippet, there is no check to ensure that the `msg.value` is greater than or equal to the `gasValue`.

### Recommendation

Add an explicit check to ensure that `msg.value` is equal to or greater than `gasValue` in the `deployAndRegisterRemoteStandardizedToken` function and any other functions that require a `gasValue`.

```solidity
require(msg.value >= gasValue, "Insufficient funds for specified gasValue");
```

This would enforce the requirement stated in the documentation and prevent the function from being called with an insufficient `msg.value`. It would also make the function's behavior more predictable and user-friendly, as the function call would fail immediately if the `msg.value` is less than `gasValue`, instead of failing later in the `gasService.payNativeGasForContractCall` function.

## QA-11 - `Timelock.sol` should include a few more sanity checks

### Vulnerability Details

In the Timelock.sol contract here are error definitions for `InvalidTimeLockHash()`, `TimeLockAlreadyScheduled()`, and `TimeLockNotReady()` which are used throughout to revert transactions under specific conditions. These cover several important cases and contribute to making the contract safer.

However, I believe there are missing checks for certain edge cases:

1. **`eta` far in the future:** The `_scheduleTimeLock` function currently does not have a check for `eta` values that are unreasonably far in the future. There should be a maximum limit to how far in the future a timelock can be scheduled to prevent potential misuse.

2. **`minimumTimeLockDelay` in the constructor:** There isn't a check for a sensible value for `minimumTimeDelay` during construction of the contract.

### Recommendation

This should be thought through and if suggestions are going to be implemented, then here is an example of how to implement the check for the `eta` value:

```solidity
uint256 constant MAXIMUM_TIME_LOCK_DELAY = 365 days;

// Within the _scheduleTimeLock function
if (eta > block.timestamp + MAXIMUM_TIME_LOCK_DELAY) revert EtaTooFarInTheFuture();
```

Additionally here's how the `minimumTimeLockDelay` could be checked in the constructor:

```solidity
uint256 constant MINIMUM_MINIMUM_TIME_LOCK_DELAY = 1 hours;
uint256 constant MAXIMUM_MINIMUM_TIME_LOCK_DELAY = 30 days;

constructor(uint256 minimumTimeDelay) {
    if (minimumTimeDelay < MINIMUM_MINIMUM_TIME_LOCK_DELAY || minimumTimeDelay > MAXIMUM_MINIMUM_TIME_LOCK_DELAY) revert InvalidMinimumTimeDelay();

    _minimumTimeLockDelay = minimumTimeDelay;
}
```

## QA-12 - Typos that should be fixed

### Vulnerability Details

- Instance [1](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L380)
  Change

* can be calculated ahead of time) then a mint/burn TokenManager is used. Otherwise a lock/unlcok TokenManager is used.
  To:
* can be calculated ahead of time) then a mint/burn TokenManager is used. Otherwise a _lock/unlock_ TokenManager is used.

- Instance [2](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interfaces/ITokenManager.sol#L75-L80)
  Change  
   _ @return the amount of token actually given, which will onle be differen than `amount` in cases where the token takes some on-transfer fee.
  To:
  _ @return the amount of token actually given, which will _only_ be _different_ than `amount` in cases where the token takes some on-transfer fee.

### Recommendation

Make the necessary fixes

## QA-13 - Pragma version should not be floating

### Vulnerability Details

Take this line from MultisigBase.sol as an example

```solidity
pragma solidity ^0.8.0;
```

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

### Recommendation

Suggest updating the pragma version to a newer version
