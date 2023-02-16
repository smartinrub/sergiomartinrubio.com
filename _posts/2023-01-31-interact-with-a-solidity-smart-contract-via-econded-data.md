---
title: Interact with a Solidity Smart Contract via Encoded Data
image: https://lh3.googleusercontent.com/pw/AMWts8CMbytfVOrkw8apCc2DK1BoaEkw7hGIyQAp-lT_4L05FgFJ7Virq7xRMbKUId7x8teaxxsJ21_880XlfQM0rvKdt1VuriAO8389X-RTGFdrDL9MPPwaLbeiXOazr0d-y6OY5tvJQjvUIQ_JEG4IEx5Q=w2562-h1698-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Solidity

mermaid: false
layout: post
---

In most cases you are likely to [interact with a Solidity Smart contract through the human readable ABI (Application Binary Interface)](https://sergiomartinrubio.com/articles/deploy-your-first-smart-contract-with-ethersjs/#interacting-with-the-smart-contract) json file, which is the recommended way to interact with deployed contracts. However, you might not have the ABI of the deployed contract. What should I do then? Keep reading to know an alternative!

## Encoded Function Invocation

On a EVM (Ethereum Virtual Machine) level there is no functions and contracts, it's just bytes, and that's why Solidity Smart Contracts are encoded bytes. This means the EVM can understand encoded bytes of strings, so you can invoke a smart contract function by simply providing the encoded version of a function signature (e.g. `0x30c48a31`).

> The Solidity compiler generates [opcodes](https://sergiomartinrubio.com/articles/smart-contracts-advanced-development/#gas-cost-and-opcodes) from source code, encoding creates byte codes from [opcodes](https://sergiomartinrubio.com/articles/smart-contracts-advanced-development/#gas-cost-and-opcodes).

### Call Function

There are 3 options for invoking functions via their encoded version:

- `address.call(bytes)`: invokes code from a contract given an encoded version of the operation.
- [address.delegatecall(bytes)](https://solidity-by-example.org/delegatecall/){:target="_blank"}: same as `address.call(bytes)`, but changes are persisted on the caller smart contract and the callee's state is not modified. This is usually used for creating proxy contracts as a way of upgrading a smart contract by switching between different implementations. This requires to deploy the callee's contract first and storage slots must match between the caller and the callee, on in other words, the first storage variable in the caller contract will be used as the first storage variable in the callee contract, regardless of the variable name. However, you have to make sure that the types between the contracts match, otherwise you might encounter unexpected results.

In the example bellow when `hello` is invoked on the `A`, `sender` and `value` are set on `A`.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract B {
    address public sender;
    string public value;

    function world(string memory _value) public {
        value = _value;
        sender = msg.sender;
    }
}

contract A {
    address public sender; // will store the first store variable value from B
    string public value; // will store the second store variable value from B

    function hello(address _contract, string memory _value) public returns(bool, bytes memory) {
        (bool success, bytes memory data) = _contract.delegatecall(
            abi.encodeWithSignature("world(string)", _value)
        );
        return (success, data);
    }
}
```

On the other hand, if we replace `delegatecall` with `call` when we call `hello` on `A`, `sender` and `value` are set on `B`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract B {
    address public sender;
    string public value;

    function world(string memory _value) public {
        value = _value;
        sender = msg.sender;
    }
}

contract A {
    address public sender;
    string public value;

    function hello(address _contract, string memory _value) public returns(bool, bytes memory) {
        (bool success, bytes memory data) = _contract.call(
            abi.encodeWithSignature("world(string)", _value)
        );
        return (success, data);
    }
}
```

- `address.staticcall(bytes)`: same as `address.call(bytes)` with the only difference that the callee's function cannot modify the state.

The `hello` function on `A` will return `false` as the success value, since the `world` function on `B` is modifying `value` and `sender`.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract B {
    address public sender;
    string public value;

    function world(string memory _value) public {
        value = _value;
        sender = msg.sender;
    }
}

contract A {
    address public sender;
    string public value;

    function hello(address _contract, string memory _value) public view returns(bool, bytes memory) {
        (bool success, bytes memory data) = _contract.staticcall(
            abi.encodeWithSignature("world(string)", _value)
        );
        return (success, data);
    }
}
```

> You can use the `view` modifier when your function is using `staticcall`.

### Encoding Techniques

In the previous section we used the `abi.encodeWithSignature(string memory signature, ...)` to encode a smart contract function signature and its arguments, however there are [other encoding techniques available in Solidity](https://docs.soliditylang.org/en/develop/cheatsheet.html){:target="_blank"}.

- `abi.encode(...) returns (bytes memory)`. Encodes the provided argument and returns the encoded bytes. For example the function below will return the encoded bytes of the value `hello world`: `0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000b68656c6c6f20776f726c64000000000000000000000000000000000000000000`

```solidity
function encodeString() public pure returns(bytes memory) {
    bytes memory someString = abi.encode("hello world);
    return someString;
}
```

- `abi.encodePacked(...) returns (bytes memory)`. Also encodes the provided argument, but it creates a compact version by removing the leading and trailing zeros. For example the function below will return the packed encoding of `hello world`: `0x68656c6c6f20776f726c64`. [Packed encoded can be ambiguous in some scenarios](https://docs.soliditylang.org/en/v0.6.10/abi-spec.html#abi-packed-mode){:target="_blank"}.

```solidity
function encodePackedString() public pure returns(bytes memory) {
    bytes memory someString = abi.encodePacked("hello world");
    return someString;
}
```

- `abi.encodeWithSelector(functionPointer.selector, (...))`. On the example below you can see how `encodeWithSelector` is invoked with the byte representation of the function signature (`"setString(string)"`) and the function argument (`value`).

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract CallFunction {
    string public s_value;

    function setString(string memory _value) public {
        s_value = _value;
    }

    function callWithSelectorEncoder(string memory value) public returns(bytes4, bool) {
        (bool success, bytes memory returnData) = address(this).call(
            abi.encodeWithSelector(
                bytes4(keccak256(bytes("setString(string)"))), 
                value
            )
        );
        return(bytes4(returnData), success);
    }
}
```

- `abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)`. Same as `abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)`. Use when the signature is known and don't want to calculate the selector.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract CallFunction {
    string public s_value;

    function setString(string memory _value) public {
        s_value = _value;
    }

    function callWithSignatureEncoder(string memory value) public returns(bytes4, bool) {
        (bool success, bytes memory returnData) = address(this).call(
            abi.encodeWithSignature("setString(string)", value)
        );
        return(bytes4(returnData), success);
    }
}
```

For decoding we can use `abi.decode(bytes memory encodedData, (...)) returns (...)`, where you can provide on the second argument the type(s)

```solidity
function encodeString() public pure returns(bytes memory) {
    bytes memory someString = abi.encode("hello", "world");
    return someString;
}

function decodeString() public pure returns(string memory, string memory) {
    return abi.decode(encodeString(), (string, string));
}
```

## Strings

### Comparison

Comparing strings also require using their encoded format.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.12;

contract StringOperations {
    function compareString(string memory _otherString) public pure returns(bool) {
        return keccak256(abi.encodePacked("World")) == keccak256(abi.encodePacked(_otherString));
    }
}
```

but this does not compile:

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.12;

contract StringOperations {
    function compareString(string memory _otherString) public pure returns(bool) {
        return "World" == _otherString;
    }
}
```

### Concatenation

For versions of Solidity previous to `v0.8.12` you need to use the encoded format of the strings for concatenation.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract StringOperations {
    function concatenateStrings() public pure returns(string memory) {
        return string(abi.encodePacked("Hello ", "World"));
    }
}
```

As of Solidity `v0.8.12` or greater the `string.concat(a,b)` function is the preferred method.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.12;

contract StringOperations {
    function concatenateStrings() public pure returns(string memory) {
        return string.concat("Hello ", "World");
    }
}
```

## Conclusion

Encoding data in Solidity can be handy in some scenarios and provide more flexibility. We've seen how you can interact with strings or call smart contract functions through their encoded signature. Other languages like Java provide similar techniques like reflection.

However there are some downsides about interacting directly with the EVM (Ethereum Virtual Machine) via the `call` function:
- Types are not checked at compilation time. 
- Reverts are not bubbled up. You don't receive information about why an call was reverted, and you just get a `success=false`.
- Typos like functions existence are not checked at compilation time.

Photo by <a href="https://unsplash.com/@luk10?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Lukas Tennie</a> on <a href="https://unsplash.com/photos/DAWnMmUSMdU?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
  