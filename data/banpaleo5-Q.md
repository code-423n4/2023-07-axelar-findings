# Table of Contents
| Number | Issues Details                                                                                         | Count |
| :----- | :----------------------------------------------------------------------------------------------------- | :---- |
| [L-1] | Incorporating Value Comparison in `Set` Functions | 8 |
| [L-2] | Possible Front-Running Vulnerability in Initializing Functions | 2 |
| [N-1] | Emitting Events for Significant Parameter Changes | 8 |
| [N-2] | Employing Cast to `bytes` or `bytes32` for Semantic Clarity | 3 |
| [N-3] | Update OpenZeppelin Contract Dependency to Latest Version | 1 |
| [N-4] | Encourage Uniform Naming Convention for Immutables | 1 |
| [N-5] | Events Should Include Emitted Parameters | 1 |
| [N-6] | Excessive Line Length | 3 |
| [N-7] | Removing Empty Blocks or Emitting Relevant Information | 6 |

## [L-1]</a><a name="L-1"> Incorporating Value Comparison in `Set` Functions

To enhance the functionality of set functions, it is recommended to incorporate a comparison mechanism to verify that the current value and the new value are distinct.

<details>
<summary><i>There are 8 instances of this issue:</i></summary>

```solidity
File: Operatable.sol
function setOperator(address operator_) external onlyOperator {
        _setOperator(operator_);
    }
}
```

```solidity
File: InterchainTokenService.sol
function setPaused(bool paused) external onlyOwner {
        _setPaused(paused);
    }
```

```solidity
File: Distributable.sol
function setDistributor(address distr) external onlyDistributor {
        _setDistributor(distr);
    }
}
```

```solidity
File: InterchainProposalExecutor.sol
function setWhitelistedProposalSender(
        string calldata sourceChain,
        address sourceSender,
        bool whitelisted
    ) external override onlyOwner {
        whitelistedSenders[sourceChain][sourceSender] = whitelisted;
        emit WhitelistedProposalSenderSet(sourceChain, sourceSender, whitelisted);
    }
```

```solidity
File: Pausable.sol
function setPaused(bool paused) external {
        _setPaused(paused);
    }
```

```solidity
File: InterchainProposalExecutor.sol
function setWhitelistedProposalCaller(
        string calldata sourceChain,
        address sourceCaller,
        bool whitelisted
    ) external override onlyOwner {
        whitelistedCallers[sourceChain][sourceCaller] = whitelisted;
        emit WhitelistedProposalCallerSet(sourceChain, sourceCaller, whitelisted);
    }
```

```solidity
File: Upgradable.sol
function setup(bytes calldata data) external override onlyProxy {
        _setup(data);
    }
```

```solidity
File: InterchainTokenService.sol
function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {
        uint256 length = tokenIds.length;
        if (length != flowLimits.length) revert LengthMismatch();
        for (uint256 i; i < length; ++i) {
            ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenIds[i]));
            tokenManager.setFlowLimit(flowLimits[i]);
        }
    }
```
</details>




## [L-2]</a><a name="L-2"> Possible Front-Running Vulnerability in Initializing Functions

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: ConstAddressDeployer.sol
function deployAndInit(
```

```solidity
File: InitProxy.sol
function init(
```
</details>



## [N-1]</a><a name="N-1"> Emitting Events for Significant Parameter Changes

It is important to emit events when modifying state variables to enable effective monitoring using off-chain monitoring tools. Emitting events facilitates the tracking of activities related to critical parameter changes.

<details>
<summary><i>There are 8 instances of this issue:</i></summary>

```solidity
File: AxelarGateway.sol
function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {
```

```solidity
File: StandardizedToken.sol
function setup(bytes calldata params) external override onlyProxy {
```

```solidity
File: TokenManager.sol
function setup(bytes calldata params) external override onlyProxy {
```

```solidity
File: FixedProxy.sol
function setup(bytes calldata setupParams) external {}
```

```solidity
File: InterchainTokenService.sol
function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {
```

```solidity
File: Distributable.sol
function setDistributor(address distr) external onlyDistributor {
```

```solidity
File: Upgradable.sol
function setup(bytes calldata data) external override onlyProxy {
```

```solidity
File: InterchainProposalExecutor.sol
function setWhitelistedProposalCaller(
```
</details>




## [N-2]</a><a name="N-2"> Employing Cast to `bytes` or `bytes32` for Semantic Clarity

To provide clearer semantic meaning and reduce reviewer confusion, it is recommended to use a cast to `bytes` or `bytes32` on a single argument rather than using `abi.encodePacked()`. This approach makes the intended operation more explicit and easier to comprehend.

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: TokenManagerDeployer.sol
bytes memory bytecode = abi.encodePacked(type(TokenManagerProxy).creationCode, args);
```

```solidity
File: AxelarGateway.sol
bytes32 salt = keccak256(abi.encodePacked(symbol));
```

```solidity
File: AxelarGateway.sol
bytes32 commandHash = keccak256(abi.encodePacked(commands[i]));
```
</details>



## [N-3]</a><a name="N-3"> Update OpenZeppelin Contract Dependency to Latest Version

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: package.json
    "@openzeppelin/contracts": "^4.9.2"
```
</details>





## [N-4]</a><a name="N-4"> Encourage Uniform Naming Convention for Immutables 

In the current code, certain immutables are named with capital letters and underscores, while others are not. For enhanced code quality, it's recommended to adopt a consistent naming convention for all immutables.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: InterchainTokenService.sol
address internal immutable implementationLockUnlock;
```

```solidity
File: InterchainTokenService.sol
address internal immutable implementationMintBurn;
```

```solidity
File: InterchainTokenService.sol
address internal immutable implementationLiquidityPool;
```

```solidity
File: InterchainTokenService.sol
IAxelarGasService public immutable gasService;
```

```solidity
File: InterchainTokenService.sol
IRemoteAddressValidator public immutable remoteAddressValidator;
```

```solidity
File: InterchainTokenService.sol
address public immutable tokenManagerDeployer;
```

```solidity
File: InterchainTokenService.sol
address public immutable standardizedTokenDeployer;
```

```solidity
File: InterchainTokenService.sol
Create3Deployer internal immutable deployer;
```

```solidity
File: InterchainTokenService.sol
bytes32 internal immutable chainNameHash;
```

```solidity
File: InterchainTokenService.sol
bytes32 internal immutable chainName;
```
</details>




## [N-5]</a><a name="N-5"> Events Should Include Emitted Parameters

Certain events lack emitted parameters. It is advisable to incorporate parameters reflecting changes in state or values when triggering events.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: Pausable.sol
emit TestEvent()
```
</details>





## [N-6]</a><a name="N-6"> Excessive Line Length

The lines in the code exceed the recommended maximum length, affecting readability and maintainability.

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: AxelarGateway.sol
/// @dev Storage slot with the address of the current implementation. `keccak256('eip1967.proxy.implementation') - 1`.
```

```solidity
File: AxelarGateway.sol
// AUDIT: If `newImplementation.setup` performs `selfdestruct`, it will result in the loss of _this_ implementation (thereby losing the gateway)
```

```solidity
File: AxelarGateway.sol
// Mark that this symbol is an external token, which is needed to differentiate between operations on mint and burn.
```

```solidity
File: InterchainTokenService.sol
// There could be a way to rewrite the following using assembly switch statements, which would be more gas efficient,
```

```solidity
File: InterchainTokenService.sol
// but accessing immutable variables and/or enum values seems to be tricky, and would reduce code readability.
```

```solidity
File: InterchainProposalExecutor.sol
// Whitelisted proposal callers. The proposal caller is the contract that calls the `InterchainProposalSender` at the source chain.
```

```solidity
File: InterchainProposalExecutor.sol
// Whitelisted proposal senders. The proposal sender is the `InterchainProposalSender` contract address at the source chain.
```

```solidity
File: InterchainProposalExecutor.sol
// You can add your own logic here to handle the failure of the target contract execution. The code below is just an example.
```
</details>



## [N-7]</a><a name="N-7"> Removing Empty Blocks or Emitting Relevant Information

It is recommended to remove empty blocks in the codebase or consider emitting relevant information within them. Empty blocks serve no functional purpose and can lead to confusion or unnecessary clutter in the code. If the blocks were intended for future implementation, it is suggested to emit relevant information or comments to provide clarity on their purpose or intended functionality. This promotes cleaner code and improves the overall maintainability and readability of the software.

<details>
<summary><i>There are 6 instances of this issue:</i></summary>

```solidity
File: InterchainTokenServiceProxy.sol
) FinalProxy(implementationAddress, owner, abi.encodePacked(operator)) {}
```

```solidity
File: TokenManagerMintBurn.sol
constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}
```

```solidity
File: InterchainGovernance.sol
receive() external payable {}
```

```solidity
File: TokenManagerLiquidityPool.sol
constructor(address interchainTokenService_) TokenManagerAddressStorage(interchainTokenService_) {}
```

```solidity
File: FixedProxy.sol
function setup(bytes calldata setupParams) external {}
```

```solidity
File: FinalProxy.sol
) Proxy(implementationAddress, owner, setupParams) {}
```
</details>


