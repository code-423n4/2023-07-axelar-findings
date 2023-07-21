### 1. No check to ensure the remote custom token manager is created with the same custom token Id from the origin chain. If different tokenId is used, user cannot send token to the destination chain.

When a custom token manager is deployed on the origin chain, and remote custom token manager is deployed on a destination chain. Then the `salt` used for both functions have to be the same to compute the same `tokenId`. If the `salt` used are different, `tokenId` would be different, and as a result, token manager is not able to send token to the destination chain.

```solidity
//InterchainTokenService.sol
    function deployCustomTokenManager(
        bytes32 salt,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) public payable notPaused returns (bytes32 tokenId) {
        address deployer_ = msg.sender;
|>      tokenId = getCustomTokenId(deployer_, salt);
        _deployTokenManager(tokenId, tokenManagerType, params);
        emit CustomTokenIdClaimed(tokenId, deployer_, salt);
    }
```
```solidity
//InterchainTokenService.sol
    function deployRemoteCustomTokenManager(
        bytes32 salt,
        string calldata destinationChain,
        TokenManagerType tokenManagerType,
        bytes calldata params,
        uint256 gasValue
    ) external payable notPaused returns (bytes32 tokenId) {
        address deployer_ = msg.sender;
|>      tokenId = getCustomTokenId(deployer_, salt);
        _deployRemoteTokenManager(tokenId, destinationChain, gasValue, tokenManagerType, params);
        emit CustomTokenIdClaimed(tokenId, deployer_, salt);
    }
```
If `tokenId` is different for a remote chain, then when token manager tries to send token to the destination chain, only the source chain `tokenId` will be used.

```solidity
//InterchainTokenService.sol-transmitSendToken()
...
|>      payload = abi.encode(SELECTOR_SEND_TOKEN_WITH_DATA, tokenId, destinationAddress, amount, sourceAddress.toBytes(), metadata);
        _callContract(destinationChain, payload, msg.value, sourceAddress);
...
```
Afterward, when remote request is processed on the destination chain, source chain `tokenId` will be used to fetch token manager contract, which will cause an error reverting the destination chain transaction. Send token will fail on the destination chain.

Recommendation: 
When `deployRemoteCustomTokenManager()` is called, first verify if the source chain custom token manager is deployed with computed token id from `salt`. If not deployed, revert the transaction because it means either user is attempting to evoke a cross-chain deployment without first deploying the source chain, OR the `salt` user entered for `deployRemoteCustomTokenManager()` is inconsistent from the source chain.

### 2. Insufficient check for implementation and proxy id matching

In Proxy.sol, `constructor()` function, proxy `contractId` and implementation `contractId` are checked to make sure the ids match each other through `id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id`.

This check might be insufficient depending on the exact needs of proxy and corresponding implementation contract, because when proxy id is bytes32(0), but the implementation id is not bytes32(0), in which case implementation id is not matching proxy id, but the check will still pass allowing mismatched proxy and implementation to be created.

```solidity
//Proxy.sol- constructor()
...
if (id != bytes32(0) && IUpgradable(implementationAddress).contractId() != id) revert InvalidImplementation();
...
```
Recommendation: 
Consider change the condition to `id != IUpgradable(implementationAddress).contractId()`.
