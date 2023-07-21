### Gas Optimizations List:
| Number | Optimization Details | Instances |
|:--:|:-------| :-----:|
| [G-01] | Using immutable instead of constant for inline keccak256 hashes of strings |16|

## [G-01] Use immutable instead of constant for inline keccak256 hashes of strings

##### Description:
For inline keccak256 hashes of strings, using immutable instead of constant is more gas efficient.
Though after changing them to immutable, the state mutability of functions in which these hashes are used, need to be changed to view from pure.

##### Instances:

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L44C1-L57C98

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L62C1-L64C105


###### Savings:

Gas savings for using immutable instead of constant obtained from tests is ``22 gas`` per call. So, for 16 instances it will be ``352 gas`` per call. 