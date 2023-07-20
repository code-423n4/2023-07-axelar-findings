## InterchainGovernance
1) trere's the receive function but only one function(executeProposal) to take out ETH sent to "InterchainGovernance" contract and the amount sent is arbitrary and not related to the balance of the contract,making the ETH received locked forever
## Gateway
1) should check that the address used in the constructor are != addr(0)
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L64
2) Attacker could bypass mint limit of a token,by deploying the same token with same tokenAddress but different symbol and mint from the new one,but will be minted by the "good" one,because the check of the mint limit will be done only on the symbol,so limit on the symbol and not on the address,multiple tokens could point to the same address but having different symbol,making the limit unlimited.
could be resolved by considering even the tokenAddress on the validation checks before mint,tokenAddress will be managed as symbols and NEEDS to be unique.
3) no check for address!= addr(0) in _mintToken function (line 512) that could lead to minted tokens to addr(0)
4) remove all deprecated functions:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L213-L226
## Interchain Proposal Sender
1) could born soft/hard fork problems related to destinationChain by changing the chainID of specific chain,should be inserted into the gasService a way to change an existing chainID to another,making the contract upgradable
2) the msg.value should be checked that will satisfy the requirement of the proposal txn in the target chain and if less will revert
## Interchain Proposal Executor
1) cannot send and receive funds due to the lack of payable in the functions and missing receive/fallback function,making the _execute unable to send value unlike what the comments wanted to do:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L39
## Create3Deployer
1) should insert a mapping that tracks "salt" variable x each msg.sender,making it more easy to track and making sure that "`salt` must not have been used already by the same `msg.sender`." --line 46 and 27
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L46C10-L46C74
## Base Proxy
1) cannot withdraw ETH from contract,for lack of related function
## Fixed Proxy
1) Remember to override receive,if not Ethers will be locked there