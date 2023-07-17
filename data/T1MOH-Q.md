## 1. LOW. _OWNER_SLOT in BaseProxy.sol and Upgradable.sol is not compatible with EIP1967
### Proof of Concept
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Upgradable.sol#L11
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/BaseProxy.sol#L16
This is current value
```solidity
    // keccak256('owner')
    bytes32 internal constant _OWNER_SLOT = 0x02016836a56b71f0d02689e69e326f4f4c1b9057164ef592671cf0d37c8040c0;
```
According to [EIP1967](https://eips.ethereum.org/EIPS/eip-1967#admin-address) it should be
```
Storage slot 0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103 (obtained as bytes32(uint256(keccak256('eip1967.proxy.admin')) - 1)).
```

## 2. LOW. msg.value can temporary stuck in MultisigBase.sol
### Proof of Concept
Modifier `onlySigners()` after voting executes action if there are enough votes. Note that if votes not enough, it just returns without execution:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L39-L77
```solidity
    modifier onlySigners() {
        if (!signers.isSigner[msg.sender]) revert NotSigner();

        bytes32 topic = keccak256(msg.data);
        Voting storage voting = votingPerTopic[signerEpoch][topic];

        // Check that signer has not voted, then record that they have voted.
        if (voting.hasVoted[msg.sender]) revert AlreadyVoted();

        voting.hasVoted[msg.sender] = true;

        // Determine the new vote count.
        uint256 voteCount = voting.voteCount + 1;

        // Do not proceed with operation execution if insufficient votes.
        if (voteCount < signers.threshold) {
            // Save updated vote count.
            voting.voteCount = voteCount;
            return;
        }

        // Clear vote count and voted booleans.
        voting.voteCount = 0;

        uint256 count = signers.accounts.length;

        for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

        emit MultisigOperationExecuted(topic);

        _;
    }
```

However some actions can require sending msg.value:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L52
```solidity
    function executeMultisigProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable onlySigners {
        ...

        _call(target, callData, nativeValue);

        emit MultisigExecuted(proposalHash, target, callData, nativeValue);
    }
```

Imagine 2 concurrent transactions to execute payable action. First transaction executes and all votes are cleared. Then second transaction is just 1st to vote on a new action, and sent msg.value is stuck in MultisigBase.sol. However it can be resqued via another action

### Recommended steps
Keep mapping for such accidental ether, and add claim function:
```solidity
    mapping(address => uint256) public pendingAmounts;
```
```solidity
modifier onlySigners() {
        ...

        // Do not proceed with operation execution if insufficient votes.
        if (voteCount < signers.threshold) {
            // Save updated vote count.
            voting.voteCount = voteCount;

            pendingAmounts[msg.sender] += msg.value;
            return;
        }

        ...

        _;
    }
```
```solidity
function claim() external {
    uint256 amount = pendingAmounts[msg.sender];
    require(amount != 0);
    pendingAmounts[msg.sender] = 0;
    payable(msg.sender).call{value: amount}("");
}
```
But want to notice that in this case external call to signer will be performed, potentially it is dangerous

## 3. LOW. FinalProxy.sol is not EIP1967 compatible
### Proof of Concept
Final implementation address is not stored, but computed via CREATE3. But according to EIP1967 implementation must be stored in slot `bytes32(uint256(keccak256('eip1967.proxy.implementation')) - 1)`

```solidity
    function implementation() public view override(BaseProxy, IProxy) returns (address implementation_) {
        implementation_ = _finalImplementation();
        if (implementation_ == address(0)) {
            implementation_ = super.implementation();
        }
    }

    function _finalImplementation() internal view virtual returns (address implementation_) {
        /**
         * @dev Computing the address is cheaper than using storage
         */
        implementation_ = Create3.deployedAddress(address(this), FINAL_IMPLEMENTATION_SALT);

        if (implementation_.code.length == 0) implementation_ = address(0);
    }
```
### Recommended steps
Use storage instead of computing address. Store implementation always in slot 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc. And store boolean flag `isFinal` for remembering that final upgrade was performed

## 4. LOW. Incorrect selector is used when performing setup of TokenManager.sol
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/proxies/TokenManagerProxy.sol#L18-L38
### Proof of Concept
Function selector `TokenManagerProxy.setup.selector` is used for setup of TokenManager. Now they are identical, but it can be different in future. Because setupping contract is TokenManager.sol and therefore selector `TokenManager.setup.selector` should be used
```solidity
    constructor(
        address interchainTokenServiceAddress_,
        uint256 implementationType_,
        bytes32 tokenId_,
        bytes memory params
    ) {
        interchainTokenServiceAddress = IInterchainTokenService(interchainTokenServiceAddress_);
        implementationType = implementationType_;
        tokenId = tokenId_;
        address impl = _getImplementation(IInterchainTokenService(interchainTokenServiceAddress_), implementationType_);

        (bool success, ) = impl.delegatecall(abi.encodeWithSelector(TokenManagerProxy.setup.selector, params));
        if (!success) revert SetupFailed();
    }
```

### Recommended steps
Refactor
```solidity
-       (bool success, ) = impl.delegatecall(abi.encodeWithSelector(TokenManagerProxy.setup.selector, params));
+       (bool success, ) = impl.delegatecall(abi.encodeWithSelector(TokenManager.setup.selector, params));
```

## 5. R. `getSignerVotesCount()` logic can be much simpler
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L119-L129
```solidity
    function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
-       uint256 length = signers.accounts.length;
-       uint256 voteCount;
-       for (uint256 i; i < length; ++i) {
-           if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
-              voteCount++;
-           }
-       }

-       return voteCount;
+       return votingPerTopic[signerEpoch][topic].voteCount;
    }
```
