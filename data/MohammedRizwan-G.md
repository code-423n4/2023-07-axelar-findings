### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | Save gas by catching function arguments in local variables | 1 |
| [G&#x2011;02] | Avoid emitting block.timestamp in events | 1 |
| [G&#x2011;03] | _call() function can be refactored to save gas | 1 |
| [G&#x2011;04] | Use gas efficient Solady library for create3 | 1 |



### [G&#x2011;01]  Save gas by catching function arguments in local variables
In _rotateSigners(), the argument for newThreshold can be catched in local variables which can be further used in functions. This will avoid repeated calling of function arguments which can cause more than twice amount of gas by current implementation. Below recommendation will result in lots of gas saving.

There is 1 instance of this issue:
In MultisigBase.sol at [L-149](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L149-L179)

### Recommended Mitigation steps

```Solidity
File: /contracts/cgp/auth/MultisigBase.sol

    function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
        uint256 length = signers.accounts.length;
+       uint256 _newThreshold = newThreshold;

        // Clean up old signers.
        for (uint256 i; i < length; ++i) {
            signers.isSigner[signers.accounts[i]] = false;
        }

        length = newAccounts.length;

-        if (newThreshold > length) revert InvalidSigners();
+        if (_newThreshold > length) revert InvalidSigners();


-        if (newThreshold == 0) revert InvalidSignerThreshold();
+        if (_newThreshold == 0) revert InvalidSignerThreshold();


        ++signerEpoch;

        signers.accounts = newAccounts;
-        signers.threshold = newThreshold;
+        signers.threshold = _newThreshold;

        for (uint256 i; i < length; ++i) {
            address account = newAccounts[i];

            // Check that the account wasn't already set as a signer for this epoch.
            if (signers.isSigner[account]) revert DuplicateSigner(account);
            if (account == address(0)) revert InvalidSigners();

            signers.isSigner[account] = true;
        }

        emit SignersRotated(newAccounts, newThreshold);
    }
```

### [G&#x2011;02]  Avoid emitting block.timestamp in events
While the event is emitted in blockchain, an event also consist of block.timestamp Therefore a gas can be saved by removing the explicitly block.timestamp from event.

There is 1 instance of this issue:
In InterchainGovernance.sol at [L-78](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L78)

### Recommended Mitigation steps

```Solidity
File: contracts/cgp/governance/InterchainGovernance.sol

    function executeProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

        _finalizeTimeLock(proposalHash);
        _call(target, callData, nativeValue);

-        emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);
+        emit ProposalExecuted(proposalHash, target, callData, nativeValue);
    }
````

### [G&#x2011;03]  _call() function can be refactored to save gas
In Caller.sol,  _call() function NativeValue argument can be catched as local variable to save gas. This function has been used at lot of places in contracts. This will save considerable amount of gas whereever it is used.

There is 1 instance of this issue:

```Solidity
File: contracts/cgp/util/Caller.sol

    function _call(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) internal {
+       uint256 _nativeValue = nativeValue;
-        if (nativeValue > address(this).balance) revert InsufficientBalance();
+        if (_nativeValue > address(this).balance) revert InsufficientBalance();

-        (bool success, ) = target.call{ value: nativeValue }(callData);
+        (bool success, ) = target.call{ value: _nativeValue }(callData); 
        if (!success) {
            revert ExecutionFailed();
        }
    }
```

### [G&#x2011;04]  Use gas efficient Solady library for create3
In contracts, create3(), create2(), create() functions are used and has been coded which consumes good amount of bytes, however these can be imported from Solady library which is gas efficient and secured.

There is 1 instance of this issue:
[In gmp-sdk deploy contracts](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy)
