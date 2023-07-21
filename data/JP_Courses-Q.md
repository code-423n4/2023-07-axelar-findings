0. https://github.com/code-423n4/2023-07-axelar/blob/9048517392e03720a0cd32d5b79db84f5e5f7c8e/contracts/its/interchain-token/InterchainToken.sol#L92-L102
/// @audit H/M?: where's the case for < ???; <= is covered, but < is not covered, why not???

1. https://github.com/code-423n4/2023-07-axelar/blob/9048517392e03720a0cd32d5b79db84f5e5f7c8e/contracts/its/interchain-token/InterchainToken.sol#L95
/// @audit H/M?: only covers case for finite allowance that requires approval, doesnt cover case for infinite allowance if needs approval.

2. https://github.com/code-423n4/2023-07-axelar/blob/9048517392e03720a0cd32d5b79db84f5e5f7c8e/contracts/its/interchain-token/InterchainToken.sol#L88-L90
https://github.com/code-423n4/2023-07-axelar/blob/9048517392e03720a0cd32d5b79db84f5e5f7c8e/contracts/its/interchain-token/InterchainToken.sol#L94
/// @audit H/M: no zero amounts checks for both _allowance & amount. max damage?; also, initially it seems attacker can call this function successfully and cause token transfer?

3. https://github.com/code-423n4/2023-07-axelar/blob/9048517392e03720a0cd32d5b79db84f5e5f7c8e/contracts/its/interchain-token/InterchainToken.sol#L92-L102
/// @audit H/M: L100 should be moved to end of if block above it
/// @audit part of above comment: elseif should be added in place of above approve statement(L100) to cover case for < , to do this: _approve(sender, address(tokenManager), allowance_);

Recommendation:

        ITokenManager tokenManager = getTokenManager();
        if (tokenManagerRequiresApproval()) {
            uint256 allowance_ = allowance[sender][address(tokenManager)];  
            if (allowance_ != type(uint256).max) {  
                if (allowance_ >= type(uint256).max - amount) { /// @audit replaced > with >= otherwise case = is excluded
                    allowance_ = type(uint256).max - amount;
                    
                    _approve(sender, address(tokenManager), allowance_ + amount);
                    
                } else if (allowance_ < type(uint256).max - amount) {
                		_approve(sender, address(tokenManager), allowance_);  
                } 
            }   
        }

===============================

1. https://github.com/code-423n4/2023-07-axelar/blob/2f9b234bb8222d5fbe934beafede56bfb4522641/contracts/gmp-sdk/util/Bytes32String.sol#L23-L38
/// @audit QA: It is recommended to include a check to ensure the length of the converted string does not exceed the maximum length of the original string.

2. The rate limiting function of the gateways could theoretically cause a bottleneck or even lead to accidental DoS on the network due to massive queues, unless the rate limiting is applied PER sender and not PER overall sent messages?

It's essential to consider that if rate limiting on the Axelar network's gateways is applied on an overall basis rather than per sender, there is a risk of potential bottlenecks and accidental Denial of Service (DoS) conditions due to massive queues of pending transactions. To address this, implementing rate limiting on a per-sender basis is crucial to ensure a more balanced distribution of transactions and resources, enhancing overall network performance and security.

3. Key rotations on the network.

Recommendation:

Increased Rotation Frequency: Consider increasing the frequency of key rotations to reduce the time window during which an attacker could gain control over the compromised validators. Frequent key rotations enhance the overall security posture of the network.
The protocol developers recommend validators to rotate their tofnd mnemonic once every 2 months. While this is a good practice for enhancing security, it is crucial to ensure that the rotation frequency is not too predictable or long. Randomizing the timing of mnemonic rotations further would make it more challenging for attackers to exploit a fixed rotation schedule.