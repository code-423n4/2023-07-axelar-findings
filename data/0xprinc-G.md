## 1. No need to make `struct Signer` in `MultisigBase.sol`
The struct has only a single instance of use to make the signer in the whole repo. 
Consider declaring `accounts`, `threshold` and `isSigner` without the use of struct. This will save gas while deploying.


## 2. The return variable can be declared inside `returns()` while declaring the function.
`MultiBase.sol/getSignerVotesCount()` has return variable named as `voteCount` which can be declared while declaring the function as follows ike this to save gas:

```Solidity
function getSignerVotesCount(bytes32 topic) external view override returns (uint256 voteCount) {
```


## 3. In `InterchainProposalSender.sol/sendProposals()`, `interchainCalls.length` should be stored in a variable and also passed as a variable in `revertIfInvalidFee()` function
This should be done as the length of the argument `interchainCalls` is fetched `2*length` times and this can be mitigated by storing this in a variable.