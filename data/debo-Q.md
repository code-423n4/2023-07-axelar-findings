## L-01
# SWC-127 Arbitrary Jump with Function Type Variable
```URL
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42-L52
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
## Description
Multiple calls are executed in the same transaction.

This call is executed following another call within the same transaction. It is possible that the call never gets executed if a prior call fails permanently. This might be caused intentionally by a malicious callee. If possible, refactor the code such that each transaction only executes one external call or make sure that all callees can be trusted (i.e. they're part of your own codebase).

External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically (this also reduces the chance of problems with the gas limit).

# Remediation
It is recommended to follow call best practices:

Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop
Always assume that external calls can fail
Implement the contract logic to handle failed calls