### GAS-1 Redundant type casting
`token` is already IERC20 type.
```solidity
51:        return IERC20(token).balanceOf(address(this)) - balance;
62:        return IERC20(token).balanceOf(address(this)) - balance;
66:        return IERC20(token).balanceOf(to) - balance;
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L51
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L62
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L66


### GAS-2 State variables can be immutable
These state variable are initialized in constructor and no more changed.
```solidity
39:    IAxelarGateway public gateway;
40:    IAxelarGasService public gasService;
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L39-L40