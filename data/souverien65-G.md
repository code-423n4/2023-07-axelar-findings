

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
