## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#low1-dangerous-use-of-the-burn-function) | Dangerous use of the `burn` function | 1 |
| [LOW&#x2011;2](#low2-decimals-not-part-of-erc20-standard) | `decimals()` not part of ERC20 standard | 1 |
| [LOW&#x2011;3](#low3-remove-unused-code) | Remove unused code | 1 |
| [LOW&#x2011;4](#low10-unnecessary-low-level-calls-should-be-avoided) | Unnecessary Low level calls should be avoided | 1 |

Total: 4 contexts over 4 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#nc1-avoid-the-use-of-sensitive-terms) | Avoid the use of sensitive terms | 42 |
| [NC&#x2011;2](#nc2-do-not-calculate-constants) | Do not calculate constants | 22 |
| [NC&#x2011;3](#nc3-blocktimestamp-is-already-used-when-emitting-events-no-need-to-input-timestamp) | `block.timestamp` is already used when emitting events, no need to input timestamp | 1 |
| [NC&#x2011;4](#nc4-generate-perfect-code-headers-for-better-solidity-code-layout-and-readability) | Generate perfect code headers for better solidity code layout and readability | 31 |
| [NC&#x2011;5](#nc5-override-function-arguments-that-are-unused-should-have-the-variable-name-removed-or-commented-out-to-avoid-compiler-warnings) | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 2 |
| [NC&#x2011;6](#nc3-using-underscore-at-the-end-of-variable-name) | Using underscore at the end of variable name | 16 |
| [NC&#x2011;7](#nc15-use-abiencodecall-instead-of-abiencodesignatureabiencodeselector) | Use `abi.encodeCall()` instead of `abi.encodeSignature()`/`abi.encodeSelector()` | 7 |
| [NC&#x2011;8](#nc18-use-named-function-calls) | Use named function calls | 9 |
| [NC&#x2011;9](#nc19-use-smtchecker) | Use SMTChecker | 1 |
| [NC&#x2011;10](#nc10-cast-to-bytes-or-bytes32-for-clearer-semantic-meaning) | Cast to `bytes` or `bytes32` for clearer semantic meaning | 4 |
| [NC&#x2011;11](#nc11-utilizing-delegatecall-within-a-loop) | Utilizing `delegatecall` within a loop | 1 |

Total: 136 contexts over 11 issues

## Low Risk Issues


### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Dangerous use of the `burnToken` function

The function `burnToken` is used by users to burn an option position by minting the required liquidity and unlocking the collateral. As how the function is designed right now in order to do that, the user needs to send his shares to the contract balance. This is simply too risky, as anyone can call the function and basically burn the shares deposited by the users, before they even get the chance to call the function first.

Instead of the need to send the shares to the contract balance, the function can be refactored to check the balance of shares the user posses and to burn them in the moment of execution or on top of that to input a uint value of how many shares the user wants to burn.

#### <ins>Proof Of Concept</ins>


```solidity
File: AxelarGateway.sol


function burnToken(bytes calldata params, bytes32) external onlySelf {
        (string memory symbol, bytes32 salt) = abi.decode(params, (string, bytes32));

        address tokenAddress = tokenAddresses(symbol);

        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

        if (_getTokenType(symbol) == TokenType.External) {
            address depositHandlerAddress = _getCreate2Address(salt, keccak256(abi.encodePacked(type(DepositHandler).creationCode)));

            if (_hasCode(depositHandlerAddress)) return;

            DepositHandler depositHandler = new DepositHandler{ salt: salt }();

            (bool success, bytes memory returnData) = depositHandler.execute(
                tokenAddress,
                abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))
            );

            if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

            
            depositHandler.destroy(address(this));
        } else {
            IBurnableMintableCappedERC20(tokenAddress).burn(salt);
        }
    }

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L427







### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> `decimals()` not part of ERC20 standard

`decimals()` is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

#### <ins>Proof Of Concept</ins>


```solidity
File: InterchainTokenService.sol

737: decimals = token.decimals()
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L737










### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Remove unused code
This code is not used in the main project contract files, remove it or add event-emit
Code that is not in use, suggests that they should not be present and could potentially contain insecure functionalities.

#### <ins>Proof Of Concept</ins>


```solidity
File: InterchainGovernance.sol

function _executeWithToken
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L155








### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Unnecessary Low level calls should be avoided

Avoid making unnecessary low-level calls to the system whenever possible, as they function differently from contract-type calls. For instance:
Using `address.call(abi.encodeWithSelector("fancy(bytes32)", mybytes)` does not confirm whether the target is genuinely a contract, whereas `ContractInterface(address).fancy(mybytes)` does.

Moreover, when invoking functions declared as `view`/`pure`, the Solidity compiler executes a `staticcall`, offering extra security guarantees that are absent in low-level calls. Additionally, return values must be manually decoded when making low-level calls.

Note: If a low-level call is required, consider using `Contract.function.selector` instead of employing a hardcoded ABI string for encoding.

#### <ins>Proof Of Concept</ins>


```solidity
File: AxelarGateway.sol


function execute(bytes calldata input) external override {
        (bytes memory data, bytes memory proof) = abi.decode(input, (bytes, bytes));

        bytes32 messageHash = ECDSA.toEthSignedMessageHash(keccak256(data));

        
        bool allowOperatorshipTransfer = IAxelarAuth(AUTH_MODULE).validateProof(messageHash, proof);

        uint256 chainId;
        bytes32[] memory commandIds;
        string[] memory commands;
        bytes[] memory params;

        (chainId, commandIds, commands, params) = abi.decode(data, (uint256, bytes32[], string[], bytes[]));

        if (chainId != block.chainid) revert InvalidChainId();

        uint256 commandsLength = commandIds.length;

        if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();

        for (uint256 i; i < commandsLength; ++i) {
            bytes32 commandId = commandIds[i];

            if (isCommandExecuted(commandId)) continue; 

            bytes4 commandSelector;
            bytes32 commandHash = keccak256(abi.encodePacked(commands[i]));

            if (commandHash == SELECTOR_DEPLOY_TOKEN) {
                commandSelector = AxelarGateway.deployToken.selector;
            } else if (commandHash == SELECTOR_MINT_TOKEN) {
                commandSelector = AxelarGateway.mintToken.selector;
            } else if (commandHash == SELECTOR_APPROVE_CONTRACT_CALL) {
                commandSelector = AxelarGateway.approveContractCall.selector;
            } else if (commandHash == SELECTOR_APPROVE_CONTRACT_CALL_WITH_MINT) {
                commandSelector = AxelarGateway.approveContractCallWithMint.selector;
            } else if (commandHash == SELECTOR_BURN_TOKEN) {
                commandSelector = AxelarGateway.burnToken.selector;
            } else if (commandHash == SELECTOR_TRANSFER_OPERATORSHIP) {
                if (!allowOperatorshipTransfer) continue;

                allowOperatorshipTransfer = false;
                commandSelector = AxelarGateway.transferOperatorship.selector;
            } else {
                continue; 
            }

            
            _setCommandExecuted(commandId, true);

            (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));

            if (success) emit Executed(commandId);
            else _setCommandExecuted(commandId, false);
        }
    }

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L374



#### <ins>Recommended Mitigation Steps</ins>

When reaching out to a known contract within the system, always opt for typed contract calls (interfaces/contracts) rather than low-level calls.
This approach helps prevent mistakes, potentially unverified return values, and ensures security guarantees.




## Non Critical Issues



### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Avoid the use of sensitive terms

Use <a href="https://www.zdnet.com/article/mysql-drops-master-slave-and-blacklist-whitelist-terminology/">alternative variants</a>, e.g. allowlist/denylist instead of whitelist/blacklist

#### <ins>Proof Of Concept</ins>

```solidity
File: InterchainProposalExecutor.sol

15: * The contract maintains whitelists for proposal senders and proposal callers. Proposal senders
23: // Whitelisted proposal callers. The proposal caller is the contract that calls the `InterchainProposalSender` at the source chain.
24: mapping(string => mapping(address => bool)) public whitelistedCallers;
26: // Whitelisted proposal senders. The proposal sender is the `InterchainProposalSender` contract address at the source chain.
27: mapping(string => mapping(address => bool)) public whitelistedSenders;
34: * @dev Executes the proposal. The source address must be a whitelisted sender.
48: // Check that the source address is whitelisted
49: if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) {
50: revert NotWhitelistedSourceAddress();
56: // Check that the caller is whitelisted
57: if (!whitelistedCallers[sourceChain][interchainProposalCaller]) {
58: revert NotWhitelistedCaller();
87: * @dev Set the proposal caller whitelist status
90: * @param whitelisted The whitelist status
105: * @param whitelisted The whitelist status
92: function setWhitelistedProposalCaller(
95: bool whitelisted
110: bool whitelisted
97: whitelistedCallers[sourceChain][sourceCaller] = whitelisted;
98: emit WhitelistedProposalCallerSet(sourceChain, sourceCaller, whitelisted);
102: * @dev Set the proposal sender whitelist status
107: function setWhitelistedProposalSender(
110: bool whitelisted
112: whitelistedSenders[sourceChain][sourceSender] = whitelisted;
113: emit WhitelistedProposalSenderSet(sourceChain, sourceSender, whitelisted);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L15

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L23

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L26

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L27

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L34

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L48

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L49

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L50

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L56

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L57

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L58

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L87

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L90

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L105

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L92

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L95

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L110

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L97

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L98

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L102

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L107

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L112

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L113



```solidity
File: IInterchainProposalExecutor.sol

6: // An event emitted when the proposal caller is whitelisted
7: event WhitelistedProposalCallerSet(string indexed sourceChain, address indexed sourceCaller, bool whitelisted);
9: // An event emitted when the proposal sender is whitelisted
10: event WhitelistedProposalSenderSet(string indexed sourceChain, address indexed sourceSender, bool whitelisted);
17: // An error emitted when the proposal caller is not whitelisted
18: error NotWhitelistedCaller();
20: // An error emitted when the proposal sender is not whitelisted
21: error NotWhitelistedSourceAddress();
24: * @notice set the whitelisted status of a proposal sender which is the `InterchainProposalSender` contract address on the source chain
27: * @param whitelisted The whitelisted status
39: * @param whitelisted The whitelisted status
29: function setWhitelistedProposalSender(
32: bool whitelisted
44: bool whitelisted
36: * @notice set the whitelisted status of a proposal caller which normally set to the `Timelock` contract address on the source chain
41: function setWhitelistedProposalCaller(
44: bool whitelisted

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L6

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L7

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L9

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L10

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L17

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L18

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L20

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L21

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L24

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L27

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L39

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L29

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L32

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L44

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L36

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol#L41




### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Do not calculate constants

Due to how `constant` variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: AxelarGateway.sol

31: // bytes32 internal constant KEY_ALL_TOKENS_FROZEN = keccak256('all-tokens-frozen');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L31

```solidity
File: AxelarGateway.sol

32: // bytes32 internal constant PREFIX_TOKEN_FROZEN = keccak256('token-frozen');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L32

```solidity
File: AxelarGateway.sol

44: bytes32 internal constant PREFIX_COMMAND_EXECUTED = keccak256('command-executed');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L44

```solidity
File: AxelarGateway.sol

45: bytes32 internal constant PREFIX_TOKEN_ADDRESS = keccak256('token-address');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L45

```solidity
File: AxelarGateway.sol

46: bytes32 internal constant PREFIX_TOKEN_TYPE = keccak256('token-type');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L46

```solidity
File: AxelarGateway.sol

47: bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED = keccak256('contract-call-approved');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L47

```solidity
File: AxelarGateway.sol

48: bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED_WITH_MINT = keccak256('contract-call-approved-with-mint');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L48

```solidity
File: AxelarGateway.sol

49: bytes32 internal constant PREFIX_TOKEN_MINT_LIMIT = keccak256('token-mint-limit');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L49

```solidity
File: AxelarGateway.sol

50: bytes32 internal constant PREFIX_TOKEN_MINT_AMOUNT = keccak256('token-mint-amount');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L50

```solidity
File: FinalProxy.sol

18: bytes32 internal constant FINAL_IMPLEMENTATION_SALT = keccak256('final-implementation');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L18

```solidity
File: TimeLock.sol

13: bytes32 internal constant PREFIX_TIME_LOCK = keccak256('time-lock');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L13

```solidity
File: InterchainTokenService.sol

62: bytes32 internal constant PREFIX_CUSTOM_TOKEN_ID = keccak256('its-custom-token-id');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L62

```solidity
File: InterchainTokenService.sol

63: bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_ID = keccak256('its-standardized-token-id');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L63

```solidity
File: InterchainTokenService.sol

64: bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_SALT = keccak256('its-standardized-token-salt');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L64

```solidity
File: InterchainTokenService.sol

71: bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L71

```solidity
File: InterchainTokenServiceProxy.sol

12: bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/InterchainTokenServiceProxy.sol#L12

```solidity
File: RemoteAddressValidatorProxy.sol

12: bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/RemoteAddressValidatorProxy.sol#L12

```solidity
File: StandardizedTokenProxy.sol

14: bytes32 private constant CONTRACT_ID = keccak256('standardized-token');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/StandardizedTokenProxy.sol#L14

```solidity
File: RemoteAddressValidator.sol

21: bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L21

```solidity
File: StandardizedToken.sol

27: bytes32 private constant CONTRACT_ID = keccak256('standardized-token');
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L27

```solidity
File: FlowLimit.sol

15: uint256 internal constant PREFIX_FLOW_OUT_AMOUNT = uint256(keccak256('prefix-flow-out-amount'));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L15

```solidity
File: FlowLimit.sol

16: uint256 internal constant PREFIX_FLOW_IN_AMOUNT = uint256(keccak256('prefix-flow-in-amount'));
```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L16



</details>




### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> block.timestamp is already used when emitting events, no need to input timestamp

#### <ins>Proof Of Concept</ins>

```solidity
File: InterchainGovernance.sol

78: emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L78





### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Generate perfect code headers for better solidity code layout and readability

It is recommended to use pre-made headers for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: AxelarGateway.sol

7: AxelarGateway.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L7

```solidity
File: MultisigBase.sol

5: MultisigBase.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L5

```solidity
File: AxelarServiceGovernance.sol

5: AxelarServiceGovernance.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L5

```solidity
File: InterchainGovernance.sol

7: InterchainGovernance.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L7

```solidity
File: Multisig.sol

5: Multisig.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/Multisig.sol#L5

```solidity
File: Caller.sol

5: Caller.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L5

```solidity
File: Upgradable.sol

5: Upgradable.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol#L5

```solidity
File: FinalProxy.sol

6: FinalProxy.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L6

```solidity
File: InitProxy.sol

5: InitProxy.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L5

```solidity
File: Proxy.sol

5: Proxy.sol
7: Proxy.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol#L5

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol#L7



```solidity
File: Upgradable.sol

5: Upgradable.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L5

```solidity
File: TimeLock.sol

5: TimeLock.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol#L5

```solidity
File: InterchainProposalExecutor.sol

7: InterchainProposalExecutor.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L7

```solidity
File: InterchainProposalSender.sol

7: InterchainProposalSender.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L7

```solidity
File: InterchainToken.sol

5: InterchainToken.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L5

```solidity
File: InterchainTokenService.sol

11: InterchainTokenService.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L11

```solidity
File: StandardizedTokenProxy.sol

7: StandardizedTokenProxy.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/StandardizedTokenProxy.sol#L7

```solidity
File: TokenManagerProxy.sol

6: TokenManagerProxy.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L6

```solidity
File: RemoteAddressValidator.sol

4: RemoteAddressValidator.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L4

```solidity
File: Pausable.sol

5: Pausable.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/test/utils/Pausable.sol#L5

```solidity
File: TokenManager.sol

5: TokenManager.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L5

```solidity
File: Distributable.sol

5: Distributable.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Distributable.sol#L5

```solidity
File: ExpressCallHandler.sol

5: ExpressCallHandler.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L5

```solidity
File: FlowLimit.sol

5: FlowLimit.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L5

```solidity
File: Implementation.sol

5: Implementation.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Implementation.sol#L5

```solidity
File: Multicall.sol

5: Multicall.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L5

```solidity
File: Operatable.sol

5: Operatable.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Operatable.sol#L5

```solidity
File: Pausable.sol

5: Pausable.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Pausable.sol#L5

```solidity
File: StandardizedTokenDeployer.sol

7: StandardizedTokenDeployer.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L7

```solidity
File: TokenManagerDeployer.sol

7: TokenManagerDeployer.sol

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L7



</details>






### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

#### <ins>Proof Of Concept</ins>


```solidity
File: AxelarGateway.sol

224: function admins : uint256

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L224

```solidity
File: AxelarGateway.sol

238: function tokenFrozen : string memory

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L238








### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Using underscore at the end of variable name

The use of underscore at the end of the variable name is uncommon and also suggests that the variable name was not completely changed. Consider refactoring `variableName_` to `variableName`.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: AxelarGateway.sol

68: authModule_;
69: tokenDeployerImplementation_;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L68-L69

```solidity
File: InterchainGovernance.sol

39: governanceChain_;
40: governanceAddress_;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L39-L40

```solidity
File: InterchainTokenService.sol

100: tokenManagerDeployer_;
101: standardizedTokenDeployer_;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L100-L101

```solidity
File: TokenManagerProxy.sol

32: implementationType_;
33: tokenId_;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L32-L33

```solidity
File: StandardizedToken.sol

51: distributor_;
52: tokenManager_;
56: tokenManager_;
52: tokenManager_;
56: tokenManager_;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L51

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L52

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L56

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L52

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L56



```solidity
File: TokenManager.sol

63: operator_;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L63

```solidity
File: StandardizedTokenDeployer.sol

34: implementationLockUnlockAddress_;
35: implementationMintBurnAddress_;

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L34-L35



</details>




### <a href="#qa-summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Use `abi.encodeCall()` instead of `abi.encodeSignature()`/`abi.encodeSelector()`

`abi.encodeCall()` has compiler <a href="https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3693">type safety</a>, whereas the other two functions do not

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: AxelarGateway.sol

294: (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(IAxelarGateway.setup.selector, setupParams));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L294

```solidity
File: AxelarGateway.sol

374: (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L374

```solidity
File: AxelarGateway.sol

399: abi.encodeWithSelector(ITokenDeployer.deployToken.selector, name, symbol, decimals, cap, salt)

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L399

```solidity
File: AxelarGateway.sol

443: abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L443

```solidity
File: Upgradable.sol

49: (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol#L49

```solidity
File: InitProxy.sol

58: (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, params));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L58

```solidity
File: Upgradable.sol

61: (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L61



</details>




### <a href="#qa-summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Use named function calls

Code base has an extensive use of named function calls, but it somehow missed one instance where this would be appropriate.

It should use named function calls on function call, as such:
```solidity
    library.exampleFunction{value: _data.amount.value}({
        _id: _data.id,
        _amount: _data.amount.value,
        _token: _data.token,
        _example: "",
        _metadata: _data.metadata
    });
```

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: Caller.sol

18: (bool success, ) = target.call{ value: nativeValue }(callData);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Caller.sol#L18

```solidity
File: InterchainProposalExecutor.sol

76: (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76

```solidity
File: InterchainProposalSender.sol

92: gasService.payNativeGasForContractCall{ value: interchainCall.gas }(
                address(this),
                interchainCall.destinationChain,
                interchainCall.destinationContract,
                payload,
                msg.sender
            );

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L92

```solidity
File: InterchainToken.sol

66: tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);
104: tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L66

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token/InterchainToken.sol#L104



```solidity
File: InterchainTokenService.sol

715: gasService.payNativeGasForContractCall{ value: gasValue }(
                address(this),
                destinationChain,
                destinationAddress,
                payload,
                refundTo
            );

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L715

```solidity
File: TokenManager.sol

92: interchainTokenService.transmitSendToken{ value: msg.value }(
            _getTokenId(),
            sender,
            destinationChain,
            destinationAddress,
            amount,
            metadata
        );
145: interchainTokenService.transmitSendToken{ value: msg.value }(
            _getTokenId(),
            sender,
            destinationChain,
            destinationAddress,
            amount,
            metadata
        );

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L92

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L145



```solidity
File: TokenManager.sol

119: interchainTokenService.transmitSendToken{ value: msg.value }(
            _getTokenId(),
            sender,
            destinationChain,
            destinationAddress,
            amount,
            abi.encodePacked(version, data)
        );

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L119



</details>





### <a href="#qa-summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19




### <a href="#qa-summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Cast to `bytes` or `bytes32` for clearer semantic meaning

Using a <a href="https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739">cast</a> on a single argument, rather than abi.encodePacked() makes the intended operation more clear, leading to less reviewer confusion.

#### <ins>Proof Of Concept</ins>

```solidity
File: AxelarGateway.sol

350: bytes32 commandHash = keccak256(abi.encodePacked(commands[i]));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L350

```solidity
File: AxelarGateway.sol

396: bytes32 salt = keccak256(abi.encodePacked(symbol));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L396

```solidity
File: AxelarGateway.sol

435: address depositHandlerAddress = _getCreate2Address(salt, keccak256(abi.encodePacked(type(DepositHandler).creationCode)));

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L435

```solidity
File: TokenManagerDeployer.sol

39: bytes memory bytecode = abi.encodePacked(type(TokenManagerProxy).creationCode, args);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol#L39





### <a href="#qa-summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Utilizing `delegatecall` within a loop

Using `delegatecall` in a `for` loop can lead to high gas costs, as `delegatecall` is an expensive operation and its costs compound when used in a loop. Additionally, it can pose security risks including reentrancy attacks, as it executes code in the calling contract's context. The function selector collisions can also lead to unpredictable behaviour. To mitigate these risks, control the loop's iterations, apply a reentrancy guard, strictly audit contracts called via `delegatecall`, and consider alternatives like `call` or proxy patterns if the use case allows. Always thoroughly vet contracts involved in `delegatecall` operations. 

#### <ins>Proof Of Concept</ins>


```solidity
File: Multicall.sol

25: (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol#L25






