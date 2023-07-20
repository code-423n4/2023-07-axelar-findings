## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Implementation.sol

```solidity
// place this modifier before the constructor
26:    modifier onlyProxy() {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

```solidity
// place this state variable before the constructor
22:    uint256 internal constant TOKEN_ADDRESS_SLOT = 0xc4e632779a6a7838736dd7e5e6a0eadf171dd37dfb6230720e265576dfcf42ba;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol

```solidity
// place these modifier before the constructor
35:    modifier onlyService() {
43:    modifier onlyToken() {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol

```solidity
// place these modifier before the constructor
123:    modifier onlyRemoteService(string calldata sourceChain, string calldata sourceAddress) {
132:    modifier onlyTokenManager(bytes32 tokenId) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol

```solidity
// place these modifier before the constructor
29:    modifier onlyProxy() {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol

```solidity
// place these modifier before the constructor
72:    modifier onlySelf() {
78:    modifier onlyGovernance() {
87:    modifier onlyMintLimiter() {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol

```solidity
// place this modifier before the constructor
44:    modifier onlySigners() {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Operatable.sol

```solidity
// external functions should come before all the other
51:    function setOperator(address operator_) external onlyOperator {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol

```solidity
// internal functions are coming after public and internal ones, should come before
63:    function getFlowOutAmount() external view returns (uint256 flowOutAmount) {
75:    function getFlowInAmount() external view returns (uint256 flowInAmount) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol

```solidity
// public functions are coming after internal functions
148:    function getExpressReceiveToken(
171:    function getExpressReceiveTokenWithData(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol

```solidity
// public function should come after external ones
29:    function distributor() public view returns (address distr) {

// internal functions should come for last since there are no private functions in this contract
39:    function _setDistributor(address distributor_) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

```solidity
// internal functions coming before external and public ones
36:    function _setup(bytes calldata params) internal override {
47:    function _setLiquidityPool(address liquidityPool_) internal {

// public functions should come after external and before internal ones
57:    function liquidityPool() public view returns (address liquidityPool_) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol

```solidity
// public functions should now come after external ones
40:    function getTokenManager() public view override returns (ITokenManager) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

```solidity
// internal functions should come for last since there are no private ones in this contract
40:    function _setup(bytes calldata params) internal override {
52:    function _lowerCase(string memory s) internal pure returns (string memory) {

// external functions should come first now
69:    function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool) {
95:    function removeTrustedAddress(string calldata chain) external onlyOwner {
106:    function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {
119:    function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {
133:    function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/TokenManagerProxy.sol

```solidity
// public functions should come after external functions after new best practices 
44:    function implementation() public view returns (address impl) {

// internal functions should come for last since there are no private ones in this contract
54:    function _getImplementation(IInterchainTokenService interchainTokenServiceAddress_, uint256 implementationType_)

// receive and fallback should come right after the constructor
72:    fallback() external payable virtual {
94:    receive() external payable virtual {}
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol

```solidity
// public functions should come after external functions after new best practices 
152:    function getChainName() public view returns (string memory name) {
151:    function getTokenManagerAddress(bytes32 tokenId) public view returns (address tokenManagerAddress) {
170:    function getValidTokenManagerAddress(bytes32 tokenId) public view returns (address tokenManagerAddress) {
191:    function getStandardizedTokenAddress(bytes32 tokenId) public view returns (address tokenAddress) {
202:    function getCanonicalTokenId(address tokenAddress) public view returns (bytes32 tokenId) {
213:    function getCustomTokenId(address sender, bytes32 salt) public pure returns (bytes32 tokenId) {
241:    function getParamsLockUnlock(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
251:    function getParamsMintBurn(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
262:    function getParamsLiquidityPool(
325:    function deployRemoteCanonicalToken(
343:    function deployCustomTokenManager(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol

```solidity
// public functions should come after external functions after new best practices
21:    function getTokenManager() public view virtual returns (ITokenManager tokenManager);
32:    function tokenManagerRequiresApproval() public view virtual returns (bool);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FixedProxy.sol

```solidity
// receive and fallback functions should come before all the other functions 
37:    fallback() external payable virtual {
59:    receive() external payable virtual {}
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol

```solidity
// internal function coming before public one
57:    function _finalImplementation() internal view virtual returns (address implementation_) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol

```solidity
// public functions should come after external functions after new best practices
39:    function implementation() public view returns (address implementation_) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/BaseProxy.sol

```solidity
// public functions should come after external functions after new best practices
22:    function implementation() public view virtual returns (address implementation_) {

// receive and fallback functions should come before all the external ones 
46:    fallback() external payable virtual {
67:    receive() external payable virtual {}
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

```solidity
// external functions should come before the internal ones
92:    function setWhitelistedProposalCaller(
107:    function setWhitelistedProposalSender(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/Multisig.sol

```solidity
// place this right after the constructor
41:    receive() external payable {}
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/ITokenManagerDeployer.sol

```solidity
18:     function deployer() external view returns (Create3Deployer);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IStandardizedTokenDeployer.sol

```solidity
// @return missing
18:    function deployer() external view returns (Create3Deployer);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IPausable.sol

```solidity
11:    event PausedSet(bool paused);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IOperatable.sol

```solidity
5: interface IOperatable {
8:    event OperatorChanged(address operator);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IImplementation.sol

```solidity
5: interface IImplementation {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IFlowLimit.sol

```solidity
8:    event FlowLimitSet(uint256 flowLimit);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IExpressCallHandler.sol

```solidity
5: interface IExpressCallHandler {
9:    event ExpressReceive(
16:    event ExpressExecutionFulfilled(
24:    event ExpressReceiveWithData(
34:    event ExpressExecutionWithDataFulfilled(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IDistributable.sol

```solidity
5: interface IDistributable {
8:    event DistributorChanged(address distributor);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/ITokenManager.sol

```solidity
// @return missing
31:    function implementationType() external pure returns (uint256);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

```solidity
28:    function implementationType() external pure returns (uint256) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerMintBurn.sol

```solidity
25:    function implementationType() external pure returns (uint256) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

```solidity
24:    function implementationType() external pure returns (uint256) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedTokenMintBurn.sol

```solidity
7:  contract StandardizedTokenMintBurn is StandardizedToken {
8:    function tokenManagerRequiresApproval() public pure override returns (bool) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedTokenLockUnlock.sol

```solidity
7:  contract StandardizedTokenLockUnlock is StandardizedToken {
8:    function tokenManagerRequiresApproval() public pure override returns (bool) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol

```solidity
// @return missing
32:    function contractId() external pure returns (bytes32) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IRemoteAddressValidator.sol

```solidity
14:    event TrustedAddressAdded(string souceChain, string sourceAddress);
15:    event TrustedAddressRemoved(string souceChain);
16:    event GatewaySupportedChainAdded(string chain);
17:    event GatewaySupportedChainRemoved(string chain);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

```solidity
40:    function _setup(bytes calldata params) internal override {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol

```solidity
// @return missing
31:    function contractId() external pure returns (bytes32) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol

```solidity
// @param operator missing
19:    constructor(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol

```solidity
14: interface IInterchainTokenService is ITokenManagerType, IExpressCallHandler, IAxelarExecutable, IPausable, IMulticall {
32:    event TokenSent(bytes32 tokenId, string destinationChain, bytes destinationAddress, uint256 indexed amount);
33:    event TokenSentWithData(
41:    event TokenReceived(bytes32 indexed tokenId, string sourceChain, address indexed destinationAddress, uint256 indexed amount);
42:    event TokenReceivedWithData(
50:    event RemoteTokenManagerDeploymentInitialized(
57:    event RemoteStandardizedTokenAndManagerDeploymentInitialized(
67:    event TokenManagerDeployed(bytes32 indexed tokenId, TokenManagerType indexed tokenManagerType, bytes params);
68:    event StandardizedTokenDeployed(
76:    event CustomTokenIdClaimed(bytes32 indexed tokenId, address indexed deployer, bytes32 indexed salt);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/Bytes32String.sol

```solidity
5:  library StringToBytes32 {
8    function toBytes32(string memory str) internal pure returns (bytes32) {
23:  library Bytes32ToString {
24:    function toTrimmedString(bytes32 stringData) internal pure returns (string memory converted) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol

```solidity
// @return missing
144:    function contractId() external pure returns (bytes32) {

555:    function _setup(bytes calldata params) internal override {
559:    function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)
726:    function _validateToken(address tokenAddress)
872:    function _decodeMetadata(bytes calldata metadata) internal pure returns (uint32 version, bytes calldata data) {
880:    function _expressExecuteWithInterchainTokenToken(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IFinalProxy.sol

```solidity
8:  interface IFinalProxy is IProxy {
9:    function isFinal() external view returns (bool);
11:    function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) external returns (address);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IInitProxy.sol

```solidity
8:  interface IInitProxy is IProxy {
9:    function init(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IUpgradable.sol

```solidity
8:  interface IUpgradable is IOwnable {
14:    event Upgraded(address indexed newImplementation);
16:    function implementation() external view returns (address);
18:    function upgrade(
24:    function setup(bytes calldata data) external;
26:    function contractId() external pure returns (bytes32);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IProxy.sol

```solidity
6:  interface IProxy {
13:    function implementation() external view returns (address implementation_);
15:    function setup(bytes calldata setupParams) external;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol

```solidity
15:    event Deployed(bytes32 indexed bytecodeHash, bytes32 indexed salt, address indexed deployedAddress);

// @params and return missing
67:    function deployedAddress(address sender, bytes32 salt) external view returns (address) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

```solidity
5:  contract ConstAddressDeployer {
10:    event Deployed(bytes32 indexed bytecodeHash, bytes32 indexed salt, address indexed deployedAddress);

// @params and @return missing
58:    function deployedAddress(

80:    function _deploy(bytes memory bytecode, bytes32 salt) internal returns (address deployedAddress_) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/lib/InterchainCalls.sol

```solidity
5:  library InterchainCalls {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

```solidity
12:    event ProposalExecuted(bytes32 indexed payloadHash);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

```solidity
29:    constructor(address _gateway, address _owner) AxelarExecutable(_gateway) {

// @param missing
156:    function _onTargetExecutionFailed(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

```solidity
7:  interface IInterchainProposalSender {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol

```solidity
42:    constructor(address _gateway, address _gasService) {
88:    function _sendProposal(InterchainCalls.InterchainCall memory interchainCall) internal {
104:    function revertIfInvalidFee(InterchainCalls.InterchainCall[] calldata interchainCalls) private {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol

```solidity
17:  contract AxelarGateway is IAxelarGateway, IGovernable, AdminMultisigBase {
64:    constructor(address authModule_, address tokenDeployerImplementation_) {
72:    modifier onlySelf() {
78:    modifier onlyGovernance() {
87:    modifier onlyMintLimiter() {
97:    function sendToken(
107:    function callContract(
115:    function callContractWithToken(
126:    function isContractCallApproved(
137:    function isContractCallAndMintApproved(
151:    function validateContractCall(
162:    function validateContractCallAndMint(
183:    function authModule() public view override returns (address) {
187:    function governance() public view override returns (address) {
191:    function mintLimiter() public view override returns (address) {
195:    function tokenDeployer() public view returns (address) {
199:    function tokenMintLimit(string memory symbol) public view override returns (uint256) {
203:    function tokenMintAmount(string memory symbol) public view override returns (uint256) {
219:    function adminThreshold(uint256) external pure override returns (uint256) {
224:    function admins(uint256) external pure override returns (address[] memory) {
228:    function implementation() public view override returns (address) {
232:    function tokenAddresses(string memory symbol) public view override returns (address) {
238:    function tokenFrozen(string memory) external pure override returns (bool) {
242:    function isCommandExecuted(bytes32 commandId) public view override returns (bool) {
246:    function contractId() public pure returns (bytes32) {
254:    function transferGovernance(address newGovernance) external override onlyGovernance {
260:    function transferMintLimiter(address newMintLimiter) external override onlyMintLimiter {
266:    function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {
280:    function upgrade(
307:    function setup(bytes calldata params) external override {
323:    function execute(bytes calldata input) external override {
385:    function deployToken(bytes calldata params, bytes32) external onlySelf {
321:    function mintToken(bytes calldata params, bytes32) external onlySelf {
427:    function burnToken(bytes calldata params, bytes32) external onlySelf {
455:    function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf {
469:    function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf {
495:    function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {
505:    function _hasCode(address addr) internal view returns (bool) {
512:    function _mintToken(
530:    function _burnTokenFrom(
556:    function _getTokenMintLimitKey(string memory symbol) internal pure returns (bytes32) {
560:    function _getTokenMintAmountKey(string memory symbol, uint256 day) internal pure returns (bytes32) {
565:    function _getTokenTypeKey(string memory symbol) internal pure returns (bytes32) {
569:    function _getTokenAddressKey(string memory symbol) internal pure returns (bytes32) {
573:    function _getIsCommandExecutedKey(bytes32 commandId) internal pure returns (bytes32) {
577:    function _getIsContractCallApprovedKey(
587:    function _getIsContractCallApprovedWithMintKey(
615:    function _getCreate2Address(bytes32 salt, bytes32 codeHash) internal view returns (address) {
619:    function _getTokenType(string memory symbol) internal view returns (TokenType) {
627:    function _setTokenMintLimit(string memory symbol, uint256 limit) internal {
633:    function _setTokenMintAmount(string memory symbol, uint256 amount) internal {
640:    function _setTokenType(string memory symbol, TokenType tokenType) internal {
644:    function _setTokenAddress(string memory symbol, address tokenAddress) internal {
648:    function _setCommandExecuted(bytes32 commandId, bool executed) internal {
652:    function _setContractCallApproved(
662:    function _setContractCallApprovedWithMint(
677:    function _setImplementation(address newImplementation) internal {
681:    function _transferGovernance(address newGovernance) internal {
687:    function _transferMintLimiter(address newMintLimiter) internal {


// @return missing
209:    function allTokensFrozen() external pure override returns (bool) {
214:    function adminEpoch() external pure override returns (uint256) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol

```solidity
7:  contract Caller is ICaller {

// @params missing
11:    function _call(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/ICaller.sol

```solidity
5:  interface ICaller {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol

```solidty
// @params and @return missing
92:    function _getTimeLockEta(bytes32 hash) internal view returns (uint256 eta) {

// @params missing
103:    function _setTimeLockEta(bytes32 hash, uint256 eta) private {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol

```solidity
13:    struct Voting {
18:    struct Signers {

// @params missing
103:    function isSigner(address account) external view override returns (bool) {
111:    function hasSignerVoted(address account, bytes32 topic) external view override returns (bool) {
119:    function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
149:    function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol

```solidity
20:    event MultisigOperationExecuted(bytes32 indexed operationHash);
22:    event SignersRotated(address[] newAccounts, uint256 newThreshold);

// @param missing
50:    function isSigner(address account) external view returns (bool);
56:    function hasSignerVoted(address account, bytes32 topic) external view returns (bool);
62:    function getSignerVotesCount(bytes32 topic) external view returns (uint256);
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IAxelarServiceGovernance.sol

```solidity
15:    event MultisigApproved(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
16:    event MultisigCancelled(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);
17:    event MultisigExecuted(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);

// @param missing
24:    function executeMultisigProposal(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol

```solidity
// @params and @return missing
143:    function _getProposalHash(

// @params missing
155:    function _executeWithToken(
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IInterchainGovernance.sol

```solidity
18:    event ProposalScheduled(bytes32 indexed proposalHash, address indexed target, bytes callData, uint256 value, uint256 indexed eta);
19:    event ProposalCancelled(bytes32 indexed proposalHash, address indexed target, bytes callData, uint256 value, uint256 indexed eta);
20:    event ProposalExecuted(bytes32 indexed proposalHash, address indexed target, bytes callData, uint256 value, uint256 indexed timestamp);
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Implementation.sol

```solidity
// private and internal `variables` should preppend with `underline`
13:    address private immutable implementationAddress;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/RemoteAddressValidatorProxy.sol

```solidity
// private and internal `functions` should preppend with `underline`
30:    function contractId() internal pure override returns (bytes32) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol

```solidity
// private and internal `functions` should preppend with `underline`
29:    function contractId() internal pure override returns (bytes32) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/libraries/AddressBytesUtils.sol

```solidity
// private and internal `functions` should preppend with `underline`
17:    function toAddress(bytes memory bytesAddress) internal pure returns (address addr) {
30:    function toBytes(address addr) internal pure returns (bytes memory bytesAddress) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/Bytes32String.sol

```solidity
// private and internal `functions` should preppend with `underline`
8:    function toBytes32(string memory str) internal pure returns (bytes32) {
24:    function toTrimmedString(bytes32 stringData) internal pure returns (string memory converted) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol

```solidity
//  private and internal `variables` should preppend with `underline`
51:    address internal immutable implementationLockUnlock;
52:    address internal immutable implementationMintBurn;
53:    address internal immutable implementationLiquidityPool;
58:    Create3Deployer internal immutable deployer;
59:    bytes32 internal immutable chainNameHash;
60:    bytes32 internal immutable chainName;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol

```solidity
// private and internal `variables` should preppend with `underline`
15:    address internal immutable implementationAddress;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/BaseProxy.sol

```solidity
// private and internal `functions` should preppend with `underline`
39:    function contractId() internal pure virtual returns (bytes32) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol

```solidity
// private and internal `functions` should preppend with `underline`
51:    function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
71:    function deployedAddress(address sender, bytes32 salt) internal pure returns (address deployed) {
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol

```solidity
// private and internal `functions` should preppend with `underline`
104:    function revertIfInvalidFee(InterchainCalls.InterchainCall[] calldata interchainCalls) private {
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/ITokenManagerDeployer.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/TokenManagerDeployer.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IStandardizedTokenDeployer.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/StandardizedTokenDeployer.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IPausable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Pausable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IOperatable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Operatable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IMulticall.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Multicall.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IImplementation.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Implementation.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IFlowLimit.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IExpressCallHandler.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code˜
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/ExpressCallHandler.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code˜
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IDistributable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code˜
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code˜
3: pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/ITokenManager.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3: pragma solidity ^0.8.0;
```

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerMintBurn.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```
 
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IStandardizedToken.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedTokenMintBurn.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedTokenLockUnlock.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IRemoteAddressValidator.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/ITokenManagerProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/TokenManagerProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IStandardizedTokenProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/RemoteAddressValidatorProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/libraries/AddressBytesUtils.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/Bytes32String.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainToken.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FixedProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IFinalProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IInitProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IUpgradable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Upgradable.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/Proxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/BaseProxy.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/lib/InterchainCalls.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
2:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol

```solidity
// this version is different from the rest of the repo. also lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.9;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/util/Caller.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/ICaller.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/ITimeLock.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/Multisig.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisig.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IAxelarServiceGovernance.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IInterchainGovernance.sol

```solidity
// lock the pragma to a fixed version without ^ for a safer code
3:  pragma solidity ^0.8.0;
```

---

### Initialized variables [6]

- Variables are default initialized with 0 for `uint / int`, 0x0 for `address` and false for `bool`

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol

```solidity
// initializing variable with its default value
118:        uint32 version = 0;
```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol

```solidity
// initializing variables with its default value
63:        for (uint256 i = 0; i < interchainCalls.length; ) {
105:        uint256 totalGas = 0;
106:        for (uint256 i = 0; i < interchainCalls.length; ) {
```

