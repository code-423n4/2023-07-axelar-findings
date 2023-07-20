## [L-01] The variable ```data``` is written twice without being read in between the write operations.
### Proof Of Concept 
Code Snippet & Link : [InterchainTokenService.sol#L874-875](https://github.com/code-423n4/2023-07-axelar/blob/be5fd29162cc329c3f8a0ce73681fb980af8028f/contracts/its/interchain-token-service/InterchainTokenService.sol#L874-L875)
``` solidity
function _decodeMetadata(bytes calldata metadata) internal pure returns (uint32 version, bytes calldata data) {
        assembly {
            data.length := sub(metadata.length, 4)
            data.offset := add(metadata.offset, 4)
            version := calldataload(sub(metadata.offset, 28))
        }
    }
```
```data.length := sub(metadata.length, 4)```
Here, the length property of data is assigned the value of ```metadata.length``` minus 4. This line writes a value to data.
```data.offset := add(metadata.offset, 4)```
The offset property of data is assigned the value of metadata.offset plus 4. This line also writes a value to data.
Both of these assignments modify the data variable, but there is no read operation or usage of data in between these two assignments. As a result, the first assignment becomes redundant and has no effect on the execution of the code.
## Tools Used 
Static Analyzer 
## Recommended Mitigation Steps
Remove the first assignment to ```data``` since it is overwritten by the second assignment.
##