**ConstAddressDeployer.sol**
- L25/47/63/67 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**Create3Deployer.sol**
- L30/54/68 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**Create3.sol**
- L72/74 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**Proxy.sol**
- L5 - Iproxy interface is imported that is never used, therefore adding them generate extra gas expense in the deploy and also reduces the understanding of the contract when the code is viewed in the blockchain explorer.


**InterchainTokenService.sol**
- L5 - IAxelarGateway interface is imported that is never used, therefore adding them generate extra gas expense in the deploy and also reduces the understanding of the contract when the code is viewed in the blockchain explorer.


**TokenManagerProxy.sol**
- L31/32 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.


**AddressBytesUtils.sol**
- L18/21/31/34/35 - The number 20 is used multiple times, within the code and the meaning of that value is not explained. Therefore, it would be beneficial for the understanding of the code, to give a name to that variable.


**RemoteAddressValidator.sol**
- L29/30 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.


**TokenManagerAddressStorage.sol**
- L6/7 - IERC20 and IAxelarGateway interfaces are imported that are never used, therefore adding them generates extra gas expense in the deploy and also reduces the understanding of the contract when the code is viewed in the blockchain explorer.


**StandardizedToken.sol**
- L5 - IERC20BurnableMintable inteface is imported that is never used, therefore adding them generate extra gas expense in the deploy and also reduces the understanding of the contract when the code is viewed in the blockchain explorer.


**IStandardizedToken.sol**
- The IStandardizedToken interface is created but is not used in the StandardizedToken contract. This can be confusing. Therefore it should be given a name according to where it is used.
yearl


**InterchainGovernance.sol**
- L41/42 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.

- L73/148 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**AxelarServiceGovernance.sol**
- L53/84 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**TimeLock.sol**
- L22 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.

- L93/104 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**AxelarGateway.sol**
- L350/396/435/616 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.


**InterchainProposalSender.sol**
- L43/44 - No validation is performed in the constructor and the variables are immutable, therefore they should be validated before setting the variable to != 0.
