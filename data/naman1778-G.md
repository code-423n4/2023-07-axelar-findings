## [G-01] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

There are 24 instances of this issue in 15 files:

    File: IMultisigBase.sol	

    73: function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/interfaces/IMultisigBase.sol

    File: MultisigBase.sol	

    142: function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol

    File: AxelarGateway.sol	

    238: function tokenFrozen(string memory) external pure override returns (bool) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: InterchainProposalSender.sol	

    80: function sendProposal(
    81:     string memory destinationChain,
    82:     string memory destinationContract,
    83:     InterchainCalls.Call[] calldata calls
    84: ) external payable override {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol

    File: IInterchainProposalSender.sol	

    15: function sendProposals(InterchainCalls.InterchainCall[] memory calls) external payable;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

    File: ConstAddressDeployer.sol	

    24: function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {

    42: function deployAndInit(
    43:     bytes memory bytecode,
    44:     bytes32 salt,
    45:     bytes calldata init
    46: ) external returns (address deployedAddress_) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

    File: Create3Deployer.sol	

    49: function deployAndInit(
    50:     bytes memory bytecode,
    51:     bytes32 salt,
    52:     bytes calldata init
    53: ) external returns (address deployedAddress_) {
    
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol

    File: Create3.sol	

    21: function deploy(bytes memory bytecode) external {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol

    File: InitProxy.sol	

    35: function init(
    36:     address implementationAddress,
    37:     address newOwner,
    38:     bytes memory params
    39: ) external {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol

    File: IInitProxy.sol	

    9: function init(
    10:     address implementationAddress,
    11:     address newOwner,
    12:     bytes memory params
    13: ) external;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/interfaces/IInitProxy.sol

    File: IFinalProxy.sol	

    11: function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) external returns (address);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/interfaces/IFinalProxy.sol

    File: InterchainTokenService.sol	

    417: function deployAndRegisterRemoteStandardizedToken(
    418:     bytes32 salt,
    419:     string calldata name,
    420:     string calldata symbol,
    421:     uint8 decimals,
    422:     bytes memory distributor,
    423:     bytes memory operator,
    424:     string calldata destinationChain,
    425:     uint256 gasValue
    426: ) external payable notPaused {

    467: function expressReceiveTokenWithData(
    468:     bytes32 tokenId,
    469:     string memory sourceChain,
    470:     bytes memory sourceAddress,
    471:     address destinationAddress,
    472:     uint256 amount,
    473:     bytes calldata data,
    474:     bytes32 commandId
    475: ) external {

    502: function transmitSendToken(
    503:     bytes32 tokenId,
    504:     address sourceAddress,
    505:     string calldata destinationChain,
    506:     bytes memory destinationAddress,
    507:     uint256 amount,
    508:     bytes calldata metadata
    509: ) external payable onlyTokenManager(tokenId) notPaused {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: IInterchainTokenService.sol	

    145: function getParamsLockUnlock(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

    153: function getParamsMintBurn(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

    162: function getParamsLiquidityPool(
    163:     bytes memory operator,
    164:     address tokenAddress,
    165:     address liquidityPoolAddress
    166: ) external pure returns (bytes memory params);

    194: function deployCustomTokenManager(
    195:     bytes32 salt,
    196:     TokenManagerType tokenManagerType,
    197:     bytes memory params
    198: ) external payable returns (bytes32 tokenId);

    245: function deployAndRegisterRemoteStandardizedToken(
    246:     bytes32 salt,
    247:     string calldata name,
    248:     string calldata symbol,
    249:     uint8 decimals,
    250:     bytes memory distributor,
    251:     bytes memory operator,
    252:     string calldata destinationChain,
    253:     uint256 gasValue
    254: ) external payable;

    272: function transmitSendToken(
    273:     bytes32 tokenId,
    274:     address sourceAddress,
    275:     string calldata destinationChain,
    276:     bytes memory destinationAddress,
    277:     uint256 amount,
    278:     bytes calldata metadata
    279: ) external payable;

    339: function expressReceiveTokenWithData(
    341:    bytes32 tokenId,
    342:    string memory sourceChain,
    343:    bytes memory sourceAddress,
    344:    address destinationAddress,
    345:    uint256 amount,
    346:    bytes calldata data,
    347:    bytes32 commandId
    348: ) external;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IInterchainTokenService.sol

    File: IRemoteAddressValidator.sol	

    32: function addTrustedAddress(string memory sourceChain, string memory sourceAddress) external;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IRemoteAddressValidator.sol

    File: IExpressCallHandler.sol	

    71: function getExpressReceiveTokenWithData(
    72:     bytes32 tokenId,
    73:     string memory sourceChain,
    74:     bytes memory sourceAddress,
    75:     address destinationAddress,
    76:     uint256 amount,
    77:     bytes calldata data,
    78:     bytes32 commandId
    79: ) external view returns (address expressCaller);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interfaces/IExpressCallHandler.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized("Naman");
            c1.optimized("Naman");
        }
    }
    
    contract Contract0 {
    
         function not_optimized(string memory a) public returns(string memory){
             return a;
         }
    }
    
    contract Contract1 {
    
         function optimized(string calldata a) public returns(string calldata){
             return a;
         }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 100747                                    | 535             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 790             | 790 | 790    | 790 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 66917                                     | 366             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 556             | 556 | 556    | 556 | 1       |

## [G-02] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

There are 31 instances of this issue in 10 files:

    File: AxelarGateway.sol	

    562: return keccak256(abi.encode(PREFIX_TOKEN_MINT_AMOUNT, symbol, day));

    584: return keccak256(abi.encode(PREFIX_CONTRACT_CALL_APPROVED, commandId, sourceChain, sourceAddress, contractAddress, payloadHash));

    598: abi.encode(
    599:     PREFIX_CONTRACT_CALL_APPROVED_WITH_MINT,
    600:    commandId,
    601:    sourceChain,
    602:    sourceAddress,
    603:    contractAddress,
    604:    payloadHash,
    605:    symbol,
    606:    amount
    607:)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: InterchainProposalSender.sol	

    89: bytes memory payload = abi.encode(msg.sender, interchainCall.calls);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol

    File: IInterchainProposalSender.sol	

    66: emit ProposalExecuted(keccak256(abi.encode(sourceChain, sourceAddress, interchainProposalCaller, payload)));

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/interfaces/IInterchainProposalSender.sol

    File: ConstAddressDeployer.sol	

    25: deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

    47: deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));

    63: bytes32 newSalt = keccak256(abi.encode(sender, salt));

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

    File: Create3Deployer.sol	

    30: bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));

    54: bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));    

    68: bytes32 deploySalt = keccak256(abi.encode(sender, salt));

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3Deployer.sol

    File: InterchainTokenService.sol	

    203: tokenId = keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_ID, chainNameHash, tokenAddress));

    214: tokenId = keccak256(abi.encode(PREFIX_CUSTOM_TOKEN_ID, sender, salt));

    242: params = abi.encode(operator, tokenAddress);

    252: params = abi.encode(operator, tokenAddress);

    267: params = abi.encode(operator, tokenAddress, liquidityPoolAddress);

    313: _deployTokenManager(tokenId, TokenManagerType.LOCK_UNLOCK, abi.encode(address(this).toBytes(), tokenAddress));

    401: _deployTokenManager(tokenId, tokenManagerType, abi.encode(msg.sender.toBytes(), tokenAddress));

    512: payload = abi.encode(SELECTOR_SEND_TOKEN, tokenId, destinationAddress, amount);

    520: payload = abi.encode(SELECTOR_SEND_TOKEN_WITH_DATA, tokenId, destinationAddress, amount, sourceAddress.toBytes(), metadata);

    696: abi.encode(operatorBytes.length == 0 ? address(this).toBytes() : operatorBytes, tokenAddress)

    755: bytes memory payload = abi.encode(SELECTOR_DEPLOY_TOKEN_MANAGER, tokenId, tokenManagerType, params);

    780: bytes memory payload = abi.encode(

    828: return keccak256(abi.encode(PREFIX_STANDARDIZED_TOKEN_SALT, tokenId));

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: ExpressCallHandler.sol	

    32: slot = uint256(keccak256(abi.encode(PREFIX_EXPRESS_RECEIVE_TOKEN, tokenId, destinationAddress, amount, commandId)));

    57: abi.encode(

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol

    File: FlowLimit.sol	

    47: slot = uint256(keccak256(abi.encode(PREFIX_FLOW_OUT_AMOUNT, epoch)));

    56: slot = uint256(keccak256(abi.encode(PREFIX_FLOW_IN_AMOUNT, epoch)));

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol

    File: StandardizedTokenDeployer.sol	

    62: bytes memory params = abi.encode(tokenManager, distributor, name, symbol, decimals, mintAmount, mintTo);

    63: bytecode = abi.encodePacked(type(StandardizedTokenProxy).creationCode, abi.encode(implementationAddress, params));

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol

    File: TokenManagerDeployer.sol	

    38: bytes memory args = abi.encode(address(this), implementationType, tokenId, params);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-03] Use *assembly* to write *address* storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

There are 14 instances of this issue in 8 files:

    File: AxelarGateway.sol	

    68: AUTH_MODULE = authModule_;

    69: TOKEN_DEPLOYER_IMPLEMENTATION = tokenDeployerImplementation_;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: Upgradable.sol	

    23: implementationAddress = address(this);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol

    File: FixedProxy.sol	

    24: implementation = implementationAddress;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FixedProxy.sol

    File: InterchainTokenService.sol	

    100: tokenManagerDeployer = tokenManagerDeployer_;

    101: standardizedTokenDeployer = standardizedTokenDeployer_;

    106: implementationLockUnlock = _sanitizeTokenManagerImplementation(tokenManagerImplementations, TokenManagerType.LOCK_UNLOCK);

    107: implementationMintBurn = _sanitizeTokenManagerImplementation(tokenManagerImplementations, TokenManagerType.MINT_BURN);

    108: implementationLiquidityPool = _sanitizeTokenManagerImplementation(tokenManagerImplementations, TokenManagerType.LIQUIDITY_POOL);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: RemoteAddressValidator.sol	

    29: interchainTokenServiceAddress = _interchainTokenServiceAddress;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

    File: StandardizedToken.sol	

    56: tokenManager = tokenManager_;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol

    File: Implementation.sol	

    19: implementationAddress = address(this);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Implementation.sol

    File: StandardizedTokenDeployer.sol	

    34: implementationLockUnlockAddress = implementationLockUnlockAddress_;

    35: implementationMintBurnAddress = implementationMintBurnAddress_;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }

    contract Contract0 {

        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }

    }

    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }

    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |

|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-04] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

There are 5 instances of this issue in 5 files:

    File: InterchainGovernance.sol	

    62: if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol

    File: TimeLock.sol	

    82: if (hash == 0 || eta == 0) revert InvalidTimeLockHash();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/util/TimeLock.sol

    File: AxelarGateway.sol	

    342: if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: InterchainTokenService.sol	

    92: if (
    93:     remoteAddressValidator_ == address(0) ||
    94:     gasService_ == address(0) ||
    95:     tokenManagerDeployer_ == address(0) ||
    96:     standardizedTokenDeployer_ == address(0)
    97: ) revert ZeroAddress();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: StandardizedTokenDeployer.sol	

    31: if (deployer_ == address(0) || implementationLockUnlockAddress_ == address(0) || implementationMintBurnAddress_ == address(0))

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-05] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

There are 2 instances of this issue in 1 file:

    File: FlowLimit.sol	

    64: uint256 epoch = block.timestamp / EPOCH_TIME;

    76: uint256 epoch = block.timestamp / EPOCH_TIME;

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/FlowLimit.sol

#### Test Code 

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }
    
    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |

| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-06] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 14 instances of this issue in 7 files:

    File: MultisigBase.sol	

    70: for (uint256 i; i < count; ++i) {

    122: for (uint256 i; i < length; ++i) {

    153: for (uint256 i; i < length; ++i) {

    168: for (uint256 i; i < length; ++i) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol

    File: AxelarGateway.sol	

    270: for (uint256 i; i < length; ++i) {

    344: for (uint256 i; i < commandsLength; ++i) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: InterchainProposalSender.sol	

    63: for (uint256 i = 0; i < interchainCalls.length; ) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalSender.sol

    File: InterchainProposalExecutor.sol	

    74: for (uint256 i = 0; i < calls.length; i++) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

    File: InterchainTokenService.sol	

    537: for (uint256 i; i < length; ++i) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: RemoteAddressValidator.sol	

    44: for (uint256 i; i < length; ++i) {

    56: for (uint256 i; i < length; i++) {

    108: for (uint256 i; i < length; ++i) {

    121: for (uint256 i; i < length; ++i) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

    File: Multicall.sol	

    24: for (uint256 i = 0; i < data.length; ++i) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Multicall.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-07] Use assembly to check for *address(0)*

Checking zero address can be improved by replacing the require statement with Assembly.Solidity has a lot of guardrails that can be removed (with care) for optimization purposes, especially for simple functionality like checking if an address is zero.

There are 31 instances of this issue in 14 files:

    File: InterchainGovernance.sol	

    100: if (target == address(0)) revert InvalidTarget();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol

    File: MultisigBase.sol	

    173: if (account == address(0)) revert InvalidSigners();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol

    File: AxelarGateway.sol	

    255: if (newGovernance == address(0)) revert InvalidGovernance();

    261: if (newMintLimiter == address(0)) revert InvalidMintLimiter();

    274: if (tokenAddresses(symbol) == address(0)) revert TokenDoesNotExist(symbol);

    309: if (implementation() == address(0)) revert NotProxy();

    313: if (governance_ != address(0)) _transferGovernance(governance_);

    314: if (mintLimiter_ != address(0)) _transferMintLimiter(mintLimiter_);

    392: if (tokenAddresses(symbol) != address(0)) revert TokenAlreadyExists(symbol);

    394: if (tokenAddress == address(0)) {

    432: if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

    519: if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

    537: if (tokenAddress == address(0)) revert TokenDoesNotExist(symbol);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: ConstAddressDeployer.sol	

    88: if (deployedAddress_ == address(0)) revert FailedDeploy();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol

    File: Create3.sol	

    60: if (address(deployer) == address(0)) revert DeployFailed();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/deploy/Create3.sol

    File: Proxy.sol	

    30: if (owner == address(0)) revert InvalidOwner();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Proxy.sol

    File: InitProxy.sol	

    47: if (implementation() != address(0)) revert AlreadyInitialized();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/InitProxy.sol

    File: FinalProxy.sol	

    39: if (implementation_ == address(0)) {

    49: return _finalImplementation() != address(0);

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/FinalProxy.sol

    File: InterchainTokenService.sol	

    92: if (
    93:     remoteAddressValidator_ == address(0) ||
    94:     gasService_ == address(0) ||
    95:     tokenManagerDeployer_ == address(0) ||
    96:     standardizedTokenDeployer_ == address(0)
    97: ) revert ZeroAddress();

    565: if (implementation == address(0)) revert ZeroAddress();

    609: if (expressCaller == address(0)) {

    652: if (expressCaller != address(0)) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: RemoteAddressValidator.sol	

    28: if (_interchainTokenServiceAddress == address(0)) revert ZeroAddress();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

    File: TokenManager.sol	

    28: if (interchainTokenService_ == address(0)) revert TokenLinkerZeroAddress();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol

    File: ExpressCallHandler.sol	

    91: if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

    133: if (prevExpressCaller != address(0)) revert AlreadyExpressCalled();

    212: if (expressCaller != address(0)) {

    252: if (expressCaller != address(0)) {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/ExpressCallHandler.sol

    File: StandardizedTokenDeployer.sol	

    31: if (deployer_ == address(0) || implementationLockUnlockAddress_ == address(0) || implementationMintBurnAddress_ == address(0))

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/StandardizedTokenDeployer.sol

    File: TokenManagerDeployer.sol	

    23: if (deployer_ == address(0)) revert AddressZero();

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/TokenManagerDeployer.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.solidity_notZero(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.assembly_notZero(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }


    contract Contract0 {

        error ZeroAddress();

        function solidity_notZero(address toCheck) public pure returns(bool success) {
            if(toCheck == address(0)) revert ZeroAddress();

            return true;
        }
    }

    contract Contract1{

    error ZeroAddress();
    function assembly_notZero(address toCheck) public pure returns(bool success) {
        assembly {
            if iszero(toCheck) {
                let ptr := mload(0x40)
                mstore(ptr, 0xd92e233d00000000000000000000000000000000000000000000000000000000) // selector for `ZeroAddress()`
                revert(ptr, 0x4)
            }
        }
        return true;
    }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 55905                                     | 311             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| solidity_notZero                          | 323             | 323 | 323    | 323 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 50099                                     | 281             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| assembly_notZero                          | 317             | 317 | 317    | 317 | 1       |

## [G-08] Functions guaranteed to revert when called by normal users can be marked *payable*

If a function modifier such as *onlyOwner* is used, the function will revert if a normal user tries to pay the function. Marking the function as *payable* will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are *CALLVALUE*(2),*DUP1*(3),*ISZERO*(3),*PUSH2*(3),*JUMPI*(10),*PUSH1*(3),*DUP1*(3),*REVERT*(0),*JUMPDEST*(1),*POP*(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are 32 instances of this issue in 11 files:

    File: MultisigBase.sol	

    142: function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/auth/MultisigBase.sol

    File: AxelarGateway.sol	

    254: function transferGovernance(address newGovernance) external override onlyGovernance {

    260: function transferMintLimiter(address newMintLimiter) external override onlyMintLimiter {

    266: function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {

    280: function upgrade(
    281:     address newImplementation,
    282:     bytes32 newImplementationCodeHash,
    283:     bytes calldata setupParams
    284: ) external override onlyGovernance {

    385: function deployToken(bytes calldata params, bytes32) external onlySelf {

    421: function mintToken(bytes calldata params, bytes32) external onlySelf {

    427: function burnToken(bytes calldata params, bytes32) external onlySelf {

    455: function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf {

    469: function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf {

    495: function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: InterchainProposalExecutor.sol	

    92: function setWhitelistedProposalCaller(
    93:     string calldata sourceChain,
    94:     address sourceCaller,
    95:     bool whitelisted
    96: ) external override onlyOwner {

    107: function setWhitelistedProposalSender(
    108:     string calldata sourceChain,
    109:     address sourceSender,
    110:     bool whitelisted
    111: ) external override onlyOwner {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

    File: Upgradable.sol	

    52: function upgrade(
    53:     address newImplementation,
    54:     bytes32 newImplementationCodeHash,
    55:     bytes calldata params
    56: ) external override onlyOwner {

    78: function setup(bytes calldata data) external override onlyProxy {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/gmp-sdk/upgradable/Upgradable.sol

    File: InterchainTokenService.sol	

    502: function transmitSendToken(
    503:     bytes32 tokenId,
    504:     address sourceAddress,
    505:     string calldata destinationChain,
    506:     bytes memory destinationAddress,
    507:     uint256 amount,
    508:     bytes calldata metadata
    509: ) external payable onlyTokenManager(tokenId) notPaused {

    534: function setFlowLimit(bytes32[] calldata tokenIds, uint256[] calldata flowLimits) external onlyOperator {

    547: function setPaused(bool paused) external onlyOwner {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol

    File: RemoteAddressValidator.sol	

    83: function addTrustedAddress(string memory chain, string memory addr) public onlyOwner {

    95: function removeTrustedAddress(string calldata chain) external onlyOwner {

    106: function addGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

    119: function removeGatewaySupportedChains(string[] calldata chainNames) external onlyOwner {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol

    File: StandardizedToken.sol	

    49: function setup(bytes calldata params) external override onlyProxy {

    76: function mint(address account, uint256 amount) external onlyDistributor {

    86: function burn(address account, uint256 amount) external onlyDistributor {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-implementations/StandardizedToken.sol

    File: TokenManager.sol	

    61: function setup(bytes calldata params) external override onlyProxy {

    136: function transmitInterchainTransfer(
    137:     address sender,
    138:     string calldata destinationChain,
    139:     bytes calldata destinationAddress,
    140:     uint256 amount,
    141:     bytes calldata metadata
    142: ) external payable virtual onlyToken {

    161: function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256) {

    171: function setFlowLimit(uint256 flowLimit) external onlyOperator {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/TokenManager.sol

    File: TokenManagerLiquidityPool.sol	

    67: function setLiquidityPool(address newLiquidityPool) external onlyOperator {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/token-manager/implementations/TokenManagerLiquidityPool.sol

    File: Distributable.sol	

    51: function setDistributor(address distr) external onlyDistributor {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Distributable.sol

    File: Operatable.sol	

    51: function setOperator(address operator_) external onlyOperator {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/utils/Operatable.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
        address owner = msg.sender;
        uint256 num;
    
        modifier only_owner{
             require(msg.sender==owner);
             _;
        }
    
        function not_optimized() public only_owner {
             num = 1;
        }
    }
    
    contract Contract1 {
    
        address owner = msg.sender;
        uint256 num;
    
        modifier only_owner{
             require(msg.sender==owner);
             _;
        }
    
        function optimized() public only_owner payable {
             num = 1;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 51816                                     | 196             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24356           | 24356 | 24356  | 24356 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 49416                                     | 184             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24332           | 24332 | 24332  | 24332 | 1       |

## [G-09] *internal* functions only called once can be inlined to save gas

If your internal functions are called only once, it is better to inline them to save gas. Failing to do this costs about 20 to 40 more gas.

There are 20 instances of this issue in 5 files:

    File: InterchainGovernance.sol	

    113: function _processCommand(
    114:     uint256 commandId,
    115:     address target,
    116:     bytes memory callData,
    117:     uint256 nativeValue,
    118:     uint256 eta
    119: ) internal virtual {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/InterchainGovernance.sol

   File: AxelarServiceGovernance.sol	

    72: function _processCommand(
    73:     uint256 commandId,
    74:     address target,
    75:     bytes memory callData,
    76:     uint256 nativeValue,
    77:     uint256 eta
    78: ) internal override {
    
https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/governance/AxelarServiceGovernance.sol

    File: AxelarGateway.sol	

    505: function _hasCode(address addr) internal view returns (bool) {

    615: function _getCreate2Address(bytes32 salt, bytes32 codeHash) internal view returns (address) {

    633: function _setTokenMintAmount(string memory symbol, uint256 amount) internal {

    652: function _setContractCallApproved(
    653:     bytes32 commandId,
    654:     string memory sourceChain,
    655:     string memory sourceAddress,
    656:     address contractAddress,
    657:     bytes32 payloadHash
    658: ) internal {

    662: function _setContractCallApprovedWithMint(
    663:     bytes32 commandId,
    664:     string memory sourceChain,
    665:     string memory sourceAddress,
    666:     address contractAddress,
    667:     bytes32 payloadHash,
    668:     string memory symbol,
    669:     uint256 amount
    670: ) internal {

    677: function _setImplementation(address newImplementation) internal {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/cgp/AxelarGateway.sol

    File: InterchainProposalExecutor.sol	

    73: function _executeProposal(InterchainCalls.Call[] memory calls) internal {

    126: function _beforeProposalExecuted(
    127:     string calldata sourceChain,
    128:     string calldata sourceAddress,
    129:     bytes calldata payload
    130: ) internal virtual {

    142: function _onProposalExecuted(
    143:     string calldata, /* sourceChain */
    144:     string calldata, /* sourceAddress */
    145:     address, /* caller */
    146:     bytes calldata payload
    147: ) internal virtual {

    156: function _onTargetExecutionFailed(
    157:     InterchainCalls.Call memory, /* call */
    158:     bytes memory result
    159: ) internal virtual {

    178: function _onTargetExecuted(InterchainCalls.Call memory call, bytes memory result) internal virtual {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol

    File: InterchainTokenService.sol	

    599: function _processSendTokenPayload(string calldata sourceChain, bytes calldata payload) internal {

    622: function _processSendTokenWithDataPayload(string calldata sourceChain, bytes calldata payload) internal {

    666: function _processDeployTokenManagerPayload(bytes calldata payload) internal {

    678: function _processDeployStandardizedTokenAndManagerPayload(bytes calldata payload) internal {

    748: function _deployRemoteTokenManager(
    749:     bytes32 tokenId,
    750:     string calldata destinationChain,
    751:     uint256 gasValue,
    752:     TokenManagerType tokenManagerType,
    753:     bytes memory params
    754: ) internal {

    872: function _decodeMetadata(bytes calldata metadata) internal pure returns (uint32 version, bytes calldata data) {

    880: function _expressExecuteWithInterchainTokenToken(
    881:     bytes32 tokenId,
    882:     address destinationAddress,
    883:     string memory sourceChain,
    884:     bytes memory sourceAddress,
    885:     bytes calldata data,
    886:     uint256 amount
    887: ) internal {

https://github.com/code-423n4/2023-07-axelar/tree/main/contracts/its/interchain-token-service/InterchainTokenService.sol