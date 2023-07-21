## Non Critical Issues

|       | Issue                                                                                               |
| ----- | :-------------------------------------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                                |
| NC-2  | Avoid passing as reference the `chainid`                                                            |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                                            |
| NC-4  | COMMENTED CODE                                                                                      |
| NC-5  | INCONSISTENT SOLIDITY VERSIONS                                                                      |
| NC-6  | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT            |
| NC-7  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                                       |
| NC-8  | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                                                         |
| NC-9  | NO SAME VALUE INPUT CONTROL                                                                         |
| NC-10 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                                                  |
| NC-11 | Return values of `_approve()` not checked                                                           |
| NC-12 | LINES ARE TOO LONG                                                                                  |
| NC-13 | Constants should be defined rather than using magic numbers                                         |
| NC-14 | IMPLEMENT SOME TYPE OF VERSION COUNTER THAT WILL BE INCREMENTED AUTOMATICALLY FOR CONTRACT UPGRADES |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

266:     function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {

627:     function _setTokenMintLimit(string memory symbol, uint256 limit) internal {

633:     function _setTokenMintAmount(string memory symbol, uint256 amount) internal {

640:     function _setTokenType(string memory symbol, TokenType tokenType) internal {

644:     function _setTokenAddress(string memory symbol, address tokenAddress) internal {

648:     function _setCommandExecuted(bytes32 commandId, bool executed) internal {

652:     function _setContractCallApproved(

662:     function _setContractCallApprovedWithMint(

677:     function _setImplementation(address newImplementation) internal {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

92:     function setWhitelistedProposalCaller(

107:     function setWhitelistedProposalSender(

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

29:     function setWhitelistedProposalSender(

41:     function setWhitelistedProposalCaller(

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

534:     function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {

```

```solidity
File: /contracts/its/interfaces/IDistributable.sol

21:     function setDistributor(address distributor) external;

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

286:     function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external;

313:     function setPaused(bool paused) external;

```

```solidity
File: /contracts/its/interfaces/IOperatable.sol

21:     function setOperator(address operator_) external;

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

87:     function setFlowLimit(uint256 flowLimit) external;

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

171:     function setFlowLimit(uint256 flowLimit) external onlyOperator {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

38:     function _setTokenAddress(address tokenAddress_) internal {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

47:     function _setLiquidityPool(address liquidityPool_) internal {

67:     function setLiquidityPool(address newLiquidityPool) external onlyOperator {

```

```solidity
File: /contracts/its/utils/Distributable.sol

39:     function _setDistributor(address distributor_) internal {

51:     function setDistributor(address distr) external onlyDistributor {

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

79:     function _setExpressReceiveToken(

110:     function _setExpressReceiveTokenWithData(

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

34:     function _setFlowLimit(uint256 flowLimit) internal {

```

```solidity
File: /contracts/its/utils/Operatable.sol

39:     function _setOperator(address operator_) internal {

51:     function setOperator(address operator_) external onlyOperator {

```

```solidity
File: /contracts/its/utils/Pausable.sol

41:     function _setPaused(bool paused) internal {

```

### [NC-2] Avoid passing as reference the `chainid`

#### Description:

Instead of passing chainid as reference you could get current `chainid` using `block.chainid`

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

338:         if (chainId != block.chainid) revert InvalidChainId();

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] COMMENTED CODE

#### **Proof Of Concept**

```solidity
File: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

72:                             keccak256(bytecode) // init code hash

```

### [NC-5] INCONSISTENT SOLIDITY VERSIONS

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

3: pragma solidity ^0.8.9;

```

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/governance/AxelarServiceGovernance.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/governance/Multisig.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/IAxelarServiceGovernance.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/ICaller.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/IMultisig.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/IMultisigBase.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/util/Caller.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3Deployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IFinalProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IInitProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/ITimeLock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IUpgradable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/FinalProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/FixedProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/InitProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/Proxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/lib/InterchainCalls.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interchain-token/InterchainToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IDistributable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IExpressCallHandler.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IFlowLimit.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IImplementation.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IInterchainToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IMulticall.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IOperatable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IPausable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IRemoteAddressValidator.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IStandardizedToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManagerDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManagerProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManagerType.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/libraries/AddressBytesUtils.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/InterchainTokenServiceProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/RemoteAddressValidatorProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/StandardizedTokenProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/TokenManagerProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenLockUnlock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenMintBurn.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerMintBurn.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Distributable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Implementation.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Multicall.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Operatable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Pausable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/StandardizedTokenDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/TokenManagerDeployer.sol

3: pragma solidity ^0.8.0;

```

### [NC-6] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### Description:

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

WConstants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

44:     bytes32 internal constant PREFIX_COMMAND_EXECUTED = keccak256('command-executed');

45:     bytes32 internal constant PREFIX_TOKEN_ADDRESS = keccak256('token-address');

46:     bytes32 internal constant PREFIX_TOKEN_TYPE = keccak256('token-type');

47:     bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED = keccak256('contract-call-approved');

48:     bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED_WITH_MINT = keccak256('contract-call-approved-with-mint');

49:     bytes32 internal constant PREFIX_TOKEN_MINT_LIMIT = keccak256('token-mint-limit');

50:     bytes32 internal constant PREFIX_TOKEN_MINT_AMOUNT = keccak256('token-mint-amount');

52:     bytes32 internal constant SELECTOR_BURN_TOKEN = keccak256('burnToken');

53:     bytes32 internal constant SELECTOR_DEPLOY_TOKEN = keccak256('deployToken');

54:     bytes32 internal constant SELECTOR_MINT_TOKEN = keccak256('mintToken');

55:     bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL = keccak256('approveContractCall');

56:     bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL_WITH_MINT = keccak256('approveContractCallWithMint');

57:     bytes32 internal constant SELECTOR_TRANSFER_OPERATORSHIP = keccak256('transferOperatorship');

```

```solidity
File: /contracts/gmp-sdk/upgradable/FinalProxy.sol

18:     bytes32 internal constant FINAL_IMPLEMENTATION_SALT = keccak256('final-implementation');

```

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

13:     bytes32 internal constant PREFIX_TIME_LOCK = keccak256('time-lock');

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

62:     bytes32 internal constant PREFIX_CUSTOM_TOKEN_ID = keccak256('its-custom-token-id');

63:     bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_ID = keccak256('its-standardized-token-id');

64:     bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_SALT = keccak256('its-standardized-token-salt');

71:     bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

```

```solidity
File: /contracts/its/proxies/InterchainTokenServiceProxy.sol

12:     bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

```

```solidity
File: /contracts/its/proxies/RemoteAddressValidatorProxy.sol

12:     bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');

```

```solidity
File: /contracts/its/proxies/StandardizedTokenProxy.sol

14:     bytes32 private constant CONTRACT_ID = keccak256('standardized-token');

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

21:     bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

27:     bytes32 private constant CONTRACT_ID = keccak256('standardized-token');

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

15:     uint256 internal constant PREFIX_FLOW_OUT_AMOUNT = uint256(keccak256('prefix-flow-out-amount'));

16:     uint256 internal constant PREFIX_FLOW_IN_AMOUNT = uint256(keccak256('prefix-flow-in-amount'));

```

### [NC-7] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

3: pragma solidity ^0.8.9;

```

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/governance/AxelarServiceGovernance.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/governance/Multisig.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/IAxelarServiceGovernance.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/ICaller.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/IMultisig.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/interfaces/IMultisigBase.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/cgp/util/Caller.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3Deployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IFinalProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IInitProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/ITimeLock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/interfaces/IUpgradable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/FinalProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/FixedProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/InitProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/Proxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/interchain-governance-executor/lib/InterchainCalls.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interchain-token/InterchainToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IDistributable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IExpressCallHandler.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IFlowLimit.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IImplementation.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IInterchainToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IMulticall.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IOperatable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IPausable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IRemoteAddressValidator.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IStandardizedToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManagerDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManagerProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/interfaces/ITokenManagerType.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/libraries/AddressBytesUtils.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/InterchainTokenServiceProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/RemoteAddressValidatorProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/StandardizedTokenProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/proxies/TokenManagerProxy.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenLockUnlock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenMintBurn.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerMintBurn.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Distributable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Implementation.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Multicall.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Operatable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/Pausable.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/StandardizedTokenDeployer.sol

3: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/its/utils/TokenManagerDeployer.sol

3: pragma solidity ^0.8.0;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-8] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

266:     function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

92:     function setWhitelistedProposalCaller(

107:     function setWhitelistedProposalSender(

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

29:     function setWhitelistedProposalSender(

41:     function setWhitelistedProposalCaller(

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

534:     function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {

547:     function setPaused(bool paused) external onlyOwner {

```

```solidity
File: /contracts/its/interfaces/IDistributable.sol

21:     function setDistributor(address distributor) external;

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

286:     function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external;

313:     function setPaused(bool paused) external;

```

```solidity
File: /contracts/its/interfaces/IOperatable.sol

21:     function setOperator(address operator_) external;

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

87:     function setFlowLimit(uint256 flowLimit) external;

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

171:     function setFlowLimit(uint256 flowLimit) external onlyOperator {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

67:     function setLiquidityPool(address newLiquidityPool) external onlyOperator {

```

```solidity
File: /contracts/its/utils/Distributable.sol

51:     function setDistributor(address distr) external onlyDistributor {

```

```solidity
File: /contracts/its/utils/Operatable.sol

51:     function setOperator(address operator_) external onlyOperator {

```

### [NC-9] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

29:         interchainTokenServiceAddress = _interchainTokenServiceAddress;

```

### [NC-10] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: hardhat.config.ts

solidity: {
        version: '0.8.9',
        settings: {
            evmVersion: process.env.EVM_VERSION || 'london',
            optimizer: {
                enabled: true,
                runs: 1000,
```

### [NC-11] Return values of `_approve()` not checked

#### Description:

Not all IERC20 implementations `revert()` when there's a failure in `_approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

#### **Proof Of Concept**

```solidity
File: /contracts/its/interchain-token/InterchainToken.sol

61:                 _approve(sender, address(tokenManager), allowance_ + amount);

89:             _approve(sender, msg.sender, _allowance - amount);

100:                 _approve(sender, address(tokenManager), allowance_ + amount);

```

### [NC-12] LINES ARE TOO LONG

#### Description:

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

5: import { SafeTokenCall, SafeTokenTransfer, SafeTokenTransferFrom } from '../gmp-sdk/util/SafeTransfer.sol';

10: import { IBurnableMintableCappedERC20 } from './interfaces/IBurnableMintableCappedERC20.sol';

35:     bytes32 internal constant KEY_IMPLEMENTATION = bytes32(0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc);

38:     bytes32 internal constant KEY_GOVERNANCE = bytes32(0xabea6fd3db56a6e6d0242111b43ebb13d1c42709651c032c7894962023a1f909);

41:     bytes32 internal constant KEY_MINT_LIMITER = bytes32(0x627f0c11732837b3240a2de89c0b6343512886dd50978b99c76a68c6416a4d92);

44:     bytes32 internal constant PREFIX_COMMAND_EXECUTED = keccak256('command-executed');

45:     bytes32 internal constant PREFIX_TOKEN_ADDRESS = keccak256('token-address');

47:     bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED = keccak256('contract-call-approved');

48:     bytes32 internal constant PREFIX_CONTRACT_CALL_APPROVED_WITH_MINT = keccak256('contract-call-approved-with-mint');

49:     bytes32 internal constant PREFIX_TOKEN_MINT_LIMIT = keccak256('token-mint-limit');

50:     bytes32 internal constant PREFIX_TOKEN_MINT_AMOUNT = keccak256('token-mint-amount');

55:     bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL = keccak256('approveContractCall');

56:     bytes32 internal constant SELECTOR_APPROVE_CONTRACT_CALL_WITH_MINT = keccak256('approveContractCallWithMint');

57:     bytes32 internal constant SELECTOR_TRANSFER_OPERATORSHIP = keccak256('transferOperatorship');

66:         if (tokenDeployerImplementation_.code.length == 0) revert InvalidTokenDeployer();

88:         if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();

104:         emit TokenSent(msg.sender, destinationChain, destinationAddress, symbol, amount);

112:         emit ContractCall(msg.sender, destinationChain, destinationContractAddress, keccak256(payload), payload);

123:         emit ContractCallWithToken(msg.sender, destinationChain, destinationContractAddress, keccak256(payload), payload, symbol, amount);

133:         return getBool(_getIsContractCallApprovedKey(commandId, sourceChain, sourceAddress, contractAddress, payloadHash));

147:                 _getIsContractCallApprovedWithMintKey(commandId, sourceChain, sourceAddress, contractAddress, payloadHash, symbol, amount)

157:         bytes32 key = _getIsContractCallApprovedKey(commandId, sourceChain, sourceAddress, msg.sender, payloadHash);

170:         bytes32 key = _getIsContractCallApprovedWithMintKey(commandId, sourceChain, sourceAddress, msg.sender, payloadHash, symbol, amount);

199:     function tokenMintLimit(string memory symbol) public view override returns (uint256) {

203:     function tokenMintAmount(string memory symbol) public view override returns (uint256) {

204:         return getUint(_getTokenMintAmountKey(symbol, block.timestamp / 6 hours));

224:     function admins(uint256) external pure override returns (address[] memory) {

232:     function tokenAddresses(string memory symbol) public view override returns (address) {

242:     function isCommandExecuted(bytes32 commandId) public view override returns (bool) {

254:     function transferGovernance(address newGovernance) external override onlyGovernance {

260:     function transferMintLimiter(address newMintLimiter) external override onlyMintLimiter {

266:     function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {

274:             if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

285:         if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();

287:         if (AxelarGateway(newImplementation).contractId() != contractId()) revert InvalidImplementation();

294:             (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(IAxelarGateway.setup.selector, setupParams));

311:         (address governance_, address mintLimiter_, bytes memory newOperatorsData) = abi.decode(params, (address, address, bytes));

324:         (bytes memory data, bytes memory proof) = abi.decode(input, (bytes, bytes));

329:         bool allowOperatorshipTransfer = IAxelarAuth(AUTH_MODULE).validateProof(messageHash, proof);

336:         (chainId, commandIds, commands, params) = abi.decode(data, (uint256, bytes32[], string[], bytes[]));

342:         if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();

347:             if (isCommandExecuted(commandId)) continue; /* Ignore if duplicate commandId received */

358:             } else if (commandHash == SELECTOR_APPROVE_CONTRACT_CALL_WITH_MINT) {

359:                 commandSelector = AxelarGateway.approveContractCallWithMint.selector;

374:             (bool success, ) = address(this).call(abi.encodeWithSelector(commandSelector, params[i], commandId));

386:         (string memory name, string memory symbol, uint8 decimals, uint256 cap, address tokenAddress, uint256 mintLimit) = abi.decode(

392:         if (tokenAddresses(symbol) != address(0)) revert TokenAlreadyExists(symbol);

398:             (bool success, bytes memory data) = TOKEN_DEPLOYER_IMPLEMENTATION.delegatecall(

399:                 abi.encodeWithSelector(ITokenDeployer.deployToken.selector, name, symbol, decimals, cap, salt)

409:             if (tokenAddress.code.length == uint256(0)) revert TokenContractDoesNotExist(tokenAddress);

422:         (string memory symbol, address account, uint256 amount) = abi.decode(params, (string, address, uint256));

428:         (string memory symbol, bytes32 salt) = abi.decode(params, (string, bytes32));

435:             address depositHandlerAddress = _getCreate2Address(salt, keccak256(abi.encodePacked(type(DepositHandler).creationCode)));

443:                 abi.encodeWithSelector(IERC20.transfer.selector, address(this), IERC20(tokenAddress).balanceOf(address(depositHandler)))

446:             if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

455:     function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf {

463:         ) = abi.decode(params, (string, string, address, bytes32, bytes32, uint256));

465:         _setContractCallApproved(commandId, sourceChain, sourceAddress, contractAddress, payloadHash);

466:         emit ContractCallApproved(commandId, sourceChain, sourceAddress, contractAddress, payloadHash, sourceTxHash, sourceEventIndex);

469:     function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf {

479:         ) = abi.decode(params, (string, string, address, bytes32, string, uint256, bytes32, uint256));

481:         _setContractCallApprovedWithMint(commandId, sourceChain, sourceAddress, contractAddress, payloadHash, symbol, amount);

495:     function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {

509:         return codehash != bytes32(0) && codehash != 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;

543:             IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);

545:             IERC20(tokenAddress).safeCall(abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount));

547:             IERC20(tokenAddress).safeTransferFrom(sender, IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0)), amount);

556:     function _getTokenMintLimitKey(string memory symbol) internal pure returns (bytes32) {

560:     function _getTokenMintAmountKey(string memory symbol, uint256 day) internal pure returns (bytes32) {

565:     function _getTokenTypeKey(string memory symbol) internal pure returns (bytes32) {

569:     function _getTokenAddressKey(string memory symbol) internal pure returns (bytes32) {

573:     function _getIsCommandExecutedKey(bytes32 commandId) internal pure returns (bytes32) {

584:         return keccak256(abi.encode(PREFIX_CONTRACT_CALL_APPROVED, commandId, sourceChain, sourceAddress, contractAddress, payloadHash));

615:     function _getCreate2Address(bytes32 salt, bytes32 codeHash) internal view returns (address) {

616:         return address(uint160(uint256(keccak256(abi.encodePacked(bytes1(0xff), address(this), salt, codeHash)))));

619:     function _getTokenType(string memory symbol) internal view returns (TokenType) {

633:     function _setTokenMintAmount(string memory symbol, uint256 amount) internal {

637:         _setUint(_getTokenMintAmountKey(symbol, block.timestamp / 6 hours), amount);

640:     function _setTokenType(string memory symbol, TokenType tokenType) internal {

644:     function _setTokenAddress(string memory symbol, address tokenAddress) internal {

659:         _setBool(_getIsContractCallApprovedKey(commandId, sourceChain, sourceAddress, contractAddress, payloadHash), true);

672:             _getIsContractCallApprovedWithMintKey(commandId, sourceChain, sourceAddress, contractAddress, payloadHash, symbol, amount),

688:         emit MintLimiterTransferred(getAddress(KEY_MINT_LIMITER), newMintLimiter);

```

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

95:     function signerAccounts() external view override returns (address[] memory) {

111:     function hasSignerVoted(address account, bytes32 topic) external view override returns (bool) {

119:     function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {

123:             if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {

142:     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {

149:     function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {

```

```solidity
File: /contracts/cgp/governance/AxelarServiceGovernance.sol

5: import { IAxelarServiceGovernance } from '../interfaces/IAxelarServiceGovernance.sol';

14: contract AxelarServiceGovernance is InterchainGovernance, MultisigBase, IAxelarServiceGovernance {

40:     ) InterchainGovernance(gateway, governanceChain, governanceAddress, minimumTimeDelay) MultisigBase(cosigners, threshold) {}

53:         bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

84:         bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

89:             emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);

91:         } else if (command == ServiceGovernanceCommand.CancelTimeLockProposal) {

94:             emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);

96:         } else if (command == ServiceGovernanceCommand.ApproveMultisigProposal) {

101:         } else if (command == ServiceGovernanceCommand.CancelMultisigApproval) {

104:             emit MultisigCancelled(proposalHash, target, callData, nativeValue);

```

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

5: import { AxelarExecutable } from '../../gmp-sdk/executable/AxelarExecutable.sol';

7: import { IInterchainGovernance } from '../interfaces/IInterchainGovernance.sol';

15: contract InterchainGovernance is AxelarExecutable, TimeLock, Caller, IInterchainGovernance {

57:         return _getTimeLockEta(_getProposalHash(target, callData, nativeValue));

73:         bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

78:         emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);

92:         if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)

95:         (uint256 command, address target, bytes memory callData, uint256 nativeValue, uint256 eta) = abi.decode(

130:             emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);

135:             emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);

```

```solidity
File: /contracts/cgp/governance/Multisig.sol

20:     constructor(address[] memory accounts, uint256 threshold) MultisigBase(accounts, threshold) {}

```

```solidity
File: /contracts/cgp/interfaces/IAxelarServiceGovernance.sol

15:     event MultisigApproved(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);

16:     event MultisigCancelled(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);

17:     event MultisigExecuted(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);

```

```solidity
File: /contracts/cgp/interfaces/IMultisigBase.sol

56:     function hasSignerVoted(address account, bytes32 topic) external view returns (bool);

62:     function getSignerVotesCount(bytes32 topic) external view returns (uint256);

73:     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external;

```

```solidity
File: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

10:     event Deployed(bytes32 indexed bytecodeHash, bytes32 indexed salt, address indexed deployedAddress);

24:     function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {

25:         deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

47:         deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

80:     function _deploy(bytes memory bytecode, bytes32 salt) internal returns (address deployedAddress_) {

85:             deployedAddress_ := create2(0, add(bytecode, 32), mload(bytecode), salt)

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3.sol

41:     bytes32 internal constant DEPLOYER_BYTECODE_HASH = keccak256(type(CreateDeployer).creationCode);

51:     function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {

71:     function deployedAddress(address sender, bytes32 salt) internal pure returns (address deployed) {

72:         address deployer = address(uint160(uint256(keccak256(abi.encodePacked(hex'ff', sender, salt, DEPLOYER_BYTECODE_HASH)))));

74:         deployed = address(uint160(uint256(keccak256(abi.encodePacked(hex'd6_94', deployer, hex'01')))));

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3Deployer.sol

15:     event Deployed(bytes32 indexed bytecodeHash, bytes32 indexed salt, address indexed deployedAddress);

29:     function deploy(bytes calldata bytecode, bytes32 salt) external returns (address deployedAddress_) {

67:     function deployedAddress(address sender, bytes32 salt) external view returns (address) {

```

```solidity
File: /contracts/gmp-sdk/interfaces/IFinalProxy.sol

11:     function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) external returns (address);

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

14:     bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

16:     bytes32 internal constant _OWNER_SLOT = 0x02016836a56b71f0d02689e69e326f4f4c1b9057164ef592671cf0d37c8040c0;

22:     function implementation() public view virtual returns (address implementation_) {

51:             let result := delegatecall(gas(), implementation_, 0, calldatasize(), 0, 0)

```

```solidity
File: /contracts/gmp-sdk/upgradable/FinalProxy.sol

18:     bytes32 internal constant FINAL_IMPLEMENTATION_SALT = keccak256('final-implementation');

37:     function implementation() public view override(BaseProxy, IProxy) returns (address implementation_) {

57:     function _finalImplementation() internal view virtual returns (address implementation_) {

61:         implementation_ = Create3.deployedAddress(address(this), FINAL_IMPLEMENTATION_SALT);

72:     function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {

79:         finalImplementation_ = Create3.deploy(FINAL_IMPLEMENTATION_SALT, bytecode);

81:             (bool success, ) = finalImplementation_.delegatecall(abi.encodeWithSelector(BaseProxy.setup.selector, setupParams));

```

```solidity
File: /contracts/gmp-sdk/upgradable/FixedProxy.sol

43:             let result := delegatecall(gas(), implementation_, 0, calldatasize(), 0, 0)

```

```solidity
File: /contracts/gmp-sdk/upgradable/InitProxy.sol

50:         if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

58:             (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, params));

```

```solidity
File: /contracts/gmp-sdk/upgradable/Proxy.sol

33:         if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();

41:             (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, setupParams));

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

14:     bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

57:         if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();

58:         if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();

61:             (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));

```

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

38:     function getTimeLock(bytes32 hash) external view override returns (uint256) {

50:     function _scheduleTimeLock(bytes32 hash, uint256 eta) internal returns (uint256) {

92:     function _getTimeLockEta(bytes32 hash) internal view returns (uint256 eta) {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

7: import { IInterchainProposalExecutor } from './interfaces/IInterchainProposalExecutor.sol';

22: contract InterchainProposalExecutor is IInterchainProposalExecutor, AxelarExecutable, Ownable {

49:         if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) {

54:         (address interchainProposalCaller, InterchainCalls.Call[] memory calls) = abi.decode(payload, (address, InterchainCalls.Call[]));

64:         _onProposalExecuted(sourceChain, sourceAddress, interchainProposalCaller, payload);

66:         emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));

76:             (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

98:         emit WhitelistedProposalCallerSet(sourceChain, sourceCaller, whitelisted);

113:         emit WhitelistedProposalSenderSet(sourceChain, sourceSender, whitelisted);

178:     function _onTargetExecuted(InterchainCalls.Call memory call, bytes memory result) internal virtual {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

6: import { IAxelarGasService } from '../gmp-sdk/interfaces/IAxelarGasService.sol';

7: import { IInterchainProposalSender } from './interfaces/IInterchainProposalSender.sol';

59:     function sendProposals(InterchainCalls.InterchainCall[] calldata interchainCalls) external payable override {

85:         _sendProposal(InterchainCalls.InterchainCall(destinationChain, destinationContract, msg.value, calls));

88:     function _sendProposal(InterchainCalls.InterchainCall memory interchainCall) internal {

92:             gasService.payNativeGasForContractCall{ value: interchainCall.gas }(

101:         gateway.callContract(interchainCall.destinationChain, interchainCall.destinationContract, payload);

104:     function revertIfInvalidFee(InterchainCalls.InterchainCall[] calldata interchainCalls) private {

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

7:     event WhitelistedProposalCallerSet(string indexed sourceChain, address indexed sourceCaller, bool whitelisted);

10:     event WhitelistedProposalSenderSet(string indexed sourceChain, address indexed sourceSender, bool whitelisted);

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

15:     function sendProposals(InterchainCalls.InterchainCall[] memory calls) external payable;

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

6: import { IAxelarGasService } from '../../gmp-sdk/interfaces/IAxelarGasService.sol';

7: import { AxelarExecutable } from '../../gmp-sdk/executable/AxelarExecutable.sol';

11: import { IInterchainTokenService } from '../interfaces/IInterchainTokenService.sol';

12: import { ITokenManagerDeployer } from '../interfaces/ITokenManagerDeployer.sol';

13: import { IStandardizedTokenDeployer } from '../interfaces/IStandardizedTokenDeployer.sol';

14: import { IRemoteAddressValidator } from '../interfaces/IRemoteAddressValidator.sol';

15: import { IInterchainTokenExpressExecutable } from '../interfaces/IInterchainTokenExpressExecutable.sol';

21: import { StringToBytes32, Bytes32ToString } from '../../gmp-sdk/util/Bytes32String.sol';

62:     bytes32 internal constant PREFIX_CUSTOM_TOKEN_ID = keccak256('its-custom-token-id');

63:     bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_ID = keccak256('its-standardized-token-id');

64:     bytes32 internal constant PREFIX_STANDARDIZED_TOKEN_SALT = keccak256('its-standardized-token-salt');

69:     uint256 private constant SELECTOR_DEPLOY_AND_REGISTER_STANDARDIZED_TOKEN = 4;

71:     bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

98:         remoteAddressValidator = IRemoteAddressValidator(remoteAddressValidator_);

104:         if (tokenManagerImplementations.length != uint256(type(TokenManagerType).max) + 1) revert LengthMismatch();

106:         implementationLockUnlock = _sanitizeTokenManagerImplementation(tokenManagerImplementations, TokenManagerType.LOCK_UNLOCK);

107:         implementationMintBurn = _sanitizeTokenManagerImplementation(tokenManagerImplementations, TokenManagerType.MINT_BURN);

108:         implementationLiquidityPool = _sanitizeTokenManagerImplementation(tokenManagerImplementations, TokenManagerType.LIQUIDITY_POOL);

123:     modifier onlyRemoteService(string calldata sourceChain, string calldata sourceAddress) {

124:         if (!remoteAddressValidator.validateSender(sourceChain, sourceAddress)) revert NotRemoteService();

133:         if (msg.sender != getTokenManagerAddress(tokenId)) revert NotTokenManager();

161:     function getTokenManagerAddress(bytes32 tokenId) public view returns (address tokenManagerAddress) {

170:     function getValidTokenManagerAddress(bytes32 tokenId) public view returns (address tokenManagerAddress) {

172:         if (ITokenManagerProxy(tokenManagerAddress).tokenId() != tokenId) revert TokenManagerDoesNotExist(tokenId);

180:     function getTokenAddress(bytes32 tokenId) external view returns (address tokenAddress) {

191:     function getStandardizedTokenAddress(bytes32 tokenId) public view returns (address tokenAddress) {

202:     function getCanonicalTokenId(address tokenAddress) public view returns (bytes32 tokenId) {

203:         tokenId = keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_ID, chainNameHash, tokenAddress));

213:     function getCustomTokenId(address sender, bytes32 salt) public pure returns (bytes32 tokenId) {

223:     function getImplementation(uint256 tokenManagerType) external view returns (address tokenManagerAddress) {

226:         if (TokenManagerType(tokenManagerType) == TokenManagerType.LOCK_UNLOCK) {

228:         } else if (TokenManagerType(tokenManagerType) == TokenManagerType.MINT_BURN) {

230:         } else if (TokenManagerType(tokenManagerType) == TokenManagerType.LIQUIDITY_POOL) {

241:     function getParamsLockUnlock(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {

251:     function getParamsMintBurn(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {

275:     function getFlowLimit(bytes32 tokenId) external view returns (uint256 flowLimit) {

276:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));

285:     function getFlowOutAmount(bytes32 tokenId) external view returns (uint256 flowOutAmount) {

286:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));

295:     function getFlowInAmount(bytes32 tokenId) external view returns (uint256 flowInAmount) {

296:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));

309:     function registerCanonicalToken(address tokenAddress) external payable notPaused returns (bytes32 tokenId) {

311:         if (gateway.tokenAddresses(tokenSymbol) == tokenAddress) revert GatewayToken();

313:         _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));

332:         if (getCanonicalTokenId(tokenAddress) != tokenId) revert NotCanonicalTokenManager();

333:         (string memory tokenName, string memory tokenSymbol, uint8 tokenDecimals) = _validateToken(tokenAddress);

334:         _deployRemoteStandardizedToken(tokenId, tokenName, tokenSymbol, tokenDecimals, '', '', destinationChain, gasValue);

374:         _deployRemoteTokenManager(tokenId, destinationChain, gasValue, tokenManagerType, params);

397:         _deployStandardizedToken(tokenId, distributor, name, symbol, decimals, mintAmount, msg.sender);

399:         TokenManagerType tokenManagerType = distributor == tokenManagerAddress ? TokenManagerType.MINT_BURN : TokenManagerType.LOCK_UNLOCK;

401:         _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));

428:         _deployRemoteStandardizedToken(tokenId, name, symbol, decimals, distributor, operator, destinationChain, gasValue);

445:         if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

448:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));

451:         SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

453:         _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, caller);

476:         if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

479:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));

482:         SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

484:         _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount);

486:         _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller);

512:             payload = abi.encode(SELECTOR_SEND_TOKEN, tokenId, destinationAddress, amount);

514:             emit TokenSent(tokenId, destinationChain, destinationAddress, amount);

520:         payload = abi.encode(SELECTOR_SEND_TOKEN_WITH_DATA, tokenId, destinationAddress, amount, sourceAddress.toBytes(), metadata);

522:         emit TokenSentWithData(tokenId, destinationChain, destinationAddress, amount, sourceAddress, metadata);

534:     function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {

538:             ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenIds[i]));

559:     function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)

566:         if (ITokenManager(implementation).implementationType() != uint256(tokenManagerType)) revert InvalidTokenManagerImplementation();

579:     ) internal override onlyRemoteService(sourceChain, sourceAddress) notPaused {

587:         } else if (selector == SELECTOR_DEPLOY_AND_REGISTER_STANDARDIZED_TOKEN) {

599:     function _processSendTokenPayload(string calldata sourceChain, bytes calldata payload) internal {

600:         (, bytes32 tokenId, bytes memory destinationAddressBytes, uint256 amount) = abi.decode(payload, (uint256, bytes32, bytes, uint256));

607:         ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));

608:         address expressCaller = _popExpressReceiveToken(tokenId, destinationAddress, amount, commandId);

611:             emit TokenReceived(tokenId, sourceChain, destinationAddress, amount);

622:     function _processSendTokenWithDataPayload(string calldata sourceChain, bytes calldata payload) internal {

635:             (, tokenId, destinationAddressBytes, amount, sourceAddress, data) = abi.decode(

641:         ITokenManager tokenManager = ITokenManager(getTokenManagerAddress(tokenId));

658:         IInterchainTokenExpressExecutable(destinationAddress).executeWithInterchainToken(sourceChain, sourceAddress, data, tokenId, amount);

659:         emit TokenReceivedWithData(tokenId, sourceChain, destinationAddress, amount, sourceAddress, data);

666:     function _processDeployTokenManagerPayload(bytes calldata payload) internal {

667:         (, bytes32 tokenId, TokenManagerType tokenManagerType, bytes memory params) = abi.decode(

678:     function _processDeployStandardizedTokenAndManagerPayload(bytes calldata payload) internal {

687:         ) = abi.decode(payload, (uint256, bytes32, string, string, uint8, bytes, bytes));

690:         address distributor = distributorBytes.length > 0 ? distributorBytes.toAddress() : tokenManagerAddress;

691:         _deployStandardizedToken(tokenId, distributor, name, symbol, decimals, 0, distributor);

692:         TokenManagerType tokenManagerType = distributor == tokenManagerAddress ? TokenManagerType.MINT_BURN : TokenManagerType.LOCK_UNLOCK;

696:             abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)

713:         string memory destinationAddress = remoteAddressValidator.getRemoteAddress(destinationChain);

755:         bytes memory payload = abi.encode(SELECTOR_DEPLOY_TOKEN_MANAGER, tokenId, tokenManagerType, params);

757:         emit RemoteTokenManagerDeploymentInitialized(tokenId, destinationChain, gasValue, tokenManagerType, params);

814:             abi.encodeWithSelector(ITokenManagerDeployer.deployTokenManager.selector, tokenId, tokenManagerType, params)

827:     function _getStandardizedTokenSalt(bytes32 tokenId) internal pure returns (bytes32 salt) {

869:         emit StandardizedTokenDeployed(tokenId, name, symbol, decimals, mintAmount, mintTo);

872:     function _decodeMetadata(bytes calldata metadata) internal pure returns (uint32 version, bytes calldata data) {

888:         IInterchainTokenExpressExecutable(destinationAddress).expressExecuteWithInterchainToken(

```

```solidity
File: /contracts/its/interchain-token/InterchainToken.sol

21:     function getTokenManager() public view virtual returns (ITokenManager tokenManager);

66:         tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

104:         tokenManager.transmitInterchainTransfer{ value: msg.value }(sender, destinationChain, recipient, amount, metadata);

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

6: import { IAxelarExecutable } from '../../gmp-sdk/interfaces/IAxelarExecutable.sol';

14: interface IInterchainTokenService is ITokenManagerType, IExpressCallHandler, IAxelarExecutable, IPausable, IMulticall {

32:     event TokenSent(bytes32 tokenId, string destinationChain, bytes destinationAddress, uint256 indexed amount);

41:     event TokenReceived(bytes32 indexed tokenId, string sourceChain, address indexed destinationAddress, uint256 indexed amount);

67:     event TokenManagerDeployed(bytes32 indexed tokenId, TokenManagerType indexed tokenManagerType, bytes params);

76:     event CustomTokenIdClaimed(bytes32 indexed tokenId, address indexed deployer, bytes32 indexed salt);

82:     function tokenManagerDeployer() external view returns (address tokenManagerDeployerAddress);

88:     function standardizedTokenDeployer() external view returns (address standardizedTokenDeployerAddress);

101:     function getTokenManagerAddress(bytes32 tokenId) external view returns (address tokenManagerAddress);

108:     function getValidTokenManagerAddress(bytes32 tokenId) external view returns (address tokenManagerAddress);

115:     function getTokenAddress(bytes32 tokenId) external view returns (address tokenAddress);

122:     function getStandardizedTokenAddress(bytes32 tokenId) external view returns (address tokenAddress);

129:     function getCanonicalTokenId(address tokenAddress) external view returns (bytes32 tokenId);

137:     function getCustomTokenId(address operator, bytes32 salt) external view returns (bytes32 tokenId);

145:     function getParamsLockUnlock(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

153:     function getParamsMintBurn(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

173:     function registerCanonicalToken(address tokenAddress) external payable returns (bytes32 tokenId);

261:     function getImplementation(uint256 tokenManagerType) external view returns (address tokenManagerAddress);

286:     function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external;

293:     function getFlowLimit(bytes32 tokenId) external view returns (uint256 flowLimit);

300:     function getFlowOutAmount(bytes32 tokenId) external view returns (uint256 flowOutAmount);

307:     function getFlowInAmount(bytes32 tokenId) external view returns (uint256 flowInAmount);

```

```solidity
File: /contracts/its/interfaces/IMulticall.sol

18:     function multicall(bytes[] calldata data) external payable returns (bytes[] memory results);

```

```solidity
File: /contracts/its/interfaces/IRemoteAddressValidator.sol

25:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool);

32:     function addTrustedAddress(string memory sourceChain, string memory sourceAddress) external;

45:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress);

51:     function supportedByGateway(string calldata chainName) external view returns (bool);

63:     function removeGatewaySupportedChains(string[] calldata chainNames) external;

```

```solidity
File: /contracts/its/interfaces/IStandardizedToken.sol

14: interface IStandardizedToken is IInterchainToken, IDistributable, IERC20BurnableMintable {

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

14: interface ITokenManager is ITokenManagerType, IOperatable, IFlowLimit, IImplementation {

81:     function giveToken(address destinationAddress, uint256 amount) external returns (uint256);

```

```solidity
File: /contracts/its/libraries/AddressBytesUtils.sol

17:     function toAddress(bytes memory bytesAddress) internal pure returns (address addr) {

30:     function toBytes(address addr) internal pure returns (bytes memory bytesAddress) {

```

```solidity
File: /contracts/its/proxies/InterchainTokenServiceProxy.sol

12:     bytes32 private constant CONTRACT_ID = keccak256('interchain-token-service');

```

```solidity
File: /contracts/its/proxies/RemoteAddressValidatorProxy.sol

12:     bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');

```

```solidity
File: /contracts/its/proxies/StandardizedTokenProxy.sol

7: import { IStandardizedTokenProxy } from '../interfaces/IStandardizedTokenProxy.sol';

21:     constructor(address implementationAddress, bytes memory params) FixedProxy(implementationAddress) {

22:         if (IStandardizedToken(implementationAddress).contractId() != CONTRACT_ID) revert InvalidImplementation();

24:         (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IStandardizedToken.setup.selector, params));

```

```solidity
File: /contracts/its/proxies/TokenManagerProxy.sol

5: import { IInterchainTokenService } from '../interfaces/IInterchainTokenService.sol';

31:         interchainTokenServiceAddress = IInterchainTokenService(interchainTokenServiceAddress_);

34:         address impl = _getImplementation(IInterchainTokenService(interchainTokenServiceAddress_), implementationType_);

36:         (bool success, ) = impl.delegatecall(abi.encodeWithSelector(TokenManagerProxy.setup.selector, params));

45:         impl = _getImplementation(interchainTokenServiceAddress, implementationType);

54:     function _getImplementation(IInterchainTokenService interchainTokenServiceAddress_, uint256 implementationType_)

59:         impl = interchainTokenServiceAddress_.getImplementation(implementationType_);

78:             let result := delegatecall(gas(), implementaion_, 0, calldatasize(), 0, 0)

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

4: import { IRemoteAddressValidator } from '../interfaces/IRemoteAddressValidator.sol';

21:     bytes32 private constant CONTRACT_ID = keccak256('remote-address-validator');

30:         interchainTokenServiceAddressHash = keccak256(bytes(_lowerCase(interchainTokenServiceAddress.toString())));

41:         (string[] memory trustedChainNames, string[] memory trustedAddresses) = abi.decode(params, (string[], string[]));

54:     function _lowerCase(string memory s) internal pure returns (string memory) {

69:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool) {

83:     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

106:     function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

119:     function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

133:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress) {

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

5: import { IERC20BurnableMintable } from '../interfaces/IERC20BurnableMintable.sol';

19: abstract contract StandardizedToken is InterchainToken, ERC20Permit, Implementation, Distributable {

54:             (tokenManager_, distributor_, tokenName, symbol, decimals) = abi.decode(params, (address, address, string, string, uint8));

63:             (, , , , , mintAmount, mintTo) = abi.decode(params, (address, address, string, string, uint8, uint256, address));

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenLockUnlock.sol

8:     function tokenManagerRequiresApproval() public pure override returns (bool) {

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenMintBurn.sol

8:     function tokenManagerRequiresApproval() public pure override returns (bool) {

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

6: import { IInterchainTokenService } from '../interfaces/IInterchainTokenService.sol';

18: abstract contract TokenManager is ITokenManager, Operatable, FlowLimit, Implementation {

28:         if (interchainTokenService_ == address(0)) revert TokenLinkerZeroAddress();

29:         interchainTokenService = IInterchainTokenService(interchainTokenService_);

161:     function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256) {

182:     function _takeToken(address from, uint256 amount) internal virtual returns (uint256);

191:     function _giveToken(address from, uint256 amount) internal virtual returns (uint256);

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

7: import { IAxelarGateway } from '../../../gmp-sdk/interfaces/IAxelarGateway.sol';

19:     constructor(address interchainTokenService_) TokenManager(interchainTokenService_) {}

22:     uint256 internal constant TOKEN_ADDRESS_SLOT = 0xc4e632779a6a7838736dd7e5e6a0eadf171dd37dfb6230720e265576dfcf42ba;

28:     function tokenAddress() public view override returns (address tokenAddress_) {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

19:     uint256 internal constant LIQUIDITY_POOL_SLOT = 0x8e02741a3381812d092c5689c9fc701c5185c1742fdf7954c4c4472be4cc4807;

26:     constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}

38:         (, address tokenAddress_, address liquidityPool_) = abi.decode(params, (bytes, address, address));

77:     function _takeToken(address from, uint256 amount) internal override returns (uint256) {

82:         SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount);

94:     function _giveToken(address to, uint256 amount) internal override returns (uint256) {

98:         SafeTokenTransferFrom.safeTransferFrom(token, liquidityPool(), to, amount);

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

8: import { SafeTokenTransferFrom, SafeTokenTransfer } from '../../../gmp-sdk/util/SafeTransfer.sol';

22:     constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}

44:     function _takeToken(address from, uint256 amount) internal override returns (uint256) {

48:         SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

60:     function _giveToken(address to, uint256 amount) internal override returns (uint256) {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerMintBurn.sol

6: import { IERC20BurnableMintable } from '../../interfaces/IERC20BurnableMintable.sol';

23:     constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}

45:     function _takeToken(address from, uint256 amount) internal override returns (uint256) {

48:         SafeTokenCall.safeCall(token, abi.encodeWithSelector(IERC20BurnableMintable.burn.selector, from, amount));

59:     function _giveToken(address to, uint256 amount) internal override returns (uint256) {

62:         SafeTokenCall.safeCall(token, abi.encodeWithSelector(IERC20BurnableMintable.mint.selector, to, amount));

```

```solidity
File: /contracts/its/utils/Distributable.sol

15:     uint256 internal constant DISTRIBUTOR_SLOT = 0x71c5a35e45a25c49e8f747acd4bcb869814b3d104c492d2554f4c46e12371f56;

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

14:     uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN = 0x67c7b41c1cb0375e36084c4ec399d005168e83425fa471b9224f6115af865619;

16:     uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN_WITH_DATA = 0x3e607cc12a253b1d9f677a03d298ad869a90a8ba4bd0fb5739e7d79db7cdeaad;

32:         slot = uint256(keccak256(abi.encode(PREFIX_EXPRESS_RECEIVE_TOKEN, tokenId, destinationAddress, amount, commandId)));

86:         uint256 slot = _getExpressReceiveTokenSlot(tokenId, destinationAddress, amount, commandId);

95:         emit ExpressReceive(tokenId, destinationAddress, amount, commandId, expressCaller);

137:         emit ExpressReceiveWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, expressCaller);

154:         uint256 slot = _getExpressReceiveTokenSlot(tokenId, destinationAddress, amount, commandId);

208:         uint256 slot = _getExpressReceiveTokenSlot(tokenId, destinationAddress, amount, commandId);

216:             emit ExpressExecutionFulfilled(tokenId, destinationAddress, amount, commandId, expressCaller);

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

14:     uint256 internal constant FLOW_LIMIT_SLOT = 0x201b7a0b7c19aaddc4ce9579b7df8d2db123805861bc7763627f13e04d8af42f;

15:     uint256 internal constant PREFIX_FLOW_OUT_AMOUNT = uint256(keccak256('prefix-flow-out-amount'));

16:     uint256 internal constant PREFIX_FLOW_IN_AMOUNT = uint256(keccak256('prefix-flow-in-amount'));

46:     function _getFlowOutSlot(uint256 epoch) internal pure returns (uint256 slot) {

55:     function _getFlowInSlot(uint256 epoch) internal pure returns (uint256 slot) {

101:         if (flowToAdd + flowAmount > flowToCompare + flowLimit) revert FlowLimitExceeded();

```

```solidity
File: /contracts/its/utils/Multicall.sol

22:     function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {

25:             (bool success, bytes memory result) = address(this).delegatecall(data[i]);

```

```solidity
File: /contracts/its/utils/Operatable.sol

15:     uint256 internal constant OPERATOR_SLOT = 0xf23ec0bb4210edd5cba85afd05127efcd2fc6a781bfed49188da1081670b22d7;

```

```solidity
File: /contracts/its/utils/Pausable.sol

14:     uint256 internal constant PAUSE_SLOT = 0xee35723ac350a69d2a92d3703f17439cbaadf2f093a21ba5bf5f1a53eb2a14d8;

```

```solidity
File: /contracts/its/utils/StandardizedTokenDeployer.sol

7: import { IStandardizedTokenDeployer } from '../interfaces/IStandardizedTokenDeployer.sol';

31:         if (deployer_ == address(0) || implementationLockUnlockAddress_ == address(0) || implementationMintBurnAddress_ == address(0))

60:         address implementationAddress = distributor == tokenManager ? implementationMintBurnAddress : implementationLockUnlockAddress;

62:             bytes memory params = abi.encode(tokenManager, distributor, name, symbol, decimals, mintAmount, mintTo);

63:             bytecode = abi.encodePacked(type(StandardizedTokenProxy).creationCode, abi.encode(implementationAddress, params));

```

```solidity
File: /contracts/its/utils/TokenManagerDeployer.sol

7: import { ITokenManagerDeployer } from '../interfaces/ITokenManagerDeployer.sol';

38:         bytes memory args = abi.encode(address(this), implementationType, tokenId, params);

39:         bytes memory bytecode = abi.encodePacked(type(TokenManagerProxy).creationCode, args);

41:         if (tokenManagerAddress.code.length == 0) revert TokenManagerDeploymentFailed();

```

### [NC-13] Constants should be defined rather than using magic numbers

#### **Proof Of Concept**

```solidity
File: /contracts/its/libraries/AddressBytesUtils.sol

31:         bytesAddress = new bytes(20);

```

### [NC-14] IMPLEMENT SOME TYPE OF VERSION COUNTER THAT WILL BE INCREMENTED AUTOMATICALLY FOR CONTRACT UPGRADES

#### Description:

I suggest implementing some kind of version counter that will be incremented automatically when you upgrade the contract.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

280:     function upgrade(

```

```solidity
File: /contracts/gmp-sdk/interfaces/IUpgradable.sol

18:     function upgrade(

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

52:     function upgrade(

```

## Low Issues

|      | Issue                                                                          |
| ---- | :----------------------------------------------------------------------------- |
| L-1  | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE                                 |
| L-2  | DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH       |
| L-3  | Initializers could be front-run                                                |
| L-4  | UNIFY RETURN CRITERIA                                                          |
| L-5  | REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR                                     |
| L-6  | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-7  | A SINGLE POINT OF FAILURE                                                      |
| L-8  | Timestamp Dependence                                                           |
| L-9  | UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT                        |
| L-10 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                             |
| L-11 | USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION            |

### [L-1] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

Changing critical addresses in contracts should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1488) and [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2369))

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

644:     function _setTokenAddress(string memory symbol, address tokenAddress) internal {

652:     function _setContractCallApproved(

662:     function _setContractCallApprovedWithMint(

677:     function _setImplementation(address newImplementation) internal {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

92:     function setWhitelistedProposalCaller(

107:     function setWhitelistedProposalSender(

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

38:     function _setTokenAddress(address tokenAddress_) internal {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

47:     function _setLiquidityPool(address liquidityPool_) internal {

67:     function setLiquidityPool(address newLiquidityPool) external onlyOperator {

```

```solidity
File: /contracts/its/utils/Distributable.sol

39:     function _setDistributor(address distributor_) internal {

51:     function setDistributor(address distr) external onlyDistributor {

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

79:     function _setExpressReceiveToken(

110:     function _setExpressReceiveTokenWithData(

```

```solidity
File: /contracts/its/utils/Operatable.sol

39:     function _setOperator(address operator_) internal {

51:     function setOperator(address operator_) external onlyOperator {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-2] DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

35:     bytes32 internal constant KEY_IMPLEMENTATION = bytes32(0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc);

38:     bytes32 internal constant KEY_GOVERNANCE = bytes32(0xabea6fd3db56a6e6d0242111b43ebb13d1c42709651c032c7894962023a1f909);

41:     bytes32 internal constant KEY_MINT_LIMITER = bytes32(0x627f0c11732837b3240a2de89c0b6343512886dd50978b99c76a68c6416a4d92);

509:         return codehash != bytes32(0) && codehash != 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;

616:         return address(uint160(uint256(keccak256(abi.encodePacked(bytes1(0xff), address(this), salt, codeHash)))));

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

14:     bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

16:     bytes32 internal constant _OWNER_SLOT = 0x02016836a56b71f0d02689e69e326f4f4c1b9057164ef592671cf0d37c8040c0;

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

14:     bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

22:     uint256 internal constant TOKEN_ADDRESS_SLOT = 0xc4e632779a6a7838736dd7e5e6a0eadf171dd37dfb6230720e265576dfcf42ba;

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

19:     uint256 internal constant LIQUIDITY_POOL_SLOT = 0x8e02741a3381812d092c5689c9fc701c5185c1742fdf7954c4c4472be4cc4807;

```

```solidity
File: /contracts/its/utils/Distributable.sol

15:     uint256 internal constant DISTRIBUTOR_SLOT = 0x71c5a35e45a25c49e8f747acd4bcb869814b3d104c492d2554f4c46e12371f56;

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

14:     uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN = 0x67c7b41c1cb0375e36084c4ec399d005168e83425fa471b9224f6115af865619;

16:     uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN_WITH_DATA = 0x3e607cc12a253b1d9f677a03d298ad869a90a8ba4bd0fb5739e7d79db7cdeaad;

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

14:     uint256 internal constant FLOW_LIMIT_SLOT = 0x201b7a0b7c19aaddc4ce9579b7df8d2db123805861bc7763627f13e04d8af42f;

```

```solidity
File: /contracts/its/utils/Operatable.sol

15:     uint256 internal constant OPERATOR_SLOT = 0xf23ec0bb4210edd5cba85afd05127efcd2fc6a781bfed49188da1081670b22d7;

```

```solidity
File: /contracts/its/utils/Pausable.sol

14:     uint256 internal constant PAUSE_SLOT = 0xee35723ac350a69d2a92d3703f17439cbaadf2f093a21ba5bf5f1a53eb2a14d8;

```

#### Recommended Mitigation Steps:

Consider using a more generic implementation that can handle different hash functions and lengths and allow the user to choose.

### [L-3] Initializers could be front-run

#### Description:

Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment.

#### **Proof Of Concept**

```solidity
File: /contracts/gmp-sdk/interfaces/IInitProxy.sol

9:     function init(

```

```solidity
File: /contracts/gmp-sdk/upgradable/InitProxy.sol

35:     function init(

```

### [L-4] UNIFY RETURN CRITERIA

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

132:     ) external view override returns (bool) {

144:     ) external view override returns (bool) {

156:     ) external override returns (bool valid) {

169:     ) external override returns (bool valid) {

183:     function authModule() public view override returns (address) {

187:     function governance() public view override returns (address) {

191:     function mintLimiter() public view override returns (address) {

195:     function tokenDeployer() public view returns (address) {

199:     function tokenMintLimit(string memory symbol) public view override returns (uint256) {

203:     function tokenMintAmount(string memory symbol) public view override returns (uint256) {

209:     function allTokensFrozen() external pure override returns (bool) {

214:     function adminEpoch() external pure override returns (uint256) {

219:     function adminThreshold(uint256) external pure override returns (uint256) {

224:     function admins(uint256) external pure override returns (address[] memory) {

228:     function implementation() public view override returns (address) {

232:     function tokenAddresses(string memory symbol) public view override returns (address) {

238:     function tokenFrozen(string memory) external pure override returns (bool) {

242:     function isCommandExecuted(bytes32 commandId) public view override returns (bool) {

246:     function contractId() public pure returns (bytes32) {

505:     function _hasCode(address addr) internal view returns (bool) {

556:     function _getTokenMintLimitKey(string memory symbol) internal pure returns (bytes32) {

560:     function _getTokenMintAmountKey(string memory symbol, uint256 day) internal pure returns (bytes32) {

565:     function _getTokenTypeKey(string memory symbol) internal pure returns (bytes32) {

569:     function _getTokenAddressKey(string memory symbol) internal pure returns (bytes32) {

573:     function _getIsCommandExecutedKey(bytes32 commandId) internal pure returns (bytes32) {

583:     ) internal pure returns (bytes32) {

595:     ) internal pure returns (bytes32) {

615:     function _getCreate2Address(bytes32 salt, bytes32 codeHash) internal view returns (address) {

619:     function _getTokenType(string memory symbol) internal view returns (TokenType) {

```

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

87:     function signerThreshold() external view override returns (uint256) {

95:     function signerAccounts() external view override returns (address[] memory) {

103:     function isSigner(address account) external view override returns (bool) {

111:     function hasSignerVoted(address account, bytes32 topic) external view override returns (bool) {

119:     function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {

```

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

56:     ) external view returns (uint256) {

147:     ) internal pure returns (bytes32) {

```

```solidity
File: /contracts/cgp/interfaces/IMultisigBase.sol

32:     function signerEpoch() external view returns (uint256);

38:     function signerThreshold() external view returns (uint256);

44:     function signerAccounts() external view returns (address[] memory);

50:     function isSigner(address account) external view returns (bool);

56:     function hasSignerVoted(address account, bytes32 topic) external view returns (bool);

62:     function getSignerVotesCount(bytes32 topic) external view returns (uint256);

```

```solidity
File: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

24:     function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {

46:     ) external returns (address deployedAddress_) {

62:     ) external view returns (address deployedAddress_) {

80:     function _deploy(bytes memory bytecode, bytes32 salt) internal returns (address deployedAddress_) {

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3.sol

51:     function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {

71:     function deployedAddress(address sender, bytes32 salt) internal pure returns (address deployed) {

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3Deployer.sol

29:     function deploy(bytes calldata bytecode, bytes32 salt) external returns (address deployedAddress_) {

53:     ) external returns (address deployedAddress_) {

67:     function deployedAddress(address sender, bytes32 salt) external view returns (address) {

```

```solidity
File: /contracts/gmp-sdk/interfaces/IFinalProxy.sol

9:     function isFinal() external view returns (bool);

11:     function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) external returns (address);

```

```solidity
File: /contracts/gmp-sdk/interfaces/IProxy.sol

13:     function implementation() external view returns (address implementation_);

```

```solidity
File: /contracts/gmp-sdk/interfaces/ITimeLock.sol

18:     function minimumTimeLockDelay() external view returns (uint256);

25:     function getTimeLock(bytes32 hash) external view returns (uint256);

```

```solidity
File: /contracts/gmp-sdk/interfaces/IUpgradable.sol

16:     function implementation() external view returns (address);

26:     function contractId() external pure returns (bytes32);

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

22:     function implementation() public view virtual returns (address implementation_) {

39:     function contractId() internal pure virtual returns (bytes32) {

```

```solidity
File: /contracts/gmp-sdk/upgradable/FinalProxy.sol

37:     function implementation() public view override(BaseProxy, IProxy) returns (address implementation_) {

48:     function isFinal() public view returns (bool) {

57:     function _finalImplementation() internal view virtual returns (address implementation_) {

72:     function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

39:     function implementation() public view returns (address implementation_) {

```

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

29:     function minimumTimeLockDelay() external view override returns (uint256) {

38:     function getTimeLock(bytes32 hash) external view override returns (uint256) {

50:     function _scheduleTimeLock(bytes32 hash, uint256 eta) internal returns (uint256) {

92:     function _getTimeLockEta(bytes32 hash) internal view returns (uint256 eta) {

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

144:     function contractId() external pure returns (bytes32) {

152:     function getChainName() public view returns (string memory name) {

161:     function getTokenManagerAddress(bytes32 tokenId) public view returns (address tokenManagerAddress) {

170:     function getValidTokenManagerAddress(bytes32 tokenId) public view returns (address tokenManagerAddress) {

180:     function getTokenAddress(bytes32 tokenId) external view returns (address tokenAddress) {

191:     function getStandardizedTokenAddress(bytes32 tokenId) public view returns (address tokenAddress) {

202:     function getCanonicalTokenId(address tokenAddress) public view returns (bytes32 tokenId) {

213:     function getCustomTokenId(address sender, bytes32 salt) public pure returns (bytes32 tokenId) {

223:     function getImplementation(uint256 tokenManagerType) external view returns (address tokenManagerAddress) {

241:     function getParamsLockUnlock(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {

251:     function getParamsMintBurn(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {

266:     ) public pure returns (bytes memory params) {

275:     function getFlowLimit(bytes32 tokenId) external view returns (uint256 flowLimit) {

285:     function getFlowOutAmount(bytes32 tokenId) external view returns (uint256 flowOutAmount) {

295:     function getFlowInAmount(bytes32 tokenId) external view returns (uint256 flowInAmount) {

309:     function registerCanonicalToken(address tokenAddress) external payable notPaused returns (bytes32 tokenId) {

347:     ) public payable notPaused returns (bytes32 tokenId) {

371:     ) external payable notPaused returns (bytes32 tokenId) {

562:         returns (address implementation)

728:         returns (

827:     function _getStandardizedTokenSalt(bytes32 tokenId) internal pure returns (bytes32 salt) {

872:     function _decodeMetadata(bytes calldata metadata) internal pure returns (uint32 version, bytes calldata data) {

```

```solidity
File: /contracts/its/interchain-token/InterchainToken.sol

21:     function getTokenManager() public view virtual returns (ITokenManager tokenManager);

32:     function tokenManagerRequiresApproval() public view virtual returns (bool);

```

```solidity
File: /contracts/its/interfaces/IDistributable.sol

14:     function distributor() external view returns (address distributor);

```

```solidity
File: /contracts/its/interfaces/IExpressCallHandler.sol

58:     ) external view returns (address expressCaller);

79:     ) external view returns (address expressCaller);

```

```solidity
File: /contracts/its/interfaces/IFlowLimit.sol

14:     function getFlowLimit() external view returns (uint256 flowLimit);

20:     function getFlowOutAmount() external view returns (uint256 flowOutAmount);

26:     function getFlowInAmount() external view returns (uint256 flowInAmount);

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

82:     function tokenManagerDeployer() external view returns (address tokenManagerDeployerAddress);

88:     function standardizedTokenDeployer() external view returns (address standardizedTokenDeployerAddress);

94:     function getChainName() external view returns (string memory name);

101:     function getTokenManagerAddress(bytes32 tokenId) external view returns (address tokenManagerAddress);

108:     function getValidTokenManagerAddress(bytes32 tokenId) external view returns (address tokenManagerAddress);

115:     function getTokenAddress(bytes32 tokenId) external view returns (address tokenAddress);

122:     function getStandardizedTokenAddress(bytes32 tokenId) external view returns (address tokenAddress);

129:     function getCanonicalTokenId(address tokenAddress) external view returns (bytes32 tokenId);

137:     function getCustomTokenId(address operator, bytes32 salt) external view returns (bytes32 tokenId);

145:     function getParamsLockUnlock(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

153:     function getParamsMintBurn(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

166:     ) external pure returns (bytes memory params);

173:     function registerCanonicalToken(address tokenAddress) external payable returns (bytes32 tokenId);

198:     ) external payable returns (bytes32 tokenId);

214:     ) external payable returns (bytes32 tokenId);

261:     function getImplementation(uint256 tokenManagerType) external view returns (address tokenManagerAddress);

293:     function getFlowLimit(bytes32 tokenId) external view returns (uint256 flowLimit);

300:     function getFlowOutAmount(bytes32 tokenId) external view returns (uint256 flowOutAmount);

307:     function getFlowInAmount(bytes32 tokenId) external view returns (uint256 flowInAmount);

```

```solidity
File: /contracts/its/interfaces/IMulticall.sol

18:     function multicall(bytes[] calldata data) external payable returns (bytes[] memory results);

```

```solidity
File: /contracts/its/interfaces/IOperatable.sol

14:     function operator() external view returns (address operator_);

```

```solidity
File: /contracts/its/interfaces/IPausable.sol

19:     function isPaused() external view returns (bool);

```

```solidity
File: /contracts/its/interfaces/IRemoteAddressValidator.sol

25:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool);

45:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress);

51:     function supportedByGateway(string calldata chainName) external view returns (bool);

```

```solidity
File: /contracts/its/interfaces/IStandardizedToken.sol

18:     function contractId() external view returns (bytes32);

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenDeployer.sol

18:     function deployer() external view returns (Create3Deployer);

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenProxy.sol

13:     function contractId() external view returns (bytes32);

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

26:     function tokenAddress() external view returns (address);

31:     function implementationType() external pure returns (uint256);

81:     function giveToken(address destinationAddress, uint256 amount) external returns (uint256);

```

```solidity
File: /contracts/its/interfaces/ITokenManagerDeployer.sol

18:     function deployer() external view returns (Create3Deployer);

```

```solidity
File: /contracts/its/interfaces/ITokenManagerProxy.sol

17:     function implementationType() external view returns (uint256);

23:     function implementation() external view returns (address);

28:     function tokenId() external view returns (bytes32);

```

```solidity
File: /contracts/its/libraries/AddressBytesUtils.sol

17:     function toAddress(bytes memory bytesAddress) internal pure returns (address addr) {

30:     function toBytes(address addr) internal pure returns (bytes memory bytesAddress) {

```

```solidity
File: /contracts/its/proxies/InterchainTokenServiceProxy.sol

29:     function contractId() internal pure override returns (bytes32) {

```

```solidity
File: /contracts/its/proxies/RemoteAddressValidatorProxy.sol

30:     function contractId() internal pure override returns (bytes32) {

```

```solidity
File: /contracts/its/proxies/StandardizedTokenProxy.sol

31:     function contractId() external pure returns (bytes32) {

```

```solidity
File: /contracts/its/proxies/TokenManagerProxy.sol

44:     function implementation() public view returns (address impl) {

57:         returns (address impl)

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

36:     function contractId() external pure returns (bytes32) {

54:     function _lowerCase(string memory s) internal pure returns (string memory) {

69:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool) {

133:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress) {

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

32:     function contractId() external pure returns (bytes32) {

40:     function getTokenManager() public view override returns (ITokenManager) {

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenLockUnlock.sol

8:     function tokenManagerRequiresApproval() public pure override returns (bool) {

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenMintBurn.sol

8:     function tokenManagerRequiresApproval() public pure override returns (bool) {

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

53:     function tokenAddress() public view virtual returns (address);

161:     function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256) {

182:     function _takeToken(address from, uint256 amount) internal virtual returns (uint256);

191:     function _giveToken(address from, uint256 amount) internal virtual returns (uint256);

204:     function _getTokenId() internal view returns (bytes32 tokenId) {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

28:     function tokenAddress() public view override returns (address tokenAddress_) {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

28:     function implementationType() external pure returns (uint256) {

57:     function liquidityPool() public view returns (address liquidityPool_) {

77:     function _takeToken(address from, uint256 amount) internal override returns (uint256) {

94:     function _giveToken(address to, uint256 amount) internal override returns (uint256) {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

24:     function implementationType() external pure returns (uint256) {

44:     function _takeToken(address from, uint256 amount) internal override returns (uint256) {

60:     function _giveToken(address to, uint256 amount) internal override returns (uint256) {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerMintBurn.sol

25:     function implementationType() external pure returns (uint256) {

45:     function _takeToken(address from, uint256 amount) internal override returns (uint256) {

59:     function _giveToken(address to, uint256 amount) internal override returns (uint256) {

```

```solidity
File: /contracts/its/utils/Distributable.sol

29:     function distributor() public view returns (address distr) {

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

31:     ) internal pure returns (uint256 slot) {

54:     ) internal pure returns (uint256 slot) {

153:     ) public view returns (address expressCaller) {

179:     ) public view returns (address expressCaller) {

207:     ) internal returns (address expressCaller) {

239:     ) internal returns (address expressCaller) {

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

24:     function getFlowLimit() public view returns (uint256 flowLimit) {

46:     function _getFlowOutSlot(uint256 epoch) internal pure returns (uint256 slot) {

55:     function _getFlowInSlot(uint256 epoch) internal pure returns (uint256 slot) {

63:     function getFlowOutAmount() external view returns (uint256 flowOutAmount) {

75:     function getFlowInAmount() external view returns (uint256 flowInAmount) {

```

```solidity
File: /contracts/its/utils/Multicall.sol

22:     function multicall(bytes[] calldata data) public payable returns (bytes[] memory results) {

```

```solidity
File: /contracts/its/utils/Operatable.sol

29:     function operator() public view returns (address operator_) {

```

```solidity
File: /contracts/its/utils/Pausable.sol

29:     function isPaused() public view returns (bool paused) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-5] REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR

#### Description:

The correct and clear error description explains to the user why the function reverts, but the error descriptions below in the project are not self-explanatory. These error descriptions are very important in the debug features of DApps like Tenderly. Error definitions should be added to the require block, not exceeding 32 bytes.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Error definitions should be added to the require block, not exceeding 32 bytes or we should use custom errors

```solidity
File: /contracts/gmp-sdk/deploy/Create3.sol

27:                 revert(0, 0)

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

56:                 revert(0, returndatasize())

```

```solidity
File: /contracts/gmp-sdk/upgradable/FixedProxy.sol

48:                 revert(0, returndatasize())

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

164:                 revert(add(32, result), mload(result))

```

```solidity
File: /contracts/its/proxies/TokenManagerProxy.sol

83:                 revert(0, returndatasize())

```

```solidity
File: /contracts/its/utils/Multicall.sol

28:                 revert(string(result));

```

### [L-6] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

524:             IERC20(tokenAddress).safeTransfer(account, amount);

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

64:         SafeTokenTransfer.safeTransfer(token, to, amount);

```

### [L-7] A SINGLE POINT OF FAILURE

#### Description:

The owner role has a single point of failure and onlyOwner can use critical a few functions.

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

#### **Proof Of Concept**

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

56:     ) external override onlyOwner {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

96:     ) external override onlyOwner {

111:     ) external override onlyOwner {

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

547:     function setPaused(bool paused) external onlyOwner {

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

83:     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

95:     function removeTrustedAddress(string calldata chain) external onlyOwner {

106:     function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

119:     function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

```

#### Recommended Mitigation Steps:

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.

### [L-8] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

204:         return getUint(_getTokenMintAmountKey(symbol, block.timestamp / 6 hours));

637:         _setUint(_getTokenMintAmountKey(symbol, block.timestamp / 6 hours), amount);

```

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

78:         emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);

```

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

54:         uint256 minimumEta = block.timestamp + _minimumTimeLockDelay;

84:         if (block.timestamp < eta) revert TimeLockNotReady();

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

64:         uint256 epoch = block.timestamp / EPOCH_TIME;

76:         uint256 epoch = block.timestamp / EPOCH_TIME;

114:         uint256 epoch = block.timestamp / EPOCH_TIME;

127:         uint256 epoch = block.timestamp / EPOCH_TIME;

```

### [L-9] UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT

#### Description:

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

168:     receive() external payable {}

```

```solidity
File: /contracts/cgp/governance/Multisig.sol

41:     receive() external payable {}

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

67:     receive() external payable virtual {}

```

```solidity
File: /contracts/gmp-sdk/upgradable/FixedProxy.sol

59:     receive() external payable virtual {}

```

```solidity
File: /contracts/its/proxies/TokenManagerProxy.sol

94:     receive() external payable virtual {}

```

#### Recommended Mitigation Steps:

The function should call another function, otherwise it should revert

### [L-10] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

65:                 _mint(mintTo, mintAmount);

77:         _mint(account, amount);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`

### [L-11] USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION

#### Description:

transferOwnership function is used to change Ownership from Owned.sol.

Use a 2 structure transferOwnership which is safer.

safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

#### **Proof Of Concept**

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

30:         _transferOwnership(_owner);

```

#### Recommended Mitigation Steps:

Use Ownable2Step.sol
