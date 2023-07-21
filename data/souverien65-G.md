

```solidity
function _scheduleTimeLock(bytes32 hash, uint256 eta) internal returns (uint256) {
    require(hash != 0, "InvalidTimeLockHash");
    require(_getTimeLockEta(hash) == 0, "TimeLockAlreadyScheduled");

    uint256 minimumEta = block.timestamp + _minimumTimeLockDelay;
    if (eta < minimumEta) {
        eta = minimumEta;
    }

    _setTimeLockEta(hash, eta);

    return eta;
}
```

By optimizing the function, we eliminated redundant storage reads and combined conditions to reduce gas usage. Additionally, storing the `block.timestamp` in a local variable reduces read operations.

Also https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AdminMultisigBase.sol#L29 

Optimized version of the `onlyAdmin` modifier:

```solidity
modifier onlyAdmin() {
    uint256 adminEpoch = _adminEpoch();

    if (!_isAdmin(adminEpoch, msg.sender)) revert NotAdmin();

    bytes32 topic = keccak256(msg.data);

    // Check that admin has not voted, then record that they have voted.
    if (_hasVoted(adminEpoch, topic, msg.sender)) revert AlreadyVoted();

    // Determine the new vote count and update it.
    uint256 adminVoteCount = _getVoteCount(adminEpoch, topic) + 1;
    _setVoteCount(adminEpoch, topic, adminVoteCount);

    // Check if the vote count has reached the threshold.
    if (adminVoteCount < _getAdminThreshold(adminEpoch)) {
        _setHasVoted(adminEpoch, topic, msg.sender, true);
        return;
    }

    // Clear vote count and voted booleans.
    _setVoteCount(adminEpoch, topic, 0);

    uint256 adminCount = _getAdminCount(adminEpoch);

    for (uint256 i; i < adminCount; ++i) {
        _setHasVoted(adminEpoch, topic, _getAdmin(adminEpoch, i), false);
    }

    // Perform the external call or state change after the Checks-Effects-Interactions.
    _;
}
```

In this optimized version, we eliminate unnecessary storage reads and writes by checking the vote count before recording the vote. If the vote count hasn't reached the threshold, we update the vote count and the voted status. Otherwise, we clear the vote count and voted status for the next voting round.

By following this approach, we can reduce gas costs during the voting process, making the contract more gas-efficient.
