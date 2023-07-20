## InterchainGovernance
1) use "proposalHash" calculation directly in line 75,78
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L73
## AxelarServiceGorvernance
1) should directly use call of "_scheduleTimeLock" in line 87-89
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L87C19-L89
2) no need to return in 4 instances: line 90-95-100-105
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L86C34-L105
## MultiSigBase
1) should use directly ``voting.voteCount + 1`` in the instances where voteCount compare(line 59-61)
2) the add sum should be unchecked in line 124-163
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L124
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L163
## Gateway
1) directly use limits[i] instead of creating an new variable limit for a one-time instance per-cycle
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L270
2) setting "allowOperatorshipTransfer" to false is useless,in fact wont be called again as a local variable
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L365
3) no need to create a variable,could use directly "_getCreate2Address" call inside if statement as an argument of _hasCode function
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L435C45-L437
## Interchain Proposal Sender
1) totalGas add should be inserted in the unchecked block due to the uint256
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L107
2) should not continue check the length of an object inside a loop,initialize the variable with length of interchainCalls and then inside the loop make the for look something like this: 
``for(uint256 i=0; i<newVariable(uint256 newVariable = interchainCalls.length); ){...}``
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63
## Interchain Proposal Executor
1) should not continue check the length of an object inside a loop,initialize the variable with length outside loop and re-read the new variable in the loop conditions
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74
2) should directly reference and not create a new variable ( calls[i] )
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L75
## Create3Deployer
1) no need to initialize another varaible (deploySalt),use it directly making something like this:
``
deployedAddress_ = Create3.deploy(keccak256(abi.encode(msg.sender, salt)), bytecode);
``
in those two instances:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L30-L31
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L54-L55
## ConstAddressDeployer
1) no need to initialize newSalt in line 63,use it directly
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L63
## Interchain Tokens
1) use mapping reference directly without using new variable
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L86
## Interchain Token Service
1) should use directly the call or msg.sender(depend on the line) instead of creating new variable
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L372
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L398-L401
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L427-L428