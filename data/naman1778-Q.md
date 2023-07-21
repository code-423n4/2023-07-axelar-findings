## [N-01] Use a modifier for access control

There are 2 instances of this issue in 2 files:

Consider using a modifier to implement access control instead of inlining the condition/requirement in the function’s body.

    File: InitProxy.sol	

    46: if (msg.sender != owner) revert NotOwner();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol

    File: FinalProxy.sol	

    77: if (msg.sender != owner) revert NotOwner();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol

## [N-02] According to the syntax rules, use *=> mapping (* instead of *=> mapping(* using spaces as keyword

There are 3 instances of this issue in 2 files:

    File: MultisigBase.sol	

    27: mapping(uint256 => mapping(bytes32 => Voting)) public votingPerTopic;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol

    File: InterchainProposalExecutor.sol	

    24: mapping(string => mapping(address => bool)) public whitelistedCallers;

    27: mapping(string => mapping(address => bool)) public whitelistedSenders;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

## [N-03] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

There are 46 instances of this issue in 20 files:

    File: TimeLock.sol	

    95: assembly {
    96:     eta := sload(key)

    106: assembly {
    107:     sstore(key, eta)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol

    File: ConstAddressDeployer.sol	

    84:  assembly {
    85:      deployedAddress_ := create2(0, add(bytecode, 32), mload(bytecode), salt)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

    File: Create3.sol	

    24: assembly {
    25:     deployed := create(0, add(bytecode, 32), mload(bytecode))

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol

    File: BaseProxy.sol	

    23: assembly {
    24:     implementation_ := sload(_IMPLEMENTATION_SLOT)
    
    48: assembly {
    49:     calldatacopy(0, 0, calldatasize())

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/BaseProxy.sol

    File: Proxy.sol	

    35: assembly {
    36:     sstore(_IMPLEMENTATION_SLOT, implementationAddress)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol

    File: Upgradable.sol	

    40: assembly {
    41:     implementation_ := sload(_IMPLEMENTATION_SLOT)

    68: assembly {
    69:     sstore(_IMPLEMENTATION_SLOT, newImplementation)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol

    File: InitProxy.sol	

    21: assembly {
    22:     sstore(_OWNER_SLOT, caller())

    42: assembly {
    43:     owner := sload(_OWNER_SLOT)

    52: assembly {
    53:     sstore(_IMPLEMENTATION_SLOT, implementationAddress)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol

    File: FinalProxy.sol	

    74: assembly {
    75:     owner := sload(_OWNER_SLOT)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol

    File: FixedProxy.sol	

    40: assembly {
    41:     calldatacopy(0, 0, calldatasize())

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol

    File: InterchainTokenService.sol	

    603: assembly {
    604:     commandId := calldataload(4)

    630: assembly {
    631:     commandId := calldataload(4)

    873: assembly {
    874:     data.length := sub(metadata.length, 4)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: AddressBytesUtils.sol	

    20: assembly {
    21:     addr := mload(add(bytesAddress, 20))

    33: assembly {
    34:     mstore(add(bytesAddress, 20), addr4

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/libraries/AddressBytesUtils.sol

    File: TokenManagerProxy.sol	

    75: assembly {
    76:     calldatacopy(0, 0, calldatasize())

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol

    File: TokenManagerAddressStorage.sol	

    29: assembly {
    30:     tokenAddress_ := sload(TOKEN_ADDRESS_SLOT)

    39: assembly {
    40:     sstore(TOKEN_ADDRESS_SLOT, tokenAddress_)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol

    File: TokenManagerLiquidityPool.sol	

    48: assembly {
    49:     sstore(LIQUIDITY_POOL_SLOT, liquidityPool_)

    58: assembly {
    59:     liquidityPool_ := sload(LIQUIDITY_POOL_SLOT)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

    File: Distributable.sol	

    30: assembly {
    31:     distr := sload(DISTRIBUTOR_SLOT)

    40: assembly {
    41:     sstore(DISTRIBUTOR_SLOT, distributor_)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Distributable.sol

    File: ExpressCallHandler.sol	

    88: assembly {
    89:     prevExpressCaller := sload(slot)

    92: assembly {
    93:     sstore(slot, expressCaller)

    130: assembly {
    131:     prevExpressCaller := sload(slot)

    134: assembly {
    135:     sstore(slot, expressCaller)

    155: assembly {
    156:     expressCaller := sload(slot)

    189: assembly {
    190:     expressCaller := sload(slot)

    209: assembly {
    210:     expressCaller := sload(slot)

    213: assembly {
    214:     sstore(slot, 0)

    249: assembly {
    250:     expressCaller := sload(slot)

    253: assembly {
    254:     sstore(slot, 0)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol

    File: FlowLimit.sol	

    25: assembly {
    26:     flowLimit := sload(FLOW_LIMIT_SLOT)

    35: assembly {
    36:     sstore(FLOW_LIMIT_SLOT, flowLimit)

    66: assembly {
    67:     flowOutAmount := sload(slot)

    78: assembly {
    79:     flowInAmount := sload(slot)

    97: assembly {
    98:     flowToAdd := sload(slotToAdd)

    102: assembly {
    103:     sstore(slotToAdd, add(flowToAdd, flowAmount))

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol

    File: Operatable.sol	

    30: assembly {
    31:     operator_ := sload(OPERATOR_SLOT)

    40: assembly {
    41:     sstore(OPERATOR_SLOT, operator_)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Operatable.sol

    File: Pausable.sol	

    30: assembly {
    31:     paused := sload(PAUSE_SLOT)

    42: assembly {
    43:     sstore(PAUSE_SLOT, paused)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Pausable.sol

## [N-04] Inconsistent Solidity Versions

Except the below mentioned contract all other contracts are using 0.8.0 version of solidity:

There is 1 instance of this issue in 1 file:

    File: AxelarGateway.sol	

    3: pragma solidity ^0.8.9;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

## [N-05] Event is missing *indexed* fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (threefields). Each *event* should use three *indexed* fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

There are 18 instances of this issue in 9 files:

    File: IAxelarServiceGovernance.sol	

    15: event MultisigApproved(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);

    16: event MultisigCancelled(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);

    17: event MultisigExecuted(bytes32 indexed proposalHash, address indexed targetContract, bytes callData, uint256 nativeValue);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IAxelarServiceGovernance.sol

    File: IMultisigBase.sol	

    22: event SignersRotated(address[] newAccounts, uint256 newThreshold);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IMultisigBase.sol

    File: IInterchainProposalExecutor.sol	

    7: event WhitelistedProposalCallerSet(string indexed sourceChain, address indexed sourceCaller, bool whitelisted);

    10: event WhitelistedProposalSenderSet(string indexed sourceChain, address indexed sourceSender, bool whitelisted);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol

    File: IInterchainTokenService.sol	

    32: event TokenSent(bytes32 tokenId, string destinationChain, bytes destinationAddress, uint256 indexed amount);

    33: event TokenSentWithData(
    34:     bytes32 tokenId,
    35:     string destinationChain,
    36:     bytes destinationAddress,
    37:     uint256 indexed amount,
    38:     address indexed sourceAddress,
    39:     bytes data
    40: );

    67: event TokenManagerDeployed(bytes32 indexed tokenId, TokenManagerType indexed tokenManagerType, bytes params);

    68: event StandardizedTokenDeployed(
    69:     bytes32 indexed tokenId,
    70:     string name,
    71:     string symbol,
    72:     uint8 decimals,
    73:     uint256 mintAmount,
    74:     address mintTo
    75: );

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol

    File: IRemoteAddressValidator.sol	

    14: event TrustedAddressAdded(string souceChain, string sourceAddress);

    15: event TrustedAddressRemoved(string souceChain);

    16: event GatewaySupportedChainAdded(string chain);

    17: event GatewaySupportedChainRemoved(string chain);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol

    File: IDistributable.sol	

    8: event DistributorChanged(address distributor);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IDistributable.sol

    File: IFlowLimit.sol	

    8: event FlowLimitSet(uint256 flowLimit);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IFlowLimit.sol

    File: IOperatable.sol	

    8: event OperatorChanged(address operator);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IOperatable.sol

    File: IPausable.sol	

    11: event PausedSet(bool paused);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IPausable.sol

## [N-06] Contract Owner Has Too Many Privileges

The owner of the contracts has too many privileges relative to standard users. The consequence is disastrous if the contract owner’s private key has been compromised. And, in the event the key was lost or unrecoverable, no implementation upgrades and system parameter updates will ever be possible.

For a project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure; and, here are some of the incidents that could possibly transpire:

Transfer ownership and mess up with all the setter functions, hijacking the entire protocol.

Consider:

-- splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
-- clearly documenting the functions and implementations the owner can change,
-- documenting the risks associated with privileged users and single points of failure, and
-- ensuring that users are aware of all the risks associated with the system.

There are 14 instances of this issue in 5 files:

    File: AxelarGateway.sol	

    385: function deployToken(bytes calldata params, bytes32) external onlySelf {

    421: function mintToken(bytes calldata params, bytes32) external onlySelf {

    427: function burnToken(bytes calldata params, bytes32) external onlySelf {

    455: function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf {

    469: function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf {

    495: function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: InterchainProposalExecutor.sol	

    96: function setWhitelistedProposalCaller(
    97:     string calldata sourceChain,
    98:     address sourceCaller,
    99:     bool whitelisted
    100: ) external override onlyOwner {

    107: function setWhitelistedProposalSender(
    108:     string calldata sourceChain,
    109:     address sourceSender,
    110:     bool whitelisted
    111: ) external override onlyOwner {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

    File: Upgradable.sol	

    52: function upgrade(
    53:     address newImplementation,
    54:     bytes32 newImplementationCodeHash,
    55:     bytes calldata params
    56: ) external override onlyOwner {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol

    File: InterchainTokenService.sol	

    547: function setPaused(bool paused) external onlyOwner {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: RemoteAddressValidator.sol	

    83: function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

    95: function removeTrustedAddress(string calldata chain) external onlyOwner {

    106: function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

    119: function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

## [N-07] Use delete to clear variables instead of zero assignment

You can use the delete keyword instead of setting the variable as zero.

There is 1 instance of this issue in 1 file:

    File: MultisigBase.sol	
   
    66: voting.voteCount = 0;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol