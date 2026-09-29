---
title: Getting Started with Solidity
image: https://lh3.googleusercontent.com/pw/AL9nZEWeD0tHumr3N1CaaWLwGcjzrcrRXFkQFNEkaerSNBCd1TTg0mK5YSzvNHQJPn6P2KRDK3tuTAqxxr8mZR1FHR1PiVKCJGEr0WplEmJEGlQqEWJawZLCQKs5Xn-a8z5jp1gT9hb8ksXQ4sf1L7aWIE-c=w1942-h1434-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Solidity
    - Blockchain
mermaid: false
layout: post
---

[Solidity](https://docs.soliditylang.org/en/v0.8.17/){:target="_blank"} is an object oriented programming language that shares many of its syntax with other statically typed languages like *Java*, and does the process of verifying and enforcing the constraints at compile-time as opposed to run-time like *Python*.

This language is used for implementing smart contracts that are deployed on blockchains platforms like Ethereum. Therefore, Solidity has a very specific purpose that is focused on consistency and security.

A Smart Contract? What is that? Smart contracts are simply applications stored on a blockchain that run when particular conditions are met. The idea is that smart contracts automate a workflow in which participants can be immediately certain of the outcome.

But wait, What is a *blockchain*? Basically it's like a database where the information is stored in a way that makes it difficult or almost impossible to change, hack, or cheat the system.

Some of the main features of a blockchain are:

1. **Immutability**. The blockchain cannot be altered.
2. **Distributed**. A copy of the blockchain is shared by many participants.
3. **Decentralized**. There is no government or organization that is responsible for deciding what happens in the blockchain.
4. **Secured**. Records are chained together with cryptographic hashes and authorised by digital signatures, which makes tampering detectable. Note this gives integrity, not confidentiality — data on a public blockchain is readable by anyone, so it is not encrypted.

## Variables

There are three types of variables in Solidity:

- **State** variables are those variables in Solidity that are permanently stored in a contract storage.
- **Local** variables are those present while a function is executing.
- **Global** variables are those that exist in the global namespace used to get information about the blockchain.

The types in Solidity are:

- **Booleans**: `bool`
- **Integers**: `int` or `uint`, signed and unsigned respectively. You can specify how many bits you want to use with `uint8` to `uint256`. Integers get initialized to zero.
- **Address**: `address`. An address represents a 20 byte value, which is the size of an Ethereum address. Address can have the `payable` modifier so it includes two additional operations `transfer` and `send`.
- **Byte Array**: `bytes`
- **Arrays**: `uint256[]`:
  - Declaration: the brackets go *before* the name, unlike C or Java — a fixed-size array `uint256[3] myArray = [1, 2, 3];` and a dynamic array `uint256[] myArray;`.
  - Add element: `myArray.push(1)` where `1` is a value.
  - Get element: `myArray[0]` where `0` is the index.
  - Get length: `myArray.length`.
  - Reset array: `myArray = new uint256[](0)`
- **Mapping**: `mapping` types define a key value structure. e.g. `mapping(uint => address) map`
- **Enum**: This is to enumerate a variable to have only some predefined values. e.g. `enum ROLES{ADMIN, USER, ENGINEER}`
- **Struct**: Structs are used to group a number of variables together. e.g. `struct struct_type { }`

**Access modifiers:**

- `public`: anyone can see what is stored in the variable. Global scope.
- `private`: only visible inside the contract.
- `internal`: only visible inside the contract and children contracts. The default modifier is `internal` when no modifier is specified.

You can declare a variable as `constant` and this means the variable cannot be assigned again. It saves *gas* because the value is stored directly into the bytecode of the contract. The convention is to use capital letters and underscores e.g. `MINIMUM_USD`.

An alternative to `constant` is `immutable`. In this case the variable can only be assigned once at construction time. It saves *gas*. The convention is to prefix the variable with `i_` e.g. `i_owner`.

## Contracts

A contract is like a class in Java and has the following structure:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.8;

contract MyContract {
    // content of the contract
}
```

As you can see we are including on top of the contract two things:

- A license identifier: `SPDX-License-Identifier`. It is used to indicate the license we are going to use for the contract, which is `MIT`.
- The Solidity version for the contract: `pragma solidity ^0.8.8;` The caret means any `0.8.x` from `0.8.8` up to but excluding `0.9.0`, i.e. `>=0.8.8 <0.9.0`. You could also specify a range like `pragma solidity >=0.8.8 <=0.8.10;`

>Use `//` or `/*   */` for commenting.

You can use the `public`  modifier to make variables inside the contract public.

## Constructors

Similar to Java you can also declare a constructor in Solidity.
- A constructor runs only once in the entire lifetime of the contract when it's created. 
- It's used to initialize contract state.
- A contract can have only one constructor.
- In case, no constructor is defined, a default constructor is present in the contract.

```solidity
contract MyContract {
    address public immutable i_owner;

    constructor() {
        i_owner = msg.sender;
    }
}
```

## Functions

Functions are declared like this:

```solidity
function myFunction() public pure returns (uint256) {
    return 1 + 1;
}
```

### Function Built-in Access Modifiers

- `public`: makes the function visible inside and outside the contract:

```solidity
function myFunction() public {
    // content of the function
}
```

- `external`: makes the function callable only from outside the contract (internally you would have to go through `this.myFunction()`, which is itself an external call):

```solidity
function myFunction() external {
    // content of the function
}
```

>What is the difference between `public` and `external`? The difference is that in public functions, Solidity immediately copies the function argument to memory, while external functions can read directly from `calldata`. Therefore, there is a gas cost implication. Memory allocation is expensive, whereas reading from `calldata` is cheap. As for best practices, you should use external if you expect that the function will only ever be called externally, and use public if you need to call the function internally.

- `internal`: This is to make the function protected, so only the contract where the function is defined and its children contracts can see the function:

```solidity
function myFunction() internal {
    // content of the function
}
```

- `private`: The function can only be called by other functions in the same smart contract.

### Pure/View/Payable Functions

- `view`: to make the function read-only, the function can access the state but can't make any modification:

```solidity
contract MyContract {
    uint x = 1;

    function myFunction(uint y) public view returns (uint) {
        return x + y; // it's reading x from outside the function
    }
}
```

- `pure`: ensures that the function cannot read or modify the state:

```solidity
contract MyContract {
    uint x = 1;

    function myFunction(uint i, uint j) public pure returns (uint) {
        return i + j; // it's not reading any variable from outside the function
    }
}
```

- `payable`: payable functions can change the state. For any function that will induce a transfer of assets, you must use the `payable` type to the function and address.

```solidity
receive() external payable {
    
}
```

>Use an underscore (`_`) in front of a function attribute to differentiate it from the ones declared on the contract scope:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.8;

contract MyContract {
    uint myNumber; // this get assigned to zero

    function myFunction(uint256 _myNumber) public {
        myNumber = _myNumber;
    }
}
```

### Custom Function Modifiers

You can define custom function modifiers within a *contract* like this:

```solidity
modifier onlyOwner {
    require(msg.sender == i_owner, "Sender is not owner!");
    _; // call the rest of the code in the function
}
```

As you can see above we first execute some code and then we use `_`. Underscore is to tell Solidity to execute the rest of the code in the function. You could invert the order and execute first the code in the function and then the code in the modifier. 

### Functions Data Location

Variables can be declared as `storage`, `memory` or `calldata` to explicitly specify the location of the data.

`memory` and `calldata` are used for temporarily storing variables and their values. You can only specify `memory` or `calldata` for array, struct, mapping or string types.

>What is the difference between `memory` and `calldata`? `calldata` is only valid for arguments of `external` functions and behaves mostly like `memory`. Any variable defined as `calldata` cannot be modified, whereas a variable defined as `memory` can be modified within the function. As a result `calldata` variables definition incur in less gas fees than `memory` variables.

```solidity
function myFunction(uint256 _myNumber) public {
    myNumber = _myNumber;
}
```

`storage` is used to defined variables that we want to write on the blockchain. These variables are persistent since they are written to the blockchain. `storage` variables can be accessed from anywhere inside the contract, and from outside it when their visibility allows. State variables are `storage` by default (the blockchain globals such as `msg` and `block` described below are not storage — they only live for the duration of the call). Variables defined as `storage` always will incur gas fee.

### Function Returning Values

Functions can return one or multiple values. If a function returns multiple values you can assign those values to variables like this:

```solidity
(uint80 roundID, int price, uint startedAt, uint timeStamp, uint80 answeredInRound) = priceFeed.latestRoundData();
```

You can also omit those values that you don't need:

```solidity
(, int price, , ,) = priceFeed.latestRoundData();
```

## For Loops and Conditional Statements

The `for` loop syntax is like the Java one:

```solidity
for(uint256 funderIndex = 0; funderIndex < funders.length; funderIndex++) {
    address funder = funders[funderIndex];
}
```

The `if`, `else if` and `else` statements are also inherited from languages like C or Java:

```solidity
uint a = 1; 
uint b = 2;
uint c = 3;
uint result;

if( a > b && a > c) {
    result = a;
} else if( b > a && b > c ){
    result = b;
} else {
    result = c;
}      
```

## Guard Functions

`assert()` and `require()` functions are guard functions that are used to improve readability.

You can use `require()` for checking a condition and in case the condition is not met throw an error with a message and revert the transaction. The first argument is the condition and the second argument is optional and the message.

```solidity
require(msg.value.getConversionRate(msg.value) >= MINIMUM_USD, "You need to spend more ETH!");
```

`assert()` works similarly to `require()`, but it's used for internal errors.

Before Solidity 0.8.0 the difference was about gas: `assert()` consumed all the remaining gas while `require()` refunded it. Since 0.8.0 `assert()` also compiles down to `revert` and refunds the unused gas, so the difference is now about intent and the error raised: a failing `assert()` produces a `Panic(uint256)` and signals a bug that should be impossible, whereas `require()` produces an `Error(string)` and validates inputs or external conditions. 

An improvement over `assert()` and `require()` would be to use `revert()`. You can declare an error and then call `revert()` with the error when a particular condition is not met. This is more gas efficient because a revert string has to be embedded in the contract bytecode and returned on failure, whereas a custom error is identified by a four byte selector.

```solidity
error NotOwner();

contract MyContract {
    address public immutable i_owner;

    constructor() {
        i_owner = msg.sender;
    }

    modifier onlyOwner() {
        if (msg.sender != i_owner) {
            revert NotOwner();
        }
        _;
    }
}
```

## Libraries

Libraries look similar to contracts but they are not contracts and the purpose is different. A library contains a set of reusable functions that can be called in your contract, so you keep your code organized, reusable and easy to understand.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

library MyLibrary {
    function getSum(uint256 value1, uint256 value2) internal pure returns (uint256) {
        return value1 + value2;
    }
}
```

To use a library within a smart contract, we use the syntax using `MyLibrary` for *Type*. We could use `*` instead of a specific type to attach functions from the library to all types.

```solidity
import "./MyLibrary.sol";

contract MyContract {
    using MyLibrary for uint256;

    uint256 a = 3;
    uint256 b = 5;

    function sum() public view returns (uint256) {
        return a.getSum(b);
    }
}
```

As you can see above, when you call a library function, the function receives the object they are called on as its first parameter, then you can pass the rest of the parameters as function arguments.

## Type Casting

Solidity is a statically typed language, but sometimes you might need to convert one type to another explicitly.

```solidity
value = uint256(msg.value);
```

## Funding and Withdrawing Ether

### Funding

You can fund your contracts by declaring functions with the access modifier `payable`. If you try to send ether to a function without a `payable` modifier, the transaction will fail.

Call the following function with some Ether to fund the contract where the function is defined.

``` solidity
function deposit() public payable {
}
```

### Withdrawing

There are three ways of sending funds to a particular address:

1. `transfer()`. There is a gas limit of 2300 and would throw an error if the transfer fails.

```solidity
payable(msg.sender).transfer(address(this).balance);
```

2. `send()`. There is a gas limit of 2300 and returns the status as a boolean.

```solidity
bool sendSuccess = payable(msg.sender).send(address(this).balance);
require(sendSuccess, "Send failed"); // revert if it fails
```

3. `call()`. This is the recommended way of transferring Ether and no 2300 gas stipend is imposed, so you can forward as much or as little as you choose. It returns the status as a boolean.

```solidity
(bool callSuccess, bytes memory data) = payable(msg.sender).call{ gas: 10000, value: address(this).balance }("");
require(callSuccess, "Call failed"); // revert if it fails
```

## Blockchain Specific Variables

You can fetch information from the blockchain itself and use that data in your functions. The most commonly used are `msg`, `block` and `tx`.


```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract MyContract {
    // other functions

    function getGasLimit() public view returns (uint256) {
        return block.gaslimit;
    }

    function getSender() public view returns (address) {
        return msg.sender;
    }
}
```

## Conclusion 

Learning about blockchain and Solidity are skills that can really widen your horizon, not just technically but career wise as well. There's still a lot of industries where blockchain hasn't reached its true potential and learning Solidity will give you an upper hand in the coming years.
