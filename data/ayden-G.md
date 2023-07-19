1.Use bytes instead of string will save more gas.
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L21#L22

We don't need to convert string to byte in constructor anymore

```
    constructor(
        address gateway,
-       string memory governanceChain_,
-       string memory governanceAddress_,
+       bytes memory governanceChain_,
+       bytes memory governanceChain_,
        uint256 minimumTimeDelay
    ) AxelarExecutable(gateway) TimeLock(minimumTimeDelay) {
        governanceChain = governanceChain_;
        governanceAddress = governanceAddress_;
-       governanceChainHash = keccak256(bytes(governanceChain_));
-       governanceAddressHash = keccak256(bytes(governanceAddress_));
+       governanceChainHash = keccak256(governanceChain_);
+       governanceAddressHash = keccak256(governanceAddress_);
    }
```

What's more, we don't need to convert sourceChain and sourceAddress from string to bytes too.
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L80

```
    function _execute(string calldata sourceChain, string calldata sourceAddress, bytes calldata payload) internal override {    <= string to bytes
-       if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)
            revert NotGovernance();
+       if (keccak256(sourceChain) != governanceChainHash || keccak256(sourceAddress) != governanceAddressHash)
            revert NotGovernance();

        (uint256 command, address target, bytes memory callData, uint256 nativeValue, uint256 eta) = abi.decode(
            payload,
            (uint256, address, bytes, uint256, uint256)
        );

        if (target == address(0)) revert InvalidTarget();

        _processCommand(command, target, callData, nativeValue, eta);
    }
```

2.Use `else` instead of ` else if`` save more gas.
According to  the variable " `GovernanceCommand``, it's consist of two types so there is no need to use `else if` anymore.
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol#L101#L121
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L91

3.Use x=x+1 instead of x++ where x is storage variable to save more gas.
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L163

5. Use immutable keyword for gas optimization
   https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L39#L40

6.The number of voter already been count in `votingPerTopic` variable.
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol#L119#L129

```solidity
function getSignerVotesCount(bytes32 topic) external view override returns (uint256) {
-   uint256 length = signers.accounts.length;
-   uint256 voteCount;
-   for (uint256 i; i < length; ++i) {
-       if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
-            voteCount++;
-        }
-    }

-   return voteCount;

+   Voting storage voting = votingPerTopic[signerEpoch][topic];
+   return voting.voteCount
}

```

7.function only called by frontend can use external instead of public to save more gas fee.
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L241
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L251

8.`getParamsLiquidityPool` and `getParamsMintBurn` can be computed directly on the frontend to save more gas fee.

9.Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L115
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol#L89
InterchainToken.sol#L49
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L438
AxelarValuedExpressExecutable.sol#L106