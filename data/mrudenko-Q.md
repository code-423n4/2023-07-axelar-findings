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