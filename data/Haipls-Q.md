## [01] MultisigBase does not allow canceling the submitted vote to execute operation
The implementation of `MultisigBase.sol`, as well as its derived contracts `Multisig.sol`, `AxelarServiceGovernance`, `AxelarServiceGovernance`, only allows confirming a vote without the possibility to revoke it if necessary. This means that once they have cast their vote, it becomes final and unchangeable for the execution of the operation. This could potentially entail certain risks in specific situations.

For example, one of the signers could be misled and cast their vote. Later, they might desire to revoke their submission within a certain time frame, but the functionality of these contracts does not provide for such an option.

### PROOF OF CONCEPT

[`MultisigBase.sol`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol) does not include the ability to cancel the vote
```c
signerThreshold()
signerAccounts()
isSigner(address)
hasSignerVoted(address,bytes32)
getSignerVotesCount(bytes32)
rotateSigners(address[],uint256)
_rotateSigners(address[],uint256)
```

### Mittigation 
It is recommended to enable the signer to cancel his vote on the selected operation


## [02] There is no deadline for voting on a proposal
A vote for a proposal has no expiration date, and can only be rejected by a change of epoch. Which leads to a situation when there may be a proposal that will be in a suspended state and be executed at an inconvenient moment, or when it is redundant (and instead they maybe decided to execute another proposal)

### PROOF OF CONCEPT

[`MultisigBase.sol`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol)
```c
    struct Voting {
        uint256 voteCount;
        mapping(address => bool) hasVoted;
    }

    struct Signers {
        address[] accounts;
        uint256 threshold;
        mapping(address => bool) isSigner;
    }

    Signers public signers;
    uint256 public signerEpoch;
    // uint256 is for epoch, bytes32 for vote topic hash
    mapping(uint256 => mapping(bytes32 => Voting)) public votingPerTopic;
```

### Mittigation 
Add a deadline for an epocha or individually for an offer

## [03] The amount of native coins transferred is not validated, allowing any user or signer to choose the source and proportion at their discretion. 

The contracts assume that a certain amount of currency will be present in the contract, but upon executing a proposal, a portion can be provided by the caller while the rest is taken from the contract's balance. The exact amount taken from the contract is entirely at the discretion of the caller, which may lead to undesirable behavior. For example, User A was supposed to initiate the execution and transfer their funds, but they specified msg.value = 0, causing the funds to be taken from the contract's wallet instead.

### PROOF OF CONCEPT
* [`Multisig#L30`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/Multisig.sol#L30)
* [`InterchainGovernance#L68`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L68)
* [`AxelarServiceGovernance#L48`](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L48)

```c
File: contracts/cgp/governance/Multisig.sol

30:     function execute(
            address target,
            bytes calldata callData,
            uint256 nativeValue
        ) external payable onlySigners {
            _call(target, callData, nativeValue);
        }

File: contracts/cgp/governance/InterchainGovernance.sol

68: function executeProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

        _finalizeTimeLock(proposalHash);
        _call(target, callData, nativeValue);

        emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);
    }

File: contracts/cgp/governance/AxelarServiceGovernance.sol

48: function executeMultisigProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable onlySigners {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

        if (!multisigApprovals[proposalHash]) revert NotApproved();

        multisigApprovals[proposalHash] = false;

        _call(target, callData, nativeValue);

        emit MultisigExecuted(proposalHash, target, callData, nativeValue);
    }
```

### Mittigation 
In the case of multi sig wallets, this is not so critical, in others I would recommend making an additional change that will validate the payment proportion


## [04] Checking the success of the call while ignoring the returned data is not always a guarantee of a successful call result

Since there are different contracts with their implementations, these implementations can include a situation when a transaction is unsuccessful, but this does not lead to revert

The most common example is the ERC20 standard, where the `transfer/transferFrom` etc transfer methods *return a bool*, which, according to the standard, indicates success. In this case, passing the transaction is not a guarantee of achieving the planned result, such as the transfer of 100 tokens, the transaction went through, but the tokens were not transferred.

### PROOF OF CONCEPT

Instances:
* [Caller.sol#L18](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Caller.sol#L18)
* [Create3Deployer.sol#L57](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L57)
* [ConstAddressDeployer.sol#L50](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50)
* [InterchainProposalExecutor.sol#76](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L76)
```c
File: contracts/cgp/util/Caller.sol

    function _call(
        ...
18:     (bool success, ) = target.call{ value: nativeValue }(callData);
        if (!success) {
            revert ExecutionFailed();
        }
    }

File contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

    function deployAndInit(
        ...
        // solhint-disable-next-line avoid-low-level-calls
50:        (bool success, ) = deployedAddress_.call(init);
51:        if (!success) revert FailedInit();

File contracts/gmp-sdk/deploy/Create3Deployer.sol

    function deployAndInit(
        ...
57:        (bool success, ) = deployedAddress_.call(init);
58:        if (!success) revert FailedInit();


File contracts/interchain-governance-executor/InterchainProposalExecutor.sol

76: (bool success, bytes memory result) = call.target.call{ value: call.value }(call.callData);

    if (!success) {
        _onTargetExecutionFailed(call, result);
    } else {
        _onTargetExecuted(call, result);
    }

```

### Mittigation 
Depending on the case, but as an option, use an analog implementation [SafeCall](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/SafeTransfer.sol#L12), when the returned success date is also checked

## [05] The implementation of Create3/Create3Deployer/ConstAddressDeployer does not provide the ability to transfer native coins during deployment/initialization
Since the essence of the factories for Deploy is the important support of Deploy for various contracts regardless of implementation. The limitation in the impossibility of transferring the native coins during the deployment of the contract, or its initialization, is a limitation in the contracts that can be deployed by these factories, and the need for new factories that will support this functionality


### PROOF OF CONCEPT

p.s: absence payable modifiers, msg.value etc
Instances:
* [ConstAddressDeployer.sol#L50](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L50)
* [ConstAddressDeployer.sol#L85](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L85)
* [Create3.sol#L25](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3.sol#L25)
* [Create3Deployer.sol#L29](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L29)
* [Create3Deployer.sol#L49](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L49)
```c
File contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

24:    function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_)

42: function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
        deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

        // solhint-disable-next-line avoid-low-level-calls
        (bool success, ) = deployedAddress_.call(init);
        if (!success) revert FailedInit();
    }
80: assembly {
            deployedAddress_ := create2(0, add(bytecode, 32), mload(bytecode), salt)
    }

File contracts/gmp-sdk/deploy/Create3.sol

25:  deployed := create(0, add(bytecode, 32), mload(bytecode))

File contracts/gmp-sdk/deploy/Create3Deployer.sol

29:     function deploy(bytes calldata bytecode, bytes32 salt) external returns (address deployedAddress_) {

49    function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {

```

### Mittigation 
Use the implementation/principle from the available popular libraries [CREATE2.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Create2.sol), [CREATE3.sol](https://github.com/transmissions11/solmate/blob/main/src/utils/CREATE3.sol) which do not limit the possibility of deploying contracts with a payable constructor or payable initialization functions

## [06] Methods can accept native coins, but do not forward
Methods allow you to send coins on deployment, but this only results in them being stuck on the contract

### PROOF OF CONCEPT

the presence of the `payable` modifier without the ability to process the transferred native currency
Instances:
* [TokenManagerDeployer.sol#L37](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/TokenManagerDeployer.sol#L37)
* [StandardizedTokenDeployer.sol#L37](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/StandardizedTokenDeployer.sol#L58)
```c
File: contracts/its/utils/TokenManagerDeployer.sol

37: function deployTokenManager(
    ...
    ) external payable {


File: contracts/its/utils/StandardizedTokenDeployer.sol

49: function deployStandardizedToken(
    ...
    ) external payable {
```

### Mittigation 
Remove the payable modifier, or implement deploy support with the payable constructor/initialization method
