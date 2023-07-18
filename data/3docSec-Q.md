- [low] InterchainToken.sol, functions `interchainTransfer` and `interchainTransferFrom` may end up setting this allowance to `type(uint256).max` against the user's intention

Possibly a better solution would be:
- save the current allowance
- set the allowance to "amount"
- restore the previous allowance