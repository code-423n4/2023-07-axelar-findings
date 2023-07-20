# Issue: Lack of input validation in contracts/cgp/AxelarGateway.sol
The contract doesn't perform extensive input validation for some functions. For example, in the deployToken function, the name and symbol parameters are assumed to be valid without any checks.
# Recommendation: Implement thorough input validation checks for all user-supplied parameters. Validate and sanitize the inputs to prevent potential security issues such as buffer overflows, unexpected behavior, or vulnerabilities arising from maliciously crafted inputs.

# Lack of error messages
 Some error messages are missing or not descriptive enough, which can make it difficult to identify the cause of a failed transaction or provide clear feedback to users interacting with the contract
# Recommendation: Improve error handling by providing informative and descriptive error messages. This will enhance the usability of the contract and help users understand why a transaction failed, enabling them to take appropriate actions.