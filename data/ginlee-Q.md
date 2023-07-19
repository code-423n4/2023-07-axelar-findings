[L-01]potential DOS attack in rotateSigners function 
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L142-L179

The function _rotateSigners in rotateSigners iterates through the signers.accounts array to perform cleanup operations on each signer, and also iterates through the newAccounts array to mark the new signers as valid signers. If the newAccounts array is very large, this will result in a high number of iterations, increasing the gas cost of the transaction. When the gas cost of the transaction exceeds the current block's gas limit, the transaction may fail or be unable to be included in a block, affecting the execution of the function.

Potential DoS attack: If a malicious user intentionally passes a large newAccounts array, especially a very large one, it may cause the contract's execution time to exceed expectations, potentially leading to a denial-of-service (DoS) attack. This is because other users may be unable to perform critical operations in a timely manner, resulting in the contract being unable to handle other important transactions properly.

To avoid these issues, it is recommended to limit the length of parameter arrays when designing the contract and avoid passing excessively large arrays in practical usage. 

[L-02]Potential precision loss
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/FlowLimit.sol#L64
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/FlowLimit.sol#L76
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/FlowLimit.sol#L127

no validation for uint256 epoch = block.timestamp / EPOCH_TIME, since block.timestamp may less than EPOCH_TIME, epoch will be 0 if this is the case, slot will be wrong data and affect following calculation

should count in precision loss when using division 


