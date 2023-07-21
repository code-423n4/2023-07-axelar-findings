## 1. `enum` VALUES CAN BE DIRECTLY CHECKED AGAINST `0` AND `1` TO SAVE GAS

In the `InterchainGovernance._processCommand` function, the `GovernanceCommand` enum values are checked against the derived value of `command` variable. But there is no need to create a new variable like `command` since the `commandId` can be directly checked for `0` value or `1` value to perform the same execution flow.

Hence by using the `commandId` directly by comparing it against `0` and `1` can remove extra steps in the execution flow thus saving gas.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L124
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L127
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L133

## 2. USE `delete` INSTEAD OF ASSIGNING TO DEFAULT VALUE.

When `deleting` state variable we can get a gas refund. Hence it is recommended to delete a state variable rather than setting it to a default value of `0`.

        voting.voteCount = 0; 

Above operation can be changed as follows to get a gas refund.

        delete voting.voteCount;

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L66
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L123

## 3. INPUT VALIDATION CHECKS SHOULD BE PERFORMED PRIOR TO STATE VARIABLE ASSIGNMENT

In the `MultisigBase._rotateSigners` function, there are input validation checks performed inside the `for` loop as follows:

            if (signers.isSigner[account]) revert DuplicateSigner(account);
            if (account == address(0)) revert InvalidSigners(); 

But the state variables are assigned prior to these input validation checks as shown below:

        signers.accounts = newAccounts;
        signers.threshold = newThreshold;

Hence if any of the above input validation checks revert, then the gas spend on the state variable assignment will be lost. Hence it is recommended to assign the state variables after the input validation checks for the `newAccounts` input address array.

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L165-L166
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L172-L173

## 4. NO NEED TO ASSIGN DEFAULT VALUE TO VARIABLES WHEN DECLARING VARIABLES

There are multiple occassions in the code base where default value is assigned to memory variables when declaring them. But the variables are by default assigned to default value when they are declared and hence no need to assign to defualt value explicitly.

Following is one instance of such assignment to default value.

        uint256 totalGas = 0; 

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L105
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74