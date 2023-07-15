
# Divide by zero should be avoided 

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