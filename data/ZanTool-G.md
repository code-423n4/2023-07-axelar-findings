# Issues
## 1. Use CustomError Instead of String
### Location
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/Multicall.sol#L28
### Description
When using revert, using CustomError is more gas efficient than string description. Because the error message described using CustomError is only compiled into four bytes.
### Advice
When reverting, it is recommended to use CustomError instead of ordinary strings to describe the error message.
## 2. Get Contract Balance of ETH in Assembly
### Location
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Caller.sol#L16
### Description
Using the selfbalance and balance opcodes to get the ETH balance of the contract in assembly saves gas compared to getting the ETH balance through address(this).balance and xx.balance.
### Advice
It is recommended to get the contract ETH balance in assembly.
# Tools Used
static analysis tool of ZAN (https://zan.top/home).