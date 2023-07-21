## Low Issues
### L-1 Too strict equivalence
The equivalence should be not so strict. It can be `<=`. It is not so gas-efficient but prevents revert.
```solidity
113:        if (totalGas != msg.value) {
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L113-L114

## Non Critical
### N-1 Typos
There are 26 instances
The word `multisignature` should be `multi-signature`.
```solidity
7: * @notice An interface defining the base operations for a multisignature contract.
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/interfaces/IMultisigBase.sol#L7
The word `multisignature` should be `multi-signature`.
```solidity
9: * @notice This contract implements a custom multisignature wallet where transactions must be confirmed by a
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L9
The word `a the` should be `the`.
```solidity
17:     * @notice Constructs the proxy contract with a the implementation address, owner address, and optional setup parameters.
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/Proxy.sol#L17
The word `overwitten` should be `overwritten`.
The word `specifiying` should be `specifying`.
The word `permissionlesly` should be `permissionlessly`.
The word `transfered` should be `transferred`.
```solidity
18:     * @dev Needs to be overwitten.
24:     * @notice Getter function specifiying if the tokenManager requires approval to facilitate cross-chain transfers.
28:     * TokenManager specifically to do it permissionlesly.
40:     * @param amount The amount of token to be transfered.
76:     * @param amount the amount of token to be transfered.          
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L18
The word `transfered` should be `transferred`.
```solidity
17:     * @param amount The amount of token to be transfered.
34:     * @param amount the amount of token to be transfered.  
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interfaces/IInterchainToken.sol#L17
The word `varaibles` should be `variables`.
The word `deployement` should be `deployment`.
The word `unlcok` should be `unlock`.
The word `fullfill` should be `fullfil`.
The word `reimburment` should be `reimbursement`.
The word `destiantion` should be `destination`.
```solidity
74:     * @dev All of the varaibles passed here are stored as immutable variables.
159:     * @return tokenManagerAddress deployement address of the TokenManager.
380:     * can be calculated ahead of time) then a mint/burn TokenManager is used. Otherwise a lock/unlcok TokenManager is used.
406:     * bytes then a mint/burn TokenManager is used. Otherwise a lock/unlcok TokenManager is used.
432:     * @notice Uses the caller's tokens to fullfill a sendCall ahead of time. Use this only if you have detected an outgoing
457:     * @notice Uses the caller's tokens to fullfill a callContractWithInterchainToken ahead of time. Use this only if you have
496:     * @param sourceAddress the address where the token is coming from, which will also be used for reimburment of gas.
500:     * @param metadata the data to be passed to the destiantion.             
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L74
The word `fullfill` should be `fullfil`.
```solidity
316:     * @notice Uses the caller's tokens to fullfill a sendCall ahead of time. Use this only if you have detected an outgoing sendToken that matches the parameters passed here.
330:     * @notice Uses the caller's tokens to fullfill a callContractWithInterchainToken ahead of time. Use this only if you have detected an outgoing sendToken that matches the parameters passed here.
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interfaces/IInterchainTokenService.sol#L316
The word `implementaion_` should be `implementation_`.
```solidity
73:        address implementaion_ = implementation();
78:            let result := delegatecall(gas(), implementaion_, 0, calldatasize(), 0, 0)
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/proxies/TokenManagerProxy.sol#L73
The word `souceChain` should be `sourceChain`.
```solidity
14:    event TrustedAddressAdded(string souceChain, string sourceAddress);
15:    event TrustedAddressRemoved(string souceChain);
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interfaces/IRemoteAddressValidator.sol#L14-L15
The word `onle` should be `only`.
The word `differen` should be `different`.
```solidity
159:     * @return the amount of token actually given, which will onle be differen than `amount` in cases where the token takes some on-transfer fee.
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/TokenManager.sol#L159
The word `eror` should be `error`.
```solidity
18:     * @dev Throws a NotDistributor custom eror if called by any account other than the distributor.
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/Distributable.sol#L18
### N-2 State variables should be before functions
```solidity
22:    uint256 internal constant TOKEN_ADDRESS_SLOT = 0xc4e632779a6a7838736dd7e5e6a0eadf171dd37dfb6230720e265576dfcf42ba;
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol#L22