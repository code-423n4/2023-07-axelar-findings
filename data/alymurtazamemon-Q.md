# Axelar Network - Quality Assurance Report

# Low Risk Findings

## Table Of Content

| Number                                                               | Issue                                                                                                                | Instances |
| :------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------- | --------: |
| [L-01](#l-01---should-check-proposal-existence-before-cancelling-it) | [Should check proposal existence before cancelling it](#l-01---should-check-proposal-existence-before-cancelling-it) |         2 |
| [L-02](#l-02---avoid-shadowing-variables)                            | [Avoid Shadowing Variables](#l-02---avoid-shadowing-variables)                                                       |        10 |
| [L-03](#l-03---should-validate-these-paremeters-and-variables)       | [Should Validate these Paremeters and Variables](#l-03---should-validate-these-paremeters-and-variables)             |         4 |

### [L-01] - Should check proposal existence before cancelling it.

**Details**

In the `AxelarServiceGovernance` contract's `_processCommand` function we cancel the TimeLock and Multisig proposals but we do not check whether they exist or not.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[AxelarServiceGovernance.sol - Line 91 - 95](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L91-L95)

[AxelarServiceGovernance.sol - Line 101 106](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L101-L106)

</details>

**Recommended Mitigation Steps**

If they do not exist then we should let the signers know the proposals do not exist.

### [L-02] - Avoid Shadowing Variables.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

Variables with name `gateway`, `governanceChain` and `governanceAddress` are all defined in the parent contracts.

[AxelarServiceGovernance.sol - Line 40](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L40)

Variable with name `gateway` is defined in the parent contract and shadow here.

[InterchainGovernance.sol - Line 38](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L38)

Variable with name `operator` is defined in the parent countract and shadow here.

[InterchainTokenService.sol - Line 241 - 242](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L241-L242)

[InterchainTokenService.sol - Line 251 - 252](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L251-L252)

[InterchainTokenService.sol - Line 262 - 267](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L262-L267)

[InterchainTokenService.sol - Line 422 - 428](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L422-L428)

[InterchainTokenService.sol - Line 770 - 799](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L770-L799)

Variable with name `implementation` is defined in the parent countract and shadow here.

[InterchainTokenService.sol - Line 559 - 566](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L559-L566)

Variable with name `tokenAddress` is defined in the parent countract and shadow here.

[TokenManagerLockUnlock.sol - Line 32 - 35](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerLockUnlock.sol#L32-L35)

[TokenManagerMintBurn.sol - Line 33 - 36](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L33-L36);

</details>

**Recommended Mitigation Steps**

Replace them with different meaningful names.

### [L-03] - Should Validate these Paremeters and Variables.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

Should add zero check for `governanceChain_` and `governanceAddress_` because if these will empty then hashes will also empty and it will create a bug to allow executing on wrong chains.

[InterchainGovernance.sol - Line 35 - 36](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L35-L36)

The address of the contract deployed through CREATE3 should be validate for zero address before performing any action using it.

[Create3Deployer.sol - Line 55](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L55)

[FinalProxy.sol - 79](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L79)

Should validate the `implementationAddress` for zero address.

[FinalProxy.sol - Line 26 - 30](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/FinalProxy.sol#L26-L30)

</details>

**Recommended Mitigation Steps**

Should validate for safty.

# Non Critical Findings

## Table Of Content

| Number                                                             | Issue                                                                                                            | Instances |
| :----------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------- | --------: |
| [N-01](#n-01---lack-of-indexed-events-parameters)                  | [Lack of Indexed Events Parameters](#n-01---lack-of-indexed-events-parameters)                                   |         2 |
| [N-02](#n-02---should-add-override-keyword-for-better-readability) | [Should add override keyword for better readability](#n-02---should-add-override-keyword-for-better-readability) |         1 |

### [N-01] - Lack of Indexed Events Parameters.

**Details**

To facilitate off-chain searching and filtering for specific events information try to utilize the 3 available indexed parameters for events.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

It is an important information and should be indexed.

[IMultisigBase.sol - Line 22](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol#L22)

This is important information and should be indexed so can be easily read by the outside interfaces.

[IRemoteAddressValidator.sol - Line 14 - 17](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IRemoteAddressValidator.sol#L14-L17)

</details>

**Recommended Mitigation Steps**

Add `indexed` keyword with events paramets for useful information.

### [N-02] - Should add override keyword for better readability.

**Details**

Should add the `override` keyword in the function signature so reader can understand the function is comming from parent contract.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[Distributable.sol - Line 51](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/utils/Distributable.sol#L51)

</details>
