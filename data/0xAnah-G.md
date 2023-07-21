# Axelar Gas Optimizations

## [G-01] Unused Imports
Some files were imported and were not used these files costs gas during deployment and generally this is bad coding practice

### 7 Instances
In the file below `IProxy` was imported with the statement `import { IProxy } from '../interfaces/IProxy.sol'`but was not used. consider removing this import.
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol#L5


In the file below `IAxelarGateway` was imported with the statement `import { IAxelarGateway } from '../../gmp-sdk/interfaces/IAxelarGateway.sol'`but was not used, consider removing this import.
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L5


In the file below `ITokenManagerDeployer` was imported with the statement `import { ITokenManagerDeployer } from './ITokenManagerDeployer.sol'`but was not used, consider removing this import.
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol#L9


In the file below `IERC20BurnableMintable` was imported with the statement `import { IERC20BurnableMintable } from '../interfaces/IERC20BurnableMintable.sol'`but was not used, consider removing this import.
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol#L5


In the file below `IAxelarGateway` was imported with the statement `import { IAxelarGateway } from '../../gmp-sdk/interfaces/IAxelarGateway.sol'`but was not used, consider removing this import.
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol#L5


In the file below `IERC20` was imported with the statement `import { IERC20 } from '../../../gmp-sdk/interfaces/IERC20.sol'`but was not used, consider removing this import.
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol#L6


In the file below `IAxelarGateway` was imported with the statement `import { IAxelarGateway } from '../../../gmp-sdk/interfaces/IAxelarGateway.sol';`but was not used, consider removing this import.
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerAddressStorage.sol#L7


## [G-02] Making constant and Immutable variables private will save gas during deployment. 
 When constants and immutable are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple.

### 5 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L23-#L24
```solidity
23:    bytes32 public immutable governanceChainHash;
24:    bytes32 public immutable governanceAddressHash;
```
From the contract above during compilation getter functions would be created for the public immutable variables `governanceChainHash` and `governanceAddressHash` this leads to increased deployment cost. The deployment cost could be reduced if the immutable variables are made internal or private thereby eliminating the need for the compiler to create getter functions for them, if these variable are needed externally we could have a single getter function that returns the values of these immutable variables as a tuple. `3213` gas units could be save if the code is refactored to:
```solidity
    bytes32 internal immutable governanceChainHash;
    bytes32 internal immutable governanceAddressHash;

    function getConstants() public view returns(bytes32, bytes32) {
        return (governanceChainHash, governanceAddressHash);
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L54-#L57
```solidity
54:    IAxelarGasService public immutable gasService;
55:    IRemoteAddressValidator public immutable remoteAddressValidator;
56:    address public immutable tokenManagerDeployer;
57:    address public immutable standardizedTokenDeployer;
```
From the contract above during compilation getter functions would be created for the public immutable variables `gasService`, `remoteAddressValidator`, `tokenManagerDeployer` and `standardizedTokenDeployer` this leads to increased deployment cost. The deployment cost could be reduced if the immutable variables are made internal or private thereby eliminating the need for the compiler to create getter functions for them, if these variable are needed externally we could have a single getter function that returns the values of these immutable variables as a tuple. `8515` gas units could be save if the code is refactored to:
```solidity
    IAxelarGasService internal immutable gasService;
    IRemoteAddressValidator internal immutable remoteAddressValidator;
    address internal immutable tokenManagerDeployer;
    address internal immutable standardizedTokenDeployer;

    function getConstants() public view returns(
        IAxelarGasService,
        IRemoteAddressValidator,
        address,
        address
    ) {
        return (gasService, remoteAddressValidator, tokenManagerDeployer, standardizedTokenDeployer);
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/proxies/TokenManagerProxy.sol#L14-#L16
```solidity
14:    IInterchainTokenService public immutable interchainTokenServiceAddress;
15:    uint256 public immutable implementationType;
16:    bytes32 public immutable tokenId;
```
From the contract above during compilation getter functions would be created for the public immutable variables `interchainTokenServiceAddress`, `implementationType` and `tokenId` this leads to increased deployment cost. The deployment cost could be reduced if the immutable variables are made internal or private thereby eliminating the need for the compiler to create getter functions for them, if these variable are needed externally we could have a single getter function that returns the values of these immutable variables as a tuple. `8893` gas units could be save if the code is refactored to:
```solidity
    IInterchainTokenService internal immutable interchainTokenServiceAddress;
    uint256 internal immutable implementationType;
    bytes32 internal immutable tokenId;


    function getConstants() public view returns(
        IInterchainTokenService,
        uint256,
        bytes32
    ){
        return (interchainTokenServiceAddress, implementationType, tokenId);
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L17-#L18
```solidity
17:    address public immutable interchainTokenServiceAddress;
18:    bytes32 public immutable interchainTokenServiceAddressHash;
```
From the contract above during compilation getter functions would be created for the public immutable variables `interchainTokenServiceAddress` and `interchainTokenServiceAddressHash` this leads to increased deployment cost. The deployment cost could be reduced if the immutable variables are made internal or private thereby eliminating the need for the compiler to create getter functions for them, if these variable are needed externally we could have a single getter function that returns the values of these immutable variables as a tuple. `7304` gas units could be save if the code is refactored to:
```solidity
    address internal immutable interchainTokenServiceAddress;
    bytes32 internal immutable interchainTokenServiceAddressHash;

    function getConstants() public view returns(address, bytes32) {
        return (interchainTokenServiceAddress, interchainTokenServiceAddressHash);
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol#L16-#L18
```solidity
16:    Create3Deployer public immutable deployer;
17:    address public immutable implementationMintBurnAddress;
18:    address public immutable implementationLockUnlockAddress;
```
From the contract above during compilation getter functions would be created for the public immutable variables `deployer`, `implementationMintBurnAddress` and `implementationLockUnlockAddress` this leads to increased deployment cost. The deployment cost could be reduced if the immutable variables are made internal or private thereby eliminating the need for the compiler to create getter functions for them, if these variable are needed externally we could have a single getter function that returns the values of these immutable variables as a tuple. `5005` gas units could be save if the code is refactored to:
```solidity
    Create3Deployer internal immutable deployer;
    address internal immutable implementationMintBurnAddress;
    address internal immutable implementationLockUnlockAddress;

    function getConstants() public view returns(
        Create3Deployer,
        address,
        address
    ) {
        return (deployer, implementationMintBurnAddress, implementationLockUnlockAddress);
    }
```

```
Total Gas saved: 32930
```

## [G-03] Declaring Unneccessary Variable
Some varibles were defined even though they are used once. Not defining these variables can reduce gas cost and contract size.

### 3 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L63
```solidity
    function deployedAddress(
        bytes calldata bytecode,
        address sender,
        bytes32 salt
    ) external view returns (address deployedAddress_) {
        bytes32 newSalt = keccak256(abi.encode(sender, salt));
        deployedAddress_ = address(
            uint160(
                uint256(
                    keccak256(
                        abi.encodePacked(
                            hex'ff',
                            address(this),
                            newSalt,
                            keccak256(bytecode) // init code hash
                        )
                    )
                )
            )
        );
    }

```
In the function above the `newSalt` variable was declared and used once. `1080` gas units can be saved during deployment and `13` gas units when the function is called by another contract by not declaring the `newSalt` variable thus the code could be refactored to:
```solidity
    function deployedAddress(
        bytes calldata bytecode,
        address sender,
        bytes32 salt
    ) external view returns (address deployedAddress_) {
        deployedAddress_ = address(
            uint160(
                uint256(
                    keccak256(
                        abi.encodePacked(
                            hex'ff',
                            address(this),
                            keccak256(abi.encode(sender, salt)),
                            keccak256(bytecode) // init code hash
                        )
                    )
                )
            )
        );
    }

```


- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L64
```solidity
    function getFlowOutAmount() external view returns (uint256 flowOutAmount) {
        uint256 epoch = block.timestamp / EPOCH_TIME;
        uint256 slot = _getFlowOutSlot(epoch);
        assembly {
            flowOutAmount := sload(slot)
        }
    }
```
In the function above the `epoch` variable was declared and used once. `1080` gas units can be saved during deployment and `13` gas units when the function is called by another contract by not declaring the `epoch` variable thus the code could be refactored to:
```solidity
    function getFlowOutAmount() external view returns (uint256 flowOutAmount) {
        uint256 slot = _getFlowOutSlot(block.timestamp / EPOCH_TIME);
        assembly {
            flowOutAmount := sload(slot)
        }
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol#L76
```solidity
function getFlowInAmount() external view returns (uint256 flowInAmount) {
    uint256 epoch = block.timestamp / EPOCH_TIME;
    uint256 slot = _getFlowInSlot(epoch);
    assembly {
        flowInAmount := sload(slot)
    }    
}
```
In the function above the `epoch` variable was declared and used once. `1080` gas units can be saved during deployment and `13` gas units when the function is called by another contract by not declaring the `epoch` variable thus the code could be refactored to:
```solidity
function getFlowInAmount() external view returns (uint256 flowInAmount) {
        uint256 slot = _getFlowInSlot(block.timestamp / EPOCH_TIME);
        assembly {
            flowInAmount := sload(slot)
    }
}
```
```
Total gas saved: 3279
```


## [G-04] Caching of global variables
Caching global variables is more expensive than using the actual variable. The code should be refactored to use msg.sender directly instead of caching it.

### 6 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L348

In the contract above `msg.sender` was cached in the variable ` deployer_` this operation cost `12` gas units and it could be avoided. The code should be refactored, the ` address deployer_ = msg.sender;` statement be removed and `msg.sender` be used directly in all occurence of the `deployer_` variable.

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L447

In the contract above `msg.sender` was cached in the variable ` caller` this operation cost `12` gas units and it could be avoided. The code should be refactored, the ` address caller = msg.sender;` statement be removed and `msg.sender` be used directly in all occurence of the `caller` variable.

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L89

In the contract above `msg.sender` was cached in the variable ` sender` this operation cost `12` gas units and it could be avoided. The code should be refactored, the ` address caller = msg.sender;` statement be removed and `msg.sender` be used directly in all occurence of the `sender` variable.

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L372

In the contract above `msg.sender` was cached in the variable ` deployer_` this operation cost `12` gas units and it could be avoided. The code should be refactored, the ` address deployer_ = msg.sender;` statement be removed and `msg.sender` be used directly in all occurence of the `deployer_` variable.

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L478

In the contract above `msg.sender` was cached in the variable ` caller` this operation cost `12` gas units and it could be avoided. The code should be refactored, the ` address caller = msg.sender;` statement be removed and `msg.sender` be used directly in all occurence of the `caller` variable.

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L115

In the contract above `msg.sender` was cached in the variable ` sender` this operation cost `12` gas units and it could be avoided. The code should be refactored, the ` address caller = msg.sender;` statement be removed and `msg.sender` be used directly in all occurence of the `sender` variable.

```
Total gas saved: 12 * 6 = 72 gas
```


## [G-05] Superflous Events fields
Refactor event to avoid emitting data that is already present in transaction data, `block.timestammp` does not have to be emitted since `block.timestamp` are added to the event information by default, so adding them manually will waste additional gas.

### 1 Instance
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L78

The `event ProposalExecuted()` event declared in the [IInterchainGovernance.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/IInterchainGovernance.sol#L20) included a timestamp parameter that represented the time the event was emitted. The event declaration should be refactored and the timestamp parameter removed. Then `block.timestamp` should be removed from this emit statement in the [InterchainGovernance contract](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L78
)
```
Total gas saved: 
```

## [G-06] keccak256() should only need to be called on a specific string literal once It should be saved to an immutable variable, and the variable used instead.

### 1 Instance
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L246-#L248

In the contract above the pure function `contractId()` calculates and returns the keccack256 hash of the string "axelar-gateway" this operation cost `193` gas units. The `contractId()` function is invoked in the external function `upgrade()`. This means whenever the `upgrade()` function is invoked the hash of the string "axelar-gateway" would be calculated doing it this way is very gas inefficient. The keccak256 hash should only need to be called on the string "axelar-gateway" once and it should be saved to an immutable variable, and the variable used instead. The code should be refactored and the `contractId()` function be removed and replaced with `bytes32 public immutable CONTRACT_ID = keccak256('axelar-gateway')` . The `contractId()` function invocation in the `upgrade()` function be replaced with `CONTRACT_ID` immutable variable
```solidity
287:   if (AxelarGateway(newImplementation).contractId() != CONTRACT_ID) revert InvalidImplementation();
```
```
Total gas saved: 
```



## [G-07] Can transfer 0
In Solidity, performing unnecessary operations can consume more gas than needed, leading to cost inefficiencies. For instance, if a transfer function doesn't have a zero amount check and someone calls it with a zero amount, unnecessary gas will be consumed in executing the function, even though the state of the contract remains the same. By implementing a zero amount check, such unnecessary function calls can be avoided, thereby saving gas and making the contract more efficient.

### 6 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L439-#L454
```solidity
    function expressReceiveToken(
        bytes32 tokenId,
        address destinationAddress,
        uint256 amount,
        bytes32 commandId
    ) external {
        if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

        address caller = msg.sender;
        ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
        IERC20 token = IERC20(tokenManager.tokenAddress());

        SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

        _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, caller);
    }
```
In the function above you should factor in a check for zero amount before the `SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```solidity
    function expressReceiveToken(
        bytes32 tokenId,
        address destinationAddress,
        uint256 amount,
        bytes32 commandId
    ) external {
        if (amount == 0) revert zeroAmount()
        if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

        address caller = msg.sender;
        ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
        IERC20 token = IERC20(tokenManager.tokenAddress());

        SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

        _setExpressReceiveToken(tokenId, destinationAddress, amount, commandId, caller);
    }
``` 


- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L467-#L487
```solidity
    function expressReceiveTokenWithData(
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

        _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount);

        _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller);
    }
```
In the function above you should factor in a check for zero amount before the `SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```solidity
    function expressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId
    ) external {
        if (amount == 0) revert zeroAmount();
        if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

        address caller = msg.sender;
        ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
        IERC20 token = IERC20(tokenManager.tokenAddress());

        SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

        _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount);

        _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller);
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L524
```solidity
    function _mintToken(
        string memory symbol,
        address account,
        uint256 amount
    ) internal {
        address tokenAddress = tokenAddresses(symbol);

        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

        _setTokenMintAmount(symbol, tokenMintAmount(symbol) + amount);

        if (_getTokenType(symbol) == TokenType.External) {
            IERC20(tokenAddress).safeTransfer(account, amount);
        } else {
            IBurnableMintableCappedERC20(tokenAddress).mint(account, amount);
        }
    }
```
In the function above you should factor in a check for zero amount before the `IERC20(tokenAddress).safeTransfer(account, amount)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```solidity
    function _mintToken(
        string memory symbol,
        address account,
        uint256 amount
    ) internal {
        address tokenAddress = tokenAddresses(symbol);

        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

        _setTokenMintAmount(symbol, tokenMintAmount(symbol) + amount);

        if (_getTokenType(symbol) == TokenType.External) {
            if (amount == 0) revert zeroAmount();
            IERC20(tokenAddress).safeTransfer(account, amount);
        } else {
            IBurnableMintableCappedERC20(tokenAddress).mint(account, amount);
        }
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L48
```solidity
    function _takeToken(address from, uint256 amount) internal override returns (uint256) {
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = token.balanceOf(address(this));

        SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

        // Note: This allows support for fee-on-transfer tokens
        return IERC20(token).balanceOf(address(this)) - balance;
    }
```
In the function above you should factor in a check for zero amount before the `SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```solidity
    function _takeToken(address from, uint256 amount) internal override returns (uint256) {
        if (amount == 0) revert zeroAmount();
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = token.balanceOf(address(this));

        SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

        // Note: This allows support for fee-on-transfer tokens
        return IERC20(token).balanceOf(address(this)) - balance;
    }
```


- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L64
```solidity
    function _giveToken(address to, uint256 amount) internal override returns (uint256) {
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = IERC20(token).balanceOf(to);

        SafeTokenTransfer.safeTransfer(token, to, amount);

        return IERC20(token).balanceOf(to) - balance;
    }
```
In the function above you should factor in a check for zero amount before the `SafeTokenTransfer.safeTransfer(token, to, amount)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```solidity
    function _giveToken(address to, uint256 amount) internal override returns (uint256) {
        if (amount == 0) revert zeroAmount();
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = IERC20(token).balanceOf(to);

        SafeTokenTransfer.safeTransfer(token, to, amount);

        return IERC20(token).balanceOf(to) - balance;
    }
```


- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L82
```solidity
    function _takeToken(address from, uint256 amount) internal override returns (uint256) {
        IERC20 token = IERC20(tokenAddress());
        address liquidityPool_ = liquidityPool();
        uint256 balance = token.balanceOf(liquidityPool_);

        SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount);

        // Note: This allows support for fee-on-transfer tokens
        return IERC20(token).balanceOf(liquidityPool_) - balance;
    }
```
In the function above you should factor in a check for zero amount before the `SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```solidity
    function _takeToken(address from, uint256 amount) internal override returns (uint256) {
        if (amount == 0) revert zeroAmount();
        IERC20 token = IERC20(tokenAddress());
        address liquidityPool_ = liquidityPool();
        uint256 balance = token.balanceOf(liquidityPool_);

        SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount);

        // Note: This allows support for fee-on-transfer tokens
        return IERC20(token).balanceOf(liquidityPool_) - balance;
    }
```
```
Total gas saved: 
```


## [G-08] Move lesser gas costing reverts checks to the top
Revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### 4 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L159-#L161
```solidity
    function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
        uint256 length = signers.accounts.length;

        // Clean up old signers.
        for (uint256 i; i < length; ++i) {
            signers.isSigner[signers.accounts[i]] = false;
        }

        length = newAccounts.length;

        if (newThreshold > length) revert InvalidSigners();

        if (newThreshold == 0) revert InvalidSignerThreshold();

        ++signerEpoch;

        signers.accounts = newAccounts;
        signers.threshold = newThreshold;

        for (uint256 i; i < length; ++i) {
            address account = newAccounts[i];

            // Check that the account wasn't already set as a signer for this epoch.
            if (signers.isSigner[account]) revert DuplicateSigner(account);
            if (account == address(0)) revert InvalidSigners();

            signers.isSigner[account] = true;
        }

        emit SignersRotated(newAccounts, newThreshold);
    }
```
In the function above the `if (newThreshold == 0) revert InvalidSignerThreshold()` statement should be moved to the top of the function since this statement checks if the `newThreshold` argument is equal to 0, so that in scenarios that `newThreshold` equals to 0 the function could revert without performing gas consuming operations. The code should be refactored to: 
```solidity
    function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
        if (newThreshold == 0) revert InvalidSignerThreshold();
        uint256 length = signers.accounts.length;

        // Clean up old signers.
        for (uint256 i; i < length; ++i) {
            signers.isSigner[signers.accounts[i]] = false;
        }

        length = newAccounts.length;

        if (newThreshold > length) revert InvalidSigners();

        ++signerEpoch;

        signers.accounts = newAccounts;
        signers.threshold = newThreshold;

        for (uint256 i; i < length; ++i) {
            address account = newAccounts[i];

            // Check that the account wasn't already set as a signer for this epoch.
            if (signers.isSigner[account]) revert DuplicateSigner(account);
            if (account == address(0)) revert InvalidSigners();

            signers.isSigner[account] = true;
        }

        emit SignersRotated(newAccounts, newThreshold);
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L537-#L538
```solidity
    function _burnTokenFrom(
        address sender,
        string memory symbol,
        uint256 amount
    ) internal {
        address tokenAddress = tokenAddresses(symbol);

        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
        if (amount == 0) revert InvalidAmount();

        TokenType tokenType = _getTokenType(symbol);

        if (tokenType == TokenType.External) {
            IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);
        } else if (tokenType == TokenType.InternalBurnableFrom) {
            IERC20(tokenAddress).safeCall(abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount));
        } else {
            IERC20(tokenAddress).safeTransferFrom(sender, IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0)), amount);
            IBurnableMintableCappedERC20(tokenAddress).burn(bytes32(0));
        }
    }
```
In the function above the `if (amount == 0) revert InvalidAmount()` statement should be moved to the top of the function since this statement checks if the `amount` argument is equal to 0, so that in scenarios that `amount` equals to 0 the function could revert without performing gas consuming operations. The code should be refactored to:
```solidity
    function _burnTokenFrom(
        address sender,
        string memory symbol,
        uint256 amount
    ) internal {
        if (amount == 0) revert InvalidAmount();
        address tokenAddress = tokenAddresses(symbol);

        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

        TokenType tokenType = _getTokenType(symbol);

        if (tokenType == TokenType.External) {
            IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);
        } else if (tokenType == TokenType.InternalBurnableFrom) {
            IERC20(tokenAddress).safeCall(abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount));
        } else {
            IERC20(tokenAddress).safeTransferFrom(sender, IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0)), amount);
            IBurnableMintableCappedERC20(tokenAddress).burn(bytes32(0));
        }
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol#L54-#L55
```solidity
    function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
        deployed = deployedAddress(address(this), salt);

        if (deployed.isContract()) revert AlreadyDeployed();
        if (bytecode.length == 0) revert EmptyBytecode();

        // Deploy using create2
        CreateDeployer deployer = new CreateDeployer{ salt: salt }();

        if (address(deployer) == address(0)) revert DeployFailed();

        deployer.deploy(bytecode);
    }
```
In the function above the `if (bytecode.length == 0) revert EmptyBytecode()` statement should be moved to the top of the function since this statement checks if the length of the `bytecode` argument is equal to 0, so that in scenarios that `bytecode` length equals 0 the function could revert without performing gas consuming operations. The function should be refactored to:
```solidity
    function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
        if (bytecode.length == 0) revert EmptyBytecode();
        deployed = deployedAddress(address(this), salt);

        if (deployed.isContract()) revert AlreadyDeployed();

        // Deploy using create2
        CreateDeployer deployer = new CreateDeployer{ salt: salt }();

        if (address(deployer) == address(0)) revert DeployFailed();

        deployer.deploy(bytecode);
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol#L57-#L58
```solidity
    function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner {
        if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
        if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();

        if (params.length > 0) {
            (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));

            if (!success) revert SetupFailed();
        }

        emit Upgraded(newImplementation);

        assembly {
            sstore(_IMPLEMENTATION_SLOT, newImplementation)
        }
    }
```
In the function above the `if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash()` statement should be moved to the top of the function. The execution of this statement would consume lesser gas than the `if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation()` statement. In scenarios that `if (newImplementationCodeHash != newImplementation.codehash)` results to true the function would revert without the EVM executing the `if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation()` statement which would consume a lot of gas. The function should be refactored to:
```solidity
    function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner {
        if (IUpgradable(newImplementation).contractId() != IUpgradable(this).contractId()) revert InvalidImplementation();
        if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();

        if (params.length > 0) {
            (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(this.setup.selector, params));

            if (!success) revert SetupFailed();
        }

        emit Upgraded(newImplementation);

        assembly {
            sstore(_IMPLEMENTATION_SLOT, newImplementation)
        }
    }
```
```
Total Gas saved: 
```


## [G-09] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
the recommended Mitigation Steps is that Functions guaranteed to revert when called by normal users can be marked payable.

### 17 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L142-#L144
```solidity
    function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {
        
    }
```
`21` gas units can be saved if the function definition is refactored to: 
```solidity
    function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual payable onlySigners {
        
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L385-#L419
```solidity
    function deployToken(bytes calldata params, bytes32) external onlySelf {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function deployToken(bytes calldata params, bytes32) external payable onlySelf {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L421-#L425
```solidity
    function mintToken(bytes calldata params, bytes32) external onlySelf {
        
    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function mintToken(bytes calldata params, bytes32) external payable onlySelf {
        
    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L427-#L453
```solidity
    function burnToken(bytes calldata params, bytes32) external onlySelf {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function burnToken(bytes calldata params, bytes32) external payable onlySelf {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L455-#L467
```solidity
    function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function approveContractCall(bytes calldata params, bytes32 commandId) external payable onlySelf {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L469-#L493
```solidity
    function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external payable onlySelf {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L495-#L499
```solidity
    function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function transferOperatorship(bytes calldata newOperatorsData, bytes32) external payable onlySelf {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L547-#L549
```solidity
    function setPaused(bool paused) external onlyOwner {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function setPaused(bool paused) external payable onlyOwner {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83-#L89
```solidity
    function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L95-#L100
```solidity
    function removeTrustedAddress(string calldata chain) external onlyOwner {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function removeTrustedAddress(string calldata chain) external payable onlyOwner {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L106-#L113
```solidity
    function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function addGatewaySupportedChains(string[] calldata chainNames) external payable onlyOwner {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L119-#L126
```solidity
    function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function removeGatewaySupportedChains(string[] calldata chainNames) external payable onlyOwner {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83-#L89
```solodity
    function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solodity
    function addTrustedAddress(string memory chain, string memory addr) public payable onlyOwner {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L171-#L173
```solidity
    function setFlowLimit(uint256 flowLimit) external onlyOperator {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function setFlowLimit(uint256 flowLimit) external payable onlyOperator {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L67-#L69
```solidity
    function setLiquidityPool(address newLiquidityPool) external onlyOperator {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function setLiquidityPool(address newLiquidityPool) external payable onlyOperator {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Distributable.sol#L51-#L53
```solidity
    function setDistributor(address distr) external onlyDistributor {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function setDistributor(address distr) external payable onlyDistributor {

    }
```

- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Operatable.sol#L51-#L53
```solidity
    function setOperator(address operator_) external onlyOperator {

    }
```
`21` gas units can be saved if the function definition is refactored to:
```solidity
    function setOperator(address operator_) external payable onlyOperator {

    }
```
```
Total gas saved: 17 * 21 = 357 gas units
```




## [G-10]  Using calldata for read-only arguments in external functions
Using calldata instead of memory for read-only arguments in external functions saves gas.
### 18 Instances
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L142-#L144
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L199-#L201
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L203-#L205
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol#L232-#L234
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L80-#L86
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L24-#L26
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42-#L52
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L49-#L61
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L35-#L61
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L72-#L85
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L241-#L243
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L251-#L253
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L262-#L268
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L343-#L352
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L417-#L429
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L467-#L487
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L502-#L523
- https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol#L171-#L191