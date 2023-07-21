# incorrect constant value 
File:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/ExpressCallHandler.sol#L13-L16
according to above code:
```
    // uint256(keccak256('prefix-express-give-token'));
    uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN = 0x67c7b41c1cb0375e36084c4ec399d005168e83425fa471b9224f6115af865619;
    // uint256(keccak256('prefix-express-give-token-with-data'));
    uint256 internal constant PREFIX_EXPRESS_RECEIVE_TOKEN_WITH_DATA = 0x3e607cc12a253b1d9f677a03d298ad869a90a8ba4bd0fb5739e7d79db7cdeaad;
```
By using `chisel`
➜ uint256(keccak256('prefix-express-give-token'))
Type: uint
├ Hex: 0x67c7b41c1cb0375e36084c4ec399d005168e83425fa471b9224f6115af86561b
└ Decimal: 46941069042209500805945719818346646414095732535730262426334574041245807957531
➜ uint256(keccak256('prefix-express-give-token-with-data'))
Type: uint
├ Hex: 0x3e607cc12a253b1d9f677a03d298ad869a90a8ba4bd0fb5739e7d79db7cdeaaf
└ Decimal: 28213874954636381927292981123379327098646147195163401297181417794057951636143

# incorrect constant value 
File:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/utils/Operatable.sol#L14-L15
according to above code:
```
    // uint256(keccak256('operator')) - 1
    uint256 internal constant OPERATOR_SLOT = 0xf23ec0bb4210edd5cba85afd05127efcd2fc6a781bfed49188da1081670b22d7;

```
By using `chisel`
➜ uint256(keccak256('operator')) - 1
Type: uint
├ Hex: 0x46a52cf33029de9f84853745a87af28464c80bf0346df1b32e205fc73319f621
└ Decimal: 31953739399695593666283731411274577053612996981383225307242093477592974358049

# missing validate parameters `tokenManagerDeployer_` and `standardizedTokenDeployer_` are zero
File:
    https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/interchain-token-service/InterchainTokenService.sol#L100-L101

If `tokenManagerDeployer_` or `standardizedTokenDeployer_` is zero, the function will not revert since  low level call returns true if the address doesn't exist, and  `tokenManagerDeployer` and `standardizedTokenDeployer` can't be set again

# Gas griefing/theft is possible on unsafe external call
File:
https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/cgp/util/Caller.sol#L18

    Now (bool success, ) is actually the same as writing (bool success, bytes memory data) which basically means that even though the data is omitted it doesn’t 
    mean that the contract does not handle it. Actually, the way it works is the bytes data that was returned from the receiver will be copied to memory. Memory 
    allocation becomes very costly if the payload is big, so this means that if a receiver implements a fallback function that returns a huge payload, then the  
    msg.sender of the transaction, in our case the relayer, will have to pay a huge amount of gas for copying this payload to memory.


# missing validate `impl` is zero
File:
    https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/its/proxies/TokenManagerProxy.sol#L34

If `impl` is zero, the `constructor` function will not revert since  low level call returns true if the address doesn't exist