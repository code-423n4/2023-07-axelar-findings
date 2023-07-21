## Gas optimization 

No | Issue |Instances|
|-|:-|:-:|:-:|
|G-1|Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate|2
|G-2|Use calldata instead of memory for function arguments that do not get mutated|39
|G-3| Use do while loops instead of for loops|15
|G-4|If statements that use && can be refactored into nested if statements|4
|G-5|Use nested if and, avoid multiple check combinations|5
|G-6|Use custom errors rather than revert()/require() strings to save gas|13
|G-7|Use constants instead of type(uintx).max|2



### [G-1] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

```solidity

file: contracts/cgp/auth/MultisigBase.sol

27     mapping(uint256 => mapping(bytes32 => Voting)) public votingPerTopic;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L27

```solidity

file: contracts/interchain-governance-executor/InterchainProposalExecutor.sol

24     mapping(string => mapping(address => bool)) public whitelistedCallers;

    // Whitelisted proposal senders. The proposal sender is the `InterchainProposalSender` contract address at the source chain.
27    mapping(string => mapping(address => bool)) public whitelistedSenders;

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24-L27

### [G-2] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity

file: contracts/cgp/auth/MultisigBase.sol

95     function signerAccounts() external view override returns (address[] memory) {
        return signers.accounts;
    }

142  function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {
        _rotateSigners(newAccounts, newThreshold);
    }

    /**
     * @dev Internal function that implements signer rotation logic
     */
    function _rotateSigners(address[] memory newAccounts, uint256 newThreshold) internal {
150        uint256 length = signers.accounts.length;
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L95
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142-L50

```solidity

file: contracts/cgp/interfaces/IInterchainGovernance.sol

26  function governanceChain() external view returns (string memory);

    /**
     * @notice Returns the address of the governance address.
     * @return string The address of the governance address
     */
32    function governanceAddress() external view returns (string memory);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IInterchainGovernance.sol#L26-32

```solidity

file: contracts/cgp/governance/InterchainGovernance.sol

113   function _processCommand(
        uint256 commandId,
        address target,
        bytes memory callData,
        uint256 nativeValue,
        uint256 eta
    ) internal virtual

143     function _getProposalHash(
        address target,
        bytes memory callData,
        uint256 nativeValue
    ) internal pure returns (bytes32)
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L113
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L143

```solidity

file: 

72    function _processCommand(
        uint256 commandId,
        address target,
        bytes memory callData,
        uint256 nativeValue,
        uint256 eta
    ) internal override 


```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L72

```solidity

file: contracts/cgp/interfaces/IMultisigBase.sol

44    function signerAccounts() external view returns (address[] memory);

73     function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external;
}

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol#L44
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/interfaces/IMultisigBase.sol#L73

```solidity

file: contracts/gmp-sdk/upgradable/InitProxy.sol

35    function init(
        address implementationAddress,
        address newOwner,
        bytes memory params
    )


```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/upgradable/InitProxy.sol#L35

```solidity

file: contracts/gmp-sdk/interfaces/IInitProxy.sol

9    function init(
        address implementationAddress,
        address newOwner,
        bytes memory params
    ) 

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IInitProxy.sol#L9

```solidity

file: contracts/gmp-sdk/interfaces/IFinalProxy.sol

11     function finalUpgrade(bytes memory bytecode, bytes calldata setupParams) external returns (address);

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/interfaces/IFinalProxy.sol#L11

```solidity

file: contracts/its/interchain-token-service/InterchainTokenService.sol

152     function getChainName() public view returns (string memory name) {
        name = chainName.toTrimmedString();
    }

241  function getParamsLockUnlock(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
        params = abi.encode(operator, tokenAddress);
    }

    /**
     * @notice Getter function for the parameters of a mint/burn TokenManager. Mainly to be used by frontends.
     * @param operator the operator of the TokenManager.
     * @param tokenAddress the token to be managed.
     * @return params the resulting params to be passed to custom TokenManager deployments.
     */
    function getParamsMintBurn(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
        params = abi.encode(operator, tokenAddress);
253    }

262   function getParamsLiquidityPool(
        bytes memory operator,
        address tokenAddress,
        address liquidityPoolAddress
    ) 
309   function registerCanonicalToken(address tokenAddress) external payable notPaused returns (bytes32 tokenId) {
        (, string memory tokenSymbol, ) = _validateToken(tokenAddress);    

343   function deployCustomTokenManager(
        bytes32 salt,
        TokenManagerType tokenManagerType,
        bytes memory params
    )    

417   function deployAndRegisterRemoteStandardizedToken(
        bytes32 salt,
        string calldata name,
        string calldata symbol,
        uint8 decimals,
        bytes memory distributor,
        bytes memory operator,
        string calldata destinationChain,
        uint256 gasValue
    ) 

467    function expressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId
    ) 

502    function transmitSendToken(
        bytes32 tokenId,
        address sourceAddress,
        string calldata destinationChain,
        bytes memory destinationAddress,
        uint256 amount,
        bytes calldata metadata
    )    

559 
    function _sanitizeTokenManagerImplementation(address[] memory implementaions, TokenManagerType tokenManagerType)
        internal
        pure
        returns (address implementation)  


599   function _processSendTokenPayload(string calldata sourceChain, bytes calldata payload) internal {
        (, bytes32 tokenId, bytes memory destinationAddressBytes, uint256 amount) = abi.decode(payload, (uint256, bytes32, bytes, uint256));
        bytes32 commandId;

622  function _processSendTokenWithDataPayload(string calldata sourceChain, bytes calldata payload) internal {
        bytes32 tokenId;
        uint256 amount;
        bytes memory sourceAddress;
        bytes memory data;
        address destinationAddress;
        bytes32 commandId;   

666    function _processDeployTokenManagerPayload(bytes calldata payload) internal {
        (, bytes32 tokenId, TokenManagerType tokenManagerType, bytes memory params) = abi.decode(
            payload,
            (uint256, bytes32, TokenManagerType, bytes)   

707   function _callContract(
        string calldata destinationChain,
        bytes memory payload,
        uint256 gasValue,
        address refundTo
    )    


748      function _deployRemoteTokenManager(
        bytes32 tokenId,
        string calldata destinationChain,
        uint256 gasValue,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) 

 770   function _deployRemoteStandardizedToken(
        bytes32 tokenId,
        string memory name,
        string memory symbol,
        uint8 decimals,
        bytes memory distributor,
        bytes memory operator,
        string calldata destinationChain,
        uint256 gasValue
    ) 


808     function _deployTokenManager(
        bytes32 tokenId,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) 

841      function _deployStandardizedToken(
        bytes32 tokenId,
        address distributor,
        string memory name,
        string memory symbol,
        uint8 decimals,
        uint256 mintAmount,
        address mintTo
    ) 

880 
    function _expressExecuteWithInterchainTokenToken(
        bytes32 tokenId,
        address destinationAddress,
        string memory sourceChain,
        bytes memory sourceAddress,
        bytes calldata data,
        uint256 amount
    ) 

```

https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L152
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L241-L253
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L262
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L309
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L343
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L417
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L467
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L502
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L559
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L599
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L622
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L666
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L707
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L748
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L770
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L808
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L841
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L880


```solidity

file: contracts/its/libraries/AddressBytesUtils.sol

17     function toAddress(bytes memory bytesAddress) internal pure returns (address addr) {
        if (bytesAddress.length != 20) revert InvalidBytesLength(bytesAddress);

30  function toBytes(address addr) internal pure returns (bytes memory bytesAddress) {
        bytesAddress = new bytes(20);        
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/libraries/AddressBytesUtils.sol#L17
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/libraries/AddressBytesUtils.sol#L30

```solidity

file: contracts/its/interfaces/IInterchainTokenService.sol

94    function getChainName() external view returns (string memory name);

145   function getParamsLockUnlock(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

153   function getParamsMintBurn(bytes memory operator, address tokenAddress) external pure returns (bytes memory params);

162     function getParamsLiquidityPool(
        bytes memory operator,
        address tokenAddress,
        address liquidityPoolAddress
    ) 
194    function deployCustomTokenManager(
        bytes32 salt,
        TokenManagerType tokenManagerType,
        bytes memory params
    )

245   function deployAndRegisterRemoteStandardizedToken(
        bytes32 salt,
        string calldata name,
        string calldata symbol,
        uint8 decimals,
        bytes memory distributor,
        bytes memory operator,
        string calldata destinationChain,
        uint256 gasValue
    )

271    function transmitSendToken(
        bytes32 tokenId,
        address sourceAddress,
        string calldata destinationChain,
        bytes memory destinationAddress,
        uint256 amount,
        bytes calldata metadata
    ) 

399    function expressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId
    ) 
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L94
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L153
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L162
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L194
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L245
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L271
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interfaces/IInterchainTokenService.sol#L339

### [G-3] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file: contracts/cgp/auth/MultisigBase.sol

70      for (uint256 i; i < count; ++i) {
            voting.hasVoted[signers.accounts[i]] = false;
        }

122     for (uint256 i; i < length; ++i) {
            if (votingPerTopic[signerEpoch][topic].hasVoted[signers.accounts[i]]) {
                voteCount++;
            }
        }
153    for (uint256 i; i < length; ++i) {
            signers.isSigner[signers.accounts[i]] = false;
        }  

168  for (uint256 i; i < length; ++i)               
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L70
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L122
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L153
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L168

```solidity

file: contracts/cgp/AxelarGateway.sol

270  for (uint256 i; i < length; ++i) {
            string memory symbol = symbols[i];
            uint256 limit = limits[i];

344  for (uint256 i; i < commandsLength; ++i) {
            bytes32 commandId = commandIds[i];            
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L270
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L344

```solidity

file: contracts/interchain-governance-executor/InterchainProposalSender.sol

63 for (uint256 i = 0; i < interchainCalls.length; )


106    for (uint256 i = 0; i < interchainCalls.length; )

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L63
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalSender.sol#L106

```solidity

file: contracts/interchain-governance-executor/InterchainProposalExecutor.sol

74     for (uint256 i = 0; i < calls.length; i++


```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L74

```solidity

file: contracts/its/interchain-token-service/InterchainTokenService.sol

537     for (uint256 i; i < length; ++i) 
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L537

```solidity

file: contracts/its/remote-address-validator/RemoteAddressValidator.sol

44  for (uint256 i; i < length; ++i)

56   for (uint256 i; i < length; i++) 

108  for (uint256 i; i < length; ++i)

121  for (uint256 i; i < length; ++i) 
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L44
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L56
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L108
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L121

### [G-4] If statements that use && can be refactored into nested if statements

```solidity

file: contracts/cgp/AxelarGateway.sol

88    if (msg.sender != getAddress(KEY_MINT_LIMITER) && msg.sender != getAddress(KEY_GOVERNANCE)) revert NotMintLimiter();

466       if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);

635    if (limit > 0 && amount > limit) revert ExceedMintLimit(symbol);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L88
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L446
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L635

```solidity

file: contracts/its/remote-address-validator/RemoteAddressValidator.sol

58   if ((b >= 65) && (b <= 70)) bytes(s)[i] = bytes1(b + uint8(32));

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/remote-address-validator/RemoteAddressValidator.sol#L58


### [G-5] Use nested if and, avoid multiple check combinations
Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity

file: contracts/cgp/governance/InterchainGovernance.sol

92      if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)


```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L92

```solidity

file: contracts/gmp-sdk/util/TimeLock.sol

82     if (hash == 0 || eta == 0) revert InvalidTimeLockHash();
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/util/TimeLock.sol#L82

```solidity

file: contracts/cgp/AxelarGateway.sol

342      if (commandsLength != commands.length || commandsLength != params.length) revert InvalidCommands();

466     if (!success || (returnData.length != uint256(0) && !abi.decode(returnData, (bool)))) revert BurnFailed(symbol);
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L342
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L446

```solidity

file: contracts/its/interchain-token-service/InterchainTokenService.sol

92       if (
            remoteAddressValidator_ == address(0) ||
            gasService_ == address(0) ||
            tokenManagerDeployer_ == address(0) ||
            standardizedTokenDeployer_ == address(0)
        ) 

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/its/interchain-token-service/InterchainTokenService.sol#L92


### [G-6] Use custom errors rather than revert()/require() strings to save gas

```solidity

file: contracts/cgp/auth/MultisigBase.sol

45     if (!signers.isSigner[msg.sender]) revert NotSigner();

51  if (voting.hasVoted[msg.sender]) revert AlreadyVoted();

159 
        if (newThreshold > length) revert InvalidSigners();

160        if (newThreshold == 0) revert InvalidSignerThreshold();

172  if (signers.isSigner[account]) revert DuplicateSigner(account);
            if (account == address(0)) revert InvalidSigners();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L45
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L51
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L159-L161
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L172-L173

```solidity

file: contracts/cgp/governance/InterchainGovernance.sol

92    if (keccak256(bytes(sourceChain)) != governanceChainHash || keccak256(bytes(sourceAddress)) != governanceAddressHash)
            revert NotGovernance();

100   if (target == address(0)) revert InvalidTarget();

120  if (commandId > uint256(type(GovernanceCommand).max)) {
            revert InvalidCommand();

162           revert TokenNotSupported();        
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L92
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L100
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L120
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L162

```solidity

file: contracts/interchain-governance-executor/InterchainProposalExecutor.sol

49       revert NotWhitelistedSourceAddress();
        }

57  if (!whitelistedCallers[sourceChain][interchainProposalCaller]) {
            revert NotWhitelistedCaller();

163           assembly {
                revert(add(32, result), mload(result))
            }
        } else {
            // There is no failure data, just revert with no reason.
            revert ProposalExecuteFailed();
        }
170    }
```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L49
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L57
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L163-L170

```solidity

file: contracts/gmp-sdk/deploy/Create3Deployer.sol

58   if (!success) revert FailedInit();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3Deployer.sol#L58

```solidity

file: contracts/gmp-sdk/deploy/Create3.sol#L54

54 
        if (deployed.isContract()) revert AlreadyDeployed();
        if (bytecode.length == 0) revert EmptyBytecode();

        // Deploy using create2
        CreateDeployer deployer = new CreateDeployer{ salt: salt }();

60        if (address(deployer) == address(0)) revert DeployFailed();

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/gmp-sdk/deploy/Create3.sol#L54-L60


### [G-7] Use constants instead of type(uintx).max

```solidity

file: contracts/cgp/governance/InterchainGovernance.sol

120   if (commandId > uint256(type(GovernanceCommand).max)) 

```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/InterchainGovernance.sol#L120

```solidity

file: contracts/cgp/governance/AxelarServiceGovernance.sol#L79

79    if (commandId > uint256(type(ServiceGovernanceCommand).max)) 


```
https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/governance/AxelarServiceGovernance.sol#L79

