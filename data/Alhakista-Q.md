# Issue: Lack of input validation in contracts/cgp/AxelarGateway.sol
The contract doesn't perform extensive input validation for some functions. For example, in the deployToken function, the name and symbol parameters are assumed to be valid without any checks.
# Recommendation: Implement thorough input validation checks for all user-supplied parameters. Validate and sanitize the inputs to prevent potential security issues such as buffer overflows, unexpected behavior, or vulnerabilities arising from maliciously crafted inputs.

# Lack of error messages
 Some error messages are missing or not descriptive enough, which can make it difficult to identify the cause of a failed transaction or provide clear feedback to users interacting with the contract
# Recommendation: Improve error handling by providing informative and descriptive error messages. This will enhance the usability of the contract and help users understand why a transaction failed, enabling them to take appropriate actions.

# Unchecked Array Iteration in 
contracts/interchain-governance-executor/InterchainProposalSender.sol
contracts/interchain-governance-executor/InterchainProposalExecutor.sol
In the 'sendProposals', 'revertIfInvalidFee' and  '_executeProposal' functions, the array iteration is done using an unchecked loop. This means there is no bound check on the array length, which could lead to unintended behavior if the array is empty or contains a large number of elements. If the array size is excessive, it could cause the contract to run out of gas and result in a transaction failure. It's important to ensure that the loop conditions are properly checked to prevent such issues.

# Unchecked External Call Result in contracts/interchain-governance-executor/InterchainProposalSender.sol
In the _sendProposal function, the contract calls the gateway.callContract function without checking its return value. If the callContract function returns false, it indicates a failure in the external call, which could potentially lead to issues if not handled properly. Always ensure to check the return value of external calls and handle failures accordingly.

# Invalid Fee Handling in contracts/interchain-governance-executor/InterchainProposalSender.sol
The revertIfInvalidFee function checks if the total gas sent with the transaction matches the total gas specified in the interchainCalls array. However, this may not be the most suitable approach to handle fees. Comparing gas directly with the value may lead to precision issues, and it might be better to consider alternative fee models.

# Transaction Ordering Dependence in contracts/interchain-governance-executor/InterchainProposalSender.sol
The contract does not implement any mechanism to prevent transaction ordering dependence vulnerabilities. Suppose the contract relies on external calls (e.g., the gasService.payNativeGasForContractCall and gateway.callContract functions) that alter contract state or perform sensitive operations. In that case, malicious actors could manipulate the order of transactions to exploit race conditions or perform attacks.

# Access Control Issues 
The contract uses an Ownable modifier to control certain functions' access. However, the contract does not specify any access control mechanism for executing proposals or setting the whitelist status of proposal callers and senders. This may lead to potential vulnerabilities if unauthorized parties can execute proposals or modify the whitelists. It is essential to implement access control mechanisms like role-based access or permissioned functions to prevent unauthorized access.

# Potential Reentrancy Vulnerability
The '_onTargetExecutionFailed' function allows for reversion with a reason if the target contract execution fails. Reverting with a reason involves reading from memory, which could potentially lead to a reentrancy vulnerability if there are any external calls or state changes after the revert statement. To avoid this, ensure that external calls are made after the revert statement, and critical state changes are handled before reverting.