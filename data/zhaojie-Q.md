## `newThreshold` should be set separately

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L142

`MultisigBase` has a function `rotateSigners` that sets the signer's `account` and `Threshold`,
the `rotateSigners` function set both Accounts and Threshold together.

But setting Accounts is an important operation that requires care,
if we just want to modify Threshold, it is not necessary to pass Accounts, and it is recommended to add another function to set Threshold separately.


```
    function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {
        _rotateSigners(newAccounts, newThreshold);
    }
```