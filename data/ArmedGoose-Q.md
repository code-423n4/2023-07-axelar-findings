[L-01] Pragma specifies 0.8.0 and above which may fail on some networks (e.g. Arbitrum)
Pragma has been set to ^0.8.0 allowing the contracts to be compiled with any 0.8.x compiler. The problem with this is that Arbitrum is NOT compatible with 0.8.20 and newer, so deploying on that network may cause problems. The default behavior of compiler is to use the latest version which would mean by default it will be compiled with the 0.8.20 version, which in turn will produce broken code.
```
cd contracts
grep -Ril "\^0\.8\.0" .

./gmp-sdk/upgradable/InitProxy.sol
./gmp-sdk/upgradable/FinalProxy.sol
./gmp-sdk/upgradable/Proxy.sol
./gmp-sdk/upgradable/BaseProxy.sol
./gmp-sdk/upgradable/FixedProxy.sol
./gmp-sdk/upgradable/Upgradable.sol
./gmp-sdk/util/Bytes32String.sol
./gmp-sdk/util/Ownable.sol
./gmp-sdk/util/ContractAddress.sol
./gmp-sdk/util/SafeTransfer.sol
./gmp-sdk/util/TimeLock.sol
./gmp-sdk/util/AddressString.sol
./gmp-sdk/interfaces/ITimeLock.sol
./gmp-sdk/interfaces/IAxelarExecutable.sol
./gmp-sdk/interfaces/IProxy.sol
./gmp-sdk/interfaces/IAxelarGateway.sol
./gmp-sdk/interfaces/IAxelarExpressExecutable.sol
./gmp-sdk/interfaces/IAxelarValuedExpressExecutable.sol
./gmp-sdk/interfaces/IUpgradable.sol
./gmp-sdk/interfaces/IAxelarGasService.sol
./gmp-sdk/interfaces/IFinalProxy.sol
./gmp-sdk/interfaces/IOwnable.sol
./gmp-sdk/interfaces/IERC20MintableBurnable.sol
./gmp-sdk/interfaces/IInitProxy.sol
./gmp-sdk/executable/AxelarExecutable.sol
./gmp-sdk/express/AxelarValuedExpressExecutable.sol
./gmp-sdk/express/ExpressExecutorTracker.sol
./gmp-sdk/express/AxelarExpressExecutable.sol
./gmp-sdk/test/MockDepositReceiver.sol
./gmp-sdk/test/TimeLockTest.sol
./gmp-sdk/test/ProxyTest.sol
./gmp-sdk/test/gmp/DestinationChainSwapExecutable.sol
./gmp-sdk/test/gmp/ExpressExecutableTest.sol
./gmp-sdk/test/gmp/DestinationChainSwapExpressDisabled.sol
./gmp-sdk/test/gmp/ExecutableSample.sol
./gmp-sdk/test/gmp/DestinationChainTokenSwapper.sol
./gmp-sdk/test/gmp/SourceChainSwapCaller.sol
./gmp-sdk/test/gmp/ValuedExpressExecutableTest.sol
./gmp-sdk/test/gmp/DestinationChainSwapExpress.sol
./gmp-sdk/test/MockGasService.sol
./gmp-sdk/test/UpgradableTest.sol
./gmp-sdk/test/ERC20MintableBurnable.sol
./gmp-sdk/test/ERC20.sol
./gmp-sdk/test/proxy/InvalidProxyImplementation.sol
./gmp-sdk/test/proxy/InvalidSetupProxyImplementation.sol
./gmp-sdk/test/proxy/ProxyImplementation.sol
./gmp-sdk/test/proxy/FixedImplementation.sol
./gmp-sdk/test/proxy/DifferentProxyImplementation.sol
./gmp-sdk/test/ERC20MintableBurnableInit.sol
./gmp-sdk/test/MockGateway.sol
./gmp-sdk/deploy/ConstAddressDeployer.sol
./gmp-sdk/deploy/Create3.sol
./gmp-sdk/deploy/Create3Deployer.sol
./cgp/governance/InterchainGovernance.sol
./cgp/governance/Multisig.sol
./cgp/governance/AxelarServiceGovernance.sol
./cgp/util/Caller.sol
./cgp/util/Upgradable.sol
./cgp/interfaces/IInterchainGovernance.sol
./cgp/interfaces/IDepositServiceBase.sol
./cgp/interfaces/IMultisigBase.sol
./cgp/interfaces/IAxelarDepositService.sol
./cgp/interfaces/IMultisig.sol
./cgp/interfaces/ICaller.sol
./cgp/interfaces/IAxelarServiceGovernance.sol
./cgp/interfaces/IGovernable.sol
./cgp/auth/MultisigBase.sol
./cgp/test/Target.sol
./cgp/test/DepositReceiver.sol
./cgp/test/TestInterchainGovernance.sol
./cgp/test/TestMultiSigBase.sol
./cgp/test/TestServiceGovernance.sol
./interchain-governance-executor/InterchainProposalExecutor.sol
./interchain-governance-executor/interfaces/IInterchainProposalSender.sol
./interchain-governance-executor/interfaces/IInterchainProposalExecutor.sol
./interchain-governance-executor/InterchainProposalSender.sol
./interchain-governance-executor/lib/InterchainCalls.sol
./interchain-governance-executor/test/TestProposalExecutor.sol
./interchain-governance-executor/test/DummyState.sol
./its/interfaces/IMockAxelarGateway.sol
./its/interfaces/IStandardizedToken.sol
./its/interfaces/IRemoteAddressValidator.sol
./its/interfaces/IMulticall.sol
./its/interfaces/IOperatable.sol
./its/interfaces/IInterchainTokenExpressExecutable.sol
./its/interfaces/ITokenManagerDeployer.sol
./its/interfaces/IStandardizedTokenProxy.sol
./its/interfaces/IExpressCallHandler.sol
./its/interfaces/IFlowLimit.sol
./its/interfaces/ITokenManagerType.sol
./its/interfaces/IERC20BurnableMintable.sol
./its/interfaces/IPausable.sol
./its/interfaces/IImplementation.sol
./its/interfaces/IERC20Named.sol
./its/interfaces/IInterchainTokenExecutable.sol
./its/interfaces/IStandardizedTokenDeployer.sol
./its/interfaces/IInterchainTokenService.sol
./its/interfaces/ITokenManagerProxy.sol
./its/interfaces/IDistributable.sol
./its/interfaces/IInterchainToken.sol
./its/interfaces/ITokenManager.sol
./its/interchain-token/InterchainToken.sol
./its/remote-address-validator/RemoteAddressValidator.sol
./its/proxies/InterchainTokenServiceProxy.sol
./its/proxies/StandardizedTokenProxy.sol
./its/proxies/TokenManagerProxy.sol
./its/proxies/RemoteAddressValidatorProxy.sol
./its/libraries/AddressBytesUtils.sol
./its/examples/InterchainTokenExpressExecutable.sol
./its/examples/InterchainTokenExecutable.sol
./its/token-implementations/StandardizedTokenMintBurn.sol
./its/token-implementations/StandardizedTokenLockUnlock.sol
./its/token-implementations/ERC20Permit.sol
./its/token-implementations/StandardizedToken.sol
./its/token-implementations/ERC20.sol
./its/token-manager/implementations/TokenManagerLockUnlock.sol
./its/token-manager/implementations/TokenManagerAddressStorage.sol
./its/token-manager/implementations/TokenManagerMintBurn.sol
./its/token-manager/implementations/TokenManagerLiquidityPool.sol
./its/token-manager/TokenManager.sol
./its/test/AxelarGasService.sol
./its/test/InterchainExecutableTest.sol
./its/test/MockAxelarGateway.sol
./its/test/InterchainTokenTest.sol
./its/test/utils/ImplementationTest.sol
./its/test/utils/ExpressCallHandlerTest.sol
./its/test/utils/Pausable.sol
./its/test/utils/MulticallTest.sol
./its/test/utils/NakedProxy.sol
./its/test/utils/DistributableTest.sol
./its/test/utils/FlowLimitTest.sol
./its/test/utils/AdminableTest.sol
./its/interchain-token-service/InterchainTokenService.sol
./its/utils/Distributable.sol
./its/utils/Implementation.sol
./its/utils/StandardizedTokenDeployer.sol
./its/utils/TokenManagerDeployer.sol
./its/utils/Pausable.sol
./its/utils/FlowLimit.sol
./its/utils/ExpressCallHandler.sol
./its/utils/Multicall.sol
./its/utils/Operatable.sol
```

Recommendation: Consider using `pragma solidity >=0.8.0 <=0.8.19`. Reference: https://developer.arbitrum.io/solidity-support


[L-02] MultisigBase.sol does not enforce proper multisig threshold
in [MultisigBase.sol](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L159-L161) there are two checks when retating signers: if threshold is larger than length and if its not zero. But there is no requirement of having a reasonable threshold, which is usually should be `numberOfSigners / 2 + 1`. At current state, it still could result in having 10 signers and a threshold of 1, which would be far from a secure multisig.
Recommendation: Enforce a secure threshold e.g. `numberOfSigners / 2 + 1` instead of just non-zero.


[I-01] Redundant external wrapper in MultiSigBase.sol
Function [rotateSigners](https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/auth/MultisigBase.sol#L142) do not add any value to its external subroutine, and has the same arguments. This means that these two could be merged instead and just `_rotateSigners` could become `rotateSigners` external and current `rotateSigners` could be removed.

