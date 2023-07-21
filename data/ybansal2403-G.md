| Number | Optimization Details                                                                                       | Instances |
| :----: | :--------------------------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Use constants instead of type(uintx).max or type(uintx).min                  |     7     |
| [G-02] | Non efficient zero initialization                                                                          |     6     |
| [G-03] | Using ternary operator instead of if else saves gas                                                        |     4     |
| [G-04] | Make 3 event parameters indexed when possible                                                              |     7     |
| [G-05] | x + y is more efficient than using += for state variables (likewise for -=)|     1     |
| [G-06] | Use calldata instead of memory for function arguments that do not get mutated                                        |     37     |
| [G-07] | Do not calculate constants variables                                        |     8     |
| [G-08] | Use unchecked{} whenever underflow/overflow not possible                                                   |     1     |
| [G-09] | Use storage instead of memory for structs/arrays                                                           |     2     |
| [G-10] | The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement                         |     2     |
| [G-11] | <= or >= is more efficient than < or >                                        |     5     |
| [G-12] | Calling .length in a for loop wastes gas                                        |     1     |
| [G-13] | Use assembly to check for the zero address                                        |     19     |



Total 13 issues

## [G-01]  Use constants instead of type(uintx).max or type(uintx).min 

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

_Total 7 instances - 1 file:_

### Instance#1-3:
Findings are marked with <=FOUND

```solidity
File: contracts/its/interchain-token/InterchainToken.sol
43:function interchainTransfer(
44:        string calldata destinationChain,
45:        bytes calldata recipient,
46:        uint256 amount,
47:        bytes calldata metadata
48:    ) external payable {
49:        address sender = msg.sender;
50:        ITokenManager tokenManager = getTokenManager();
51:        /**
52:         * @dev if you know the value of `tokenManagerRequiresApproval()` you can just skip the if statement and just do nothing or _approve.
53:         */
54:        if (tokenManagerRequiresApproval()) {
55:            uint256 allowance_ = allowance[sender][address(tokenManager)];
56:            if (allowance_ != type(uint256).max) {//<=FOUND
57:                if (allowance_ > type(uint256).max - amount) {//<=FOUND
58:                    allowance_ = type(uint256).max - amount;//<=FOUND
59:                }
60:
61:                _approve(sender, address(tokenManager), allowance_ + amount);
62:            }
63:        }
64:
65:        // Metadata semantics are defined by the token service and thus should be passed as-is.
66:        tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);
67:    }
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L43C5-L67C6

### Instance#4-7:
Findings are marked with <=FOUND

```solidity
File: contracts/its/interchain-token/InterchainToken.sol
88:if (_allowance != type(uint256).max) {//<=FOUND
89:            _approve(sender, msg.sender, _allowance - amount);
90:        }
91:
92:        ITokenManager tokenManager = getTokenManager();
93:        if (tokenManagerRequiresApproval()) {
94:            uint256 allowance_ = allowance[sender][address(tokenManager)];
95:            if (allowance_ != type(uint256).max) {//<=FOUND
96:                if (allowance_ > type(uint256).max - amount) {//<=FOUND
97:                    allowance_ = type(uint256).max - amount;//<=FOUND
98:                }
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L88C9-L98C18


## [G-02] Non efficient zero initialization

Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

_Total 6 instances - 4 file:_

### Instance#1:

```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
74: for (uint256 i = 0; i < calls.length; i++) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74C8-L74C53

### Instance#2:

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
63: for (uint256 i = 0; i < interchainCalls.length; ) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63C9-L63C60

### Instance#3-4:

findings are marked with <=FOUND

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
105: uint256 totalGas = 0;//<=FOUND
106:        for (uint256 i = 0; i < interchainCalls.length; ) {//<=FOUND
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L105C9-L106C60

### Instance#5:

```solidity
File: contracts/its/token-manager/TokenManager.sol
118:uint32 version = 0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L118C9-L118C28

### Instance#6:

```solidity
File: contracts/its/utils/Multicall.sol
24: for (uint256 i = 0; i < data.length; ++i) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Multicall.sol#L24

## [G-03]  Using ternary operator instead of if else saves gas

_Total 4 instances - 3 file:_

### Instance#1:

```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
78:if (!success) {
79:                _onTargetExecutionFailed(call, result);
80:            } else {
81:                _onTargetExecuted(call, result);
82:            }
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L78C13-L82C14

### Instance#2:

```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
161:if (result.length > 0) {
162:            // The failure data is a revert reason string.
163:            assembly {
164:                revert(add(32, result), mload(result))
165:            }
166:        } else {
167:            // There is no failure data, just revert with no reason.
168:            revert ProposalExecuteFailed();
169:        }
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L161C9-L169C10

### Instance#3:

```solidity
File: contracts/its/token-manager/TokenManager.sol
68:if (operatorBytes.length == 0) {
69:            operator_ = address(interchainTokenService);
70:        } else {
71:            operator_ = operatorBytes.toAddress();
72:        }
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L68C9-L72C10

### Instance#4:

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
609:if (expressCaller == address(0)) {
610:            amount = tokenManager.giveToken(destinationAddress, amount);
611:            emit TokenReceived(tokenId, sourceChain, destinationAddress, amount);
612:        } else {
613:            amount = tokenManager.giveToken(expressCaller, amount);
614:        }
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L609C9-L614C10

## [G-04] Make 3 event parameters indexed when possible

Using the indexed keyword for value types such as uint, bool, and address saves gas costs, as seen in the example below. However, this is only the case for value types, whereas indexing bytes and strings are more expensive than their unindexed version.

_Total 7 instances - 3 file:_

### Instance#1:

```solidity
File: contracts/cgp/interfaces
/IAxelarServiceGovernance.sol
15: event MultisigApproved(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
16:event MultisigCancelled(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
17:event MultisigExecuted(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IAxelarServiceGovernance.sol#L15C1-L17C127

### Instance#2-3:

Finding are marked with <=FOUND

```solidity
File: contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol
7:    event WhitelistedProposalCallerSet(string indexed sourceChain, address indexed sourceCaller, bool whitelisted);//<=FOUND
8:
9:    // An event emitted when the proposal sender is whitelisted
10:    event WhitelistedProposalSenderSet(string indexed sourceChain, address indexed sourceSender, bool whitelisted);//<=FOUND
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L7C1-L10C116

### Instance#4-5:

Finding are marked with <=FOUND

```solidity
File: contracts/its/interfaces/IInterchainTokenService.sol
32: event TokenSent(bytes32 tokenId, string destinationChain, bytes destinationAddress, uint256 indexed amount);//<=FOUND
33:    event TokenSentWithData(//<=FOUND
34:        bytes32 tokenId,
35:        string destinationChain,
36:        bytes destinationAddress,
37:        uint256 indexed amount,
38:        address indexed sourceAddress,
39:        bytes data
40:    );
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L32C4-L40C7

### Instance#6:

Finding are marked with <=FOUND

```solidity
File: contracts/its/interfaces/IInterchainTokenService.sol
50:event RemoteTokenManagerDeploymentInitialized(//<=FOUND
51:        bytes32 indexed tokenId,
52:        string destinationChain,
53:        uint256 indexed gasValue,
54:        TokenManagerType indexed tokenManagerType,
55:        bytes params
56:    );
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L50C5-L56C7

### Instance#7:

Finding are marked with <=FOUND

```solidity
File: contracts/its/interfaces/IInterchainTokenService.sol
67:event TokenManagerDeployed(bytes32 indexed tokenId, TokenManagerType indexed tokenManagerType, bytes params);
68:    event StandardizedTokenDeployed(//<=FOUND
69:        bytes32 indexed tokenId,
70:        string name,
71:        string symbol,
72:        uint8 decimals,
73:        uint256 mintAmount,
74:        address mintTo
75:    );
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L67C5-L75C7

## [G-05] x + y is more efficient than using += for state variables (likewise for -=)

In instances found where either += or -= are used against state variables use x = x + y instead

_Total 1 instances - 1 file:_

### Instance#1:

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
107: totalGas += interchainCalls[i].gas;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L107C13-L107C48 


## [G-06] Use calldata instead of memory for function arguments that do not get mutated

It is generally cheaper to load variables directly from calldata, rather than copying them to memory. Only use memory if the variable needs to be modified.

_Total 37 instances - 9 file:_

### Instance#1:

```solidity
File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
24:function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L24C5-L24C5

### Instance#2:

```solidity
File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
43:bytes memory bytecode,
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L43C9-L43C31

### Instance#3:

```solidity
File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
80:function _deploy(bytes memory bytecode, bytes32 salt) internal returns (address deployedAddress_) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L80

### Instance#4:

```solidity
File: contracts/gmp-sdk/deploy/Create3Deployer.sol
50:bytes memory bytecode,
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L50C9-L50C31

### Instance#5:

```solidity
File: contracts/gmp-sdk/deploy/Create3.sol
21: function deploy(bytes memory bytecode) external {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L21C4-L21C54

### Instance#6:

```solidity
File: contracts/gmp-sdk/deploy/Create3.sol
51:  function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L51

### Instance#7:
Findings are marked as <=FOUND

```solidity
File: contracts/gmp-sdk/upgradable/InitProxy.sol
35:function init(
36:        address implementationAddress,
37:        address newOwner,
38:        bytes memory params//<=FOUND
39:    ) external {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L35C4-L39C17

### Instance#8:

```solidity
File: contracts/gmp-sdk/upgradable/FinalProxy.sol
72:function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L72C5-L72C125

### Instance#9:

```solidity
File: contracts/its/libraries/AddressBytesUtils.sol
17:function toAddress(bytes memory bytesAddress) internal pure returns (address addr) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/libraries/AddressBytesUtils.sol#L17C5-L17C5

### Instance#10:

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
83:function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83C5-L83C91

### Instance#11-13:

Findings are marked as <=FOUND

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
46:function _getExpressReceiveTokenWithDataSlot(
47:        bytes32 tokenId,
48:        string memory sourceChain,//<=FOUND
49:        bytes memory sourceAddress,//<=FOUND
50:        address destinationAddress,
51:        uint256 amount,
52:        bytes memory data,//<=FOUND
53:        bytes32 commandId
54:    ) internal pure returns (uint256 slot) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L46C5-L54C45

### Instance#14-15:

Findings are marked as <=FOUND

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
110:function _setExpressReceiveTokenWithData(
111:        bytes32 tokenId,
112:        string memory sourceChain,//<=FOUND
113:        bytes memory sourceAddress,//<=FOUND
114:        address destinationAddress,
115:        uint256 amount,
116:        bytes calldata data,
117:        bytes32 commandId,
118:        address expressCaller
119:    ) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L110C5-L119C17

### Instance#16-17:

Findings are marked as <=FOUND

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
171:    function getExpressReceiveTokenWithData(
172:        bytes32 tokenId,
173:        string memory sourceChain,//<=FOUND
174:        bytes memory sourceAddress,//<=FOUND
175:        address destinationAddress,
176:        uint256 amount,
177:        bytes calldata data,
178:        bytes32 commandId
179:    ) public view returns (address expressCaller) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L110C5-L119C17

### Instance#18-20:

Findings are marked as <=FOUND

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
231:    function _popExpressReceiveTokenWithData(
232:        bytes32 tokenId,
233:        string memory sourceChain,//<=FOUND
234:        bytes memory sourceAddress,//<=FOUND
235:        address destinationAddress,
236:        uint256 amount,
237:        bytes memory data,//<=FOUND
238:        bytes32 commandId
239:    ) internal returns (address expressCaller) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L231C5-L239C49

### Instance#21:

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
241:function getParamsLockUnlock(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L241C5-L241C122

### Instance#22:

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
251:function getParamsMintBurn(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L251C5-L251C120

### Instance#23:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
262:function getParamsLiquidityPool(
263:        bytes memory operator,//<=FOUND
264:        address tokenAddress,
265:        address liquidityPoolAddress
266:    ) public pure returns (bytes memory params) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L262C5-L266C50

### Instance#24-25:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
417:function deployAndRegisterRemoteStandardizedToken(
418:        bytes32 salt,
419:        string calldata name,
420:        string calldata symbol,
421:        uint8 decimals,
422:        bytes memory distributor,//<=FOUND
423:        bytes memory operator,//<=FOUND
424:        string calldata destinationChain,
425:        uint256 gasValue
426:    ) external payable notPaused {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L417C5-L426C35

### Instance#26:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
502: function transmitSendToken(
503:        bytes32 tokenId,
504:        address sourceAddress,
505:        string calldata destinationChain,
506:        bytes memory destinationAddress,//<=FOUND
507:        uint256 amount,
508:        bytes calldata metadata
509:    ) external payable onlyTokenManager(tokenId) notPaused {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L502C4-L509C61

### Instance#27:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
707:function _callContract(
708:        string calldata destinationChain,
709:        bytes memory payload,//<=FOUND
710:        uint256 gasValue,
711:        address refundTo
712:    ) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L707C5-L712C17

### Instance#28:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
748:function _deployRemoteTokenManager(
749:        bytes32 tokenId,
750:        string calldata destinationChain,
751:        uint256 gasValue,
752:        TokenManagerType tokenManagerType,
753:        bytes memory params//<=FOUND
754:    ) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L748C5-L754C17

### Instance#29-32:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
770:function _deployRemoteStandardizedToken(
771:        bytes32 tokenId,
772:        string memory name,//<=FOUND
773:        string memory symbol,//<=FOUND
774:        uint8 decimals,
775:        bytes memory distributor,//<=FOUND
776:        bytes memory operator,//<=FOUND
777:        string calldata destinationChain,
778:        uint256 gasValue
779:    ) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L770C5-L779C17

### Instance#33:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
808:    function _deployTokenManager(
809:        bytes32 tokenId,
810:        TokenManagerType tokenManagerType,
811:        bytes memory params//<=FOUND
812:    ) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L808C1-L812C17

### Instance#34-35:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
841:    function _deployStandardizedToken(
842:        bytes32 tokenId,
843:        address distributor,
844:        string memory name,//<=FOUND
845:        string memory symbol,//<=FOUND 
846:        uint8 decimals,
847:        uint256 mintAmount,
848:        address mintTo
849:    ) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L841C1-L849C17

### Instance#36-37:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
880:    function _expressExecuteWithInterchainTokenToken(
881:        bytes32 tokenId,
882:        address destinationAddress,
883:        string memory sourceChain,//<=FOUND
884:        bytes memory sourceAddress,//<=FOUND
885:        bytes calldata data,
886:        uint256 amount
887:    ) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L880C1-L887C17


## [G-07] Do not calculate constants variables
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

_Total 12 instances - 8 file:_

### Instance#1:

```solidity
File: contracts/gmp-sdk/upgradable/FinalProxy.sol   
18: bytes32 internal constant FINAL_IMPLEMENTATION_SALT = keccak256('final-implementation');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L18C4-L18C93

### Instance#2:

```solidity
File: contracts/its/proxies/InterchainTokenServiceProxy.sol   
12: bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol#L12C5-L12C82

### Instance#3:

```solidity
File: contracts/its/proxies/RemoteAddressValidatorProxy.sol   
12: bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/RemoteAddressValidatorProxy.sol#L12C5-L12C82

### Instance#4:

```solidity
File: contracts/its/proxies/StandardizedTokenProxy.sol   
14:  bytes32 private constant CONTRACT_ID = keccak256('standardized-token');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol#L14C4-L14C76

### Instance#5:

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol   
21:  bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L21C5-L21C82

### Instance#6:

```solidity
File: contracts/its/token-implementations/StandardizedToken.sol   
27:  bytes32 private constant CONTRACT_ID = keccak256('standardized-token');
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L27C5-L27C76

### Instance#7-8:

Findings are marked as <=FOUND

```solidity
File: contracts/its/utils/FlowLimit.sol  
15:    uint256 internal constant PREFIX_FLOW_OUT_AMOUNT = uint256(keccak256('prefix-flow-out-amount'));//<=FOUND
16:    uint256 internal constant PREFIX_FLOW_IN_AMOUNT = uint256(keccak256('prefix-flow-in-amount'));////<=FOUND
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol#L15C1-L16C99

### Instance#9-11:

Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol  
62:   bytes32 internal constant PREFIX_CUSTOM_TOKEN_ID = keccak256('its-custom-token-id');//<=FOUND
63:    bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_ID = keccak256('its-standardized-token-id');//<=FOUND
64:    bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_SALT = keccak256('its-standardized-token-salt');//<=FOUND
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L62C2-L64C105

### Instance#12:

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol  
71:   bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service'); 
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L71C4-L71C82

## [G-08]  Use unchecked{} whenever underflow/overflow not possible

Using unchecked increments in Solidity can save on gas fees by bypassing built-in overflow checks, thus optimizing gas usage, but requires careful assessment of potential risks and edge cases to avoid unintended consequences.

_Total 1 instances - 1 file:_

### Instance#1:

```solidity
File: contracts/cgp/auth/MultisigBase.sol
70:for (uint256 i; i < count; ++i) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L70

## [G-09] Use storage instead of memory for structs/arrays

In Solidity, using storage instead of memory for structs and arrays in function parameters can result in gas savings. When data is passed as a storage reference, the function operates directly on the original data stored in contract storage, without needing to create a copy. Conversely, when memory is used, a copy of the data is created, which incurs additional gas costs. However, it's essential to note that while using storage references can save gas, it also means that any changes made to the data inside the function will modify the original data stored in the contract, which may not always be the desired behavior. Therefore, developers should carefully consider the implications of using storage versus memory based on the specific requirements of their functions.

_Total 2 instances - 1 file:_

### Instance#1-2:

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
534:function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L534C5-L534C110

## [G-10] The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement

Using a double if statement instead of logical AND (&&) can provide similar short-circuiting behavior whereas double if is slightly more efficient.

_Total 2 instances - 2 file:_

### Instance#1:

```solidity
File: contracts/cgp/AxelarGateway.sol
87:    modifier onlyMintLimiter() {
88:        if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();
89:
90:        _;
91:    }
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L87C1-L91C6

### Instance#2:

```solidity
File: contracts/gmp-sdk/upgradable/Proxy.sol
33: if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Proxy.sol#L33C8-L33C119

## [G-11]  <= or >= is more efficient than < or >

Make such found comparisons to the <=/>= equivalent when comparing against integer literals

_Total 5 instances - 4 file:_

### Instance#1:

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
91:if (interchainCall.gas > 0) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L91C9-L91C38

### Instance#2:

```solidity
File: contracts/gmp-sdk/upgradable/Upgradable.sol
60:if (params.length > 0) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L60C9-L60C33

### Instance#3:

```solidity
File: contracts/its/token-implementations/StandardizedToken.sol
64:if (mintAmount > 0) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L64C13-L64C34

### Instance#4:

```solidity
File:contracts/its/interchain-token-service/InterchainTokenService.sol
519: if (version > 0) revert InvalidMetadataVersion(version);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L519C9-L519C65

### Instance#5:

```solidity
File:contracts/its/interchain-token-service/InterchainTokenService.sol
714:if (gasValue > 0) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L714C9-L714C28

## [G-12] Calling .length in a for loop wastes gas

Rather than calling .length for an array in a for loop declaration, it is far more gas efficient to cache this length before and use that instead. This will prevent the array length from being called every loop iteration

_Total 1 instance - 1 file:_

### Instance#1:

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol
106:for (uint256 i = 0; i < interchainCalls.length; ) {
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106C9-L106C60

## [G-13] Use assembly to check for the zero address

ing assembly for address comparisons in Solidity can save gas because it allows for more direct access to the Ethereum Virtual Machine (EVM), reducing the overhead of higher-level operations. Solidity's high-level abstraction simplifies coding but can introduce additional gas costs. Using assembly for simple operations like address comparisons can be more gas-efficient.

_Total 19 instances -9 file:_

### Instance#1:

```solidity
File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
88:if (deployedAddress_ == address(0)) revert FailedDeploy();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L88C9-L88C67

### Instance#2:

```solidity
File: contracts/gmp-sdk/upgradable/Proxy.sol
30:if (owner == address(0)) revert InvalidOwner();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Proxy.sol#L30

### Instance#3:

```solidity
File: contracts/gmp-sdk/upgradable/InitProxy.sol
47: if (implementation() != address(0)) revert AlreadyInitialized();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L47

### Instance#4:

```solidity
File: contracts/its/remote-address-validator/RemoteAddressValidator.sol
28: if (_interchainTokenServiceAddress == address(0)) revert ZeroAddress();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L28C9-L28C80

### Instance#5:

```solidity
File: contracts/its/token-manager/TokenManager.sol
28: if (interchainTokenService_ == address(0)) revert TokenLinkerZeroAddress();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L28C9-L28C84

### Instance#6:

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
91: if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L91C9-L91C76

### Instance#7:

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
133: if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L133C9-L133C76

### Instance#8:

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
212: if (expressCaller != address(0)) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L212C9-L212C43

### Instance#9:

```solidity
File: contracts/its/utils/ExpressCallHandler.sol
252: if (expressCaller != address(0)) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol#L252C9-L252C43

### Instance#10-12:

```solidity
File: contracts/its/utils/StandardizedTokenDeployer.sol
31: if (deployer_ == address(0) || implementationLockUnlockAddress_ == address(0) || implementationMintBurnAddress_ == address(0))
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/StandardizedTokenDeployer.sol#L31C9-L31C135

### Instance#13:

```solidity
File: contracts/its/utils/TokenManagerDeployer.sol
23:  if (deployer_ == address(0)) revert AddressZero();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/TokenManagerDeployer.sol#L23C8-L23C59

### Instance#14-17:
Findings are marked as <=FOUND

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
92:if (
93:            remoteAddressValidator_ == address(0) ||//<=FOUND
94:            gasService_ == address(0) ||//<=FOUND
95:            tokenManagerDeployer_ == address(0) ||//<=FOUND
96:            standardizedTokenDeployer_ == address(0)//<=FOUND
97:        ) revert ZeroAddress();
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L92C9-L97C32

### Instance#18:

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
609: if (expressCaller == address(0)) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L609C8-L609C43

### Instance#19:

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol
652:if (expressCaller != address(0)) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L652C13-L652C47

