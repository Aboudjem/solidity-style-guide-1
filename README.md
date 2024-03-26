# Coinbase Solidity Style Guide

This is a guide for Coinbase engineers developing EVM-based smart contracts. We use Solidity when developing such contracts, so we call it a "Solidity Style Guide." This guide also covers development and testing practices. We are sharing this publicly incase it is useful to others.

## Why?

We should be as specific and thorough as possible when defining our style, testing, and development practices. Any time we save not having to debate these things on pull requests is productive time that can go into other discussion and review.

## 1. Style

### A. Unless an exception is specifically noted, we follow the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html).

### B. Exceptions

#### 1. Names of internal functions in a library should not have an underscore prefix.

The style guide states

> Underscore Prefix for Non-external Functions and Variables

One of the motivations for this rule is that it is a helpful visual clue.

> Leading underscores allow you to immediately recognize the intent of such functions...

We agree that a leading underscore is a useful visual clue, and this is why we oppose using them for internal library functions that can be called from other contracts. Visually, it looks wrong.

```
Library._function()
```

or

```
using Library for bytes
bytes._function()
```

Note, we cannot address this by insisting on the use public functions. Whether a library functions are internal or external has important implications. From the [documentation](https://docs.soliditylang.org/en/latest/contracts.html#libraries)

> ... the code of internal library functions that are called from a contract and all functions called from therein will at compile time be included in the calling contract, and a regular JUMP call will be used instead of a DELEGATECALL.

Developers may prefer internal functions because they are more gas efficient to call.

If a function should never be called from another contract, it should be marked private and its name should have a leading underscore.

### C. Additions

#### 1. Prefer custom errors.

Custom errors are in some cases more gas efficient and allow passing useful information.

#### 2. Custom error names should be CapWords style.

For example, `InsufficientBalance`.

#### 3. Event names should be past tense.

For example, `UpdatedOwner` not `UpdateOwner`.

Events should track things that _happened_ and so should be past tense. Using past tense also helps avoid naming collisions with structs or functions.

We are aware this does not follow precedent from early ERCs, like [ERC-20](https://eips.ethereum.org/EIPS/eip-20). However it does align with some more recent high profile Solidity, e.g. [1](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/976a3d53624849ecaef1231019d2052a16a39ce4/contracts/access/Ownable.sol#L33), [2](https://github.com/aave/aave-v3-core/blob/724a9ef43adf139437ba87dcbab63462394d4601/contracts/interfaces/IAaveOracle.sol#L25-L31), [3](https://github.com/ProjectOpenSea/seaport/blob/1d12e33b71b6988cbbe955373ddbc40a87bd5b16/contracts/zones/interfaces/PausableZoneEventsAndErrors.sol#L25-L41).

#### 4. Avoid using assembly.

Assembly code is hard to read and audit. We should avoid it unless the gas savings are very consequential, e.g. > 25%.

#### 5. Avoid unnecessary named return arguments.

In short functions, named return arguments are unnecessary.

NO:

```solidity
function add(uint a, uint b) public returns (uint result) {
  result = a + b;
}
```

Named return arguments can be helpful in functions with multiple returned values.

```solidity
function validate(UserOperation calldata userOp) external returns (bytes memory context, uint256 validationData)
```

However, it is important to be explicit when returning early.\
NO:

```solidity
function validate(UserOperation calldata userOp) external returns (bytes memory context, uint256 validationData) {
  context = "";
  validationData = 1;

  if (condition) {
    return;
  }
}
```

YES:

```solidity
function validate(UserOperation calldata userOp) external returns (bytes memory context, uint256 validationData) {
  context = "";
  validationData = 1;

  if (condition) {
    return (context, validationData);
  }
}
```

#### 6. Avoid leading verbs in function names.

NO:

- getBalanceOf
- doValidate

YES:

- balanceOf
- validate

#### 7. Prefer composition over inheritence.

Excessive use of inheritence can make code harder to reason about and audit. When possible, prefer composing logic in a single contract.

#### 8. Avoid writing interfaces.

Interfaces separate NatSpec from contract logic, requiring readers to do more work to understand the code. For this reason, they should be avoided.

#### 9. Avoid unnecessary version Pragma constraints.

While the main contracts we deploy should specify a single Solidity version, all supporting contracts and libraries should have as open a Pragma as possible. A good rule of thumb is to the next major version. For example

```solidity
pragma solidity ^0.8.0;
```

#### 10. Prefer struct and error defintions in a contract.

Defining errors and structs within a contract makes their usage elsewhere easier to follow.

NO:

```
error MyEvent();

struct MyStruct {};

contract MyContract {}
```

YES:

```
contract MyContract {
  error MyEvent();
  struct MyStruct {};
}
```

## 2. Development

### A. Use [Forge](https://github.com/foundry-rs/foundry/tree/master/crates/forge) for testing and dependency management.

### B. Testing

#### 1. Test file names should follow Solidity Style Guide conventions and also have `.t` before `.sol`.

For example, `ERC20.t.sol`

#### 2. Test contract names should follow include the name of the contract or function being tested, followed by "Test".

For example,

- `ERC20Test`
- `TransferFromTest`

#### 3. Test names should follow the convention `test_functionName_outcome_optionalContext`

For example

1. `test_transferFrom_debitsFromAccountBalance`
2. `test_transferFrom_debitsFromAccountBalance_whenCalledViaPermit`

If the contract is named after a function, then function name can be omitted.

```
contract TransferFromTest {
  function test_debitsFromAccountBalance() ...
}
```

### C. Upgradability

#### 1. Prefer [ERC-7201](https://eips.ethereum.org/EIPS/eip-7201) "Namespaced Storage Layout" convention to avoid storage collisions.
