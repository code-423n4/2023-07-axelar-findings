1.Use common function `_getProposalHash`
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L65

2.Ensure that at least one signer address is set.
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L35#L37

```
    constructor(address[] memory accounts, uint256 threshold) {
        require(accounts.length>0);
        _rotateSigners(accounts, threshold);
    }
```

3.`addTrustedAddress` maybe overwrite the previous address because it hasn't been confirmed whether there is a value previously
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83#L89

4.Lack of zero address check
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L56