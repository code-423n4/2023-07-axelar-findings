### Gas Optimizations List

| Number | Optimization Details                                                                                                     | Instances |
| :----: | :----------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Use `delete` for state variables to reinitialize with their default value rather than assigning default value saves gas. |     2     |
| [G-02] | Use assembly for `address(0)` comparison to save gas                                                                     |    13     |
| [G-03] | Do not emit `block.timestamp` since it is already included in event data and can be accessed from event data.            |     1     |
| [G-04] | State variables can be re-arranged to be packed into fewer storage slots                                                 |     1     |
| [G-05] | Do not calculate constant variables                                                                                      |    28     |
| [G-06] | Missing `zero-address` check in `constructor`                                                                            |     4     |
| [G-07] | Use nested if and, avoid multiple check combinations using &&                                                            |     1     |
| [G-08] | Using ternary operator instead of if else saves gas                                                                      |     2     |
| [G-09] | Use unchecked{} whenever underflow/overflow not possible                                                                 |     2     |

Total 9 issues.

Note: Here some gas findings may be similar to automated report but all the instances are different that were missed in auto report.

## [G-01] Use `delete` for state variables to reinitialize with their default value rather than assigning default value saves gas.

This gas-saving behavior is due to how the delete operation is implemented in the Ethereum Virtual Machine (EVM).

When you use the delete keyword on a state variable, the EVM internally performs a low-level operation called SSTORE to update the variable's storage slot. The SSTORE operation clears the storage slot, effectively resetting the variable to its default value.

In contrast, if you manually assign a default value to a state variable, it involves writing a value to the storage slot. This operation is performed using the SSTORE opcode as well but with an extra cost compared to the delete operation. Writing the default value to the storage slot consumes more gas than simply clearing the storage slot with the delete operation.

_2 instances - 1 file:_

### Instance#1:

```diff
File:  contracts/cgp/auth/MultisigBase.sol

- 66: voting.voteCount = 0;
+ 66: delete voting.voteCount;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L66C9-L66C30

### Instance#2:

```diff
File: contracts/cgp/auth/MultisigBase.sol
  70:  for (uint256 i; i < count; ++i) {
- 71:            voting.hasVoted[signers.accounts[i]] = false;
+ 71:          delete voting.hasVoted[signers.accounts[i]];
  72:        }
```

## [G-02] Use assembly for `address(0)` comparison to save gas

_13 instances - 7 files:_

### Instance#1 :

```solidity
File : contracts/cgp/auth/MultisigBase.sol
173:   if (account == address(0)) revert InvalidSigners();

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L173

Recommended Changes:

```diff
+       bool isAddressZero;
+        assembly {
+            isAddressZero := eq(address(account), 0)
+        }
-    if (account == address(0)) revert InvalidSigners();
+    if(isAddressZero)   revert  InvalidSigners();
```

### Instance#2-3 :

```solidity
File: contracts/cgp/util/Upgradable.sol
25:   if (newOwner == address(0)) revert InvalidOwner();
        ...

63:   if (implementation() == address(0)) revert NotProxy();

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Upgradable.sol#L25
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Upgradable.sol#L63

Recommended Changes : Like Instance#1 assembly can be used, or separate function can be made to check for address(0) using assembly.

### Instance#4:

```solidity
File :  /contracts/cgp/AdminMultisigBase.sol

157: if (threshold == uint256(0)) revert InvalidAdminThreshold();

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AdminMultisigBase.sol#L157

### Instance#5:

```solidity
File :  /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

88:   if (deployedAddress_ == address(0)) revert FailedDeploy();

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L88C7-L88C67

### Instance#6:

```solidity
File :  /contracts/gmp-sdk/upgradable/InitProxy.sol

47:    if (implementation() != address(0)) revert AlreadyInitialized();

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L47

Recommended Changes : Like Instance#1 of [G-02] recommendation assembly can be used, with little change use `if(!isAddressZero)` instead of `if(isAddressZero)`

### Instance#7:

```solidity
File :  /contracts/gmp-sdk/upgradable/Proxy.sol

30:    if (owner == address(0)) revert InvalidOwner();

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Proxy.sol#L30

Recommended Changes : Like Instance#1 of [G-02] recommendation assembly can be used

### Instance#8-13:

```solidity
File:  contracts/its/interchain-token-service/InterchainTokenService.sol
92: if (
93:            remoteAddressValidator_ == address(0) ||
94:            gasService_ == address(0) ||
95:            tokenManagerDeployer_ == address(0) ||
96:            standardizedTokenDeployer_ == address(0)
        ) revert ZeroAddress();

        ...

609:     if (expressCaller == address(0)) {
        ...

652:     if (expressCaller != address(0)) {

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L92C6-L97C32

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L609

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L652

## [G-03] : Do not emit `block.timestamp` since it is already included in event data and can be accessed from event data.

_1 instance - 1 file:_

### Instance#1 :

```solidity
File: contracts/cgp/governance/InterchainGovernance.sol

78: emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L78C9-L78C93

## [G‑04] State variables can be re-arranged to be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

_1 instance - 1 file:_

### Instance#1 : `tokenManager` and `decimals` can be declared one after another so that they can be fit into one storage slot.

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol
22:  address public tokenManager;
     string public name;
     string public symbol;
25:  uint8 public decimals;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L22C5-L25C27

Recommended Changes:

```diff
  22:  address public tokenManager;
+      uint8 public decimals;
       string public name;
       string public symbol;
- 25:  uint8 public decimals;
```

## [G-05] Do not calculate constant variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas each time of use.

_Total 28 instances - 8 files:_

### Instance#1-7 : Assign direct simple constant value to constant types after calculating this value off chain(remix OR using vscode)

```solidity
File: contracts/cgp/AdminMultisigBase.sol
19:  bytes32 internal constant KEY_ADMIN_EPOCH = keccak256('admin-epoch');

21:  bytes32 internal constant PREFIX_ADMIN = keccak256('admin');
22:  bytes32 internal constant PREFIX_ADMIN_COUNT = keccak256('admin-count');
23:  bytes32 internal constant PREFIX_ADMIN_THRESHOLD = keccak256('admin-threshold');
24:  bytes32 internal constant PREFIX_ADMIN_VOTE_COUNTS = keccak256('admin-vote-counts');
25:  bytes32 internal constant PREFIX_ADMIN_VOTED = keccak256('admin-voted');
26:  bytes32 internal constant PREFIX_IS_ADMIN = keccak256('is-admin');

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AdminMultisigBase.sol#L19C5-L26C71

Recommended Changes : Assign direct hash value and comment these calculating lines to show that assigning hash belongs to which string.

### Instance#8-19 :

```solidity
File: /contracts/cgp/AxelarGateway.sol

44:    bytes32 internal constant PREFIX_COMMAND_EXECUTED = keccak256('command-executed');
45:    bytes32 internal constant PREFIX_TOKEN_ADDRESS = keccak256('token-address');
46:    bytes32 internal constant PREFIX_TOKEN_TYPE = keccak256('token-type');
47:    bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED = keccak256('contract-call-approved');
48:    bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED_WITH_MINT = keccak256('contract-call-approved-with-mint');
49:      bytes32 internal constant PREFIX_TOKEN_MINT_LIMIT = keccak256('token-mint-limit');
50:      bytes32 internal constant PREFIX_TOKEN_MINT_AMOUNT = keccak256('token-mint-amount');

52:      bytes32 internal constant SELECTOR_BURN_TOKEN = keccak256('burnToken');
53:      bytes32 internal constant SELECTOR_DEPLOY_TOKEN = keccak256('deployToken');
54:      bytes32 internal constant SELECTOR_MINT_TOKEN = keccak256('mintToken');
55:      bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL = keccak256('approveContractCall');
56:      bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL_WITH_MINT = keccak256('approveContractCallWithMint');
57:      bytes32 internal constant SELECTOR_TRANSFER_OPERATORSHIP = keccak256('transferOperatorship');

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L44C5-L58C1

Recommended Changes : Assign direct hash value and comment these calculating lines to show that assigning hash belongs to which string.

### Instance#20 :

```solidity
File: /contracts/gmp-sdk/deploy/Create3.sol

41: bytes32 internal constant DEPLOYER_BYTECODE_HASH = keccak256(type(CreateDeployer).creationCode);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L41

Recommended Changes : Assign direct hash value and comment these calculating line to show that assigning hash belongs to which string.

### Instance#21 :

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

13:  bytes32 internal constant PREFIX_TIME_LOCK = keccak256('time-lock');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L13

Recommended Changes : Assign direct hash value and comment these calculating line to show that assigning hash belongs to which string.

### Instance#22-25 :

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

62:    bytes32 internal constant PREFIX_CUSTOM_TOKEN_ID = keccak256('its-custom-token-id');
63:    bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_ID = keccak256('its-standardized-token-id');
64:    bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_SALT = keccak256('its-standardized-token-salt');

    ...

71:  bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L62C5-L64C105

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L71C4-L71C82

Recommended Changes : Assign direct hash value and comment these calculating lines to show that assigning hash belongs to which string.

### Instance#26:

```solidity
File: contracts/its/proxies/InterchainTokenServiceProxy.sol

12:   bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol#L12

Recommended Changes : Assign direct hash value and comment these calculating line to show that assigning hash belongs to which string.

### Instance#27:

```solidity
File: contracts/its/proxies/RemoteAddressValidatorProxy.sol

12:  bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/RemoteAddressValidatorProxy.sol#L12C5-L12C82

Recommended Changes : Assign direct hash value and comment these calculating line to show that assigning hash belongs to which string.

### Instance#28:

```solidity
File: contracts/its/proxies/StandardizedTokenProxy.sol

14:   bytes32 private constant CONTRACT_ID = keccak256('standardized-token');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol#L14C4-L14C76

Recommended Changes : Assign direct hash value and comment these calculating line to show that assigning hash belongs to which string.

## [G-06] Missing `zero-address` check in `constructor`

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wastes gas as it requires the redeployment of the contract.

Recommendation : Check input address if it is zero, using assembly to save more gas see: [G-02]

_4 instances - 3 files:_

### Instance#1: `implementationAddress` can be checked for address(0)

```solidity
File : /contracts/gmp-sdk/upgradable/FixedProxy.sol

23: constructor(address implementationAddress) {
24:        implementation = implementationAddress;
25:    }

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FixedProxy.sol#L23C5-L25C6

### Instance#2-3: `_gateway` and `_gasService` can be checked for address(0) before assigning to state variables.

```solidity
File : /contracts/interchain-governance-executor/InterchainProposalSender.sol

42:   constructor(address _gateway, address _gasService) {
43:       gateway = IAxelarGateway(_gateway);
44:        gasService = IAxelarGasService(_gasService);
45:    }

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L42C5-L45C6

### Instance#4: `interchainTokenServiceAddress_` must be checked for address(0) before assigning to immutable variable `interchainTokenServiceAddress` .

```solidity
File : contracts/its/proxies/TokenManagerProxy.sol

25:  constructor(
26:        address interchainTokenServiceAddress_,
        uint256 implementationType_,
        bytes32 tokenId_,
        bytes memory params
    ) {
31:        interchainTokenServiceAddress = IInterchainTokenService(interchainTokenServiceAddress_);

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/TokenManagerProxy.sol#L25C4-L31C97

## [G-07] Use nested if and, avoid multiple check combinations using &&

Using nested if is cheaper gas wise than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

_1 instance - 1 file:_

### Instance#1:

```solidity
File : contracts/gmp-sdk/upgradable/Proxy.sol

33:    if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Proxy.sol#L33

Recommended Changes: Use nested if

```diff
- 33:  if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
+   if (id != bytes32(0)) {
+            if (IUpgradable(implementationAddress).contractId() != id) {
+                revert InvalidImplementation();
+            }
+        }

```

## [G-08] Using ternary operator instead of if else saves gas

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes conditional stack manipulation.
The exact amount of gas saved by using the ternary operator instead of the if-else construct in Solidity can vary depending on the specific scenario, the complexity of the surrounding code, and the conditions involved

_2 instances - 2 files:_

### Instance#1 :

```solidity
File : contracts/interchain-governance-executor/InterchainProposalExecutor.sol
78: if (!success) {
                _onTargetExecutionFailed(call, result);
            } else {
                _onTargetExecuted(call, result);
82:          }

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L78C13-L82C14

Recommended Changes :

```diff
- 78: if (!success) {
-                _onTargetExecutionFailed(call, result);
-            } else {
-                _onTargetExecuted(call, result);
- 82:          }

+     !success ? _onTargetExecutionFailed(call, result) : _onTargetExecuted(call, result);
```

### Instance#2 :

```solidity
File: /contracts/its/token-manager/TokenManager.sol

68:  if (operatorBytes.length == 0) {
69:            operator_ = address(interchainTokenService);
70:        } else {
71:            operator_ = operatorBytes.toAddress();
72:        }
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L68C9-L72C10

Recommended Changes :

```diff
- 68:  if (operatorBytes.length == 0) {
- 69:            operator_ = address(interchainTokenService);
- 70:        } else {
- 71:            operator_ = operatorBytes.toAddress();
- 72:        }
+       operatorBytes.length == 0 ? operator_ = address(interchainTokenService) : operator_ = operatorBytes.toAddress();
```

## [G-09] Use unchecked{} whenever underflow/overflow not possible

Because of previous condition check before the operation(+/-), it can be decided that underflow/overflow not possible.

_2 instances - 1 file:_

### Instance#1 : `type(uint256).max - amount` can't underflow because `amount` is type of uint256 so it can't go above it's max value `type(uint256).max` so this subtraction can't be less than zero this means no chances for underflow .So this subtraction can be unchecked and cached rather than re-calculating same thing.

```solidity
File: contracts/its/interchain-token/InterchainToken.sol

43: function interchainTransfer(
        string calldata destinationChain,
        bytes calldata recipient,
46:        uint256 amount,
          ...

56:            if (allowance_ != type(uint256).max) {
57:                if (allowance_ > type(uint256).max - amount) {
58:                    allowance_ = type(uint256).max - amount;
                }

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L56C12-L60C1

Recommended Changes :

```diff
  56:            if (allowance_ != type(uint256).max) {
- 57:                if (allowance_ > type(uint256).max - amount) {
- 58:                    allowance_ = type(uint256).max - amount;
+                  unchecked{
+                     uint256 diffAmount = type(uint256).max - amount;
+                    }
+                             if (allowance_ > diffAmount) {
+                    allowance_ = diffAmount;
                }
           ...
```

### Instance#2 : `type(uint256).max - amount` can't underflow same as above instance#1

```solidity
File: contracts/its/interchain-token/InterchainToken.sol

83:   uint256 amount

     ...

95:  if (allowance_ != type(uint256).max) {
96:               if (allowance_ > type(uint256).max - amount) {
97:                    allowance_ = type(uint256).max - amount;
                }

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L95C13-L98C18

Recommended Changes : Same as above instance#1 recommendation changes. `type(uint256).max - amount` subtraction can be unchecked and cached rather than re-calculating same thing.
