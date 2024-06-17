# Aderyn Analysis Report

This report was generated by [Aderyn](https://github.com/Cyfrin/aderyn), a static analysis tool built by [Cyfrin](https://cyfrin.io), a blockchain security company. This report is not a substitute for manual audit or security review. It should not be relied upon for any purpose other than to assist in the identification of potential security vulnerabilities.
# Table of Contents

- [Summary](#summary)
  - [Files Summary](#files-summary)
  - [Files Details](#files-details)
  - [Issue Summary](#issue-summary)
- [High Issues](#high-issues)
  - [H-1: Arbitrary `from` passed to `transferFrom` (or `safeTransferFrom`)](#h-1-arbitrary-from-passed-to-transferfrom-or-safetransferfrom)
- [Low Issues](#low-issues)
  - [L-1: Centralization Risk for trusted owners](#l-1-centralization-risk-for-trusted-owners)
  - [L-2: Unsafe ERC20 Operations should not be used](#l-2-unsafe-erc20-operations-should-not-be-used)
  - [L-3: Missing checks for `address(0)` when assigning values to address state variables](#l-3-missing-checks-for-address0-when-assigning-values-to-address-state-variables)
  - [L-4: `public` functions not used internally could be marked `external`](#l-4-public-functions-not-used-internally-could-be-marked-external)
  - [L-5: Event is missing `indexed` fields](#l-5-event-is-missing-indexed-fields)
  - [L-6: PUSH0 is not supported by all chains](#l-6-push0-is-not-supported-by-all-chains)


# Summary

## Files Summary

| Key | Value |
| --- | --- |
| .sol Files | 4 |
| Total nSLOC | 101 |


## Files Details

| Filepath | nSLOC |
| --- | --- |
| src/L1BossBridge.sol | 64 |
| src/L1Token.sol | 8 |
| src/L1Vault.sol | 12 |
| src/TokenFactory.sol | 17 |
| **Total** | **101** |


## Issue Summary

| Category | No. of Issues |
| --- | --- |
| High | 1 |
| Low | 6 |


# High Issues

## H-1: Arbitrary `from` passed to `transferFrom` (or `safeTransferFrom`)

Passing an arbitrary `from` address to `transferFrom` (or `safeTransferFrom`) can lead to loss of funds, because anyone can transfer tokens from the `from` address if an approval is made.  

- Found in src/L1BossBridge.sol [Line: 74](src/L1BossBridge.sol#L74)

	```solidity
	        token.safeTransferFrom(from, address(vault), amount);
	```



# Low Issues

## L-1: Centralization Risk for trusted owners

Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

- Found in src/L1BossBridge.sol [Line: 27](src/L1BossBridge.sol#L27)

	```solidity
	contract L1BossBridge is Ownable, Pausable, ReentrancyGuard {
	```

- Found in src/L1BossBridge.sol [Line: 49](src/L1BossBridge.sol#L49)

	```solidity
	    function pause() external onlyOwner {
	```

- Found in src/L1BossBridge.sol [Line: 53](src/L1BossBridge.sol#L53)

	```solidity
	    function unpause() external onlyOwner {
	```

- Found in src/L1BossBridge.sol [Line: 57](src/L1BossBridge.sol#L57)

	```solidity
	    function setSigner(address account, bool enabled) external onlyOwner {
	```

- Found in src/L1Vault.sol [Line: 12](src/L1Vault.sol#L12)

	```solidity
	contract L1Vault is Ownable {
	```

- Found in src/L1Vault.sol [Line: 19](src/L1Vault.sol#L19)

	```solidity
	    function approveTo(address target, uint256 amount) external onlyOwner {
	```

- Found in src/TokenFactory.sol [Line: 11](src/TokenFactory.sol#L11)

	```solidity
	contract TokenFactory is Ownable {
	```

- Found in src/TokenFactory.sol [Line: 23](src/TokenFactory.sol#L23)

	```solidity
	    function deployToken(string memory symbol, bytes memory contractBytecode) public onlyOwner returns (address addr) {
	```



## L-2: Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

- Found in src/L1BossBridge.sol [Line: 99](src/L1BossBridge.sol#L99)

	```solidity
	                abi.encodeCall(IERC20.transferFrom, (address(vault), to, amount))
	```

- Found in src/L1Vault.sol [Line: 20](src/L1Vault.sol#L20)

	```solidity
	        token.approve(target, amount);
	```



## L-3: Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

- Found in src/L1BossBridge.sol [Line: 58](src/L1BossBridge.sol#L58)

	```solidity
	        signers[account] = enabled;
	```

- Found in src/L1Vault.sol [Line: 16](src/L1Vault.sol#L16)

	```solidity
	        token = _token;
	```



## L-4: `public` functions not used internally could be marked `external`

Instead of marking a function as `public`, consider marking it as `external` if it is not used internally.

- Found in src/TokenFactory.sol [Line: 23](src/TokenFactory.sol#L23)

	```solidity
	    function deployToken(string memory symbol, bytes memory contractBytecode) public onlyOwner returns (address addr) {
	```

- Found in src/TokenFactory.sol [Line: 31](src/TokenFactory.sol#L31)

	```solidity
	    function getTokenAddressFromSymbol(string memory symbol) public view returns (address addr) {
	```



## L-5: Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

- Found in src/L1BossBridge.sol [Line: 40](src/L1BossBridge.sol#L40)

	```solidity
	    event Deposit(address from, address to, uint256 amount);
	```

- Found in src/TokenFactory.sol [Line: 14](src/TokenFactory.sol#L14)

	```solidity
	    event TokenDeployed(string symbol, address addr);
	```



## L-6: PUSH0 is not supported by all chains

Solc compiler version 0.8.20 switches the default target EVM version to Shanghai, which means that the generated bytecode will include PUSH0 opcodes. Be sure to select the appropriate EVM version in case you intend to deploy on a chain other than mainnet like L2 chains that may not support PUSH0, otherwise deployment of your contracts will fail.

- Found in src/L1BossBridge.sol [Line: 15](src/L1BossBridge.sol#L15)

	```solidity
	pragma solidity 0.8.20;
	```

- Found in src/L1Token.sol [Line: 2](src/L1Token.sol#L2)

	```solidity
	pragma solidity 0.8.20;
	```

- Found in src/L1Vault.sol [Line: 2](src/L1Vault.sol#L2)

	```solidity
	pragma solidity 0.8.20;
	```

- Found in src/TokenFactory.sol [Line: 2](src/TokenFactory.sol#L2)

	```solidity
	pragma solidity 0.8.20;
	```


