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






# Revert on Transfer to the Zero Address

Revert on Zero Value Approvals

Revert on Zero Value Transfers

Approval Race Protections
Some tokens (e.g. USDT, KNC) do not allow approving an amount M > 0 when an existing amount N > 0 is already approved. This is to protect from an ERC20 attack vector described here.

Tokens with Blocklists

Pausable Tokens

Missing Return Values- some tokens not return any values 



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

##

## [NC-3] For Critical functions should emit events



