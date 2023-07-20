## [L-01] The variable ```data``` is written twice without being read in between the write operations.
#### Proof Of Concept 
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
#### Tools Used 
Slither 
#### Recommended Mitigation Steps
Remove the first assignment to ```data``` since it is overwritten by the second assignment.
## [L-02] ```tokenAddress``` local variable of ```TokenManagerMintBurn._setup``` shadows ```tokenAddress()``` function in ```TokenManager``` contract.  
#### Proof Of Concept
Code snippet from ```TokenManagerMintBurn._setup```
``` solidity
function _setup(bytes calldata params) internal override {
        ......
        (, address tokenAddress) = abi.decode(params, (bytes, address));
        ......
    }
```
Code snippet from ```TokenManager```
``` solidity
function tokenAddress() public view virtual returns (address);
```
Here, local variable of TokenManagerMintBurn._setup is shadowing tokenAddress() function of TokenManager. 
#### Tools Used 
Slither
#### Recommended Mitigation Step 
Rename the local variables that shadow another component.
#### Code Link 
[TokenManagerMintBurn.sol#L35](https://github.com/code-423n4/2023-07-axelar/blob/be5fd29162cc329c3f8a0ce73681fb980af8028f/contracts/its/token-manager/implementations/TokenManagerMintBurn.sol#L35)
[TokenManager.sol#L53](https://github.com/code-423n4/2023-07-axelar/blob/be5fd29162cc329c3f8a0ce73681fb980af8028f/contracts/its/token-manager/TokenManager.sol#L53)
#### Reference 
https://github.com/crytic/slither/wiki/Detector-Documentation#local-variable-shadowing
## [L-03] ```tokenManager_``` Is Missing Zero Address Validation
#### Proof Of Concept 
Code Snippet
``` solidity
{
            address distributor_;
            address tokenManager_;
            string memory tokenName;
            (tokenManager_, distributor_, tokenName, symbol, decimals) = abi.decode(params, (address, address, string, string, uint8));
            _setDistributor(distributor_);
            tokenManager = tokenManager_;
            _setDomainTypeSignatureHash(tokenName);
            name = tokenName;
}
```
Here ```tokenManager = tokenManager_;``` Bob Specify tokenManager Mistankenly to zero address. Now Zero Address is assigned as tokenManager.
#### Tools Used 
Slither
#### Code Link 
[StandardizedToken.sol#L56](https://github.com/code-423n4/2023-07-axelar/blob/be5fd29162cc329c3f8a0ce73681fb980af8028f/contracts/its/token-implementations/StandardizedToken.sol#L56)
#### Recommended Mitigation Step 
Check that the address is not zero.
#### Reference 
[Zero Address Check](https://github.com/crytic/slither/wiki/Detector-Documentation#missing-zero-address-validation)
## [L-04] Using ```Block.timestamp``` in ```if``` Condition Can Be dangerous
#### Proof Of Concept 
``` solidity
function _finalizeTimeLock(bytes32 hash) internal {
        .......

        if (block.timestamp < eta) revert TimeLockNotReady();

        .......
    }  
```
Miner can manipulate ```block.timestamp``` to bypass this if condtion and possible scenario of bypassing this condition would be.
- ```Premature Finalization```: If the manipulated ```block.timestamp``` is less than the expected eta value, the condition block.timestamp < eta will evaluate to ``true``` even though the required time delay has not actually passed. This could allow the ```finalization``` of a timelock before the intended waiting period is complete, bypassing the ```desired delay```.

- ```Delayed Finalization```: Conversely, if the manipulated ```block.timestamp``` is greater than or equal to the expected eta value, the condition block.timestamp < eta will evaluate to false, and the ```TimeLockNotReady()``` revert statement will be triggered. This will ```prevent``` the finalization of the timelock even if the required time delay has actually passed.

#### Tools Used 
Slither
#### Recommended Mitigation Step 
Avoid Relying on ```Block.timestamp```
#### Code Link
[TimeLock.sol#L84](https://github.com/code-423n4/2023-07-axelar/blob/be5fd29162cc329c3f8a0ce73681fb980af8028f/contracts/gmp-sdk/util/TimeLock.sol#L84)
#### Reference 
[Block.timestamp](https://github.com/crytic/slither/wiki/Detector-Documentation#block-timestamp)
## [NC-01] ```contractId()``` of ```InterchainTokenServiceProxy``` Is Never Used And Should Be Removed 
#### Proof Of Concept 
``` solidity 
function contractId() internal pure override returns (bytes32) {
        return CONTRACT_ID;
    }
```
This Function Is Never Used 
#### Tools Used 
Slither Detection
#### Recommended Mitigation Step
Removed Unsused or Dead Code for Code Efficiency and Optimization
#### Code Link
[InterchainTokenServiceProxy.sol#L29](https://github.com/code-423n4/2023-07-axelar/blob/be5fd29162cc329c3f8a0ce73681fb980af8028f/contracts/its/proxies/InterchainTokenServiceProxy.sol#L29)
#### Reference 
[DeadCode](https://github.com/crytic/slither/wiki/Detector-Documentation#dead-code)