# Low

## Users funds are locked when Interchain token has not been deployed on that chain

If a user tries to make an interchainTokenTransfer with a canonical or standardized token that has not yet been deployed on destination chain, user's tokens are collected on source chain, but his transfer won't be completed on destination chain, and user won't be able to claim his tokens on source chain

## `deployRemoteCaonicalToken`, `deployRemoteCustomTokenManager` and `deployAndRegisterRemoteStandardizedToken` allow user to specify any gasValue which may allow draining of InterchainTokenService contract

The `gasValue` parameter is put in place to allow multichain calls to be able to specify amount of gas. But a malicious user may call any of those functions with a very high gasValue(even though he does not attach any msg.value with his call), to drain InterchainTokenService of its native value

## call will return true success value if contract being called does not exist.

There are many instances where a call or delegatecall is made without first checking if the contract being called exists. This will cause the transaction to execute successfully in those scenarios because it will always return true success value despite it not executing anything.

## Erc777 canonical tokens won't work as expected

If a canonical token is deployed for an ERC777 token, `takeToken` will fail because there will be an attempt to call `tokensReceived` hook on TokenManager. Also, `giveToken` will call `tokensReceived` hook on recipient contract, which opens risks of reentrancy

## While making interchain transfers, users are allowed to input a destination address of 0

Users are allowed to input a 0 address as the destination address while making interchain transfers, which would lead to burning of the tokens on destination chain, and making tokens to be locked on source chain. This should not be allowed because only the distributor of the token should be allowed to burn tokens.

## users are allowed to make an interchainTransfer of 0 tokens

Users should not be allowwed to make interchain transfers of 0 amounts

## Standardized tokens will not circulate or be distributed if it is of type MINT_BURN on all chains it is deployed.

Unlike canonical tokens, which are given a TokenManagerType of LOCK_UNLOCK on the chain where they are originally deployed, Standardized token can be made to be of type MINT_BURN on all chains it is deployed. This will prevent the token from being distributed because token manager is the distributor, and can only mint new tokens when it receives that token from an interchain transfer. So, if there is no distributor aside the Token manager, that token cannot be distributed

## Governance can destroy AxelarGateway if newImplementation#setup contains self destruct code

If governance calls AxelarGateway#upgrade with a `newImplementation` whose functions that will be interacted with contains any self-destruct code, it will lead to destruction of AxelarGateway(which is a core contract in the protocol)
Although very unlikely to happen, impact is very high.

## There is no use case of `deployCustomTokenManager`

All other functions that deploys tokens automatically deploys a token manager with it. So there is no practical scenario where this will be used.

## validateSender is not very correct.

```solidity
function validateSender(string calldata sourceChain, string calldata sourceAddress) external view returns (bool) {
    string memory sourceAddressLC = _lowerCase(sourceAddress);
    bytes32 sourceAddressHash = keccak256(bytes(sourceAddressLC));
@>  if (sourceAddressHash == interchainTokenServiceAddressHash) {
        return true;
    }
    return sourceAddressHash == remoteAddressHashes[sourceChain];
}
```

`if (sourceAddressHash == interchainTokenServiceAddressHash) {return true;}` should be removed. sourceAddress from chainB can have same address as interchainTokenServiceAddress on chainA, and not be an interchainTokenService. Deployer of interchainTokenService can deploy malicious contract on chainB to the same address as interchainTokenService on chainA(due to create3), and will be able to call critical functions in interchainTokenService on chainA.

# QA

## User can register a standardized token as a canonical token

## There should be an eta and deadline when making interchain proposals.

When making interchain proposals in `InterchainProposalSender.sol`, users should be able to specify an ETA and deadline because there could be times when executing a particular proposal could be favourable

## Standardized token deployer can deploy multiple standardized token managers for a tokenId on the same chain

## wrong naming of variable in `InterchainTokenService#deployRemoteCanonicalToken`

In the first line: `address tokenAddress = getValidTokenManagerAddress(tokenId);`, variable should be named tokenManagerAddress, not tokenAddress

## Express caller is not incentivized in any way to make an express call
