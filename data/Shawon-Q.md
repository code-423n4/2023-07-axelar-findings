Minor suggestion:

Inside `AdminMultisigBase.sol` 

46: if (adminVoteCount < _getAdminThreshold(adminEpoch)) return;

there is no return value in this if statement. It will get confusing when this execution fails as there will be no results shown to identify the reason behind this failure.
add a return value to make this if statement.