# [L-01] missing overwrite keyword in payNativeGasForContractCallWithToken
https://github.com/code-423n4/2023-07-axelar/blob/acda3c34e5eeecccbd939eb8bcfc10c6e0215192/contracts/its/test/AxelarGasService.sol#L172

The `payNativeGasForContractCallWithToken` missing `overwrite` keyword when it inherits from interface `IAxelarGasService`

https://github.com/code-423n4/2023-07-axelar/blob/acda3c34e5eeecccbd939eb8bcfc10c6e0215192/contracts/its/test/AxelarGasService.sol#L213
https://github.com/code-423n4/2023-07-axelar/blob/acda3c34e5eeecccbd939eb8bcfc10c6e0215192/contracts/gmp-sdk/test/MockGasService.sol#L206

The `collectFees` missing `overwrite` keyword when it inherits from interface `IAxelarGasService`
# [L-02] Refactor event and use only one event
there is many events in `IAxelarGasService.sol`
```
GasPaidForContractCall,
GasPaidForContractCallWithToken,
NativeGasPaidForContractCall,
NativeGasPaidForContractCallWithToken,
GasPaidForExpressCallWithToken,
NativeGasPaidForExpressCallWithToken
```
There have the same parameters each time. Why not add an event type parameter that will be `indexed` and use one event (this will reduce deployment cost)

# [L-03] Duplicated code in _rotateSigners method. There is logic where signers.isSigner array, but this logic is done in `onlySigners` - modifier. So this code not needed 
Mitigation steps: 
- remove the following code in _rotateSigners method
```
uint256 length = signers.accounts.length;

// Clean up old signers.
for (uint256 i; i < length; ++i) {
    signers.isSigner[signers.accounts[i]] = false;
}
``` 
https://github.com/code-423n4/2023-07-axelar/blob/acda3c34e5eeecccbd939eb8bcfc10c6e0215192/contracts/cgp/auth/MultisigBase.sol#L154

# [L-04] `minimumTimeDelay` should be bounded in TimeLock contract. 
The InterchainGovernance::constructor method sets a `minimumTimeDelay` that is unbounded, meaning it can possibly be a really huge number. While this likeability is still low, it seems dangerous. I suggest you remove to upper time delay value