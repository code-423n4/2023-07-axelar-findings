## 1. Include the logic written in `MultisigBase.sol/onlySigners()` modifier into a function so that wherever the modifier is used, the codesize is not increased.

```Solidity
    modifier onlySigners() {
        signerCheck();
        _;

    function signerCheck() public view {
        // onlySigner() logic
    }

}
```


## 2. Repeating comments of multiple functions in the contract and their interfaces.
`Multibase.sol` and `IMultibase.sol` have their comments repeating in many functions and even same comments in `isSigner()` and `hasSignerVoted()`.
Also in `Multisig.sol/execute()`, `Timelock.sol/minimumTimeLockDelay()` which have repeating comments with its interface.
The comments should be coveredin interface itself and `@dev` comments should be included in child contract.


## 3. Better to use `uint ChainId` to represent a chain rather than string naming the chain in `InterchainProposalExecutor.sol`
mappings named as `whitelistedCallers` and `whitelistedSenders` also include the string datatypes for representing chains.