# Low Risk Finding
 
## [L‑01] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() [instead]

```
bytes32 key = keccak256(abi.encodePacked(PREFIX_TIME_LOCK, hash));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L93

```
bytes32 key = keccak256(abi.encodePacked(PREFIX_TIME_LOCK, hash));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L104

```
bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L73

```
return keccak256(abi.encodePacked(target, callData, nativeValue));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L148

```
  bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L53

```
bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L84

and 
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3.sol#L72C8-L74C106


## [L‑02] incorrect declaration of variable in function parameter `uint256` 
`uint256` should have a name
```
 function _executeWithToken(
        string calldata, /* sourceChain */
        string calldata, /* sourceAddress */
        bytes calldata, /* payload */
        string calldata, /* tokenSymbol */
        uint256          /* amount */    // <== Error
    ) internal pure override {
        revert TokenNotSupported();
    }
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L155C4-L163C6

## [L‑03] Empty receive()/payable fallback() function does not authenticate requests
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.
168: `receive() external payable {}`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L168

41: `receive() external payable {}`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/Multisig.sol#L41

67:`receive() external payable virtual {}`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/BaseProxy.sol#L67

## [L‑04] Use Ownable2Step rather than Ownable
`Ownable2Step` and `Ownable2StepUpgradeable` prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.
12: `abstract contract Upgradable is Ownable, IUpgradable {`
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L12

## [L‑05]  Functions without implementation must be marked virtual.
````
 function setWhitelistedProposalSender(
        string calldata sourceChain,
        address sourceSender,
        bool whitelisted
    ) external;
````

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L29C4-L33C16

```
 function setWhitelistedProposalCaller(
        string calldata sourceChain,
        address sourceCaller,
        bool whitelisted
    ) external;
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L41C4-L45C16



# Non-Critical Finding

## [N-01] Use a non-legacy and more battle-tested version
Use 0.8.10

## [S-02] Generate perfect code headers every time
Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [N-03] NatSpec comments should be increased in contracts
> all contest

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity Official documentation

in complext project such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## [N-04] Include return parameters in NatSpec comments
> all contest

## [N-05] Assembly Codes Specific – Should Have Comments
>Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.
>This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.
>Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.
```
 assembly {
            eta := sload(key)
        }
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/TimeLock.sol#L95C8-L97C10

```
 assembly {
            sstore(key, eta)
        }
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/TimeLock.sol#L106C8-L108C10

```
assembly {
            deployed := create(0, add(bytecode, 32), mload(bytecode))
            if iszero(deployed) {
                revert(0, 0)
            }
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3.sol#L24C9-L28C14

```
 assembly {
            implementation_ := sload(_IMPLEMENTATION_SLOT)
        }
```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/BaseProxy.sol#L23C8-L25C10

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/libraries/AddressBytesUtils.sol#L20C5-L23C6

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/libraries/AddressBytesUtils.sol#L33C9-L36C10

## [N-06] Use underscores for number literals not for varaible name
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol

## [N-07] Use of bytes.concat() instead of abi.encodePacked()

``` 
23:  ) FinalProxy(implementationAddress, owner, abi.encodePacked(operator)) {}
```

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)