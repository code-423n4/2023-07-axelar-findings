gas:
1.
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L73-L84

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/AddressString.sol#L15

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/AddressString.sol#L39

use unchecked block with ++i instead of i++ or just ++i in for loop

```
for (uint256 i = 0; i < length; ) {
     // logic ...
     unchecked {
          ++i;
     }
}
```