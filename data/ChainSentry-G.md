1. Use immutable for OpenZeppelin AccessControl's Roles Declarations

Using immutable for OpenZeppelin AccessControl's roles declarations can lead to gas optimizations. This issue was found in multiple files and lines of code, including but not limited to:

- `cgp/AdminMultisigBase.sol::19`
- `cgp/AdminMultisigBase.sol::21`
- `cgp/AdminMultisigBase.sol::22`
- `cgp/AdminMultisigBase.sol::23`
- `cgp/AdminMultisigBase.sol::24`

2. Use Shift Right/Left instead of Division/Multiplication if possible

Using shift right/left instead of division/multiplication where possible can lead to gas optimizations. This issue was found in multiple files and lines of code, including but not limited to:

- `cgp/ECDSA.sol::56`
- `cgp/ERC20.sol::15`
- `cgp/interfaces/IERC20.sol::49`