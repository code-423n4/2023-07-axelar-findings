<h1>Use calldata instead of memory for function arguments that do not get mutated</h1>
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. Using memory costs more gas than calldata here.

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L512C2-L516C17

 function _mintToken(
        string memory symbol,

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L532C8-L532C30

 function _burnTokenFrom(
        address sender,
        string memory symbol,
        uint256 amount
    ) internal {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L556C5-L556C91

function _getTokenMintLimitKey(string memory symbol) internal pure returns (bytes32) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L560C5-L560C105

function _getTokenMintAmountKey(string memory symbol, uint256 day) internal pure returns (bytes32) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L565

    function _getTokenTypeKey(string memory symbol) internal pure returns (bytes32) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L569

    function _getTokenAddressKey(string memory symbol) internal pure returns (bytes32) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L579C6-L580C37

 function _getIsContractCallApprovedKey(
        bytes32 commandId,
        string memory sourceChain,
        string memory sourceAddress,
        address contractAddress,
        bytes32 payloadHash
    ) internal pure returns (

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L587C3-L593C30

  function _getIsContractCallApprovedWithMintKey(
        bytes32 commandId,
        string memory sourceChain,
        string memory sourceAddress,
        address contractAddress,
        bytes32 payloadHash,
        string memory symbol,
        uint256 amount
    ) internal pure returns (bytes32) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L619C2-L619C85

   function _getTokenType(string memory symbol) internal view returns (TokenType) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L627C4-L627C80

  function _setTokenMintLimit(string memory symbol, uint256 limit) internal {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L633C4-L633C82

  function _setTokenMintAmount(string memory symbol, uint256 amount) internal {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L640C3-L640C81

  function _setTokenType(string memory symbol, TokenType tokenType) internal {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L644

    function _setTokenAddress(string memory symbol, address tokenAddress) internal {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L654C8-L655C37

   function _setContractCallApproved(
        bytes32 commandId,
        string memory sourceChain,
        string memory sourceAddress,
        address contractAddress,
        bytes32 payloadHash
    ) internal {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L662C3-L668C30

    function _setContractCallApprovedWithMint(
        bytes32 commandId,
        string memory sourceChain,
        string memory sourceAddress,
        address contractAddress,
        bytes32 payloadHash,
        string memory symbol,
        uint256 amount
    ) internal {
        _setBool(
            _getIsContractCallApprovedWithMintKey(commandId, sourceChain, sourceAddress, contractAddress, payloadHash, symbol, amount),
            true
        );
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalSender.sol#L80C1-L87C1

     */
    function sendProposal(
        string memory destinationChain,
        string memory destinationContract,
        InterchainCalls.Call[] calldata calls
    ) external payable override {
        _sendProposal(InterchainCalls.InterchainCall(destinationChain, destinationContract, msg.value, calls));
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/ConstAddressDeployer.sol#L24C3-L26C6

   function deploy(bytes memory bytecode, bytes32 salt) external returns (address deployedAddress_) {
        deployedAddress_ = _deploy(bytecode, keccak256(abi.encode(msg.sender, salt)));
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3Deployer.sol#L50C3-L50C31

  function deployAndInit(
        bytes memory bytecode,
        bytes32 salt,
        bytes calldata init
    ) external returns (address deployedAddress_) {
        bytes32 deploySalt = keccak256(abi.encode(msg.sender, salt));
        deployedAddress_ = Create3.deploy(deploySalt, bytecode);

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/deploy/Create3.sol#L51C4-L63C6

 function deploy(bytes32 salt, bytes memory bytecode) internal returns (address deployed) {
        deployed = deployedAddress(address(this), salt);

        if (deployed.isContract()) revert AlreadyDeployed();
        if (bytecode.length == 0) revert EmptyBytecode();

        // Deploy using create2
        CreateDeployer deployer = new CreateDeployer{ salt: salt }();

        if (address(deployer) == address(0)) revert DeployFailed();

        deployer.deploy(bytecode);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/InitProxy.sol#L38C8-L38C28

 function init(
        address implementationAddress,
        address newOwner,
        bytes memory params
    ) external {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L241C4-L241C122

   function getParamsLockUnlock(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
        params = abi.encode(operator, tokenAddress);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L251C4-L252C53

   function getParamsMintBurn(bytes memory operator, address tokenAddress) public pure returns (bytes memory params) {
        params = abi.encode(operator, tokenAddress);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L263C3-L263C31

    function getParamsLiquidityPool(
        bytes memory operator,
        address tokenAddress,
        address liquidityPoolAddress
    ) public pure returns (bytes memory params) {
        params = abi.encode(operator, tokenAddress, liquidityPoolAddress);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L346C5-L346C28

    function deployCustomTokenManager(
        bytes32 salt,
        TokenManagerType tokenManagerType,
        bytes memory params
    ) public payable notPaused returns (bytes32 tokenId) {
        address deployer_ = msg.sender;
        tokenId = getCustomTokenId(deployer_, salt);
        _deployTokenManager(tokenId, tokenManagerType, params);
        emit CustomTokenIdClaimed(tokenId, deployer_, salt);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L422C6-L423C31

   function deployAndRegisterRemoteStandardizedToken(
        bytes32 salt,
        string calldata name,
        string calldata symbol,
        uint8 decimals,
        bytes memory distributor,
        bytes memory operator,
        string calldata destinationChain,
        uint256 gasValue
    ) external payable notPaused {
        bytes32 tokenId = getCustomTokenId(msg.sender, salt);
        _deployRemoteStandardizedToken(tokenId, name, symbol, decimals, distributor, operator, destinationChain, gasValue);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L469C8-L470C36

 function expressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId
    ) external {
        if (gateway.isCommandExecuted(commandId)) revert AlreadyExecuted(commandId);

        address caller = msg.sender;
        ITokenManager tokenManager = ITokenManager(getValidTokenManagerAddress(tokenId));
        IERC20 token = IERC20(tokenManager.tokenAddress());

        SafeTokenTransferFrom.safeTransferFrom(token, caller, destinationAddress, amount);

        _expressExecuteWithInterchainTokenToken(tokenId, destinationAddress, sourceChain, sourceAddress, data, amount);

        _setExpressReceiveTokenWithData(tokenId, sourceChain, sourceAddress, destinationAddress, amount, data, commandId, caller);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/ExpressCallHandler.sol#L48C3-L54C45

  function _getExpressReceiveTokenWithDataSlot(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes memory data,
        bytes32 commandId
    ) internal pure returns (uint256 slot) {
        slot = uint256(
            keccak256(
                abi.encode(
                    PREFIX_EXPRESS_RECEIVE_TOKEN_WITH_DATA,
                    tokenId,
                    sourceChain,
                    sourceAddress,
                    destinationAddress,
                    amount,
                    data,
                    commandId
                )
            )
        );
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/ExpressCallHandler.sol#L112C7-L113C36

 function _setExpressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId,
        address expressCaller
    ) internal {
        uint256 slot = _getExpressReceiveTokenWithDataSlot(
            tokenId,
            sourceChain,
            sourceAddress,
            destinationAddress,
            amount,
            data,
            commandId
        );
    
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/ExpressCallHandler.sol#L173C8-L174C36

function getExpressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes calldata data,
        bytes32 commandId
    ) public view returns (address expressCaller) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/ExpressCallHandler.sol#L231C4-L239C49

    function _popExpressReceiveTokenWithData(
        bytes32 tokenId,
        string memory sourceChain,
        bytes memory sourceAddress,
        address destinationAddress,
        uint256 amount,
        bytes memory data,
        bytes32 commandId
    ) internal returns (address expressCaller) {

<h1>Functions guaranteed to revert when called by normal users can be marked payable</h1>

If a function modifier or require such as onlyOwner or onlySigners is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L254C4-L254C90

  function transferGovernance(address newGovernance) external override onlyGovernance {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L260C3-L260C93

  function transferMintLimiter(address newMintLimiter) external override onlyMintLimiter {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L266C2-L266C122

    function setTokenMintLimits(string[] calldata symbols, uint256[] calldata limits) external override onlyMintLimiter {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L280C3-L284C41

    function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata setupParams
    ) external override onlyGovernance {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L385C3-L385C77

 function deployToken(bytes calldata params, bytes32) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L421C5-L421C75

 function mintToken(bytes calldata params, bytes32) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L427C3-L427C75

  function burnToken(bytes calldata params, bytes32) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L455C3-L455C95

  function approveContractCall(bytes calldata params, bytes32 commandId) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L469C4-L469C103

 function approveContractCallWithMint(bytes calldata params, bytes32 commandId) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/AxelarGateway.sol#L495

 function transferOperatorship(bytes calldata newOperatorsData, bytes32) external onlySelf {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L142C3-L142C110

  function rotateSigners(address[] memory newAccounts, uint256 newThreshold) external virtual onlySigners {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/TokenManager.sol#L61C4-L61C72

    function setup(bytes calldata params) external override onlyProxy {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/TokenManager.sol#L161C4-L161C108

 function giveToken(address destinationAddress, uint256 amount) external onlyService returns (uint256) {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/token-manager/TokenManager.sol#L171C2-L173C6

    function setFlowLimit(uint256 flowLimit) external onlyOperator {
        _setFlowLimit(flowLimit);
    }


https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L92C4-L96C36

    function setWhitelistedProposalCaller(
        string calldata sourceChain,
        address sourceCaller,
        bool whitelisted
    ) external override onlyOwner {

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L107C4-L114C6

    function setWhitelistedProposalSender(
        string calldata sourceChain,
        address sourceSender,
        bool whitelisted
    ) external override onlyOwner {
        whitelistedSenders[sourceChain][sourceSender] = whitelisted;
        emit WhitelistedProposalSenderSet(sourceChain, sourceSender, whitelisted);
    }

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/upgradable/Upgradable.sol#L52C4-L56C36

  function upgrade(
        address newImplementation,
        bytes32 newImplementationCodeHash,
        bytes calldata params
    ) external override onlyOwner {

<h1> Multiple address mappings can be combined into a single mapping of an address to a struct</h1>

You can combine multiple mappings into structs. This will result in cheaper storage reads since multiple mappings are accessed in functions and those values are now occupying the same storage slot, meaning the slot will become warm after the first SLOAD. In addition, when writing to and reading from the struct values we will avoid a Gsset (20000 gas) and Gcoldsload (2100 gas) since multiple struct values are now occupying the same slot.

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/interchain-governance-executor/InterchainProposalExecutor.sol#L24C3-L27C75

   mapping(string => mapping(address => bool)) public whitelistedCallers;

    // Whitelisted proposal senders. The proposal sender is the `InterchainProposalSender` contract address at the source chain.
    mapping(string => mapping(address => bool)) public whitelistedSenders;

You can do this instead:

struct Whitelists {
    bool whitelistedCallers;
    bool whitelistedSenders;
}

 mapping(uint256 => mapping(address => Whitelists)) public whitelists;


