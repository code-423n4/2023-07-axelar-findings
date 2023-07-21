###  Gas Optimizations List
 --------------------------------------------------
- [x] **Gas 01** → State variables only set in the constructor should be declared immutable
- [x] **Gas 02** → Prioritize function parameter checking
- [x] **Gas 03** → Do not calculate constants variables
- [x] **Gas 04** → Calling msg.sender is cheaper than accessing a memory variable
- [x] **Gas 05** → Superfluous event fields
- [x] **Gas 06** → Use nested if and, avoid multiple check combinations
- [x] **Gas 07** → Using fixed bytes is cheaper than using string



### [G-01] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

There are 4 instances on the subject:

```solidity
contracts\cgp\governance\InterchainGovernance.sol:
 
  21:     string public governanceChain;
  22:     string public governanceAddress;



  32       */
  33:     constructor(
  34:         address gateway,
  35:         string memory governanceChain_,
  36:         string memory governanceAddress_,
  37:         uint256 minimumTimeDelay
  38:     ) AxelarExecutable(gateway) TimeLock(minimumTimeDelay) {
  39:         governanceChain = governanceChain_;
  40:         governanceAddress = governanceAddress_;

```

```diff
contracts\cgp\governance\InterchainGovernance.sol:
 
- 21:     string public governanceChain;
+ 21:     string public immutable governanceChain;
- 22:     string public governanceAddress;
+ 22:     string public immutable governanceAddress;


  32       */
  33:     constructor(
  34:         address gateway,
  35:         string memory governanceChain_,
  36:         string memory governanceAddress_,
  37:         uint256 minimumTimeDelay
  38:     ) AxelarExecutable(gateway) TimeLock(minimumTimeDelay) {
  39:         governanceChain = governanceChain_;
  40:         governanceAddress = governanceAddress_;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L21C29-L21C29-L22



```solidity
contracts\interchain-governance-executor\InterchainProposalSender.sol:


  39:     IAxelarGateway public gateway;
  40:     IAxelarGasService public gasService;
 

  42:     constructor(address _gateway, address _gasService) {
  43:         gateway = IAxelarGateway(_gateway);
  44:         gasService = IAxelarGasService(_gasService);
  45:     }

```


```diff
contracts\interchain-governance-executor\InterchainProposalSender.sol:

- 39:     IAxelarGateway public gateway;
+ 39:     IAxelarGateway public immutable gateway;
- 40:     IAxelarGasService public gasService;
+ 40:     IAxelarGasService public immutable gasService;


  42:     constructor(address _gateway, address _gasService) {
  43:         gateway = IAxelarGateway(_gateway);
  44:         gasService = IAxelarGasService(_gasService);
  45:     }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L39-L40


### [G-02] Prioritize function parameter checking

When there is a check for a function parameter, performing this check first will save gas. In case the condition is not fulfilled, gas will not be used in vain, as the transaction will be reverted.


```solidity
contracts\gmp-sdk\deploy\Create3.sol:

  50       */
  51:     function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
  52:         deployed = deployedAddress(address(this), salt);
  53: 
  54:         if (deployed.isContract()) revert AlreadyDeployed();
  55:         if (bytecode.length == 0) revert EmptyBytecode();
  56: 
  57:         // Deploy using create2
  58:         CreateDeployer deployer = new CreateDeployer{ salt: salt }();
  59: 
  60:         if (address(deployer) == address(0)) revert DeployFailed();
  61: 
  62:         deployer.deploy(bytecode);
  63:     }
  64  

```

```diff
contracts\gmp-sdk\deploy\Create3.sol:

  50       */
  51:     function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
+ 55:         if (bytecode.length == 0) revert EmptyBytecode();
  52:         deployed = deployedAddress(address(this), salt);
  53: 
  54:         if (deployed.isContract()) revert AlreadyDeployed();
- 55:         if (bytecode.length == 0) revert EmptyBytecode();
  56: 
  57:         // Deploy using create2
  58:         CreateDeployer deployer = new CreateDeployer{ salt: salt }();
  59: 
  60:         if (address(deployer) == address(0)) revert DeployFailed();
  61: 
  62:         deployer.deploy(bytecode);
  63:     }
  64  

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L55


### [G-03] Do not calculate constants variables

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

There are 2 instances on the subject:

**Deploy gas saved: ~1.2k** 

```solidity
contracts\its\utils\FlowLimit.sol:

  15:     uint256 internal constant PREFIX_FLOW_OUT_AMOUNT = uint256(keccak256('prefix-flow-out-amount'));

  16:     uint256 internal constant PREFIX_FLOW_IN_AMOUNT = uint256(keccak256('prefix-flow-in-amount'));

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol#L15-L16

```diff
contracts\its\utils\FlowLimit.sol:


- 15:     uint256 internal constant PREFIX_FLOW_OUT_AMOUNT = uint256(keccak256('prefix-flow-out-amount'));
+         // uint256(keccak256('prefix-flow-out-amount'));
+ 15:     uint256 internal constant PREFIX_FLOW_OUT_AMOUNT = 50253035344105178570952279334888603022121336219150220430721657185541570781535;
  

- 16:     uint256 internal constant PREFIX_FLOW_IN_AMOUNT = uint256(keccak256('prefix-flow-in-amount'));
+         // uint256(keccak256('prefix-flow-in-amount'));
+ 16:     uint256 internal constant PREFIX_FLOW_IN_AMOUNT = 13176762390203660117732781612158659424434310574419307873666982947178483190627;

```

### [G-04] Calling msg.sender is cheaper than accessing a memory variable

It is slightly more gas-optimized to call msg.sender directly instead of caching it.

There are 7 instances on the subject:

<details><summary>Click to show findings</summary>

```solidity
contracts\its\interchain-token\InterchainToken.sol:

  43:     function interchainTransfer(
  44:         string calldata destinationChain,
  45:         bytes calldata recipient,
  46:         uint256 amount,
  47:         bytes calldata metadata
  48:     ) external payable {
  49:         address sender = msg.sender;
  50:         ITokenManager tokenManager = getTokenManager();
  51:         /**
  52:          * @dev if you know the value of `tokenManagerRequiresApproval()` you can just skip the if statement and just do nothing or _approve.
  53:          */
  54:         if (tokenManagerRequiresApproval()) {
  55:             uint256 allowance_ = allowance[sender][address(tokenManager)];
  56:             if (allowance_ != type(uint256).max) {
  57:                 if (allowance_ > type(uint256).max - amount) {
  58:                     allowance_ = type(uint256).max - amount;
  59:                 }
  60: 
  61:                 _approve(sender, address(tokenManager), allowance_ + amount);
  62:             }
  63:         }
  64: 
  65:         // Metadata semantics are defined by the token service and thus should be passed as-is.
  66:         tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);
  67:     }

```


```diff
contracts\its\interchain-token\InterchainToken.sol:

  43:     function interchainTransfer(
  44:         string calldata destinationChain,
  45:         bytes calldata recipient,
  46:         uint256 amount,
  47:         bytes calldata metadata
  48:     ) external payable {
- 49:         address sender = msg.sender;
  50:         ITokenManager tokenManager = getTokenManager();
  51:         /**
  52:          * @dev if you know the value of `tokenManagerRequiresApproval()` you can just skip the if statement and just do nothing or _approve.
  53:          */
  54:         if (tokenManagerRequiresApproval()) {
- 55:             uint256 allowance_ = allowance[sender][address(tokenManager)];
+ 55:             uint256 allowance_ = allowance[msg.sender][address(tokenManager)];
  56:             if (allowance_ != type(uint256).max) {
  57:                 if (allowance_ > type(uint256).max - amount) {
  58:                     allowance_ = type(uint256).max - amount;
  59:                 }
  60: 
- 61:                 _approve(sender, address(tokenManager), allowance_ + amount);
+ 61:                 _approve(msg.sender, address(tokenManager), allowance_ + amount);
  62:             }
  63:         }
  64: 
  65:         // Metadata semantics are defined by the token service and thus should be passed as-is.
- 66:         tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);
+ 66:         tokenManager.transmitInterchainTransfer{ value: msg.value }(msg.sender, destinationChain, recipient, amount, metadata);
  67:     }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L43-L67



```solidity
contracts\its\interchain-token-service\InterchainTokenService.sol:

  343:     function deployCustomTokenManager(
  344:         bytes32 salt,
  345:         TokenManagerType tokenManagerType,
  346:         bytes memory params
  347:     ) public payable notPaused returns (bytes32 tokenId) {
  348:         address deployer_ = msg.sender;
  349:         tokenId = getCustomTokenId(deployer_, salt);
  350:         _deployTokenManager(tokenId, tokenManagerType, params);
  351:         emit CustomTokenIdClaimed(tokenId, deployer_, salt);
  352:     }

```


```diff
contracts\its\interchain-token-service\InterchainTokenService.sol:

  343:     function deployCustomTokenManager(
  344:         bytes32 salt,
  345:         TokenManagerType tokenManagerType,
  346:         bytes memory params
  347:     ) public payable notPaused returns (bytes32 tokenId) {
- 348:         address deployer_ = msg.sender;
- 349:         tokenId = getCustomTokenId(deployer_, salt);
+ 349:         tokenId = getCustomTokenId(msg.sender, salt);
  350:         _deployTokenManager(tokenId, tokenManagerType, params);
- 351:         emit CustomTokenIdClaimed(tokenId, deployer_, salt);
+ 351:         emit CustomTokenIdClaimed(tokenId, msg.sender, salt);
  352:     }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L343-L352


```solidity
contracts\its\interchain-token-service\InterchainTokenService.sol:

  365:     function deployRemoteCustomTokenManager(
  366:         bytes32 salt,
  367:         string calldata destinationChain,
  368:         TokenManagerType tokenManagerType,
  369:         bytes calldata params,
  370:         uint256 gasValue
  371:     ) external payable notPaused returns (bytes32 tokenId) {
  372:         address deployer_ = msg.sender;
  373:         tokenId = getCustomTokenId(deployer_, salt);
  374:         _deployRemoteTokenManager(tokenId, destinationChain, gasValue, tokenManagerType, params);
  375:         emit CustomTokenIdClaimed(tokenId, deployer_, salt);
  376:     }

```

```diff
contracts\its\interchain-token-service\InterchainTokenService.sol:

  365:     function deployRemoteCustomTokenManager(
  366:         bytes32 salt,
  367:         string calldata destinationChain,
  368:         TokenManagerType tokenManagerType,
  369:         bytes calldata params,
  370:         uint256 gasValue
  371:     ) external payable notPaused returns (bytes32 tokenId) {
- 372:         address deployer_ = msg.sender;
- 373:         tokenId = getCustomTokenId(deployer_, salt);
+ 373:         tokenId = getCustomTokenId(deployer_, salt);  
  374:         _deployRemoteTokenManager(tokenId, destinationChain, gasValue, tokenManagerType, params);
- 375:         emit CustomTokenIdClaimed(tokenId, deployer_, salt);
+ 375:         emit CustomTokenIdClaimed(tokenId, deployer_, salt);
  376:     }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L365-L376



```solidity
contracts\its\interchain-token-service\InterchainTokenService.sol:

  439:     function expressReceiveToken(
  440:         bytes32 tokenId,
  441:         address destinationAddress,
  442:         uint256 amount,
  443:         bytes32 commandId
  444:     ) external {
  445:         if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);
  446: 
  447:         address caller = msg.sender;
  448:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
  449:         IERC20 token = IERC20(tokenManager.tokenAddress());
  450: 
  451:         SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
  452: 
  453:         _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, caller);
  454:     }

```


```diff
contracts\its\interchain-token-service\InterchainTokenService.sol:

  439:     function expressReceiveToken(
  440:         bytes32 tokenId,
  441:         address destinationAddress,
  442:         uint256 amount,
  443:         bytes32 commandId
  444:     ) external {
  445:         if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);
  446: 
- 447:         address caller = msg.sender;
  448:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
  449:         IERC20 token = IERC20(tokenManager.tokenAddress());
  450: 
- 451:         SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
+ 451:         SafeTokenTransferFrom.safeTransferFrom(token, msg.sender, destinationAddress, amount);
  452: 
- 453:         _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, caller);
+ 453:         _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, msg.sender);
  454:     }

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L439-L454



```solidity
contracts\its\interchain-token-service\InterchainTokenService.sol:

  467:     function expressReceiveTokenWithData(
  468:         bytes32 tokenId,
  469:         string memory sourceChain,
  470:         bytes memory sourceAddress,
  471:         address destinationAddress,
  472:         uint256 amount,
  473:         bytes calldata data,
  474:         bytes32 commandId
  475:     ) external {
  476:         if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);
  477: 
  478:         address caller = msg.sender;
  479:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
  480:         IERC20 token = IERC20(tokenManager.tokenAddress());
  481: 
  482:         SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
  483: 
  484:         _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount);
  485: 
  486:         _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller);
  487:     }

```


```diff
contracts\its\interchain-token-service\InterchainTokenService.sol:

  467:     function expressReceiveTokenWithData(
  468:         bytes32 tokenId,
  469:         string memory sourceChain,
  470:         bytes memory sourceAddress,
  471:         address destinationAddress,
  472:         uint256 amount,
  473:         bytes calldata data,
  474:         bytes32 commandId
  475:     ) external {
  476:         if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);
  477: 
- 478:         address caller = msg.sender;
  479:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
  480:         IERC20 token = IERC20(tokenManager.tokenAddress());
  481: 
- 482:         SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);
+ 482:         SafeTokenTransferFrom.safeTransferFrom(token, msg.sender, destinationAddress, amount);
  483: 
  484:         _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount);
  485: 
- 486:         _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller);
+ 486:         _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, msg.sender);  
  487:     }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L467-L487

```solidity
contracts\its\token-manager\TokenManager.sol:

   83:     function sendToken(
   84:         string calldata destinationChain,
   85:         bytes calldata destinationAddress,
   86:         uint256 amount,
   87:         bytes calldata metadata
   88:     ) external payable virtual {
   89:         address sender = msg.sender;
   90:         amount = _takeToken(sender, amount);
   91:         _addFlowOut(amount);
   92:         interchainTokenService.transmitSendToken{ value: msg.value }(
   93:             _getTokenId(),
   94:             sender,
   95:             destinationChain,
   96:             destinationAddress,
   97:             amount,
   98:             metadata
   99:         );
  100:     }

```

```diff
contracts\its\token-manager\TokenManager.sol:

   83:     function sendToken(
   84:         string calldata destinationChain,
   85:         bytes calldata destinationAddress,
   86:         uint256 amount,
   87:         bytes calldata metadata
   88:     ) external payable virtual {
-  89:         address sender = msg.sender;
-  90:         amount = _takeToken(sender, amount);
+  90:         amount = _takeToken(msg.sender, amount);   
   91:         _addFlowOut(amount);
   92:         interchainTokenService.transmitSendToken{ value: msg.value }(
   93:             _getTokenId(),
-  94:             sender,
+  94:             msg.sender,   
   95:             destinationChain,
   96:             destinationAddress,
   97:             amount,
   98:             metadata
   99:         );
  100:     }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L83-L100

```solidity
contracts\its\token-manager\TokenManager.sol:

  109:     function callContractWithInterchainToken(
  110:         string calldata destinationChain,
  111:         bytes calldata destinationAddress,
  112:         uint256 amount,
  113:         bytes calldata data
  114:     ) external payable virtual {
  115:         address sender = msg.sender;
  116:         amount = _takeToken(sender, amount);
  117:         _addFlowOut(amount);
  118:         uint32 version = 0;
  119:         interchainTokenService.transmitSendToken{ value: msg.value }(
  120:             _getTokenId(),
  121:             sender,
  122:             destinationChain,
  123:             destinationAddress,
  124:             amount,
  125:             abi.encodePacked(version, data)
  126:         );
  127:     }

```

```diff
contracts\its\token-manager\TokenManager.sol:

  109:     function callContractWithInterchainToken(
  110:         string calldata destinationChain,
  111:         bytes calldata destinationAddress,
  112:         uint256 amount,
  113:         bytes calldata data
  114:     ) external payable virtual {
- 115:         address sender = msg.sender;
- 116:         amount = _takeToken(sender, amount);
+ 116:         amount = _takeToken(msg.sender, amount);  
  117:         _addFlowOut(amount);
  118:         uint32 version = 0;
  119:         interchainTokenService.transmitSendToken{ value: msg.value }(
  120:             _getTokenId(),
- 121:             sender,
+ 121:             msg.sender,  
  122:             destinationChain,
  123:             destinationAddress,
  124:             amount,
  125:             abi.encodePacked(version, data)
  126:         );
  127:     }

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L109-L127


### [G-05] Superfluous event fields

``block.timestamp`` and ``block.number`` are added to event information by default so adding them manually wastes gas.


```solidity
contracts\cgp\interfaces\IInterchainGovernance.sol:

  20:     event ProposalExecuted(bytes32 indexed proposalHash, address indexed target, bytes callData, uint256 value, uint256 indexed timestamp);

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IInterchainGovernance.sol#L20



### [G-06] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.


There are 6 instances on the subject:


```solidity
contracts\cgp\AxelarGateway.sol:

   88:         if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();
  
  446:             if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);
  
  635:         if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L88
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L446
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L635



```solidity
contracts\gmp-sdk\upgradable\InitProxy.sol:

  50:         if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L50



```solidity
contracts\gmp-sdk\upgradable\Proxy.sol:

  33:         if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Proxy.sol#L33



```solidity
contracts\its\remote-address-validator\RemoteAddressValidator.sol:

  58:             if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L58


### [G-07] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.


There are 2 instances on the subject:

```solidity
contracts\cgp\governance\InterchainGovernance.sol:

  21:     string public governanceChain;

  22:     string public governanceAddress;

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L21-L22