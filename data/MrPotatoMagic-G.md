# Gas Optimizations Report

## [G-01] Function parameters can be made calldata to save gas

Function parameters that are not being modified can be made calldata instead of memory.
There are 2 instances of this:

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L532

Deployment cost: 3980826 - 3970282 = 10554 gas saved 

string memory variable on Line 532 can be made calldata.
```solidity
File: contracts/cgp/AxelarGateway.sol
530: function _burnTokenFrom(
531:         address sender,
532:         string memory symbol, //@audit make this calldata
533:         uint256 amount
534:     ) 
```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L142

Deployment cost: 816076 - 805683 = 10393 gas saved
Function cost: 75010 - 74425 = 585 gas saved (per rotateSigners() function call)

address[] memory newAccounts can be made calldata.
```solidity
File: contracts/cgp/auth/MultisigBase.sol
142: function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {
```

## [G-02] Mapping can be made private to save gas

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L22

Deployment cost: 2177589 - 2164879 = 12710 gas saved
```solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol
22: mapping(bytes32 => bool) public multisigApprovals;
```

## [G-03] Else statement can be removed to save gas

There are 3 instances of this:

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L132

The GovernanceCommand enum has only two members, thus adding just one check for command is enough. If the check is false, we know the command entered is for the 2nd member (CancelTimelockProposal).

Deployment cost: 1241231 - 1237343 = 3888 gas saved
Function cost: 30777 - 30714 = 63 gas saved per [_execute() function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L87) call (the if block below is in an internal function [_processCommand()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L113), which is only called by the _execute function)

Instead of this:
```solidity
File: contracts/cgp/governance/InterchainGovernance.sol
127: if (command == GovernanceCommand.ScheduleTimeLockProposal) {
128:             eta = _scheduleTimeLock(proposalHash, eta);
129: 
130:             emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);
131:             return;
132:         } else if (command == GovernanceCommand.CancelTimeLockProposal) {
133:             _cancelTimeLock(proposalHash);
134: 
135:             emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);
136:             return;
137:         }
```
Use this:
```solidity
File: contracts/cgp/governance/InterchainGovernance.sol
127: if (command == GovernanceCommand.ScheduleTimeLockProposal) {
128:             eta = _scheduleTimeLock(proposalHash, eta);
129: 
130:             emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);
131:             return;
132: }
133:    _cancelTimeLock(proposalHash);
134: 
135:    emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);
136:    return;
```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L101

The ServiceGovernanceCommand enum has only four members, thus adding three checks for command is enough. If the first three checks are false, we know the command entered is for the 4th member (CancelMultisigApproval).

Deployment cost: 2177589 - 2173718 = 3871 gas saved
Function cost: approximately 60 gas saved per [_execute() function](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L87) call (the if block below is in an internal function [_processCommand()](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L72), which is only called by the _execute function.)


Instead of this:
```solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol
86: if (command == ServiceGovernanceCommand.ScheduleTimeLockProposal) {
87:             eta = _scheduleTimeLock(proposalHash, eta);
88: 
89:             emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);
90:             return;
91:         } else if (command == ServiceGovernanceCommand.CancelTimeLockProposal) {
92:             _cancelTimeLock(proposalHash);
93: 
94:             emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);
95:             return;
96:         } else if (command == ServiceGovernanceCommand.ApproveMultisigProposal) {
97:             multisigApprovals[proposalHash] = true;
98: 
99:             emit MultisigApproved(proposalHash, target, callData, nativeValue);
100:             return;
101:         } else if (command == ServiceGovernanceCommand.CancelMultisigApproval) {
102:             multisigApprovals[proposalHash] = false;
103: 
104:             emit MultisigCancelled(proposalHash, target, callData, nativeValue);
105:             return;
106:         }
```
Use this:
```solidity
File: contracts/cgp/governance/AxelarServiceGovernance.sol
86: if (command == ServiceGovernanceCommand.ScheduleTimeLockProposal) {
87:             eta = _scheduleTimeLock(proposalHash, eta);
88: 
89:             emit ProposalScheduled(proposalHash, target, callData, nativeValue, eta);
90:             return;
91:         } else if (command == ServiceGovernanceCommand.CancelTimeLockProposal) {
92:             _cancelTimeLock(proposalHash);
93: 
94:             emit ProposalCancelled(proposalHash, target, callData, nativeValue, eta);
95:             return;
96:         } else if (command == ServiceGovernanceCommand.ApproveMultisigProposal) {
97:             multisigApprovals[proposalHash] = true;
98: 
99:             emit MultisigApproved(proposalHash, target, callData, nativeValue);
100:             return;
101:         }
102:            multisigApprovals[proposalHash] = false;
103: 
104:            emit MultisigCancelled(proposalHash, target, callData, nativeValue);
105:            return;      
```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L78

Deployment cost: 1223359 - 1222279 = 1080 gas saved

Instead of this:
```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
78: if (!success) {
79:      _onTargetExecutionFailed(call, result);
80: } else {
81:      _onTargetExecuted(call, result);
82: }
```
Use this:
```solidity
File: contracts/interchain-governance-executor/InterchainProposalExecutor.sol
78: if (!success) {
79:      _onTargetExecutionFailed(call, result);
80: }
81: _onTargetExecuted(call, result);
```

### Total deployment cost: 42496 gas saved
### Total function execution cost: 708 gas per call 