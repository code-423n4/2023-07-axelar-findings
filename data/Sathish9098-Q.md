# LOW FINDINGS
##

## [L-1] ``_getTokenMintLimitKey,_getTokenTypeKey,_getTokenAddressKey,_getIsCommandExecutedKey`` functions not prevent the duplicate ``symbol`` values 

### Impact
If the functions ``_getTokenMintLimitKey``, ``_getTokenTypeKey``, ``_getTokenAddressKey``, and ``_getIsCommandExecutedKey`` do not incorporate any uniqueness, and ``symbol`` is the only varying parameter, then there will be ``hash collisions`` for ``duplicate symbol`` values.

### 
```solidity
FILE: 2023-07-axelar/contracts/cgp/AxelarGateway.sol

 function _getTokenMintLimitKey(string memory symbol) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked(PREFIX_TOKEN_MINT_LIMIT, symbol));
    }

  function _getTokenTypeKey(string memory symbol) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked(PREFIX_TOKEN_TYPE, symbol));
    }

    function _getTokenAddressKey(string memory symbol) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked(PREFIX_TOKEN_ADDRESS, symbol));
    }

    function _getIsCommandExecutedKey(bytes32 commandId) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked(PREFIX_COMMAND_EXECUTED, commandId));
    }


```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L556-L558

### Recommended Mitigation
Unique nonce or timestamp in the ``abi.encodePacked`` process to ensure that each call produces a unique hash

```diff

uint256 private nonce;

function _getTokenMintLimitKey(string memory symbol) private returns (bytes32) {
    return keccak256(abi.encodePacked(PREFIX_TOKEN_MINT_LIMIT, symbol, nonce++));
}

```
##

## [L-2] Lack of pause state Check in ``setFlowLimit()`` function Poses security risk in contract

### Impact 

The problem is the services paused by ``onlyOwner`` . But ``FlowLimit`` set by ``onlyOperator``this will lead to unintended consequences.  

Allowing ``setFlowLimit()`` to execute when the contract is paused can lead to unintended consequences and may compromise the expected behavior of the paused contract. Depending on how the contract utilizes the ``FlowLimit``, this can introduce potential security vulnerabilities or operational issues.

### POC

```solidity
FILE: 2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

 function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {
        uint256 length = tokenIds.length;
        if (length != flowLimits.length) revert LengthMismatch();
        for (uint256 i; i < length; ++i) {
            ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenIds[i]));
            tokenManager.setFlowLimit(flowLimits[i]);
        }
    }

```

### Recommended Mitigation
Incorporate the ``notPaused`` modifier in the ``setFlowLimit()`` function

##

## [L-3] ``OnlyOwner`` allows malicious address as a trusted address

### Impact
he function allows the contract owner to add any address as a trusted address, regardless of whether the address is actually controlled by the interchain token service. This could allow an attacker to gain control of the contract by adding a malicious address as a trusted address.

### POC

```solidity
FILE: 2023-07-axelar/contracts/its/remote-address-validator/RemoteAddressValidator.sol

 function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {
        if (bytes(chain).length == 0) revert ZeroStringLength();
        if (bytes(addr).length == 0) revert ZeroStringLength();
        remoteAddressHashes[chain] = keccak256(bytes(_lowerCase(addr)));
        remoteAddresses[chain] = addr;
        emit TrustedAddressAdded(chain, addr);
    }

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L83-L89

### Recommended Mitigation
``addTrustedAddress`` function could be modified to require that the address to be added be signed by the ``interchain token service``. This would ensure that only ``trusted addresses`` are added to the contract.

##

## [L-4] Avoid Hardcoded Values 

### Impact
Two constants ``PREFIX_EXPRESS_RECEIVE_TOKEN`` and ``PREFIX_EXPRESS_RECEIVE_TOKEN_WITH_DATA`` are hardcoded. This means that anyone who knows the contract code will know what the prefixes are. This could make it easier for attackers to forge transactions that look like they were sent from the Axelar gateway. 

``codehash != 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470`` hardcoded value check is not appreciated . This will cause problem in future .

Possible Problems:

- Hardcoded values can make your contract more vulnerable to attack.
- Hardcoded values can make your contract more difficult to update.
- Hardcoded values can make your contract less flexible.

### POC


```solidity
FILE: Breadcrumbs2023-07-axelar/contracts/cgp/AxelarGateway.sol

509: return codehash != bytes32(0) && codehash != 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470; 


```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L35

```solidity
FILE: 2023-07-axelar/contracts/its/utils/ExpressCallHandler.sol

 uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN = 0x67c7b41c1cb0375e36084c4ec399d005168e83425fa471b9224f6115af865619;
    // uint256(keccak256('prefix-express-give-token-with-data'));
    uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN_WITH_DATA = 0x3e607cc12a253b1d9f677a03d298ad869a90a8ba4bd0fb5739e7d79db7cdeaad;

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/ExpressCallHandler.sol#L14-L16


### Recommended Mitigation
It would be better to generate the prefixes dynamically and in lowercase. This would make the contract more secure and more flexible

##

## [L-5] ``sstore`` instruction is a mutable instruction, which means that it can be overwritten by another transaction

### Impact
The ``_setExpressReceiveToken`` function uses the ``sstore`` assembly instruction to store the value of the ``expressCaller`` variable in the contract storage. The ``sstore`` instruction is a mutable instruction, which means that it can be overwritten by another transaction. This means that if an attacker were to send a transaction that overwrites the value of the ``expressCaller`` variable, they could effectively steal the tokens that were sent to the destination address

### POC

```solidity
FILE: 2023-07-axelar/contracts/its/utils/ExpressCallHandler.sol

 if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();
        assembly {
            sstore(slot, expressCaller)
        }

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/ExpressCallHandler.sol#L91-L94

### Recommended Mitigation
``_setExpressReceiveToken`` function, the ``sstore`` instruction should be used with the lock modifier. The lock modifier prevents the value of the ``expressCaller`` variable from being overwritten by another transaction

##

## [L-6] Use ``IERC20.allowance()`` function to check the ``allowance`` instead of a ``custom mapping``

### Impact
Custom mappings consume storage on the Ethereum blockchain. If you have a large number of users or tokens, maintaining allowances in a mapping can lead to higher storage costs. Multiple concurrent transactions attempting to update allowances, there may be potential concurrency issues or race conditions.When using a custom mapping, you need to ensure that the allowances are synchronized and consistent between different parts of the contract

### POC
```solidity
FILE: 2023-07-axelar/contracts/its/interchain-token/InterchainToken.sol

55: uint256 allowance_ = allowance[sender][address(tokenManager)];

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L55

### Recommended Mitigation
Use ``IERC20.allowance()`` function for handling token allowances.

##

## [L-7] Use safeIncreaseAllowance()/safeDecreaseAllowance() instead of normal approve() function

### Impact
The ``safeIncreaseAllowance()`` and ``safeDecreaseAllowance()`` functions are more secure than the ``approve()`` function. The ``approve()`` function allows an account to approve another account to transfer an unlimited amount of tokens on their behalf. This could be exploited by an attacker to transfer more tokens than the sender intended.

### POC

```solidity
FILE: Breadcrumbs2023-07-axelar/contracts/its/interchain-token/InterchainToken.sol

61: _approve(sender, address(tokenManager), allowance_ + amount);

100: _approve(sender, address(tokenManager), allowance_ + amount);

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L100

### Recommended Mitigation
``safeIncreaseAllowance()`` and ``safeDecreaseAllowance()`` functions prevent this by ensuring that the amount of tokens that an account is approved to transfer cannot be increased or decreased by more than the current allowance.

##

## [L-8] Revert on Zero Value Transfers

### Impact
Some tokens revert when transferring a zero value amount. Many ERC-20 and ERC-721 token contracts implement a safeguard that reverts transactions which attempt to transfer tokens with zero amount. This is because such transfers are often the result of programming errors. The OpenZeppelin library, a popular choice for implementing these standards, includes this safeguard. For token contract developers who want to avoid unintentional transfers with zero amount, it's good practice to include a condition that reverts the transaction if the amount is zero.

### POC

```solidity

451: SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L439-L451

```solidity

482: SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L467-L482

```solidity

48: SafeTokenTransferFrom.safeTransferFrom(token, from, address(this), amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L44-L48

```solidity

82: SafeTokenTransferFrom.safeTransferFrom(token, from, liquidityPool_, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L77-L82

```solidity
98:  SafeTokenTransferFrom.safeTransferFrom(token, liquidityPool(), to, amount);

```
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol#L94-L98

### Recommended Mitigation
Make sure ``amount > 0`` before calling transfer function

##

## [L-9] Revert on Zero Value Approvals

### Impact
Some tokens (e.g. BNB) revert when approving a zero value amount (i.e. a call to approve(address, 0)).
Integrators may need to add special cases to handle this logic if working with such a token.

### POC

```solidity

61: _approve(sender, address(tokenManager), allowance_ + amount);

100: _approve(sender, address(tokenManager), allowance_ + amount);

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L100

##

## [L-10] ``registerCanonicalToken()`` Not checking whether the ``tokenaddress`` already managed by a token manager

### Impact
If two different token managers are managing the ``same token address``, there is a ``risk`` of ``conflicting behavior``. For example, one token manager might allow transfers while another token manager might disallow transfers. This could lead to errors or unexpected behavior for users.

### POC
```solidity
FILE: 2023-07-axelar/contracts/its/interchain-token-service/InterchainTokenService.sol

 function registerCanonicalToken(address tokenAddress) external payable notPaused returns (bytes32 tokenId) {
        (, string memory tokenSymbol, ) = _validateToken(tokenAddress);
        if (gateway.tokenAddresses(tokenSymbol) == tokenAddress) revert GatewayToken();
        tokenId = getCanonicalTokenId(tokenAddress);
        _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));
    }

```
### Recommended Mitigation
Add ``isTokenManaged()`` function to check the token address existence 

```diff

function registerCanonicalToken(address tokenAddress) external payable notPaused returns (bytes32 tokenId) {
(, string memory tokenSymbol, ) = _validateToken(tokenAddress);
if (gateway.tokenAddresses(tokenSymbol) == tokenAddress) revert GatewayToken();
+ if (isTokenManaged(tokenAddress)) revert TokenAlreadyManaged();
tokenId = getCanonicalTokenId(tokenAddress);
_deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));

}

```

##

## [L-11] Unused/empty receive()/fallback() function

### 
If the intention is for the Ether to be used, the function should call another function or emit something, otherwise it should revert (e.g. require(msg.sender == address(weth)))

### POC

```solidity
FILE: 2023-07-axelar/contracts/cgp/governance/InterchainGovernance.sol

168: receive() external payable {}

```

##

## [L-12] ``_approve()`` doesn’t check return value

### Impact
if the approve() function fails, the caller will not be notified of the failure

### POC

```solidity

61: _approve(sender, address(tokenManager), allowance_ + amount);

100: _approve(sender, address(tokenManager), allowance_ + amount);

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L100

### Recommended Mitigation
Add the control to check _approve() status 

##

## [L-13] Eth that is accidentally sent to a ``receive()`` function cannot be withdrawn

### Impact
The ``receive()`` function is used to handle incoming ether (native currency) transfers to a contract. However, when tokens (other than the native currency) are accidentally sent to the receive() function, they are effectively trapped within the contract, and there is ``no built-in mechanism`` to ``withdraw`` or ``recover them``.

### POC

```solidity
FILE: 2023-07-axelar/contracts/cgp/governance/InterchainGovernance.sol

168: receive() external payable {}

```

### Recommended Mitigation
Add Recovery mechanism to recover Eth sent accidently to the contract

##

## [L-14] Lack of access control in ``executeProposal()`` function

### Impact
There is a lack of access control in the ``executeProposal()`` function of the Interchain Governance contract. Without access control, any external account can call this function and execute governance proposals, which may lead to unauthorized or malicious actions being taken

### POC

```solidity
FILE: 2023-07-axelar/contracts/cgp/governance/InterchainGovernance.sol

function executeProposal(
        address target,
        bytes calldata callData,
        uint256 nativeValue
    ) external payable {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue));

        _finalizeTimeLock(proposalHash);
        _call(target, callData, nativeValue);

        emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);
    }

```

### Recommended Mitigation
Add modifiers like ``onlyOwner`` to execute the proposals 

##

## [L-15] ``sourceChain`` and ``sourceAddress`` string comparison is vulnerable to timing attacks

### Impact
In the ``_execute`` function, the contract uses string comparison to verify if the provided ``sourceChain`` and ``sourceAddress`` match the expected governance chain and address. This method is vulnerable to timing attacks, where an attacker could exploit the time taken for the comparison to determine partial information about the comparison result.

### POC

```solidity
FILE: 2023-07-axelar/contracts/cgp/governance/InterchainGovernance.sol

 if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)
            revert NotGovernance();

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L92-L93

### Recommended Mitigation
Use ``bytes32`` for ``governanceChainHash`` and ``governanceAddressHash``, and use == for direct comparison

##

## [L-16] ``InterchainProposalSender.sol contract lacks nonce handling mechanism to prevent replay attacks

### Impact

The sendProposal function in the InterchainProposalSender contract can be susceptible to replay attacks without a nonce mechanism because it allows users to broadcast the same proposal multiple times.

 leading to the following issues:

 - Double Execution
 - Unintended Cost
 - Invalid Replays

### POC 

```solidity

function sendProposals(InterchainCalls.InterchainCall[] calldata interchainCalls) external payable override {
        // revert if the sum of given fees are not equal to the msg.value
        revertIfInvalidFee(interchainCalls);

        for (uint256 i = 0; i < interchainCalls.length; ) {
            _sendProposal(interchainCalls[i]);
            unchecked {
                ++i;
            }
        }
    }

```
### Recommended Mitigation
Add nonce mechanism to ``sendProposal `` function

##

## [L-17] Contract ``RemoteAddressValidator.sol`` is inherited ``upgradable` but  not actually ungradable contract

### Impact
The absence of an ``initialize`` function suggests that the contract is not intended to be upgradable using a proxy contract. Instead, it follows a traditional deployment approach, where the constructor is used to set up the initial state of the contract. Once deployed, the state and behavior of the contract are immutable, and any changes or updates would require deploying a new instance of the contract.

### POC 
```solidity
FILE: 2023-07-axelar/contracts/its/remote-address-validator/RemoteAddressValidator.sol

12: contract RemoteAddressValidator is IRemoteAddressValidator, Upgradable {

```
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L12


# NON CRITICAL FINDINGS

##

## [NC-1] Events is missing indexed fields

Index event fields make the field more quickly accessible to off-chain.

Each event should use three indexed fields if there are three or more fields.

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/interfaces/IAxelarServiceGovernance.sol#L15-L17

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/interfaces/IMultisigBase.sol#L22

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interfaces/IInterchainTokenService.sol#L32-L76

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interfaces/IExpressCallHandler.sol#L9-L43

## 

## [NC-2] Shorter the inheritance list

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L37-L44







