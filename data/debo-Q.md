## L-01
# SWC-127 Arbitrary Jump with Function Type Variable
```URL
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42-L52
```
```sol
    function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
        deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));


        // solhint-disable-next-line avoid-low-level-calls
        (bool success, ) = deployedAddress_.call(init);
        if (!success) revert FailedInit();
    }
```
## Description
The caller can redirect execution to arbitrary bytecode locations.

It is possible to redirect the control flow to arbitrary locations in the code. This may allow an attacker to bypass security controls or manipulate the business logic of the smart contract. Avoid using low-level-operations and assembly to prevent this issue.

Solidity supports function types. That is, a variable of function type can be assigned with a reference to a function with a matching signature. The function saved to such variable can be called just like a regular function.

The problem arises when a user has the ability to arbitrarily change the function type variable and thus execute random code instructions. As Solidity doesn't support pointer arithmetics, it's impossible to change such variable to an arbitrary value. However, if the developer uses assembly instructions, such as mstore or assign operator, in the worst case scenario an attacker is able to point a function type variable to any code instruction, violating required validations and required state changes.

## Remediation
The use of assembly should be minimal. A developer should not allow a user to assign arbitrary values to function type variables.

## L-02
# SWC-113 DoS with Failed Call
```URL
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42-L52
```
```sol
    function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
        deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));


        // solhint-disable-next-line avoid-low-level-calls
        (bool success, ) = deployedAddress_.call(init);
        if (!success) revert FailedInit();
    }
```
## Description
Multiple calls are executed in the same transaction.

This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they're part of your own codebase).

External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically (this also reduces the chance of problems with the gas limit).

# Remediation
It is recommended to follow call best practices:

Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop
Always assume that external calls can fail
Implement the contract logic to handle failed calls.

## L-03
# SWC-118 Incorrect Constructor Name
```URL
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/Multicall.sol#L22-L33
```
```sol
    function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {
        results = new bytes[](data.length);
        for (uint256 i = 0; i < data.length; ++i) {
            (bool success, bytes memory result) = address(this).delegatecall(data[i]);


            if (!success) {
                revert(string(result));
            }


            results[i] = result;
        }
    }
```
## Description
Until Solidity version 0.4.21 the constructor can only be defined as a function with the exact same name as the contract class. The function "multicall" looks similar to a constructor for "Multicall" but is not a constructor. Please rename to avoid confusion.

Constructors are special functions that are called only once during the contract creation. They often perform critical, privileged actions such as setting the owner of the contract. Before Solidity version 0.4.22, the only way of defining a constructor was to create a function with the same name as the contract class containing it. A function meant to become a constructor becomes a normal, callable function if its name doesn't exactly match the contract name. This behavior sometimes leads to security issues, in particular when smart contract code is re-used with a different name but the name of the constructor function is not changed accordingly.

## Remediation
Solidity version 0.4.22 introduces a new constructor keyword that make a constructor definitions clearer. It is therefore recommended to upgrade the contract to a recent version of the Solidity compiler and change to the new constructor declaration.