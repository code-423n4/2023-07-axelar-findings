## Gas Optimizations

| Number | Issue | Instances | Estimated Gas Saved |
| --- | --- | --- | --- |
| [G-01](#) | Using storage instead of memory for structs/arrays saves gas | 2 | 4200 |
| [G-02](#) |  Use calldata instead of memory for function parameters | 3 | 300 |
| [G-03](#) | call environment variables directly instead of caching them | 7 | 21 |
| [G-04](#) | Cache state variables outside of loop to avoid reading/writing storage on every iteration | 1 | 100 |
| [G-05](#) | Multiple accesses of a mapping/array should use a local variable cache | 1 | 100 |
| [G-06](#) | Can Make The Variable Outside The Loop To Save Gas | 4 | 400 |
| [G-07](#) | IF's/require() statements that check input arguments should be at the top of the function | 2 | 4200 |
| [G-08](#) | <array>.length should not be looked up in every loop of a for-loop | 1 | 100 |


**Total gas saved across all listed functions: 9421**

## [G-1] **Using storage instead of memory for structs/arrays saves gas**

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L75

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

75: InterchainCalls.Call memory call = calls[i];
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L271C47-L271C47

```solidity
File: /contracts/cgp/AxelarGateway.sol

271: string memory symbol = symbols[i];
```

## [G-2 ] Use calldata instead of memory for function parameters

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142C4-L144C6

```solidity
File: /contracts/cgp/auth
/MultisigBase.sol

142: function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {
143:        _rotateSigners(newAccounts, newThreshold);
144:    }
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L50

```solidity
contracts/gmp-sdk/deploy/Create3Deployer.sol

50: function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42

```solidity
File: contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

42: function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
```

## [G-3] **call environment variables directly instead of cac**hing them

In the instance below, instead of caching `msg.sender` and incurring unnecessary stack manipulation, we can call `msg.sender` directly. `msg.sender` costs 2 Gas while the extra stack manipulation will cost 3 Gas per `DUP`.

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/TokenManager.sol#L89

```solidity
File: contracts/its/token-manager/TokenManager.sol

89: address sender = msg.sender;
115: address sender = msg.sender;
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token/InterchainToken.sol#L49C8-L49C37

```solidity
File: contracts/its/interchain-token/InterchainToken.sol

49: address sender = msg.sender;
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L348C9-L348C40

```solidity
File: contracts/its/interchain-token-service/InterchainTokenService.sol

348: address deployer_ = msg.sender;
372: address deployer_ = msg.sender;
447: address caller = msg.sender;
478: address caller = msg.sender;
```

## [G-4] Cache state variables outside of loop to avoid reading/writing storage on every iteration

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a `Gwarmaccess (100 gas)` per loop iteration.

Total Instances: `1`

Estimated Gas Saved: `100` (This will be more if there are > 1 loop iterations for each instance)

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L119C5-L129C6

```solidity
File: contracts/cgp/auth/MultisigBase.sol

119: function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
120:        uint256 length = signers.accounts.length;
121:        uint256 voteCount;
122;        for (uint256 i; i < length; ++i) {
123:            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) { //@audit cache state variables outside of loop
124:                voteCount++;
125:            }
126:        }

128:        return voteCount;
129:    }
```

```diff
File: contracts/cgp/auth/MultisigBase.sol

119: function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
120:        uint256 length = signers.accounts.length;
121:        uint256 voteCount;
+122:      Voting storage voting = votingPerTopic[signerEpoch][topic];
123:        for (uint256 i; i < length; ++i) {
-124:            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
+124:            if (voting.hasVoted[signers.accounts[i]]) {
125:               voteCount++;
126:            }
127:        }

129:        return voteCount;
130:    }
```

## [G-5] Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. **Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

`AxelarServiceGovernance.sol.executeMultisigProposal() :multisigApprovals[proposalHash]` should cached

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L55

```solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol

48: function executeMultisigProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable onlySigners {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

        if (!multisigApprovals[proposalHash]) revert NotApproved(); //@audit initial call

        multisigApprovals[proposalHash] = false;     //@audit second call

        _call(target, callData, nativeValue);

        emit MultisigExecuted(proposalHash, target, callData, nativeValue);
    }
```

## [G-6] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L169

```solidity
File: contracts/cgp/auth/MultisigBase.sol

169: address account = newAccounts[i];
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L272C12-L272C39

```solidity
File: contracts/cgp/AxelarGateway.sol

272: uint256 limit = limits[i];
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L345C1-L346C1

```solidity
File: contracts/cgp/AxelarGateway.sol

345: bytes32 commandId = commandIds[i];
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L57C13-L57C42

```solidity
File: /contracts/its/remote-address-validator/RemoteAddressValidator.sol

57: uint8 b = uint8(bytes(s)[i]);
```

## [G-7] IF's/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

**reorder the if statement to have cheaper one before the gas consuming one**

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L538

```solidity
File: contracts/cgp/AxelarGateway.sol

530: function _burnTokenFrom(
531:        address sender,
532:       string memory symbol,
533:        uint256 amount
534:    ) internal {
535:        address tokenAddress = tokenAddresses(symbol);
536:
537:        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
538:        if (amount == 0) revert InvalidAmount();
539:
540:        TokenType tokenType = _getTokenType(symbol);
541:
542:        if (tokenType == TokenType.External) {
543:            IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);
544:        } else if (tokenType == TokenType.InternalBurnableFrom) {
545:            IERC20(tokenAddress).safeCall(abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount));
546:        } else {
547:            IERC20(tokenAddress).safeTransferFrom(sender, IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0)), amount);
548:            IBurnableMintableCappedERC20(tokenAddress).burn(bytes32(0));
549:   }
550:    }
```

```diff
File: contracts/cgp/AxelarGateway.sol

530: function _burnTokenFrom(
531:        address sender,
532:       string memory symbol,
533:        uint256 amount
534:    ) internal {
+           if (amount == 0) revert InvalidAmount();
535:        address tokenAddress = tokenAddresses(symbol);
536:
537:        if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);
-538:        if (amount == 0) revert InvalidAmount();
539:
540:        TokenType tokenType = _getTokenType(symbol);
541:
542:        if (tokenType == TokenType.External) {
543:            IERC20(tokenAddress).safeTransferFrom(sender, address(this), amount);
544:        } else if (tokenType == TokenType.InternalBurnableFrom) {
545:            IERC20(tokenAddress).safeCall(abi.encodeWithSelector(IBurnableMintableCappedERC20.burnFrom.selector, sender, amount));
546:        } else {
547:            IERC20(tokenAddress).safeTransferFrom(sender, IBurnableMintableCappedERC20(tokenAddress).depositAddress(bytes32(0)), amount);
548:            IBurnableMintableCappedERC20(tokenAddress).burn(bytes32(0));
549:   }
550:    }
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142C4-L144C6

```solidity
File: /contracts/cgp/auth/MultisigBase.sol

149: function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
150:        uint256 length = signers.accounts.length;

159: if (newThreshold > length) revert InvalidSigners();
160:
161:        if (newThreshold == 0) revert InvalidSignerThreshold(); //@audit
162:
163:        ++signerEpoch;

178: emit SignersRotated(newAccounts, newThreshold);
179:    }
```

```diff
File: /contracts/cgp/auth/MultisigBase.sol

149: function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
+            if (newThreshold == 0) revert InvalidSignerThreshold();
150:        uint256 length = signers.accounts.length;

159: if (newThreshold > length) revert InvalidSigners();
160:
-161:        if (newThreshold == 0) revert InvalidSignerThreshold(); 
162:
163:        ++signerEpoch;

178: emit SignersRotated(newAccounts, newThreshold);
179:    }
```

## [G-8]`**<array>.length` should not be looked up in every loop of a `for`-loop**

The overheads outlined below are PER LOOP, excluding the first loop

storage arrays incur a Gwarmaccess (100 gas) memory arrays use MLOAD (3 gas) calldata arrays use CALLDATALOAD (3 gas)

Caching the length changes each of these to a DUP (3 gas), and gets rid of the extra DUP needed to store the stack offset

`gas save 100`

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106C7-L106C60

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol

106: for (uint256 i = 0; i < interchainCalls.length; ) {
```