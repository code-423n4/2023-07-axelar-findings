Local variable assignment
---------------
Catch frequently used storage variables in memory/stack, converting multiple SLOAD into 1 SLOAD.

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L119C5-L129C6
`votingPerTopic[signerEpoch][topic]` has been used multiple times in a loop

```solidity
function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
        uint256 length = signers.accounts.length;
        uint256 voteCount;
        for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }

        return voteCount;
    }
```
```
diff --git a/a/contracts/cgp/auth/MultisigBase.sol b/b/contracts/cgp/auth/MultisigBase.sol
index 8d0ea00..9995d2f 100755
--- a/a/contracts/cgp/auth/MultisigBase.sol
+++ b/b/contracts/cgp/auth/MultisigBase.sol
@@ -119,8 +119,9 @@ contract MultisigBase is IMultisigBase {
     function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
         uint256 length = signers.accounts.length;
         uint256 voteCount;
+        Voting storage voting = votingPerTopic[signerEpoch][topic];
         for (uint256 i; i < length; ++i) {
-            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
+            if (voting.hasVoted[signers.accounts[i]]) {
                 voteCount++;
             }
         }
```

***
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L104C5-L116C6
`interchainCalls.length` has been used multiple times in a loop
```solidity
function revertIfInvalidFee(InterchainCalls.InterchainCall[] calldata interchainCalls) private {
        uint256 totalGas = 0;
        for (uint256 i = 0; i < interchainCalls.length; ) {
            totalGas += interchainCalls[i].gas;
            unchecked {
                ++i;
            }
        }


        if (totalGas != msg.value) {
            revert InvalidFee();
        }
 }
```
```
diff --git a/a/contracts/interchain-governance-executor/InterchainProposalSender.sol b/b/contracts/interchain-governance-executor/InterchainProposalSender.sol
index c637e9c..5bf1621 100755
--- a/a/contracts/interchain-governance-executor/InterchainProposalSender.sol
+++ b/b/contracts/interchain-governance-executor/InterchainProposalSender.sol
@@ -103,7 +103,8 @@ contract InterchainProposalSender is IInterchainProposalSender {

     function revertIfInvalidFee(InterchainCalls.InterchainCall[] calldata interchainCalls) private {
         uint256 totalGas = 0;
-        for (uint256 i = 0; i < interchainCalls.length; ) {
+        uint256 length = interchainCalls.length;
+        for (uint256 i = 0; i < length; ) {
             totalGas += interchainCalls[i].gas;
             unchecked {
                 ++i;
```

Use calldata instead of memory for function parameters
--------------
It is generally cheaper to load variables directly from calldata, rather than copying them to memory. Only use memory if the variable needs to be modified.

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L530C5-L550C6
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
```
diff --git a/a/contracts/cgp/AxelarGateway.sol b/b/contracts/cgp/AxelarGateway.sol
index 832588f..8bc016e 100755
--- a/a/contracts/cgp/AxelarGateway.sol
+++ b/b/contracts/cgp/AxelarGateway.sol
@@ -529,7 +529,7 @@ contract AxelarGateway is IAxelarGateway, IGovernable, AdminMultisigBase {

     function _burnTokenFrom(
         address sender,
-        string memory symbol,
+        string calldata symbol,
         uint256 amount
     ) internal {
         address tokenAddress = tokenAddresses(symbol);
```
***
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42C5-L52C6
```solidity
    function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
        deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));


        // solhint-disable-next-line avoid-low-level-calls
        (bool success, ) = deployedAddress_.call(init);
        if (!success) revert FailedInit();
    }
```
```
diff --git a/a/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol b/b/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
index dca6e6b..5fc4129 100755
--- a/a/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
+++ b/b/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol
@@ -40,7 +40,7 @@ contract ConstAddressDeployer {
      *    as an option to not have the constructor args affect the address derived by `CREATE2`.
      */
     function deployAndInit(
-        bytes memory bytecode,
+        bytes calldata bytecode,
         bytes32 salt,
         bytes calldata init
     ) external returns (address deployedAddress_) {
```
***
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L49C5-L61C6
```solidity
    function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
        bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));
        deployedAddress_ = Create3.deploy(deploySalt, bytecode);


        (bool success, ) = deployedAddress_.call(init);
        if (!success) revert FailedInit();


        emit Deployed(keccak256(bytecode), salt, deployedAddress_);
    }
```
```
diff --git a/a/contracts/gmp-sdk/deploy/Create3Deployer.sol b/b/contracts/gmp-sdk/deploy/Create3Deployer.sol
index 0bf1f31..846b03e 100755
--- a/a/contracts/gmp-sdk/deploy/Create3Deployer.sol
+++ b/b/contracts/gmp-sdk/deploy/Create3Deployer.sol
@@ -47,7 +47,7 @@ contract Create3Deployer {
      * - `init` is used to initialize the deployed contract
      */
     function deployAndInit(
-        bytes memory bytecode,
+        bytes calldata bytecode,
         bytes32 salt,
         bytes calldata init
     ) external returns (address deployedAddress_) {
```
***
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3.sol#L51C5-L63C6

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
```
diff --git a/a/contracts/gmp-sdk/deploy/Create3.sol b/b/contracts/gmp-sdk/deploy/Create3.sol
index 2769bf7..a3c278d 100755
--- a/a/contracts/gmp-sdk/deploy/Create3.sol
+++ b/b/contracts/gmp-sdk/deploy/Create3.sol
@@ -48,7 +48,7 @@ library Create3 {
      * @param bytecode The bytecode of the contract to be deployed
      * @return deployed The address of the deployed contract
      */
-    function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
+    function deploy(bytes32 salt, bytes calldata bytecode) internal returns (address deployed) {
         deployed = deployedAddress(address(this), salt);

         if (deployed.isContract()) revert AlreadyDeployed();
```
> Notice that will effect to https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/FinalProxy.sol#L79, so we update it too
```solidity
    function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {
        address owner;
        assembly {
            owner := sload(_OWNER_SLOT)
        }
        if (msg.sender != owner) revert NotOwner();


        finalImplementation_ = Create3.deploy(FINAL_IMPLEMENTATION_SALT, bytecode);
        if (setupParams.length != 0) {
            (bool success, ) = finalImplementation_.delegatecall(abi.encodeWithSelector(BaseProxy.setup.selector, setupParams));
            if (!success) revert SetupFailed();
        }
    }
```
```
diff --git a/a/contracts/gmp-sdk/upgradable/FinalProxy.sol b/b/contracts/gmp-sdk/upgradable/FinalProxy.sol
index 050b829..f63ff0a 100755
--- a/a/contracts/gmp-sdk/upgradable/FinalProxy.sol
+++ b/b/contracts/gmp-sdk/upgradable/FinalProxy.sol
@@ -69,7 +69,7 @@ contract FinalProxy is Proxy, IFinalProxy {
      * @param setupParams The parameters to setup the final implementation contract
      * @return finalImplementation_ The address of the final implementation contract
      */
-    function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {
+    function finalUpgrade(bytes calldata bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {
         address owner;
         assembly {
             owner := sload(_OWNER_SLOT)
```
***
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/InitProxy.sol#L35C5-L61C6
```solidity
    function init(
        address implementationAddress,
        address newOwner,
        bytes memory params
    ) external {
        address owner;


        assembly {
            owner := sload(_OWNER_SLOT)
        }


        if (msg.sender != owner) revert NotOwner();
        if (implementation() != address(0)) revert AlreadyInitialized();


        bytes32 id = contractId();
        if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();


        assembly {
            sstore(_IMPLEMENTATION_SLOT, implementationAddress)
            sstore(_OWNER_SLOT, newOwner)
        }


        if (params.length != 0) {
            (bool success, ) = implementationAddress.delegatecall(abi.encodeWithSelector(IUpgradable.setup.selector, params));
            if (!success) revert SetupFailed();
        }
    }
```
```
diff --git a/a/contracts/gmp-sdk/upgradable/InitProxy.sol b/b/contracts/gmp-sdk/upgradable/InitProxy.sol
index 7bcc307..0e92ced 100755
--- a/a/contracts/gmp-sdk/upgradable/InitProxy.sol
+++ b/b/contracts/gmp-sdk/upgradable/InitProxy.sol
@@ -35,7 +35,7 @@ contract InitProxy is BaseProxy, IInitProxy {
     function init(
         address implementationAddress,
         address newOwner,
-        bytes memory params
+        bytes calldata params
     ) external {
         address owner;
```
***
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L343C5-L352C6
```solidity
    function deployCustomTokenManager(
        bytes32 salt,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) public payable notPaused returns (bytes32 tokenId) {
        address deployer_ = msg.sender;
        tokenId = getCustomTokenId(deployer_, salt);
        _deployTokenManager(tokenId, tokenManagerType, params);
        emit CustomTokenIdClaimed(tokenId, deployer_, salt);
    }
```
```
diff --git a/a/contracts/its/interchain-token-service/InterchainTokenService.sol b/b/contracts/its/interchain-token-service/InterchainTokenService.sol
index 2a13bdf..dc74375 100755
--- a/a/contracts/its/interchain-token-service/InterchainTokenService.sol
+++ b/b/contracts/its/interchain-token-service/InterchainTokenService.sol
@@ -343,7 +343,7 @@ contract InterchainTokenService is
     function deployCustomTokenManager(
         bytes32 salt,
         TokenManagerType tokenManagerType,
-        bytes memory params
+        bytes calldata params
     ) public payable notPaused returns (bytes32 tokenId) {
         address deployer_ = msg.sender;
         tokenId = getCustomTokenId(deployer_, salt);
```
***