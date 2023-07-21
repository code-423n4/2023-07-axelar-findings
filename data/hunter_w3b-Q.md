## [L-01] `AxelarServiceGovernance::executeMultisigProposal` function allows proposal execution without verifying that all authorized signers have provided their approval

The executeMultisigProposal function is designed to execute a proposal if it has been approved by all authorized signers. The approval status is checked using the multisigApprovals mapping, which tracks whether a proposal has been approved or not.
The function lacks proper validation to ensure that all authorized signers have provided their approval before proceeding with the execution. As a result, if just one signer's approval is registered in the multisigApprovals mapping, the function allows proposal execution, even if other signers have not yet provided their consent. This exposes the contract to unauthorized executions, potentially leading to unintended actions and mismanagement of funds.

### Steps to Reproduce:

1. Deploy the multisig contract and set multiple authorized signers.
2. Create a proposal and obtain approval from only one of the signers.
3. Attempt to execute the proposal using the executeMultisigProposal function

### Expected Behavior:

The function should require approval from all authorized signers before executing the proposal. If any signer has not approved the proposal, the execution should not proceed.

### Actual Behavior:

The function executes the proposal even if only one signer's approval is recorded in the multisigApprovals mapping. Other signers' approvals are not enforced, leading to unauthorized executions.

## [G-02] Publicly Accessible Contract Addresses in Contract

The contract under review exposes the addresses of the token manager deployer and standardized token deployer through public getter functions tokenManagerDeployer() and standardizedTokenDeployer(). While the exposure of these addresses may be intentional for transparency and accessibility purposes

Exposing contract addresses publicly can present several security risks, including:

a. Unauthorized Access: Any external entity can easily access and view the contract addresses, potentially leading to unauthorized interactions with the contract's critical functions.

b. Malicious Interference: Malicious actors may exploit the exposed addresses to interfere with token deployment, deploy malicious contracts, or target specific token managers for attacks.

c. Contract Vulnerability: Publicly exposing contract addresses might aid attackers in identifying potential vulnerabilities or weaknesses in the contract's design.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L82-L88

```solidity

    function tokenManagerDeployer() external view returns (address tokenManagerDeployerAddress);

    /**
     * @notice Returns the address of the standardized token deployer contract.
     * @return standardizedTokenDeployerAddress The address of the standardized token deployer contract.
     */
    function standardizedTokenDeployer() external view returns (address standardizedTokenDeployerAddress);
```

the IInterchainTokenService interface includes two getter functions, tokenManagerDeployer() and standardizedTokenDeployer(). These functions are publicly accessible, meaning anyone can call them and retrieve the addresses of the token manager deployer and standardized token deployer contracts.

Exposing these addresses publicly may allow external entities to know and interact with these contract addresses, which may lead to unintended consequences if proper security measures are not in place.
