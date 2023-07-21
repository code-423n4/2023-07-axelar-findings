# Axelar Network Gas Optimization Report

## Table Of Content

| Number                                                                                               | Issue                                                                                                                                                                                  | Instances |
| :--------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------: |
| [G-01](#g-01---should-add-check-conditions-at-the-beginning-of-the-functions-to-save-users-gas-cost) | [Should add `check` conditions at the beginning of the functions to save users gas cost](#g-01---should-add-check-conditions-at-the-beginning-of-the-functions-to-save-users-gas-cost) |         1 |
| [G-02](#g-02---multiple-reads-of-storage-variable-which-can-be-cached-to-save-users-gas-cost)        | [Multiple reads of storage variable which can be cached to save users gas cost](#g-02---multiple-reads-of-storage-variable-which-can-be-cached-to-save-users-gas-cost)                 |         2 |
| [G-03](#g-03---use-unchecked-where-possible-to-save-users-gas)                                       | [Use `unchecked` where possible to save users gas](#g-03---use-unchecked-where-possible-to-save-users-gas)                                                                             |         2 |
| [G-04](#g-04---use-calldata-data-location-for-functions-parameters-to-save-gas)                      | [use `calldata` data location for functions parameters to save gas](#g-04---use-calldata-data-location-for-functions-parameters-to-save-gas)                                           |        13 |
| [G-05](#g-05---combining-similar-mappings-using-struct)                                              | [Combining similar mappings using `struct`](#g-05---combining-similar-mappings-using-struct)                                                                                           |         2 |
| [G-06](#g-06---constants-expressions-are-expressions-not-constants)                                  | [`constants` expressions are expressions, not constants](#g-06---constants-expressions-are-expressions-not-constants)                                                                  |        10 |

### [G-01] - Should add `check` conditions at the beginning of the functions to save users gas cost.

**Details**

Adding `check` conditions at the top of the functions reduces the gas cost of callers when the function calls fails because they will not spend for other commutations.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[MultisigBase.sol - Line 149 - 161](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L149-L161)

```diff
     function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
-        uint256 length = signers.accounts.length;
-
-        // Clean up old signers.
-        for (uint256 i; i < length; ++i) {
-            signers.isSigner[signers.accounts[i]] = false;
-        }
-
-        length = newAccounts.length;

+        // * @audit Gas - these two conditions should be on top.
+        uint256 length = newAccounts.length;

         if (newThreshold > length) revert InvalidSigners();

         if (newThreshold == 0) revert InvalidSignerThreshold();

+        uint256 signersLength = signers.accounts.length;
+
+        // Clean up old signers.
+        for (uint256 i; i < signersLength; ++i) {
+            signers.isSigner[signers.accounts[i]] = false;
+        }
+
```

</details>

### [G-02] - Multiple reads of storage variable which can be cached to save users gas cost.

**Details**

SLOAD (is the opcode used of read storage variable) cost 100 units of gas and MLOAD & MSTORE (are the opcodes used to read and write to memory respectivily) cost 3, 3 units of gas respectivily.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[MultisigBase.sol - Line 149 - 161](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L149-L161)

```diff
     function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
+        // * @audit Gas - save `signers.accounts` in a local array and then use for length and below.
-        uint256 length = signers.accounts.length;
+        address[] memory _accounts = signers.accounts;
+        uint256 signersLength = _accounts.length;

        // Clean up old signers.
        for (uint256 i; i < length; ++i) {
-            signers.isSigner[signers.accounts[i]] = false;
+            signers.isSigner[_accounts[i]] = false;
        }
      }
```

[MultisigBase.sol - 119 - 129](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L119-L129)

```diff
function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
+        // * @audit Gas - save the `signers.accounts` in a local array and then use for length and for hasVotes below.
-        uint256 length = signers.accounts.length;
+        address[] memory _accounts = signers.accounts;
+        uint256 length = _accounts.length;

         uint256 voteCount;
+        uint256 _signerEpoch = signerEpoch;

        for (uint256 i; i < length; ++i) {
+            // * @audit Gas - save `signers.accounts` in a local array.
+            // * @audit Gas - save `signerEpoch` in a local variable.
-            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
+            if (votingPerTopic[_signerEpoch][topic].hasVoted[_accounts[i]]) {
                 voteCount++;
             }
         }
```

</details>

### [G-03] - Use `unchecked` where possible to save users gas.

**Details**

When `unchecked` is used variable then compiler do not execute opcodes which check over/underflow errors, it can be risky some times but we should use it when we are sure it will not exceed the limits.

And here we have a huge limit for it.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[MultisigBase.sol - Line 163](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L163)

```diff
-     ++signerEpoch;
+     unchecked {
+         ++signerEpoch;
+     }
```

[MultisigBase.sol - 122 - 124](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L122-L124)

```diff
    function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
        address[] memory _accounts = signers.accounts;

        uint256 length = _accounts.length;
        uint256 voteCount;

        uint256 _signerEpoch = signerEpoch;

-        for (uint256 i = 0; i < length; ++i) {
+        for (uint256 i = 0; i < length; ) {
            if (votingPerTopic[_signerEpoch][topic].hasVoted[_accounts[i]]) {
+                // * @audit Gas - use unchecked.
-                ++voteCount;
+                unchecked {
+                    ++voteCount;
+                }
            }
+
+            unchecked {
+                ++i;
+            }
        }

        return voteCount;
    }
```

</details>

### [G-04] - use `calldata` data location for functions parameters to save gas.

**Details**

-   Solidity docs suggests that `If you can, try to use calldata as data location because it will avoid copies and also makes sure that the data cannot be modified`.
-   And it also save gas when compare to memory data location.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[ConstAddressDeployer.sol - Line 24](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L24)

```diff
-     function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {
+     function deploy(bytes calldata bytecode, bytes32 salt) external returns (address deployedAddress_) {
```

[ConstAddressDeployer.sol - Line 42](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42)

```diff
-     function deployAndInit(bytes memory bytecode, bytes32 salt, bytes calldata init) external returns (address deployedAddress_) {
+     function deployAndInit(bytes calldata bytecode, bytes32 salt, bytes calldata init) external returns (address deployedAddress_) {
```

[FinalProxy.sol - Line 72](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L72)

```diff
-     function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {
+     function finalUpgrade(bytes calldata bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {
```

[InitProxy.sol - Line 35](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L35)

```diff
-     function init(address implementationAddress, address newOwner, bytes memory params)
+     function init(address implementationAddress, address newOwner, bytes calldata params)
```

[InterchainProposalSender.sol - Line 80 - 84](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L80-L84)

```diff
-     function sendProposal(string memory destinationChain, string memory destinationContract, InterchainCalls.Call[] calldata calls ) external payable override {
+     function sendProposal(string calldata destinationChain, string calldata destinationContract, InterchainCalls.Call[] calldata calls ) external payable override {
```

[IInterchainProposalSender.sol - Line 15](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol#L15)

```diff
-     function sendProposals(InterchainCalls.InterchainCall[] memory calls) external payable;
+     function sendProposals(InterchainCalls.InterchainCall[] calldata calls) external payable;
```

[IInterchainTokenService.sol - Line 145](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L145)

[IInterchainTokenService.sol - Line 153](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L153)

[IInterchainTokenService.sol - Line 162](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#162)

[IInterchainTokenService.sol - Line 194](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L194)

[IInterchainTokenService.sol - Line 245](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L245)

[IInterchainTokenService.sol - Line 272](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L272)

[IInterchainTokenService.sol - Line 339](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L339)

</details>

### [G-05] - Combining similar mappings using `struct`.

**Details**

-   In the `InterchainProposalExecutor.sol` we are using the `whitelistedCallers` and `whitelistedSenders` mappings to know whether the address comming from the source chain is caller or sender and we are utilizing two mappings for them which will use lots of storage slots. And more you utilize the storage slots more you increase the users gas cost in the execution.

-   In the `RemoteAddressValidator.sol` we are using the `remoteAddressHashes`, `remoteAddresses`, and `supportedByGateway` mappings to know the `remoteAddresshash`, `remoteAddress`, and `isSupportedByGateway` for a given chain. We can merge all these three into one mapping using struct.

We can merge the similar mappings using structs to get the same results.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[InterchainProposalExecutor.sol - Line 23 - 27](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L23-L27)

```diff
 contract InterchainProposalExecutor is IInterchainProposalExecutor, AxelarExecutable, Ownable {
+    // * @audit Question - Can we merge the two into one mapping using struct.
     // Whitelisted proposal callers. The proposal caller is the contract that calls the `InterchainProposalSender` at the source chain.
-    mapping(string => mapping(address => bool)) public whitelistedCallers;
-
     // Whitelisted proposal senders. The proposal sender is the `InterchainProposalSender` contract address at the source chain.
-    mapping(string => mapping(address => bool)) public whitelistedSenders;
+    struct Address {
+        bool isCaller;
+        bool isSender;
+    }
+
+    mapping(string => mapping(address => Address)) public whitelistedAddresses;
+
+    // mapping(string => mapping(address => bool)) public whitelistedSenders;

    function _execute(string calldata sourceChain, string calldata sourceAddress, bytes calldata payload) internal override {
         _beforeProposalExecuted(sourceChain, sourceAddress, payload);

         // Check that the source address is whitelisted
-        if (!whitelistedSenders[sourceChain][StringToAddress.toAddress(sourceAddress)]) {
+        if (!whitelistedAddresses[sourceChain][StringToAddress.toAddress(sourceAddress)].isSender) {
             revert NotWhitelistedSourceAddress();
         }

         // Check that the caller is whitelisted
-        if (!whitelistedCallers[sourceChain][interchainProposalCaller]) {
+        if (!whitelistedAddresses[sourceChain][interchainProposalCaller].isCaller) {
             revert NotWhitelistedCaller();
         }
    }

    function setWhitelistedProposalCaller(string calldata sourceChain, address sourceCaller, bool whitelisted) external override onlyOwner {
-        whitelistedCallers[sourceChain][sourceCaller] = whitelisted;
+        whitelistedAddresses[sourceChain][sourceCaller].isCaller = whitelisted;
         emit WhitelistedProposalCallerSet(sourceChain, sourceCaller, whitelisted);
     }

    function setWhitelistedProposalSender(string calldata sourceChain, address sourceSender, bool whitelisted) external override onlyOwner {
-        whitelistedSenders[sourceChain][sourceSender] = whitelisted;
+        whitelistedAddresses[sourceChain][sourceSender].isSender = whitelisted;
         emit WhitelistedProposalSenderSet(sourceChain, sourceSender, whitelisted);
     }
```

[RemoteAddressValidator.sol - Lines 12 - 139](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L12-L139)

```diff
 contract RemoteAddressValidator is IRemoteAddressValidator, Upgradable {
     using AddressToString for address;

+    // * @audit Gas - these three mappings can be convert to single string => struct mapping
-    mapping(string => bytes32) public remoteAddressHashes;
-    mapping(string => string) public remoteAddresses;
+    struct Chain {
+        bytes32 remoteAddresshash;
+        string remoteAddress;
+        bool isSupportedByGateway;
+    }
+    mapping(string => Chain) public chains;
+
     address public immutable interchainTokenServiceAddress;
     bytes32 public immutable interchainTokenServiceAddressHash;
-    mapping(string => bool) public supportedByGateway;

+    // add this to override required parent function.
+    function supportedByGateway(string calldata chainName) external view override returns (bool) {
+        return chains[chainName].isSupportedByGateway;
+    }

    function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool) {
        string memory sourceAddressLC = _lowerCase(sourceAddress);
        bytes32 sourceAddressHash = keccak256(bytes(sourceAddressLC));
        if (sourceAddressHash == interchainTokenServiceAddressHash) {
            return true;
        }
-        return sourceAddressHash == remoteAddressHashes[sourceChain];
+        return sourceAddressHash == chains[sourceChain].remoteAddresshash;
    }

     function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {
         if (bytes(chain).length == 0) revert ZeroStringLength();
         if (bytes(addr).length == 0) revert ZeroStringLength();
-        remoteAddressHashes[chain] = keccak256(bytes(_lowerCase(addr)));
+        chains[chain].remoteAddresshash = keccak256(bytes(_lowerCase(addr)));
-        remoteAddresses[chain] = addr;
+        chains[chain].remoteAddress = addr;
         emit TrustedAddressAdded(chain, addr);
     }

     function removeTrustedAddress(string calldata chain) external onlyOwner {
         if (bytes(chain).length == 0) revert ZeroStringLength();
-        remoteAddressHashes[chain] = bytes32(0);
+        chains[chain].remoteAddresshash = bytes32(0);
-        remoteAddresses[chain] = '';
+        chains[chain].remoteAddress = '';
         emit TrustedAddressRemoved(chain);
     }

     function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {
         uint256 length = chainNames.length;
         for (uint256 i; i < length; ++i) {
             string calldata chainName = chainNames[i];
-            supportedByGateway[chainName] = true;
+            chains[chainName].isSupportedByGateway = true;
             emit GatewaySupportedChainAdded(chainName);
         }
     }

     function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {
         uint256 length = chainNames.length;
         for (uint256 i; i < length; ++i) {
             string calldata chainName = chainNames[i];
-            supportedByGateway[chainName] = false;
+            chains[chainName].isSupportedByGateway = false;
             emit GatewaySupportedChainRemoved(chainName);
         }
     }

     function getRemoteAddress(string calldata chainName) external view returns (string memory remoteAddress) {
-        remoteAddress = remoteAddresses[chainName];
+        remoteAddress = chains[chainName].remoteAddress;
         if (bytes(remoteAddress).length == 0) {
             remoteAddress = interchainTokenServiceAddress.toString();
         }
     }
```

</details>

### [G-06] - `constants` expressions are expressions, not constants.

**Details**
A constant declared like this;

```solidity
bytes32 internal constant PREFIX_COMMAND_EXECUTED = keccak256('command-executed');
```

Is re-calculated each time it is in use. This re-calculated increases the gas cost.

The value should be converted into a constant value at compile time or should use a `immutable` variable and calculate value inside the constructor so only the deployment cost occur.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[AxelarGateway.sol - Lines 44 - 57](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L44-L57)

[FinalProxy.sol - Line 18](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L18)

[TimeLock.sol - Line 13](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L13)

[InterchainTokenService.sol - Lines 62 - 64 & 71](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L62-L64)

[InterchainTokenServiceProxy.sol - Line 12](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/InterchainTokenServiceProxy.sol#L12)

[RemoteAddressValidatorProxy.sol - Line 12](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/RemoteAddressValidatorProxy.sol#L12)

[StandardizedTokenProxy.sol - Line 14](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/proxies/StandardizedTokenProxy.sol#L14)

[RemoteAddressValidator.sol - Line 21](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L21)

[StandardizedToken.sol - Line 27](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-implementations/StandardizedToken.sol#L27)

[FlowLimit.sol - Lines 15 - 16](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/FlowLimit.sol#L15-L16)

</details>
