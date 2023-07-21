## [Low] Create3.sol doesn't support deploy with value

```solidiy
    function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) { // @audit-issue low don't support value in creation, see https://github.com/0xsequence/create3/blob/master/contracts/Create3.sol
        deployed = deployedAddress(address(this), salt);

        if (deployed.isContract()) revert AlreadyDeployed();
        if (bytecode.length == 0) revert EmptyBytecode();

        // Deploy using create2
        CreateDeployer deployer = new CreateDeployer{ salt: salt }();

        if (address(deployer) == address(0)) revert DeployFailed();

        deployer.deploy(bytecode);
    }
```

## [Low] BaseProxy should use keccak256('owner') - 1 to prevent pre-image attack

```solidity
abstract contract BaseProxy is IProxy {
    // bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1)
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
    // keccak256('owner') @audit-issue low should use -1 to protect pre-image attack
    bytes32 internal constant _OWNER_SLOT = 0x02016836a56b71f0d02689e69e326f4f4c1b9057164ef592671cf0d37c8040c0;
```

## [Low] expressReceiveTokenWithData is vulnerable to reentrancy attack, not follow CEI pattern

```solidity
    function expressReceiveTokenWithData( // @audit-issue low not follow CEI, reentrancy
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId
    ) external {
        if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

        address caller = msg.sender;
        ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
        IERC20 token = IERC20(tokenManager.tokenAddress());

        SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

        _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount); // <-- reentrancy

        _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller); // <-- modify state
    }
```

## [non-critical] ProposalCancelled event don't need eta parameter

```solidity
            emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta); //@audit-issue non-critical eta is passed, not zero
```

## [non-critical] should use _getProposalHash instead of redundant code
```solidity
    function executeProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue)); // @audit-issue non-critical should use _getProposalHash, redundant code
```

```solidity
    function _processCommand(
        uint256 commandId,
        address target,
        bytes memory callData,
        uint256 nativeValue,
        uint256 eta
    ) internal override {
        if (commandId > uint256(type(ServiceGovernanceCommand).max)) {
            revert InvalidCommand();
        }

        ServiceGovernanceCommand command = ServiceGovernanceCommand(commandId);
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue)); // @audit-issue non-critical should use _getProposalHash, redundant code
```

## [non-critical] multicall is not used

```solidity
import { ExpressCallHandler } from '../utils/ExpressCallHandler.sol';
import { Pausable } from '../utils/Pausable.sol';
import { Operatable } from '../utils/Operatable.sol';
import { Multicall } from '../utils/Multicall.sol'; // @audit-issue non-critical multicall is not used
```

## [non-critical] transmitInterchainTransfer() sendToken() lack netspec @param for metadata

```solidity
    /**
     * @notice Calls the service to initiate the a cross-chain transfer after taking the appropriate amount of tokens from the user.
     * @param destinationChain the name of the chain to send tokens to.
     * @param destinationAddress the address of the user to send tokens to.
     * @param amount the amount of tokens to take from msg.sender.
     */
    // @audit-issue non-critical lack @param for metadata
    function sendToken(
        string calldata destinationChain,
        bytes calldata destinationAddress,
        uint256 amount,
        bytes calldata metadata
```

```solidity
    /**
     * @notice Calls the service to initiate the a cross-chain transfer after taking the appropriate amount of tokens from the user. This can only be called by the token itself.
     * @param sender the address of the user paying for the cross chain transfer.
     * @param destinationChain the name of the chain to send tokens to.
     * @param destinationAddress the address of the user to send tokens to.
     * @param amount the amount of tokens to take from msg.sender.
     */
    // @audit-issue non-criticle no @param for metadata
    function transmitInterchainTransfer(
        address sender,
        string calldata destinationChain,
        bytes calldata destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) external payable virtual onlyToken {
```

## [non-critical] _giveToken typo, "from" should be "to"

```solidity
    /**
     * @notice Transfers tokens from this contract to a specific address.
     * Must be overridden in the inheriting contract.
     * @param from The address to which the tokens will be sent
     * @param amount The amount of tokens to send
     * @return uint amount of tokens sent
     */
    function _giveToken(address from, uint256 amount) internal virtual returns (uint256); // @audit-issue non-critical "from" should be "to"
```

## [non-critical] implementationType() should use enum type instead of number

In TokenManagerLiquidityPool, TokenManagerLockUnlock, TokenManagerMintBurn. 

```solidity
    function implementationType() external pure returns (uint256) {
        return 1; // @audit-issue non-critical should return TokenManagerType.MINT_BURN
    }
``` 

## [non-critical] _addFlow() lacks natspec @param for flowLimit

```solidity
    /**
     * @dev Adds a flow amount while ensuring it does not exceed the flow limit
     * @param slotToAdd The slot to add the flow to
     * @param slotToCompare The slot to compare the flow against
     * @param flowAmount The flow amount to add
     */
    // @audit-issue non-critical lack @param for flowLimit
    function _addFlow(
        uint256 flowLimit,
        uint256 slotToAdd,
        uint256 slotToCompare,
        uint256 flowAmount
    ) internal {
```