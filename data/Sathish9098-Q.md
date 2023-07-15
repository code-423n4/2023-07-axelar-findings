
# Divide by zero should be avoided 

# Event is missing the indexed field 

# Use safeTranfer/SafetranferFrom

# Return values of transfer functions should be checked 

# Return values of approve should be checked 

# Initialize functions could be frontrun 

# For Critical functions should emit events

# Use safeAllowance instead of normal approve 

# Need to approve 0 first for some of the tokens 

Revert on Transfer to the Zero Address

Revert on Zero Value Approvals

Revert on Zero Value Transfers

Approval Race Protections
Some tokens (e.g. USDT, KNC) do not allow approving an amount M > 0 when an existing amount N > 0 is already approved. This is to protect from an ERC20 attack vector described here.

Tokens with Blocklists

Pausable Tokens

Missing Return Values- some tokens not return any values 

Use ownable2step instead of normal ownable 

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

