## Summary

6/8 optimizations were benchmarked using the protocol's tests(some tests were modified a bit so that the conditions were met)  using the following config: solc version 0.8.9, optimizer on, 1000 runs





### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Use calldata instead of memory for function arguments that do not get mutated | 8 |  38956 |
| [G&#x2011;02] | Use delete instead of assigning the string to "" | 2 |  1980 |
| [G&#x2011;03] | Emit the event after the delegatecall in case it fails | 1 |  876(in the revert case)  |
| [G&#x2011;04] | Some calculations can be done in an unchecked block | 2 |  144 |
| [G&#x2011;05] | Dont cache global variables | 6 |  43 |
| [G&#x2011;06] | Dont cache the length of a calldata array | 3 |  15 |
| [G&#x2011;07] | Revert as early as possible | 3 |  * |
| [G&#x2011;08] | Checking if the commandId is bigger than the max wastes gas  | 2 |  ~200 |




Total: 27 instances over 8 issues 
Total gas saved: 42214


## Gas Optimizations

### [G&#x2011;01]  Use calldata instead of memory for function arguments that do not get mutated

 When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. Storing the bytecode in memory will cost a lot of gas because of the memory expansion. Use calldata instead of memory where you can

Note: Everything was tested and no stack too deep errors were found

*Total gas saved: Avg 38956 gas*


*Gas Savings for MultisigBase rotateSigners() obtained via protocol's tests: Avg 557 gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  90571  |  
| After  |  90014  |  


*Gas Savings for Create3 deploy() obtained via protocol's tests: Avg 20972 gas*

|        |   AVG   |
| ------ | ------- |  
| Before |  631474 |  
| After  |  610502 |  


*Gas Savings for Create3Deployer deployAndInit() obtained via protocol's tests: Avg 8087 gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  824658 |  
| After  |  816571 |  


*Gas Savings for FinalProxy finalUpgrade() obtained via protocol's tests: Avg 3623 gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  498327 |  
| After  |  494704 |  

*Gas Savings for InterchainProposalSender sendProposal() obtained via protocol's tests: Avg 93 gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  71737  |  
| After  |  71644  |  

*Gas Savings for InterchainTokenService deployCustomTokenManager() obtained via protocol's tests: Avg 5624  gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  451127  |  
| After  |  445503  |  




*There are 8 instances of this issue:*

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L24

```solidity
File: gmp-sdk/deploy/ConstAddressDeployer.sol

24: function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {

```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L42

```solidity
File: gmp-sdk/deploy/ConstAddressDeployer.sol

42: function deployAndInit(
43:        bytes memory bytecode,
44:        bytes32 salt,
45:        bytes calldata init
46:    ) external returns (address deployedAddress_) {

```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/InitProxy.sol#L35

```solidity
File: gmp-sdk/upgradable/InitProxy.sol

35: function init(
36:        address implementationAddress,
37:        address newOwner,
38:        bytes memory params
39:    ) external {

```


https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3.sol#L51

```solidity
File: gmp-sdk/deploy/Create3.sol

51: function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {

```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L50

```solidity
File: gmp-sdk/deploy/Create3Deployer.sol

49: function deployAndInit(
50:        bytes memory bytecode,
51:        bytes32 salt,
52:        bytes calldata init
53:    ) external returns (address deployedAddress_) {

```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/FinalProxy.sol#L72

```solidity
File: gmp-sdk/upgradable/FinalProxy.sol

72: function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) public returns (address finalImplementation_) {

```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L343

```solidity
File: its/interchain-token-service/InterchainTokenService.sol

343: function deployCustomTokenManager(
344:        bytes32 salt,
345:        TokenManagerType tokenManagerType,
346:        bytes memory params
347:    ) public payable notPaused returns (bytes32 tokenId) {

```


https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L80

```solidity
File: interchain-governance-executor/InterchainProposalSender.sol

80: function sendProposal(
81:        string memory destinationChain,
82:        string memory destinationContract,
83:        InterchainCalls.Call[] calldata calls
84:    ) external payable override {

```



### [G&#x2011;02]  Use delete instead of assigning the string to ""

Deleting something in storage (setting a non-zero to a zero) lowers the gas cost because you get a gas refund for freeing up storage space. However using delete and assigning a string/bytes32 to "" is not the same and delete is much cheaper here.



*Gas Savings for RemoteAddressValidator removeTrustedAddress() obtained via protocol's tests: Avg 1980 gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  35145 |  
| After  |  33165 |  



*There are 2 instances of this issue:*

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L97



https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L98


```solidity
File: its/remote-address-validator/RemoteAddressValidator.sol

95: function removeTrustedAddress(string calldata chain) external onlyOwner {
96:        if (bytes(chain).length == 0) revert ZeroStringLength();
97:        remoteAddressHashes[chain] = bytes32(0);
98:        remoteAddresses[chain] = '';
99:        emit TrustedAddressRemoved(chain);
100:    }

```


*Should be replaced with:*

```solidity
delete remoteAddressHashes[chain];
delete remoteAddresses[chain];
```



### [G&#x2011;03] Emit the event after the delegatecall in case it fails

In AxelarGateway.sol an event is emitted right before a delegatecall. It would be better to emit the event after the delegatecall because the delegatecall can easily fail and you would have to pay for everything before the REVERT opcode was hit so you would have to pay for emitting that event.

Emit the event after the delegatecall is made to save gas in the revert case


*Gas Savings for AxelarGateway.sol upgrade() in the revert case(where the delegatecall fails): 876 gas*


*There is 1 instance of this issue:*

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L289

```solidity
File: cgp/AxelarGateway.sol

280: function upgrade(
281:     address newImplementation,
282:     bytes32 newImplementationCodeHash,
283:     bytes calldata setupParams
284: ) external override onlyGovernance {
285:     if (newImplementationCodeHash != newImplementation.codehash) revert InvalidCodeHash();
286:
287:     if (AxelarGateway(newImplementation).contractId() != contractId()) revert InvalidImplementation();
288:
289:     emit Upgraded(newImplementation);
290:
291:     // AUDIT: If `newImplementation.setup` performs `selfdestruct`, it will result in the loss of _this_ implementation (thereby losing the gateway)
292:     //        if `upgrade` is entered within the context of _this_ implementation itself.
293:     if (setupParams.length != 0) {
294:         (bool success, ) = newImplementation.delegatecall(abi.encodeWithSelector(IAxelarGateway.setup.selector, setupParams));
295:
296:         if (!success) revert SetupFailed();
297:     }
298:
299:     _setImplementation(newImplementation);
300: }

```


### [G&#x2011;04]  Some calculations can be done in an unchecked block 

In InterchainToken.sol there are 2 calculations that can be done in an unchecked block and its impossible for the number to underflow. Using the unchecked block reduces gas because its skips integer over/underflow checks

*Gas Savings for InterchainToken interchainTransferFrom(): 72 gas*


*Gas Savings for InterchainToken interchainTransfer(): 72 gas*


*There are 2 instances of this issue:*


https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L57-L59

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token/InterchainToken.sol#L96-L98

The calculation for allowance_ can be put it an unchecked block because it is checked above in the if statement(allowance_ > type(uint256).max - amount)

```solidity
File: interchain-token/InterchainToken.sol

if (allowance_ > type(uint256).max - amount) {
      allowance_ = type(uint256).max - amount;
}

```


### [G&#x2011;05] Dont cache global variables

Because msg.sender is a global variable, caching it and putting it on the stack would only increase the gas costs because of the extra DUPs, SWAPs and POPs used

*Total gas saved: 45 gas*

*Gas Savings for TokenManager sendToken() obtained via protocol's tests: Avg 6 gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  153846 |  
| After  |  153840 |  

There are 5 other functions with the same gas savings as sendToken():

TokenManager.sol:  callContract()


InterchainTokenService.sol: deployCustomTokenManager(), deployRemoteCustomTokenManager(), expressReceiveToken(), expressReceiveTokenWithData()



*Gas Savings for InterchainToken interchainTransfer() obtained via protocol's tests: Avg 13 gas*

|        |   AVG   |
| ------ | ------- | 
| Before |  173832 |  
| After  |  173819 |  


*There are 6 instances of this issue:*

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/TokenManager.sol#L89

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/TokenManager.sol#L115

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L348

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L372

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L447

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L478



### [G&#x2011;06] Dont cache the length of a calldata array

When the optimizer is enabled caching a calldata array length will increase gas because of the extra DUPs, SWAPs and POPs used to declare the length variable. 


*This saves 5 gas units*


*There are 3 instances of this issue:*

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L535

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L107

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L120


### [G&#x2011;07] Revert as early as possible

Input validations should always go first because we want out tx to revert as early as possible so no operations were made before the REVERT opcode was hit and no gas was wasted on them. 

Placing the input validations in the right place can significantly decrease gas in the MultisigBase

The gas savings here depend on how big the loop is

In _rotateSigners() first a loop runs where storage operations are done and after that the newThreshold is checked. If the tx revert because the newThreshold is 0 or bigger than the length then you would have to pay for all the operations before that revert. And because storage operations and loops are very expensive this can cost a lot. Placing the newThreshold checks before the loop can decrease gas in the revert case a lot

*There are 2 instances of this issue:*

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L149-L161

```solidity
File: cgp/auth/MultisigBase.sol

function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
        uint256 length = signers.accounts.length;

        // Clean up old signers.
        for (uint256 i; i < length; ++i) {
            signers.isSigner[signers.accounts[i]] = false;
        }

        length = newAccounts.length;

        if (newThreshold > length) revert InvalidSigners();

        if (newThreshold == 0) revert InvalidSignerThreshold();
        ...

```

Replace with:

```solidity
function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {

        if (newThreshold > newAccounts.length) revert InvalidSigners();

        if (newThreshold == 0) revert InvalidSignerThreshold();
        uint256 length = signers.accounts.length;

        // Clean up old signers.
        for (uint256 i; i < length; ++i) {
            signers.isSigner[signers.accounts[i]] = false;
        }

        length = newAccounts.length;
        ...
```

### [G&#x2011;08] Checking if the commandId is bigger than the max wastes gas


In AxelarServiceGovernance and InterchainGovernance  _processCommand() there is a check if the commandId is bigger than the max of the enum. This check only wastes gas because if the commandId is bigger than the max then the tx will revert because of GovernanceCommand(commandId) or ServiceGovernanceCommand(commandId);. 

This saves ~50 gas

And because the tx reverts, the last else-if can be replaced with an else which will also save ~50 gas(in case that the last command id is used)

*There are 2 instances of this issue:*

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L72
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/InterchainGovernance.sol#L113

```solidity
if (commandId > uint256(type(ServiceGovernanceCommand).max)) {
   revert InvalidCommand();
}

//Reverts here if the commandId is bigger than the max
ServiceGovernanceCommand command = ServiceGovernanceCommand(commandId);
```