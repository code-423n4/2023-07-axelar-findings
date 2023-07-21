# Low

1. `MultisigBase#_rotateSigners` can accept any number of accounts to be set as signers by looping through them, but due to the difference between block gas limit across chains this may lead to failing to rotate signers for a big number of accounts in certain chains which will make a difference between number of signers across chains ( number of signers should be the same across chains according to devs ) .

# Non critical
1. `Multisig#execute` is a relevant function in the protocol architecture, thought it doesn’t emit any event for off-chain actors to inform them .
2.  there no reason to log eta (The time after which the proposal can be executed)  when Proposal is cancelled since it won't be executed it can be confusing . https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IInterchainGovernance.sol#L19