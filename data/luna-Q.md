
QA Report
=========

Table of content
================

* [QA Findings](#qa-findings)
	* [Missing two steps verification process](#missing-two-steps-verification-process)

# QA Findings

## Missing two steps verification process
The process of transferring ownership is dangerous since typing the wrong address can lead to severe implications. It is better to have to steps verification process with set and claim functions to decrease the chances of human error.

- [AxelarGateway.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol)
- [Ownable.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/Ownable.sol)
- [Upgradable.sol](https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/util/Upgradable.sol)

