[L] governanceAddress being passed as string memory

```
   constructor(
        address gateway,
        string memory governanceChain,
        string memory governanceAddress,
        uint256 minimumTimeDelay,
        address[] memory cosigners,
        uint256 threshold
    ) 
```

https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/governance/AxelarServiceGovernance.sol#L33

Mitigation:

Pass ```governanceAddress``` as an address, not string memory.