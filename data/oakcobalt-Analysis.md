### Any comments for the judge to contextualize your findings
-Finding edge cases during normal usage of the interchain token service that may result in unintended behaviors;
-Any vulnerabilities that might cause loss of funds for users.
-Any vulnerabilities that might cause malicious users to steal funds at the expense of other users or the protocol.

### Approach is taken in evaluating the codebase
-Manual walk-through
-Custom script testing

### Architecture recommendations
-There is one part of the design system that seems a bit overlooked is the maintenance and coordination of registries. For example, RemoteAddressValidtor.sol plays a crucial role in keeping track of supported chains and addresses on chains. The current implementation shows some vulnerabilities and the contract is also not adequately tested and integrated to interchain token service. 

### Codebase quality analysis
-High-quality code overall with clear documentation.
-Only a few unused functions or custom errors were observed.

### Centralization risks
-Low

### Mechanism review
Overall innovative cross-chain implementations. 

Interchain token service takes advantage of `Create3` deployment which simplifies custom token integration and eliminates the need for bookkeeping for users. The `multiCall` feature allows management of cross-chain tokens with ease.

It would be great to see an example implementation of a generic DAO governance integration. 

### Systemic risks
In my opinion, one critical point of potential failure of the system is the coordination of cross-chain registries, which include both gateway contracts and remote address validator contracts. What is the mechanism for updating these registries when a new chain is supported or an existing chain is removed? Are there any vulnerabilities during the update? How can a request to/execution on a chain be paused independently? And how would other chains coordinate with the pause to avoid users losing funds? There should be both on-chain and off-chain protection in place during these scenarios.
 

### Time spent:
24 hours