## `InterchainGovernance` Contract presumes validity of entered proposal before executing.
It assumes that the proposal hash (generated from `target`, `callData`, and `nativeValue`) is always correct. This could lead to unwanted or malicious execution if an invalid proposal is mistakenly accepted.

```solidity
function executeProposal( //understand this function
        address target, //only external payable function that modifies the state
        bytes calldata callData, //how about checking for validity of proposal
        uint256 nativeValue
    ) external payable {
        bytes32 proposalHash = keccak256(abi.encodePacked(target, callData, nativeValue)); //get the proposal has from encoding 3 values

        _finalizeTimeLock(proposalHash);
        _call(target, callData, nativeValue);

        emit ProposalExecuted(proposalHash, target, callData, nativeValue, block.timestamp);
    } 
```


## `minimumTimeDelay` should be checked before contract deployment
To avoid unexpected behavior, there are currently no checks for validity of the `minimumTimeDelay`

```solidity
constructor(
        address gateway,
        string memory governanceChain_,
        string memory governanceAddress_,
        uint256 minimumTimeDelay
    ) AxelarExecutable(gateway) TimeLock(minimumTimeDelay) {
        governanceChain = governanceChain_;
        governanceAddress = governanceAddress_;
        governanceChainHash = keccak256(bytes(governanceChain_));
        governanceAddressHash = keccak256(bytes(governanceAddress_));
    }
```

## potential `out-of-gas error` in InterchainProposalSender.sol if the array is too large.
If an attacker passes a large array, it can potentially lead to out-of-gas issues or denial-of-service (DoS) attacks. Consider limiting the array size or implementing batching to avoid this problem.

```solidity
function revertIfInvalidFee(InterchainCalls.InterchainCall[] calldata interchainCalls) private {
        uint256 totalGas = 0;
        for (uint256 i = 0; i < interchainCalls.length; ) {
            totalGas += interchainCalls[i].gas;
            unchecked {
                ++i;
            }
        }

        if (totalGas != msg.value) {
            revert InvalidFee();
        }
    }
```

## The `InterchainProposalSender` contract calls external contracts `AxelarGateway` without handling possible errors. 
If these external calls fail, the contract could end up in an inconsistent state or leave pending transactions.

```solidity
       gateway.callContract(interchainCall.destinationChain, interchainCall.destinationContract, payload);
    }
```

