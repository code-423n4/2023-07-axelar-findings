# Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | Calls inside loops that may address DoS | 1 |
| [L&#x2011;02] | executeMultisigProposal() can be failed due to invalid target address and nativeValue | 1 |
| [L&#x2011;03] | In InterchainGovernance.sol, minimumTimeDelay can be set to 0 | 1 |
| [L&#x2011;04] | _getProposalHash() can have hash collision | 1 |
| [L&#x2011;05] | Ethers sent to Multisig contract will be stucked forever | 1 |
| [L&#x2011;06] | _call() does not check target address can be address(0) | 1 |
| [L&#x2011;07] | return value is not checked in _call()  | 1 |
| [L&#x2011;08] | Improper Validation Of create() Return Value | 1 |
| [L&#x2011;09] | Distributor can mint tokens to his address | 1 |
| [L&#x2011;10] | Missing sanity validations in _takeToken() and _giveToken() | 2 |
| [L&#x2011;11] | Follow two steps process for distributor address transfer | 1 |
| [L&#x2011;12] | In Timelock.sol, minimumTimeDelay limits are undefined | 1 |


### [L&#x2011;01]  Calls inside loops that may address DoS
In InterchainProposalExecutor.sol contract _executeProposal() function, 
Calls to external contracts inside a loop are dangerous because it could lead to DoS if one of the calls reverts or execution runs out of gas. Such issue also introduces chance of problems with the gas limits.

**Per SWC-113:**
External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call.

Reference link- https://swcregistry.io/docs/SWC-113

**Code Reference:**
```Solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol

    function _executeProposal(InterchainCalls.Call[] memory calls) internal {
        for (uint256 i = 0; i < calls.length; i++) {
            InterchainCalls.Call memory call = calls[i];
            (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

            if (!success) {
                _onTargetExecutionFailed(call, result);
            } else {
                _onTargetExecuted(call, result);
            }
        }
    }
```

### Recommended Mitigation Steps
1) Avoid combining multiple calls in a single transaction, especially when calls are executed as part of a loop
2) Always assume that external calls can fail
3) Implement the contract logic to handle failed calls

### [L&#x2011;02]  executeMultisigProposal() can be failed due to invalid target address and nativeValue
In executeMultisigProposal(), check the target address is valid and is a contract address referring per the interface. If the nativeValue i.e ether is provided insufficient then the execution will be failed.

There is 1 instance of this issue:
In AxelarServiceGovernance.sol at [L-48](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L48)

```Solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol

    function executeMultisigProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable onlySigners {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

        if (!multisigApprovals[proposalHash]) revert NotApproved();

        multisigApprovals[proposalHash] = false;

        _call(target, callData, nativeValue);

        emit MultisigExecuted(proposalHash, target, callData, nativeValue);
    }
```

### Recommended Mitigation steps
1) Check the contract existence check as the executeMultisigProposal() has used .call low level function.
2) Check the target address is not address(0) otherwise the ether sent to target address will be lost.
3) Check the nativeValue is not 0 otherwise the execution will fail.


### [L&#x2011;03]  In InterchainGovernance.sol, minimumTimeDelay can be set to 0
minimumTimeDelay is passed in constructor via Timelock contract but it does not check that the minimumTimeDelay can be 0. If the delay is set to zero then the use of timelock is obscelate and the functions for which timelock is used will be executed immediately without any delay.

There is 1 instance of this issue:
In InterchainGovernance.sol at [L-48](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L48)

```Solidity
File: contracts/cgp/governance/InterchainGovernance.sol

    constructor(
        address gateway,
        string memory governanceChain_,
        string memory governanceAddress_,
        uint256 minimumTimeDelay
    ) AxelarExecutable(gateway) TimeLock(minimumTimeDelay) {
+       require(minimumTimeDelay != 0, "delay can not be 0");
        governanceChain = governanceChain_;
        governanceAddress = governanceAddress_;
        governanceChainHash = keccak256(bytes(governanceChain_));
        governanceAddressHash = keccak256(bytes(governanceAddress_));
    }
```

### [L&#x2011;04]  _getProposalHash() can have hash collision
Hash collision possibility increases with the dynamic data types like bytes, string, etc. To avoid hash collision in proposal hash, use abi.encode

There is 1 instance of this issue:
In InterchainGovernance.sol at [L-148](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L148)

### Recommended Mitigation steps

```Solidity
File: contracts/cgp/governance/InterchainGovernance.sol

    function _getProposalHash(
        address target,
        bytes memory callData,
        uint256 nativeValue
    ) internal pure returns (bytes32) {
-        return keccak256(abi.encodePacked(target, callData, nativeValue));
+        return keccak256(abi.encode(target, callData, nativeValue));
    }
```

### [L&#x2011;05]  Ethers sent to Multisig contract will be stucked forever
Multisig.sol contract has receive() function which can receive ethers directly sent to Multisig.sol however when by any chance the ethers are sent there is no functions to withdraw the ethers. Recommend to add the ether withdraw functions or restrict the direct receival of ether in contract.

### [L&#x2011;06]  _call() does not check target address can be address(0)
In Caller.sol, low level function call is used to transfer ether to target address but target address is not validated to address(0) which can cause permanent loss of ethers to address(0).

There is 1 instance of this issue:

### Recommended Mitigation steps

```Solidity
File: contracts/cgp/util/Caller.sol

    function _call(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) internal {
+       require(target != address(0), "invalid address");
        if (nativeValue > address(this).balance) revert InsufficientBalance();

        (bool success, ) = target.call{ value: nativeValue }(callData);
        if (!success) {
            revert ExecutionFailed();
        }
    }
```

### [L&#x2011;07]  return value is not checked in _call()
In Caller.sol, low level function call is used in which calldata is passed and accepted by call function but it does not return it. 

```Solidity
File: contracts/cgp/util/Caller.sol

    function _call(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) internal {
        if (nativeValue > address(this).balance) revert InsufficientBalance();

        (bool success, ) = target.call{ value: nativeValue }(callData);
        if (!success) {
            revert ExecutionFailed();
        }
    }
```
While passing the eth value it checks for its for success return value but it ignores the return data value check. This is in violation with solidity guidlines as the data is passed and accepted by .call function and its return value is unchecked.

Solidity documentation says,
"In order to interface with contracts that do not adhere to the ABI, or to get more direct control over the encoding, the functions call, delegatecall and staticcall are provided. They all take a single bytes memory parameter and return the success condition (as a bool) and the returned data (bytes memory). The functions abi.encode, abi.encodePacked, abi.encodeWithSelector and abi.encodeWithSignature can be used to encode structured data."

Example:
```Solidity
1   bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
2   (bool success, bytes memory returnData) = address(nameReg).call(payload);
3   require(success);
```
At L-2, returnData is used while the data is passed to .call function. Similarly the same should be taken care in in _execute() function.

### Recommended Mitigation steps
A simple solution is to import openzeppelin Address.sol and do the following changes in code.

```Solidity
File: contracts/cgp/util/Caller.sol

    function _call(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) internal {
        if (nativeValue > address(this).balance) revert InsufficientBalance();

-        (bool success, ) = target.call{ value: nativeValue }(callData);
-        if (!success) {
-            revert ExecutionFailed();
-        }
+        (bool success, bytes memory returnData) = target.call{ value: nativeValue }(callData);
+        Address.verifyCallResult(success, returndata);
    }
```

### [L&#x2011;08]  Improper Validation Of create() Return Value
In Create3.sol, CreateDeployer.deploy() is used to deploy a new contract with the specified bytecode using the CREATE opcode. The function does not revert properly if there is a failed contract deployment or revert from the create opcode as it does not properly check the returned address for bytecode. The create opcode returns the expected address which will never be the zero address.

There is 1 instance of this issue:
In CreateDeployer.deploy() at [L-21](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3.sol#L21-L31)

```Solidity
File: contracts/gmp-sdk/deploy/Create3.sol

21    function deploy(bytes memory bytecode) external {
22        address deployed;
23
24        assembly {
25            deployed := create(0, add(bytecode, 32), mload(bytecode))
26            if iszero(deployed) {
27                revert(0, 0)
28            }
29        }
30    }
```
Reference:- https://solodit.xyz/issues/m-13-improper-validation-of-create2-return-value-code4rena-mochi-mochi-contest-git

### Recommended Mitigation steps

```Solidity
File: contracts/gmp-sdk/deploy/Create3.sol

    function deploy(bytes memory bytecode) external {
        address deployed;

        assembly {
            deployed := create(0, add(bytecode, 32), mload(bytecode))
-            if iszero(deployed) {
+            if iszero(extcodesize(deployed))
                revert(0, 0)
            }
        }
    }
```

### [L&#x2011;09]  Distributor can mint tokens to his address
In StandardizedToken.sol, mint() can only be accessed by distributor address. distributor can mint unlimited tokens to his own address. Therefore  validation is required from happening this.

There is 1 instance of this issue:

```Solidity
File: contracts/its/token-implementations/StandardizedToken.sol

76    function mint(address account, uint256 amount) external onlyDistributor {
77        _mint(account, amount);
78    }
```
### Recommended Mitigation steps

```Solidity

    function mint(address account, uint256 amount) external onlyDistributor {
+        require(account != msg.sender, "self mint not allowed");
        _mint(account, amount);
    }
```

### [L&#x2011;10]  Avoid sending null tokens in _takeToken() and _giveToken()
In TokenManagerLiquidityPool.sol, Avoid sending zero amount of tokens in functions like _takeToken() and _giveToken(). 

There is [2](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L77-L101) instances of this issue:

### Recommended Mitigation steps

```Solidity
File: ontracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

    function _takeToken(address from, uint256 amount) internal override returns (uint256) {
+       require(amount != 0,"invalid amount");
        IERC20 token = IERC20(tokenAddress());

      // Some code

   }
```
```Solidity
File: ontracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

    function _giveToken(address to, uint256 amount) internal override returns (uint256) {
+       require(to != address(0), "invalid address");
+       require(amount != 0, "invalid amount");
        IERC20 token = IERC20(tokenAddress());


      // Some code

   }
```

### [L&#x2011;11]  Follow two steps process for distributor address transfer
In Distributable.sol, setDistributor() is used to change the distributor of the contract, however this only one step process which is very risky in case the wrong address is set as distributor address which means the wrong address will have a control on Distributable contract. This is not the recommended way to change such critical addresses.

There is 1 instances of this issue:

```Solidity
File: contracts/its/utils/Distributable.sol

51    function setDistributor(address distr) external onlyDistributor {
52        _setDistributor(distr);
53    }
```

### Recommended Mitigation steps
Follow two step process for critical address transfer.

### [L&#x2011;12]  In Timelock.sol, minimumTimeDelay limits are undefined
In Timelock.sol, minimumTimeDelay is passed in constructor for which limits are not defined. It means it can be set as 0 or it can be indefinately delayed. It is recommended to have some minimum and maximum limits for minimumTimeDelay.

There is 1 instance of this issue:
In Timelock.sol at [L-21](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/TimeLock.sol#L21-L23)