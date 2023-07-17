**BaseProxy.sol**
- L16 - A constant _OWNER_SLOT is created, which is never used, therefore it should be eliminated.


**Multicall.sol**
- L13 - The MulticallFailed error is created but it is never used, this generates an extra cost of gas, therefore it should be eliminated.


**ITokenManagerProxy.sol**
- L11 - The ImplementationLookupFailed error is created, but it is never used, so it should be removed.


**ITokenManager.sol**
- L17/18 - Se crean los errores TakeTokenFailed y GiveTokenFailed, pero nunca son utilizados, esto genera un gasto extra de gas, por lo tanto deberian ser eliminados.


**TokenManagerLockUnlock.sol**
- L51/66 - This operation could be unchecked: IERC20(token).balanceOf(to) - balance, because there is no possibility that after a transfer, the balance is lower, at most it could be the same.


**TokenManagerLiquidityPool.sol**
- L85/100 - This operation could be unchecked: IERC20(token).balanceOf(to) - balance, because there is no possibility that after a transfer, the balance will be less, at most it could be the same.
