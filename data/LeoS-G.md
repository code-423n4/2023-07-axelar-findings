# Summary
|        | Issue | Instances | Gas Saved |
|--------|-------|-----------|-----------|
|[G-01]|Use `calldata` instead of `memory`|1|-566|
|[G-02]|Change the optimizer runs number|-|-44 892|


The gas saved column simply adds up the evolution in the gas reports, excluding deployments and using the method described in the next section.
# Benchmark
A benchmark is performed on each optimization, using the gas report provided by hardhat. To provide the best possible quality, only optimizations that significantly improve this report are presented. This report is based on tests and therefore does not take into account all functions, this loss is accepted. But it is also subject to intrinsic variance. This means that some functions or deployment costs that are not relevant to the comparison because they vary too much are not taken into account in the presented gas report. These are listed below.
`AxelarExpressExecutableTest: execute`, `AxelarGateway: execute`, `AxelarGatewaInterchainTokenTesty: setTokenMintLimits`, `AxelarGateway: validateContractCall`, `AxelarGateway: validateContractCallAndMint`, `AxelarValuedExpressExecutableTest: execute`, `AxelarValuedExpressExecutableTest: executeWithToken`, `AxelarValuedExpressExecutableTest: expressExecute`, `ExpressCallHandlerTest: popExpressReceiveToken`, `ExpressCallHandlerTest: popExpressReceiveTokenWithData`, `ExpressCallHandlerTest: setExpressReceiveToken`, `ExpressCallHandlerTest: setExpressReceiveTokenWithData`, `InterchainProposalSender: sendProposal`, `InterchainProposalSender: sendProposals`, `InterchainTokenService: deployAndRegisterStandardizedToken`, `InterchainTokenService: deployCustomTokenManager`, `InterchainTokenService: deployRemoteCustomTokenManager`, `InterchainTokenService: expressReceiveToken`, `InterchainTokenService: expressReceiveTokenWithData`, `InterchainTokenService: multicall`, `InterchainTokenTest: approve`, `InterchainTokenTest: setDistributor`, `MockAxelarGateway: approveContractCall`, `MockGateway: approveContractCall`, `MockGateway: approveContractCallWithMint`, `AxelarExpressExecutableTest: executeWithToken`, `AxelarServiceGovernance: execute`, `BurnableMintableCappedERC20: approve`, `StandardizedTokenDeployer: deployStandardizedToken`, `InterchainTokenTest: transfer`, `BurnableMintableCappedERC20: mint`, `Create3Deployer: deploy`, `ERC20MintableBurnable: burn`, `AxelarExpressExecutableTest: expressExecuteWithToken`, `AxelarValuedExpressExecutableTest: expressExecuteWithToken`, `InterchainTokenService: deployAndRegisterRemoteStandardizedToken`, `BurnableMintableCappedERC20: transfer`,`Deployment: InterchainTokenTest`

# [G-01] Use `calldata` instead of `memory`
Using `calldata` instead of `memory` for function parameters can save gas if the argument is only read in the function
*1 instance*
- [MultisigBase.sol#L142](https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/auth/MultisigBase.sol#L142)

Applying this change, gas report evolves as follows :
||avg before|avg after|difference|
|-|:-:|:-:|:-:|
|DestinationChainSwapExecutable: executeWithToken|216884|216875|-9|
|TestMultiSigBase: rotateSigners|90571|90014|-557|
|TestServiceGovernance: executeProposalAction|47051|47009|-42|
|||||
|Deployment: Multisig|1001863|991471|-10392|
|Deployment: TestMultiSigBase|816076|805683|-10393|
|Deployment: TestServiceGovernance|2177577|2171318|-6259|

Gas saved: -566

# [G-02] Change the optimizer runs number
The number of setup runs in the optmizer mean how many times the contracts are meant to be used. A huge number of runs improves the running cost, but penalizes the deployment cost, and inversely. 
This depends on the deploier's vision and can therefore be widely discussed. It's not 100% relevant to optimize this through gas report, as it doesn't always represent a normal use of contracts. However, it is the only readily available metric and will therefore be used. With this in mind, the purpose of this optimization is simply to show the possible variations and thus highlight the importance of correctly setting up the number of runs.

*Changes are made using hardhat configuration files*

By default, it is set to 1000, and it evolves in the following way:
|Runs:| 1| 10| 100| 500| 2000| 5000| 10000| 20000| 50000| 100000|
|-|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|Usage:| -0.644%| -0.644%| -0.737%| -0.436%| +0.706%| +3.193%| +3.153%| +3.131%| +3.126%| +3.126%|
|Deployment:| -6.511%| -6.511%| -6.025%| -3.814%| +2.542%| +6.400%| +9.907%| +11.412%| +11.863%| +11.863%|

100 runs seems like a good compromise.
This number is only intended to show the potential for better cases.
By setting it to this value, the gas report evolve as follow:
||avg before|avg after|difference|
|-|:-:|:-:|:-:|
|AxelarAuthWeighted: transferOwnership|28675|28734|+59|
|AxelarExpressExecutableTest: expressExecute|60277|60409|+132|
|AxelarGateway: callContract|34582|34729|+147|
|AxelarGateway: callContractWithToken|71439|71651|+212|
|AxelarGateway: sendToken|68389|68611|+222|
|AxelarGateway: setup|46873|46885|+12|
|AxelarGateway: transferGovernance|33756|34215|+459|
|AxelarGateway: transferMintLimiter|34816|35369|+553|
|AxelarGateway: upgrade|66099|66888|+789|
|AxelarGatewayProxy: setup|24479|24523|+44|
|AxelarValuedExpressExecutableTest: setCallValue|43550|43594|+44|
|AxelarValuedExpressExecutableTest: setExpressToken|35836|35828|-8|
|BurnableMintableCappedERC20: transferOwnership|35135|35334|+199|
|Create3Deployer: deployAndInit|824646|807673|-16973|
|DestinationChainSwapExecutable: executeWithToken|216884|217547|+663|
|DestinationChainSwapExpress: executeWithToken|101976|102401|+425|
|DestinationChainSwapExpress: expressExecuteWithToken|218882|219794|+912|
|DifferentProxyImplementation: set|38820|38842|+22|
|DistributableTest: setDistributor|27847|27859|+12|
|DistributableTest: testDistributable|45555|45567|+12|
|ERC20MintableBurnable: approve|38501|38478|-23|
|ERC20MintableBurnable: mint|56989|56999|+10|
|FinalProxy: finalUpgrade|498327|483362|-14965|
|FlowLimitTest: addFlowIn|37128|37117|-11|
|FlowLimitTest: addFlowOut|37171|37227|+56|
|FlowLimitTest: setFlowLimit|56067|56119|+52|
|InterchainTokenService: deployRemoteCanonicalToken|132106|133420|+1314|
|InterchainTokenService: registerCanonicalToken|461666|440493|-21173|
|InterchainTokenService: setPaused|38416|38522|+106|
|InterchainTokenTest: interchainTransfer|173832|175590|+1758|
|InterchainTokenTest: mint|70495|70562|+67|
|InterchainTokenTest: setTokenManager|26814|26805|-9|
|MintableCappedERC20: approve|46241|46218|-23|
|MintableCappedERC20: mint|70627|70715|+88|
|MintableCappedERC20: transfer|50703|50742|+39|
|MockAxelarGateway: setTokenAddress|45264|45265|+1|
|MockGateway: deployToken|357237|351944|-5293|
|MockGateway: mintToken|107848|107943|+95|
|MulticallTest: multicall|56868|56954|+86|
|MulticallTest: multicallTest|239775|239861|+86|
|Multisig: execute|72133|72154|+21|
|OperatorableTest: setOperator|27891|27903|+12|
|OperatorableTest: testOperatorable|45599|45611|+12|
|PausableTest: setPaused|34196|34229|+33|
|PausableTest: testPaused|24118|24151|+33|
|RemoteAddressValidator: addGatewaySupportedChains|53913|53959|+46|
|RemoteAddressValidator: addTrustedAddress|84742|84869|+127|
|RemoteAddressValidator: removeGatewaySupportedChains|32013|32104|+91|
|RemoteAddressValidator: removeTrustedAddress|35145|35218|+73|
|SourceChainSwapCaller: swapToken|107716|107980|+264|
|TestInterchainGovernance: executeProposal|36548|36610|+62|
|TestInterchainGovernance: executeProposalAction|48840|48888|+48|
|TestMultiSigBase: rotateSigners|90571|90558|-13|
|TestProposalExecutor: forceExecute|82539|82733|+194|
|TestProposalExecutor: setWhitelistedProposalCaller|49295|49341|+46|
|TestProposalExecutor: setWhitelistedProposalSender|49283|49351|+68|
|TestServiceGovernance: executeMultisigProposal|73544|73611|+67|
|TestServiceGovernance: executeProposalAction|47051|47193|+142|
|TimeLockTest: cancelSetNum|22018|22072|+54|
|TimeLockTest: scheduleSetNum|44447|44534|+87|
|TimeLockTest: setNum|45415|45510|+95|
|TokenManagerLiquidityPool: callContractWithInterchainToken|153846|155532|+1686|
|TokenManagerLiquidityPool: sendToken|154119|155867|+1748|
|||||
|Deployment: AxelarAuthWeighted|1015139|952159|-62980|
|Deployment: AxelarDepositService|3062798|2876520|-186278|
|Deployment: AxelarExpressExecutableTest|1140918|1102731|-38187|
|Deployment: AxelarGasService|1109386|1070701|-38685|
|Deployment: AxelarGateway|3980849|3617795|-363054|
|Deployment: AxelarGatewayProxy|415109|393775|-21334|
|Deployment: AxelarValuedExpressExecutableTest|1399737|1347771|-51966|
|Deployment: Create3Deployer|590161|538666|-51495|
|Deployment: DestinationChainSwapExecutable|726668|706369|-20299|
|Deployment: DestinationChainSwapExpress|1454618|1402544|-52074|
|Deployment: DestinationChainTokenSwapper|305214|295068|-10146|
|Deployment: DifferentProxyImplementation|378791|369035|-9756|
|Deployment: DistributableTest|221566|204994|-16572|
|Deployment: DummyState|242276|227252|-15024|
|Deployment: ERC20MintableBurnable|594980|583244|-11736|
|Deployment: ExpressCallHandlerTest|705054|662704|-42350|
|Deployment: FinalProxy|680550|612441|-68109|
|Deployment: FixedProxy|144358|141538|-2820|
|Deployment: FlowLimitTest|275487|254324|-21163|
|Deployment: ImplementationTest|145893|137991|-7902|
|Deployment: InterchainExecutableTest|471077|455872|-15205|
|Deployment: InterchainProposalSender|601367|584248|-17119|
|Deployment: InterchainTokenService|4689381|4316756|-372625|
|Deployment: InvalidProxyImplementation|193321|183601|-9720|
|Deployment: InvalidSetupProxyImplementation|215048|197017|-18031|
|Deployment: MintableCappedERC20|899488|842658|-56830|
|Deployment: MockAxelarGateway|703398|692921|-10477|
|Deployment: MockGateway|2976353|2894158|-82195|
|Deployment: MulticallTest|532204|523474|-8730|
|Deployment: Multisig|1001863|969707|-32156|
|Deployment: NakedProxy|108908|106100|-2808|
|Deployment: OperatorableTest|221578|204994|-16584|
|Deployment: PausableTest|158623|148747|-9876|
|Deployment: ProxyImplementation|373133|363412|-9721|
|Deployment: RemoteAddressValidator|1629033|1480230|-148803|
|Deployment: RemoteAddressValidatorProxy|212280|209552|-2728|
|Deployment: SourceChainSwapCaller|590252|570001|-20251|
|Deployment: StandardizedTokenDeployer|680234|672571|-7663|
|Deployment: StandardizedTokenLockUnlock|1453378|1370670|-82708|
|Deployment: StandardizedTokenMintBurn|1411522|1328809|-82713|
|Deployment: TestInterchainGovernance|1241231|1134074|-107157|
|Deployment: TestMultiSigBase|816076|783932|-32144|
|Deployment: TestProposalExecutor|1223347|1157941|-65406|
|Deployment: TestProxy|256278|253421|-2857|
|Deployment: TestServiceGovernance|2177577|2041698|-135879|
|Deployment: TimeLockTest|408228|336724|-71504|
|Deployment: TokenDeployer|1512761|1422414|-90347|
|Deployment: TokenManagerDeployer|652320|625019|-27301|
|Deployment: TokenManagerLiquidityPool|1296127|1178899|-117228|
|Deployment: TokenManagerLockUnlock|1219554|1116496|-103058|
|Deployment: TokenManagerMintBurn|1112900|1007491|-105409|
|Deployment: UpgradableTest|574428|495975|-78453|

Gas saved: -44892

This perfectly show what happen when the runs number is reduced. Only sum function reduce

This clearly shows the impact of the number of runs. Here, only certain functions are reduced, but very sharply, while all others increase slightly. That's why this change needs to be carefully thought through. Its impact is enormous and surpasses all other imaginable optimizations.