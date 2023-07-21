### Comments/implementation mismatch in InterchainProposalSender 
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L38C10-L38C34)

The extensive comment at the beginning of the `` contract, states that:
```
 * The contract ensures the correctness of the provided proposal details and fees through a series of internal
 * functions that revert the transaction if any of the checks fail. This includes checking if the provided fees
 * are equal to the total value sent with the transaction, if the lengths of the provided arrays match, and if the
 * provided proposal arguments are valid.
```
Of the expected checks only one is performed (`revertIfInvalidFee(interchainCalls)`), and only in one of the many functions available in the contract.


### Unsafe cast to IERC20 in the `TokenManagerLockUnlock` contract
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L44)
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L60)

In two areas of the implementation, the lock-unlock manager cast the token's address to an IERC20:
```
    function _takeToken(address from, uint256 amount) internal override returns (uint256) {
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = token.balanceOf(address(this));

        SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

        // Note: This allows support for fee-on-transfer tokens
        return IERC20(token).balanceOf(address(this)) - balance;
    }
```
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L44C1-L52C6)
```
    function _giveToken(address to, uint256 amount) internal override returns (uint256) {
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = IERC20(token).balanceOf(to);

        SafeTokenTransfer.safeTransfer(token, to, amount);

        return IERC20(token).balanceOf(to) - balance;
    }
```
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L60)

The lock-unlock token manager accepts pre existing tokens and, during the registration, only the `name`, `symbol` and `decimals` values are checked (using the `_validateToken` function of the `InterchainTokenService` contract,https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L726 ). Therefor the registered token might not be compliant with a standard ERC20 implementation, possibly causing malfunctionings of the bridge.

### Improper update of FlowLimit, in both `TokenManagerLiquidityPool` and `TokenManagerLockUnlock`, when using fee-on-transfer tokens
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L94)
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L60C1-L67C6)

Both the `TokenManagerLiquidityPool` and the `TokenManagerLockUnlock` contract will perform a `balanceAfter - balanceBefore` type of computation, when giving tokens to the user, in order to support fee on transfer tokens:
```
    function _giveToken(address to, uint256 amount) internal override returns (uint256) {
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = IERC20(token).balanceOf(to);

        SafeTokenTransferFrom.safeTransferFrom(token, liquidityPool(), to, amount);

        return IERC20(token).balanceOf(to) - balance;
    }
```
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L94)

And
```
    function _giveToken(address to, uint256 amount) internal override returns (uint256) {
        IERC20 token = IERC20(tokenAddress());
        uint256 balance = IERC20(token).balanceOf(to);

        SafeTokenTransfer.safeTransfer(token, to, amount);

        return IERC20(token).balanceOf(to) - balance;
    }
```
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L60C1-L67C6)

The `_addFlowIn(amount)` function, used to update the flow count of the `FlowLimit` contract, called by `giveToken` of the `TokenManager` contract (https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/TokenManager.sol#L161), used to update flow upon giving tokens to the user, will use the returned value of `_giveToken(destinationAddress, amount)`, which as stated before uses `balanceAfter - balanceBefore` to compute it.
In case of a fee-on-transfer token this will represents the amount of token received by the user, and not the amount of token leaving the bridge, meaning that the flow counter will be improperly updated.

### Payable functions are using a user controllable parameter to send native tokens
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L48)
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L68)
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/Multisig.sol#L30C14-L30C21)

The `executeMultisigProposal` function of the `AxelarServiceGovernance` contract (https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L48), the `executeProposal` function of the `InterchainGovernance` contract (https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L68) and the ``execute function of the `Multisig` contract (https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/Multisig.sol#L30C14-L30C21) are all payable and are all using the `_call` function of the `Caller` contract (https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Caller.sol#L7C14-L7C16) to perform actions, which might include transferring native tokens. The `_call` function uses the `uint256 nativeValue` parameter to specify the amount to send:
```
    function _call(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) internal {
        if (nativeValue > address(this).balance) revert InsufficientBalance();

        (bool success, ) = target.call{ value: nativeValue }(callData);
        if (!success) {
            revert ExecutionFailed();
        }
    }
```
In case of a mismatch between the value sent by the user, to be used by call, and the amount specified as the `nativeValue` parameter (due to human mistakes, for example), the `_call` function can end up using native tokens locked in the contracts that might be stored there for other purposes

### Lack of emit/return value when performing relevant actions
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L44C14-L44C27)

The `onlySigners()` modifier of the `MultisigBase` contract (https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L44C14-L44C27), enables signer to vote on the execution of a function by calling it.
When doing so the `onlySigners()` modifier will update a vote counter until in exceed the pre determined threshold. If a vote doesn't exceed such value, the vote counter will be updated, and no value is returned nor any event is emitted:
```

        // Determine the new vote count.
        uint256 voteCount = voting.voteCount + 1;

        // Do not proceed with operation execution if insufficient votes.
        if (voteCount < signers.threshold) {
            // Save updated vote count.
            voting.voteCount = voteCount;
            return;
        }
```
Making the readability of the outcome of calling a modified function, under such circumstances very hard.

### Improper check order in `_rotateSigners`
(https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L149)

The `_rotateSigners` function of the `MultisigBase` contracts (https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L149), performs a couple of checks on the values provided as arguments:
```
    function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
        uint256 length = signers.accounts.length;

        // Clean up old signers.
        for (uint256 i; i < length; ++i) {
            signers.isSigner[signers.accounts[i]] = false;
        }

        length = newAccounts.length;

        if (newThreshold > length) revert InvalidSigners();

        if (newThreshold == 0) revert InvalidSignerThreshold();
```
This checks should be performed at the beginning of the function and not after having performed other computations increasing readability and avoiding useless computations in case of a revert.