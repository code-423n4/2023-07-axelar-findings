# call environment variables directly instead of caching them

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

`**<array>.length` should not be looked up in every loop of a `for`-loop**

The overheads outlined below are PER LOOP, excluding the first loop

storage arrays incur a Gwarmaccess (100 gas) memory arrays use MLOAD (3 gas) calldata arrays use CALLDATALOAD (3 gas)

Caching the length changes each of these to a DUP (3 gas), and gets rid of the extra DUP needed to store the stack offset

`gas save 100`

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106C7-L106C60

```solidity
File: contracts/interchain-governance-executor/InterchainProposalSender.sol

106: for (uint256 i = 0; i < interchainCalls.length; ) {
```

**Using storage instead of memory for structs/arrays saves gas**

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L75

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalExecutor.sol

75: InterchainCalls.Call memory call = calls[i];
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L80C3-L84C34

```solidity
File: /contracts/interchain-governance-executor/InterchainProposalSender.sol

80: function sendProposal(
        string memory destinationChain,
        string memory destinationContract,
        InterchainCalls.Call[] calldata calls
    ) external payable override {
```

- https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L271C47-L271C47

```solidity
File: 

271: string memory symbol = symbols[i];
```

## Cache state variables outside of loop to avoid reading/writing storage on every iteration

Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a `Gwarmaccess (100 gas)` per loop iteration.

Total Instances: `1`

Estimated Gas Saved: `2280` (This will be more if there are > 1 loop iterations for each instance)

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
        uint256 length = signers.accounts.length;
        uint256 voteCount;
+      Voting storage voting = votingPerTopic[signerEpoch][topic];
        for (uint256 i; i < length; ++i) {
-            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
+            if (voting.hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }

        return voteCount;
    }
```

## Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. **Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

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