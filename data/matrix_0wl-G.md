## Gas Optimizations

|        | Issue                                                                                                                   |
| ------ | :---------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | Use `selfbalance()` instead of `address(this).balance`                                                                  |
| GAS-2  | `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables                   |
| GAS-3  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                              |
| GAS-4  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                             |
| GAS-5  | Using bools for storage incurs overhead                                                                                 |
| GAS-6  | <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP                                                      |
| GAS-7  | Use custom errors rather than revert()/require() strings to save gas                                                    |
| GAS-8  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                       |
| GAS-9  | IT COSTS MORE GAS TO INITIALIZE NON-CONSTANT/NON-IMMUTABLE VARIABLES TO ZERO THAN TO LET THE DEFAULT OF ZERO BE APPLIED |
| GAS-10 | MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL                                                            |
| GAS-11 | Multiple address/id mappings can be combined into a single mapping of an address/id to a struct, where appropriate      |
| GAS-12 | Functions guaranteed to revert when called by normal users can be marked payable                                        |
| GAS-13 | Optimize names to save gas                                                                                              |
| GAS-14 | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops  |
| GAS-15 | Reorder structure layout                                                                                                |
| GAS-16 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                     |
| GAS-17 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead                                                  |
| GAS-18 | USE BYTES32 INSTEAD OF STRING                                                                                           |
| GAS-19 | USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT                                         |

### [GAS-1] Use `selfbalance()` instead of `address(this).balance`

#### Description:

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas.
Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/util/Caller.sol

16:         if (nativeValue > address(this).balance) revert InsufficientBalance();

```

### [GAS-2] `<x> += <y>`/`<x> -= <y>` costs more gas than `<x> = <x> + <y>`/`<x> = <x> - <y>` for state variables

#### Description:

Using the addition operator instead of plus-equals saves gas

#### **Proof Of Concept**

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

107:             totalGas += interchainCalls[i].gas;

```

### [GAS-3] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

562:         return keccak256(abi.encode(PREFIX_TOKEN_MINT_AMOUNT, symbol, day));

584:         return keccak256(abi.encode(PREFIX_CONTRACT_CALL_APPROVED, commandId, sourceChain, sourceAddress, contractAddress, payloadHash));

```

```solidity
File: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

25:         deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

47:         deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

63:         bytes32 newSalt = keccak256(abi.encode(sender, salt));

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3Deployer.sol

30:         bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));

54:         bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));

68:         bytes32 deploySalt = keccak256(abi.encode(sender, salt));

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

66:         emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

89:         bytes memory payload = abi.encode(msg.sender, interchainCall.calls);

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

203:         tokenId = keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_ID, chainNameHash, tokenAddress));

214:         tokenId = keccak256(abi.encode(PREFIX_CUSTOM_TOKEN_ID, sender, salt));

242:         params = abi.encode(operator, tokenAddress);

252:         params = abi.encode(operator, tokenAddress);

267:         params = abi.encode(operator, tokenAddress, liquidityPoolAddress);

313:         _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));

401:         _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));

512:             payload = abi.encode(SELECTOR_SEND_TOKEN, tokenId, destinationAddress, amount);

520:         payload = abi.encode(SELECTOR_SEND_TOKEN_WITH_DATA, tokenId, destinationAddress, amount, sourceAddress.toBytes(), metadata);

696:             abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)

755:         bytes memory payload = abi.encode(SELECTOR_DEPLOY_TOKEN_MANAGER, tokenId, tokenManagerType, params);

828:         return keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_SALT, tokenId));

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

32:         slot = uint256(keccak256(abi.encode(PREFIX_EXPRESS_RECEIVE_TOKEN, tokenId, destinationAddress, amount, commandId)));

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

47:         slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)));

56:         slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)));

```

```solidity
File: /contracts/its/utils/StandardizedTokenDeployer.sol

62:             bytes memory params = abi.encode(tokenManager, distributor, name, symbol, decimals, mintAmount, mintTo);

63:             bytecode = abi.encodePacked(type(StandardizedTokenProxy).creationCode, abi.encode(implementationAddress, params));

```

```solidity
File: /contracts/its/utils/TokenManagerDeployer.sol

38:         bytes memory args = abi.encode(address(this), implementationType, tokenId, params);

```

### [GAS-4] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

495:     function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {

681:     function _transferGovernance(address newGovernance) internal {

687:     function _transferMintLimiter(address newMintLimiter) internal {

```

### [GAS-5] Using bools for storage incurs overhead

#### Description:

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

15:         mapping(address => bool) hasVoted;

21:         mapping(address => bool) isSigner;

```

```solidity
File: /contracts/cgp/governance/AxelarServiceGovernance.sol

22:     mapping(bytes32 => bool) public multisigApprovals;

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

24:     mapping(string => mapping(address => bool)) public whitelistedCallers;

27:     mapping(string => mapping(address => bool)) public whitelistedSenders;

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

19:     mapping(string => bool) public supportedByGateway;

```

### [GAS-6] <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP

#### Description:

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

[Source](https://code4rena.com/reports/2022-12-backed#g14--arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)

#### **Proof Of Concept**

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

74:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

63:         for (uint256 i = 0; i < interchainCalls.length; ) {

106:         for (uint256 i = 0; i < interchainCalls.length; ) {

```

```solidity
File: /contracts/its/utils/Multicall.sol

24:         for (uint256 i = 0; i < data.length; ++i) {

```

### [GAS-7] Use custom errors rather than revert()/require() strings to save gas

#### Description:

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas.[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.
[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

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

### [GAS-8] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

72:     modifier onlySelf() {

78:     modifier onlyGovernance() {

87:     modifier onlyMintLimiter() {

```

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

44:     modifier onlySigners() {

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

29:     modifier onlyProxy() {

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

123:     modifier onlyRemoteService(string calldata sourceChain, string calldata sourceAddress) {

132:     modifier onlyTokenManager(bytes32 tokenId) {

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

35:     modifier onlyService() {

43:     modifier onlyToken() {

```

```solidity
File: /contracts/its/utils/Distributable.sol

20:     modifier onlyDistributor() {

```

```solidity
File: /contracts/its/utils/Implementation.sol

26:     modifier onlyProxy() {

```

```solidity
File: /contracts/its/utils/Operatable.sol

20:     modifier onlyOperator() {

```

```solidity
File: /contracts/its/utils/Pausable.sol

20:     modifier notPaused() {

```

### [GAS-9] IT COSTS MORE GAS TO INITIALIZE NON-CONSTANT/NON-IMMUTABLE VARIABLES TO ZERO THAN TO LET THE DEFAULT OF ZERO BE APPLIED

#### Description:

If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

#### **Proof Of Concept**

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

74:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

63:         for (uint256 i = 0; i < interchainCalls.length; ) {

105:         uint256 totalGas = 0;

106:         for (uint256 i = 0; i < interchainCalls.length; ) {

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

118:         uint32 version = 0;

```

```solidity
File: /contracts/its/utils/Multicall.sol

24:         for (uint256 i = 0; i < data.length; ++i) {

```

### [GAS-10] MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

44:     modifier onlySigners() {

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

29:     modifier onlyProxy() {

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

123:     modifier onlyRemoteService(string calldata sourceChain, string calldata sourceAddress) {

132:     modifier onlyTokenManager(bytes32 tokenId) {

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

35:     modifier onlyService() {

43:     modifier onlyToken() {

```

```solidity
File: /contracts/its/utils/Distributable.sol

20:     modifier onlyDistributor() {

```

```solidity
File: /contracts/its/utils/Implementation.sol

26:     modifier onlyProxy() {

```

```solidity
File: /contracts/its/utils/Operatable.sol

20:     modifier onlyOperator() {

```

```solidity
File: /contracts/its/utils/Pausable.sol

20:     modifier notPaused() {

```

### [GAS-11] Multiple address/id mappings can be combined into a single mapping of an address/id to a struct, where appropriate

#### Description:

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.
If both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

15:         mapping(address => bool) hasVoted;

21:         mapping(address => bool) isSigner;

27:     mapping(uint256 => mapping(bytes32 => Voting)) public votingPerTopic;

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

24:     mapping(string => mapping(address => bool)) public whitelistedCallers;

27:     mapping(string => mapping(address => bool)) public whitelistedSenders;

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

15:     mapping(string => bytes32) public remoteAddressHashes;

16:     mapping(string => string) public remoteAddresses;

```

#### Recommended Mitigation Steps:

Make mapping a struct instead

### [GAS-12] Functions guaranteed to revert when called by normal users can be marked payable

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

254:     function transferGovernance(address newGovernance) external override onlyGovernance {

260:     function transferMintLimiter(address newMintLimiter) external override onlyMintLimiter {

266:     function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {

284:     ) external override onlyGovernance {

385:     function deployToken(bytes calldata params, bytes32) external onlySelf {

421:     function mintToken(bytes calldata params, bytes32) external onlySelf {

427:     function burnToken(bytes calldata params, bytes32) external onlySelf {

455:     function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf {

469:     function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf {

495:     function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {

```

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

142:     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {

```

```solidity
File: /contracts/cgp/governance/AxelarServiceGovernance.sol

52:     ) external payable onlySigners {

```

```solidity
File: /contracts/cgp/governance/Multisig.sol

34:     ) external payable onlySigners {

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

56:     ) external override onlyOwner {

78:     function setup(bytes calldata data) external override onlyProxy {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

96:     ) external override onlyOwner {

111:     ) external override onlyOwner {

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

509:     ) external payable onlyTokenManager(tokenId) notPaused {

534:     function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {

547:     function setPaused(bool paused) external onlyOwner {

579:     ) internal override onlyRemoteService(sourceChain, sourceAddress) notPaused {

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

83:     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

95:     function removeTrustedAddress(string calldata chain) external onlyOwner {

106:     function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

119:     function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

49:     function setup(bytes calldata params) external override onlyProxy {

76:     function mint(address account, uint256 amount) external onlyDistributor {

86:     function burn(address account, uint256 amount) external onlyDistributor {

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

35:     modifier onlyService() {

43:     modifier onlyToken() {

61:     function setup(bytes calldata params) external override onlyProxy {

142:     ) external payable virtual onlyToken {

161:     function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256) {

171:     function setFlowLimit(uint256 flowLimit) external onlyOperator {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

67:     function setLiquidityPool(address newLiquidityPool) external onlyOperator {

```

```solidity
File: /contracts/its/utils/Distributable.sol

20:     modifier onlyDistributor() {

51:     function setDistributor(address distr) external onlyDistributor {

```

```solidity
File: /contracts/its/utils/Implementation.sol

26:     modifier onlyProxy() {

```

```solidity
File: /contracts/its/utils/Operatable.sol

20:     modifier onlyOperator() {

51:     function setOperator(address operator_) external onlyOperator {

```

### [GAS-13] Optimize names to save gas

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

[Solidity optimize name github](https://github.com/enzosv/solidity-optimize-name)

[Source](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

17: contract AxelarGateway is IAxelarGateway, IGovernable, AdminMultisigBase {

```

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

12: contract MultisigBase is IMultisigBase {

```

```solidity
File: /contracts/cgp/governance/AxelarServiceGovernance.sol

14: contract AxelarServiceGovernance is InterchainGovernance, MultisigBase, IAxelarServiceGovernance {

```

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

15: contract InterchainGovernance is AxelarExecutable, TimeLock, Caller, IInterchainGovernance {

```

```solidity
File: /contracts/cgp/governance/Multisig.sol

13: contract Multisig is Caller, MultisigBase, IMultisig {

```

```solidity
File: /contracts/cgp/interfaces/IAxelarServiceGovernance.sol

12: interface IAxelarServiceGovernance is IMultisigBase, IInterchainGovernance {

```

```solidity
File: /contracts/cgp/interfaces/ICaller.sol

5: interface ICaller {

```

```solidity
File: /contracts/cgp/interfaces/IMultisig.sol

12: interface IMultisig is ICaller, IMultisigBase {

```

```solidity
File: /contracts/cgp/interfaces/IMultisigBase.sol

9: interface IMultisigBase {

```

```solidity
File: /contracts/cgp/util/Caller.sol

7: contract Caller is ICaller {

```

```solidity
File: /contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

5: contract ConstAddressDeployer {

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3.sol

16: contract CreateDeployer {

```

```solidity
File: /contracts/gmp-sdk/deploy/Create3Deployer.sol

12: contract Create3Deployer {

```

```solidity
File: /contracts/gmp-sdk/interfaces/IFinalProxy.sol

8: interface IFinalProxy is IProxy {

```

```solidity
File: /contracts/gmp-sdk/interfaces/IInitProxy.sol

8: interface IInitProxy is IProxy {

```

```solidity
File: /contracts/gmp-sdk/interfaces/IProxy.sol

6: interface IProxy {

```

```solidity
File: /contracts/gmp-sdk/interfaces/ITimeLock.sol

9: interface ITimeLock {

```

```solidity
File: /contracts/gmp-sdk/interfaces/IUpgradable.sol

8: interface IUpgradable is IOwnable {

```

```solidity
File: /contracts/gmp-sdk/upgradable/BaseProxy.sol

12: abstract contract BaseProxy is IProxy {

```

```solidity
File: /contracts/gmp-sdk/upgradable/FinalProxy.sol

17: contract FinalProxy is Proxy, IFinalProxy {

```

```solidity
File: /contracts/gmp-sdk/upgradable/FixedProxy.sol

12: contract FixedProxy is IProxy {

```

```solidity
File: /contracts/gmp-sdk/upgradable/InitProxy.sol

16: contract InitProxy is BaseProxy, IInitProxy {

```

```solidity
File: /contracts/gmp-sdk/upgradable/Proxy.sol

15: contract Proxy is BaseProxy {

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

12: abstract contract Upgradable is Ownable, IUpgradable {

```

```solidity
File: /contracts/gmp-sdk/util/TimeLock.sol

12: contract TimeLock is ITimeLock {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

22: contract InterchainProposalExecutor is IInterchainProposalExecutor, AxelarExecutable, Ownable {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

38: contract InterchainProposalSender is IInterchainProposalSender {

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

5: interface IInterchainProposalExecutor {

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

7: interface IInterchainProposalSender {

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

37: contract InterchainTokenService is

```

```solidity
File: /contracts/its/interchain-token/InterchainToken.sol

15: abstract contract InterchainToken is IInterchainToken, ERC20 {

```

```solidity
File: /contracts/its/interfaces/IDistributable.sol

5: interface IDistributable {

```

```solidity
File: /contracts/its/interfaces/IExpressCallHandler.sol

5: interface IExpressCallHandler {

```

```solidity
File: /contracts/its/interfaces/IFlowLimit.sol

5: interface IFlowLimit {

```

```solidity
File: /contracts/its/interfaces/IImplementation.sol

5: interface IImplementation {

```

```solidity
File: /contracts/its/interfaces/IInterchainToken.sol

10: interface IInterchainToken is IERC20 {

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

14: interface IInterchainTokenService is ITokenManagerType, IExpressCallHandler, IAxelarExecutable, IPausable, IMulticall {

```

```solidity
File: /contracts/its/interfaces/IMulticall.sol

10: interface IMulticall {

```

```solidity
File: /contracts/its/interfaces/IOperatable.sol

5: interface IOperatable {

```

```solidity
File: /contracts/its/interfaces/IPausable.sol

10: interface IPausable {

```

```solidity
File: /contracts/its/interfaces/IRemoteAddressValidator.sol

9: interface IRemoteAddressValidator {

```

```solidity
File: /contracts/its/interfaces/IStandardizedToken.sol

14: interface IStandardizedToken is IInterchainToken, IDistributable, IERC20BurnableMintable {

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenDeployer.sol

11: interface IStandardizedTokenDeployer {

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenProxy.sol

9: interface IStandardizedTokenProxy {

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

14: interface ITokenManager is ITokenManagerType, IOperatable, IFlowLimit, IImplementation {

```

```solidity
File: /contracts/its/interfaces/ITokenManagerDeployer.sol

11: interface ITokenManagerDeployer {

```

```solidity
File: /contracts/its/interfaces/ITokenManagerProxy.sol

10: interface ITokenManagerProxy {

```

```solidity
File: /contracts/its/interfaces/ITokenManagerType.sol

9: interface ITokenManagerType {

```

```solidity
File: /contracts/its/proxies/InterchainTokenServiceProxy.sol

11: contract InterchainTokenServiceProxy is FinalProxy {

```

```solidity
File: /contracts/its/proxies/RemoteAddressValidatorProxy.sol

11: contract RemoteAddressValidatorProxy is Proxy {

```

```solidity
File: /contracts/its/proxies/StandardizedTokenProxy.sol

13: contract StandardizedTokenProxy is FixedProxy, IStandardizedTokenProxy {

```

```solidity
File: /contracts/its/proxies/TokenManagerProxy.sol

13: contract TokenManagerProxy is ITokenManagerProxy {

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

12: contract RemoteAddressValidator is IRemoteAddressValidator, Upgradable {

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

19: abstract contract StandardizedToken is InterchainToken, ERC20Permit, Implementation, Distributable {

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenLockUnlock.sol

7: contract StandardizedTokenLockUnlock is StandardizedToken {

```

```solidity
File: /contracts/its/token-implementations/StandardizedTokenMintBurn.sol

7: contract StandardizedTokenMintBurn is StandardizedToken {

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

18: abstract contract TokenManager is ITokenManager, Operatable, FlowLimit, Implementation {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

14: abstract contract TokenManagerAddressStorage is TokenManager {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

17: contract TokenManagerLiquidityPool is TokenManagerAddressStorage {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

16: contract TokenManagerLockUnlock is TokenManagerAddressStorage {

```

```solidity
File: /contracts/its/token-manager/implementations/TokenManagerMintBurn.sol

17: contract TokenManagerMintBurn is TokenManagerAddressStorage {

```

```solidity
File: /contracts/its/utils/Distributable.sol

13: contract Distributable is IDistributable {

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

12: contract ExpressCallHandler is IExpressCallHandler {

```

```solidity
File: /contracts/its/utils/FlowLimit.sol

12: contract FlowLimit is IFlowLimit {

```

```solidity
File: /contracts/its/utils/Implementation.sol

12: abstract contract Implementation is IImplementation {

```

```solidity
File: /contracts/its/utils/Multicall.sol

12: contract Multicall is IMulticall {

```

```solidity
File: /contracts/its/utils/Operatable.sol

13: contract Operatable is IOperatable {

```

```solidity
File: /contracts/its/utils/Pausable.sol

12: contract Pausable is IPausable {

```

```solidity
File: /contracts/its/utils/StandardizedTokenDeployer.sol

15: contract StandardizedTokenDeployer is IStandardizedTokenDeployer {

```

```solidity
File: /contracts/its/utils/TokenManagerDeployer.sol

15: contract TokenManagerDeployer is ITokenManagerDeployer {

```

### [GAS-14] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops

#### Description:

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

124:                 voteCount++;

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

74:         for (uint256 i = 0; i < calls.length; i++) {

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

56:         for (uint256 i; i < length; i++) {

```

### [GAS-15] Reorder structure layout

#### Description:

Structures could be optimized moving the position of certain values in order to save a lot slots.

For example Enums are represented by integers; the possibility listed first by 0, the next by 1, and so forth. An enum type just acts like uintN, where N is the smallest legal value large enough to accomodate all the possibilities.

[Source](https://ethdebug.github.io/solidity-data-representation)

[Source](hhttps://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding)

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

18:     struct Signers {
            address[] accounts;
            uint256 threshold;
            mapping(address => bool) isSigner;
        }

```

### [GAS-16] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

523:    if (_getTokenType(symbol) == TokenType.External) {
            IERC20(tokenAddress).safeTransfer(account, amount);
        } else {
            IBurnableMintableCappedERC20(tokenAddress).mint(account, amount);
        }

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

78:         if (!success) {
                _onTargetExecutionFailed(call, result);
            } else {
                _onTargetExecuted(call, result);
            }

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

68:     if (operatorBytes.length == 0) {
            operator_ = address(interchainTokenService);
        } else {
            operator_ = operatorBytes.toAddress();
        }

```

### [GAS-17] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

386:         (string memory name, string memory symbol, uint8 decimals, uint256 cap, address tokenAddress, uint256 mintLimit) = abi.decode(

388:             (string, string, uint8, uint256, address, uint256)

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

333:         (string memory tokenName, string memory tokenSymbol, uint8 tokenDecimals) = _validateToken(tokenAddress);

392:         uint8 decimals,

421:         uint8 decimals,

517:         uint32 version;

684:             uint8 decimals,

687:         ) = abi.decode(payload, (uint256, bytes32, string, string, uint8, bytes, bytes));

731:             uint8 decimals

774:         uint8 decimals,

846:         uint8 decimals,

872:     function _decodeMetadata(bytes calldata metadata) internal pure returns (uint32 version, bytes calldata data) {

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

29:     error InvalidMetadataVersion(uint32 version);

61:         uint8 tokenDecimals,

72:         uint8 decimals,

229:         uint8 decimals,

249:         uint8 decimals,

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenDeployer.sol

37:         uint8 decimals,

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

57:             uint8 b = uint8(bytes(s)[i]);

57:             uint8 b = uint8(bytes(s)[i]);

58:             if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

25:     uint8 public decimals;

54:             (tokenManager_, distributor_, tokenName, symbol, decimals) = abi.decode(params, (address, address, string, string, uint8));

63:             (, , , , , mintAmount, mintTo) = abi.decode(params, (address, address, string, string, uint8, uint256, address));

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

118:         uint32 version = 0;

```

```solidity
File: /contracts/its/utils/StandardizedTokenDeployer.sol

55:         uint8 decimals,

```

### [GAS-18] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

[Code4arena example](https://code4rena.com/reports/2022-10-holograph/#g-15-use-bytes32-instead-of-string)

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

98:         string calldata destinationChain,

99:         string calldata destinationAddress,

100:         string calldata symbol,

108:         string calldata destinationChain,

109:         string calldata destinationContractAddress,

116:         string calldata destinationChain,

117:         string calldata destinationContractAddress,

119:         string calldata symbol,

128:         string calldata sourceChain,

129:         string calldata sourceAddress,

138:         string calldata sourceChain,

139:         string calldata sourceAddress,

142:         string calldata symbol,

153:         string calldata sourceChain,

154:         string calldata sourceAddress,

164:         string calldata sourceChain,

165:         string calldata sourceAddress,

167:         string calldata symbol,

199:     function tokenMintLimit(string memory symbol) public view override returns (uint256) {

203:     function tokenMintAmount(string memory symbol) public view override returns (uint256) {

232:     function tokenAddresses(string memory symbol) public view override returns (address) {

238:     function tokenFrozen(string memory) external pure override returns (bool) {

266:     function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {

271:             string memory symbol = symbols[i];

333:         string[] memory commands;

336:         (chainId, commandIds, commands, params) = abi.decode(data, (uint256, bytes32[], string[], bytes[]));

386:         (string memory name, string memory symbol, uint8 decimals, uint256 cap, address tokenAddress, uint256 mintLimit) = abi.decode(

386:         (string memory name, string memory symbol, uint8 decimals, uint256 cap, address tokenAddress, uint256 mintLimit) = abi.decode(

388:             (string, string, uint8, uint256, address, uint256)

388:             (string, string, uint8, uint256, address, uint256)

422:         (string memory symbol, address account, uint256 amount) = abi.decode(params, (string, address, uint256));

422:         (string memory symbol, address account, uint256 amount) = abi.decode(params, (string, address, uint256));

428:         (string memory symbol, bytes32 salt) = abi.decode(params, (string, bytes32));

428:         (string memory symbol, bytes32 salt) = abi.decode(params, (string, bytes32));

457:             string memory sourceChain,

458:             string memory sourceAddress,

463:         ) = abi.decode(params, (string, string, address, bytes32, bytes32, uint256));

463:         ) = abi.decode(params, (string, string, address, bytes32, bytes32, uint256));

471:             string memory sourceChain,

472:             string memory sourceAddress,

475:             string memory symbol,

479:         ) = abi.decode(params, (string, string, address, bytes32, string, uint256, bytes32, uint256));

479:         ) = abi.decode(params, (string, string, address, bytes32, string, uint256, bytes32, uint256));

479:         ) = abi.decode(params, (string, string, address, bytes32, string, uint256, bytes32, uint256));

513:         string memory symbol,

532:         string memory symbol,

556:     function _getTokenMintLimitKey(string memory symbol) internal pure returns (bytes32) {

560:     function _getTokenMintAmountKey(string memory symbol, uint256 day) internal pure returns (bytes32) {

565:     function _getTokenTypeKey(string memory symbol) internal pure returns (bytes32) {

569:     function _getTokenAddressKey(string memory symbol) internal pure returns (bytes32) {

579:         string memory sourceChain,

580:         string memory sourceAddress,

589:         string memory sourceChain,

590:         string memory sourceAddress,

593:         string memory symbol,

619:     function _getTokenType(string memory symbol) internal view returns (TokenType) {

627:     function _setTokenMintLimit(string memory symbol, uint256 limit) internal {

633:     function _setTokenMintAmount(string memory symbol, uint256 amount) internal {

640:     function _setTokenType(string memory symbol, TokenType tokenType) internal {

644:     function _setTokenAddress(string memory symbol, address tokenAddress) internal {

654:         string memory sourceChain,

655:         string memory sourceAddress,

664:         string memory sourceChain,

665:         string memory sourceAddress,

668:         string memory symbol,

```

```solidity
File: /contracts/cgp/governance/AxelarServiceGovernance.sol

35:         string memory governanceChain,

36:         string memory governanceAddress,

```

```solidity
File: /contracts/cgp/governance/InterchainGovernance.sol

21:     string public governanceChain;

22:     string public governanceAddress;

35:         string memory governanceChain_,

36:         string memory governanceAddress_,

88:         string calldata sourceChain,

89:         string calldata sourceAddress,

156:         string calldata, /* sourceChain */

157:         string calldata, /* sourceAddress */

159:         string calldata, /* tokenSymbol */

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

24:     mapping(string => mapping(address => bool)) public whitelistedCallers;

27:     mapping(string => mapping(address => bool)) public whitelistedSenders;

42:         string calldata sourceChain,

43:         string calldata sourceAddress,

93:         string calldata sourceChain,

108:         string calldata sourceChain,

127:         string calldata sourceChain,

128:         string calldata sourceAddress,

143:         string calldata, /* sourceChain */

144:         string calldata, /* sourceAddress */

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

81:         string memory destinationChain,

82:         string memory destinationContract,

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

7:     event WhitelistedProposalCallerSet(string indexed sourceChain, address indexed sourceCaller, bool whitelisted);

10:     event WhitelistedProposalSenderSet(string indexed sourceChain, address indexed sourceSender, bool whitelisted);

30:         string calldata sourceChain,

42:         string calldata sourceChain,

```

```solidity
File: /contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

24:         string calldata destinationChain,

25:         string calldata destinationContract,

```

```solidity
File: /contracts/interchain-governance-executor/lib/InterchainCalls.sol

14:         string destinationChain;

15:         string destinationContract;

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

46:     using StringToBytes32 for string;

90:         string memory chainName_

123:     modifier onlyRemoteService(string calldata sourceChain, string calldata sourceAddress) {

123:     modifier onlyRemoteService(string calldata sourceChain, string calldata sourceAddress) {

152:     function getChainName() public view returns (string memory name) {

310:         (, string memory tokenSymbol, ) = _validateToken(tokenAddress);

327:         string calldata destinationChain,

333:         (string memory tokenName, string memory tokenSymbol, uint8 tokenDecimals) = _validateToken(tokenAddress);

333:         (string memory tokenName, string memory tokenSymbol, uint8 tokenDecimals) = _validateToken(tokenAddress);

367:         string calldata destinationChain,

390:         string calldata name,

391:         string calldata symbol,

419:         string calldata name,

420:         string calldata symbol,

424:         string calldata destinationChain,

469:         string memory sourceChain,

505:         string calldata destinationChain,

576:         string calldata sourceChain,

577:         string calldata sourceAddress,

599:     function _processSendTokenPayload(string calldata sourceChain, bytes calldata payload) internal {

622:     function _processSendTokenWithDataPayload(string calldata sourceChain, bytes calldata payload) internal {

682:             string memory name,

683:             string memory symbol,

687:         ) = abi.decode(payload, (uint256, bytes32, string, string, uint8, bytes, bytes));

687:         ) = abi.decode(payload, (uint256, bytes32, string, string, uint8, bytes, bytes));

708:         string calldata destinationChain,

713:         string memory destinationAddress = remoteAddressValidator.getRemoteAddress(destinationChain);

729:             string memory name,

730:             string memory symbol,

750:         string calldata destinationChain,

772:         string memory name,

773:         string memory symbol,

777:         string calldata destinationChain,

844:         string memory name,

845:         string memory symbol,

883:         string memory sourceChain,

```

```solidity
File: /contracts/its/interchain-token/InterchainToken.sol

44:         string calldata destinationChain,

81:         string calldata destinationChain,

```

```solidity
File: /contracts/its/interfaces/IExpressCallHandler.sol

26:         string sourceChain,

36:         string sourceChain,

73:         string memory sourceChain,

```

```solidity
File: /contracts/its/interfaces/IInterchainToken.sol

21:         string calldata destinationChain,

39:         string calldata destinationChain,

```

```solidity
File: /contracts/its/interfaces/IInterchainTokenService.sol

32:     event TokenSent(bytes32 tokenId, string destinationChain, bytes destinationAddress, uint256 indexed amount);

35:         string destinationChain,

41:     event TokenReceived(bytes32 indexed tokenId, string sourceChain, address indexed destinationAddress, uint256 indexed amount);

44:         string sourceChain,

52:         string destinationChain,

59:         string tokenName,

60:         string tokenSymbol,

64:         string destinationChain,

70:         string name,

71:         string symbol,

94:     function getChainName() external view returns (string memory name);

183:         string calldata destinationChain,

210:         string calldata destinationChain,

227:         string calldata name,

228:         string calldata symbol,

247:         string calldata name,

248:         string calldata symbol,

252:         string calldata destinationChain,

275:         string calldata destinationChain,

341:         string memory sourceChain,

```

```solidity
File: /contracts/its/interfaces/IRemoteAddressValidator.sol

14:     event TrustedAddressAdded(string souceChain, string sourceAddress);

14:     event TrustedAddressAdded(string souceChain, string sourceAddress);

15:     event TrustedAddressRemoved(string souceChain);

16:     event GatewaySupportedChainAdded(string chain);

17:     event GatewaySupportedChainRemoved(string chain);

25:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool);

25:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool);

32:     function addTrustedAddress(string memory sourceChain, string memory sourceAddress) external;

32:     function addTrustedAddress(string memory sourceChain, string memory sourceAddress) external;

38:     function removeTrustedAddress(string calldata sourceChain) external;

45:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress);

45:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress);

51:     function supportedByGateway(string calldata chainName) external view returns (bool);

57:     function addGatewaySupportedChains(string[] calldata chainNames) external;

63:     function removeGatewaySupportedChains(string[] calldata chainNames) external;

```

```solidity
File: /contracts/its/interfaces/IStandardizedTokenDeployer.sol

35:         string calldata name,

36:         string calldata symbol,

```

```solidity
File: /contracts/its/interfaces/ITokenManager.sol

40:         string calldata destinationChain,

54:         string calldata destinationChain,

69:         string calldata destinationChain,

```

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

15:     mapping(string => bytes32) public remoteAddressHashes;

16:     mapping(string => string) public remoteAddresses;

16:     mapping(string => string) public remoteAddresses;

19:     mapping(string => bool) public supportedByGateway;

41:         (string[] memory trustedChainNames, string[] memory trustedAddresses) = abi.decode(params, (string[], string[]));

41:         (string[] memory trustedChainNames, string[] memory trustedAddresses) = abi.decode(params, (string[], string[]));

41:         (string[] memory trustedChainNames, string[] memory trustedAddresses) = abi.decode(params, (string[], string[]));

41:         (string[] memory trustedChainNames, string[] memory trustedAddresses) = abi.decode(params, (string[], string[]));

54:     function _lowerCase(string memory s) internal pure returns (string memory) {

54:     function _lowerCase(string memory s) internal pure returns (string memory) {

69:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool) {

69:     function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool) {

70:         string memory sourceAddressLC = _lowerCase(sourceAddress);

83:     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

83:     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

95:     function removeTrustedAddress(string calldata chain) external onlyOwner {

106:     function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

109:             string calldata chainName = chainNames[i];

119:     function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

122:             string calldata chainName = chainNames[i];

133:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress) {

133:     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress) {

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

23:     string public name;

24:     string public symbol;

53:             string memory tokenName;

54:             (tokenManager_, distributor_, tokenName, symbol, decimals) = abi.decode(params, (address, address, string, string, uint8));

54:             (tokenManager_, distributor_, tokenName, symbol, decimals) = abi.decode(params, (address, address, string, string, uint8));

63:             (, , , , , mintAmount, mintTo) = abi.decode(params, (address, address, string, string, uint8, uint256, address));

63:             (, , , , , mintAmount, mintTo) = abi.decode(params, (address, address, string, string, uint8, uint256, address));

```

```solidity
File: /contracts/its/token-manager/TokenManager.sol

84:         string calldata destinationChain,

110:         string calldata destinationChain,

138:         string calldata destinationChain,

```

```solidity
File: /contracts/its/utils/ExpressCallHandler.sol

48:         string memory sourceChain,

112:         string memory sourceChain,

173:         string memory sourceChain,

233:         string memory sourceChain,

```

```solidity
File: /contracts/its/utils/Multicall.sol

28:                 revert(string(result));

```

```solidity
File: /contracts/its/utils/StandardizedTokenDeployer.sol

53:         string calldata name,

54:         string calldata symbol,

```

### [GAS-19] USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT

#### Description:

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

#### **Proof Of Concept**

```solidity
File: /contracts/cgp/AxelarGateway.sol

635:         if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);

```

```solidity
File: /contracts/gmp-sdk/upgradable/Upgradable.sol

60:         if (params.length > 0) {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

161:         if (result.length > 0) {

```

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

91:         if (interchainCall.gas > 0) {

```

```solidity
File: /contracts/its/interchain-token-service/InterchainTokenService.sol

519:         if (version > 0) revert InvalidMetadataVersion(version);

690:         address distributor = distributorBytes.length > 0 ? distributorBytes.toAddress() : tokenManagerAddress;

714:         if (gasValue > 0) {

```

```solidity
File: /contracts/its/token-implementations/StandardizedToken.sol

64:             if (mintAmount > 0) {

```
