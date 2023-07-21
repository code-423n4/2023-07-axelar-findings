# keccak256() Should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead.

```
File: cgp/AxelarGateway.sol

246:    function contractId() public pure returns (bytes32) {
247:        return keccak256('axelar-gateway');
248:    }
```

Link: https://github.com/code-423n4/2023-07-axelar/blob/main/contracts/cgp/AxelarGateway.sol#L247
