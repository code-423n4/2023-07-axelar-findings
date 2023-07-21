## 1. USE `abi.encode` IN PLACE OF `abi.encodePacked`.

When concatenating multiple data types to be provided as input to `keccak256` function, it is recommended to use `abi.encode` instead of `abi.encodePacked` since `abi.encodePacked` can introduce hash collisions in the event dynamic data types such as `bytes`, `arrays` or `structs` are used.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L148

## 2. USE THE SAME CODING TEMPLATE IN SIMILAR FUNCTION FOR EASE OF UNDERSTANDING

In the `InterchainGovernance.getProposalEta` function the `_getProposalHash` function is used to compute the `keccak256` hash of the encoded data of `target, callData, nativeValue`. But in the `InterchainGovernance.executeProposal` function, the same hash is calculated directly inside the function itself as shown below:

        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

Hence it is recommended to use the `_getProposalHash` function to get the hash value in the `InterchainGovernance.executeProposal` function as well for the ease of readability and clarity of the code.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L57
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L73

## 3. `abstract` KEYWORD IS MISSING IN THE ABSTRACT FUNCTION

In the `Natspec` comments for the `InterchainProposalExecutor` contract the following is mentioned:

    * This contract is abstract and some of its functions need to be implemented in a derived contract.

When a contract is abstract the `abstract` keyword is used to denote the same.

Hence it is recommended to add the `abstract` keyword before the function name as follows:

abstract contract InterchainProposalExecutor is IInterchainProposalExecutor, AxelarExecutable, Ownable {

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L22

## 4. LACK OF INPUT VALIDATION FOR CRITICAL STATE CHANGE

In the `InterchainProposalExecutor.constructor` contract, the `ownership` of the contract is set as follows inside the `constructor`.

        _transferOwnership(_owner); 

But the `_owner` is not checked for `address(0)` before calling the `_transferOwnership` function in the `openzeppelin Ownable.sol` contract. As a result if the owner is set to `address(0)` then `onlyOwner` modifier controlled critical functions will be inaccessible. This will prompt a redeployment of the contract.

Hence it is required to perform the following check before calling the `_transferOwnership` function to make sure the proper input validation is performed.

    require(_owner != address(0), "Can not be address(0)");

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L29-L31
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L54
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L24

## 5. USE TWO STEP OWNERSHIP TRANSFER INSTEAD OF SINGLE STEP OWNERSHIP TRANSFER

Here the `exisiting owner` `proposes` the `new owner` and then new owner should accept the role by calling the respective function as `msg.sender`.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L30

## 6. WRONG `Natspec` COMMENT GIVEN FOR THE FUNCTION `InterchainToken.tokenManagerRequiresApproval`

The following comment is given as the description of the `return` value of `InterchainToken.tokenManagerRequiresApproval`.

     * @return tokenManager the TokenManager called to facilitate cross chain transfers.


But the above `Natspec` comment is wrong since the function returns `boolean` value which indicates whether the `tokenManager` requires `approval` for token transfers.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L30-L32

The following `Natspec` comment is also wrong since the `ExpressCallHandler._setExpressReceiveToken` calls the `_getExpressReceiveTokenSlot` function to get the storage slot.

     * @notice Stores the address of the express caller at the storage slot determined by _getExpressSendTokenSlot

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L72
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L86
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L100
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L120

```solidity
/**
 * @title TokenManagerProxy
 * @dev This contract is a proxy for token manager contracts. It implements ITokenManagerProxy and
 * inherits from FixedProxy from the gmp sdk repo
 */
interface ITokenManagerProxy {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/ITokenManager.sol#L5-L10

     * @param amount the amount of tokens to take from msg.sender.

Above should be corrected as follows:

     * @param amount the amount of tokens to take from sender.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L134

## 7. NO CONDITIONAL CHECK FOR THE UNDERFLOW, HENCE TRANSACTION COULD REVERT WITHOUT CLEAR ERROR MESSAGE

In the `InterchainToken.interchainTransferFrom` function, the existing allowance of the msg.sender is decreased by the `amount` as follows:

        if (_allowance != type(uint256).max) {
            _approve(sender, msg.sender, _allowance - amount); 
        }

But there is no conditinal check to verify `_allowance >= amount`. If this is not the case the transaction could revert without an error message. Even in the `openzeppelin` library for `ERC20.sol`, this check is performed to confirm that `_allowance >= amount` and if not transaction reverts.

Hence it is recommended to update the above code snippet as follows to inlcude the above check.

        if (_allowance != type(uint256).max) {
            require(_allowance >= amount, "Insufficient Allowance");
            _approve(sender, msg.sender, _allowance - amount); 
        }

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L88-L90

## 8. USE NAMED MAPPING INTRODUCED FROM `solidity 0.8.18` 

Consider moving to solidity version 0.8.18 or later, and use named mappings for better understanding of the purpose of each mapping.

    mapping(string => bytes32) public remoteAddressHashes;
    mapping(string => string) public remoteAddresses;
    ...
    mapping(string => bool) public supportedByGateway;

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L15-L16
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L19    

## 9. LACK OF CONTRACT EXISTENCE CHECK BEFORE CALLING THE LOW LEVEL `call` FUNCTION

The `Caller._call` internal function is used to exeucte the low level `call` function to execute the `calldata` payload on the destination address as shown below:

        (bool success, ) = target.call{ value: nativeValue }(callData); 

But there is no check to make sure that the `target` has contract code in the destination address.
If the `nativeValue` is sent to a destination contract without contract code the transaction will succeed but the sent `native` tokens will be lost. Thus this is loss of funds to the caller of the transaction.

Hence it is recommended to check the contract code size as shown below before calling the low level `call` function. The following code snippet should be run before calling the low level `call` function inside the `Caller._call` function.

        uint256 codeSize;
        assembly {
            codeSize := extcodesize(contractAddress)
        }
        require(codeSize > 0, "Can not transfer to contract with no contract code");

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol#L11-L22
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L100
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76