## Lack of Balance Checks in Minting and Burning Functions

## Description:
The `AxelarGateway.sol` smart contract contains a critical issue where the `_mintToken()` and `_burnTokenFrom()` functions do not include balance checks. This vulnerability allows users to mint or burn tokens without verifying their actual token holdings. As a consequence, the token supply may become inaccurate, leading to potential losses for token holders.


https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L512-L528
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L530-L550


## Recommendation:
It is essential to include balance checks in the `_mintToken()` and `_burnTokenFrom()` functions to ensure that the amount of tokens being minted or burned does not exceed the account's current balance. This simple addition will prevent users from performing these operations with invalid token amounts and help maintain the integrity of the token supply.
