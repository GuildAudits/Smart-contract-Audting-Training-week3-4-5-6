# Attack Vectors in Solidity #10 Short Address Attack
### Introduction

A short address attack is a type of vulnerability that can occur in smart contracts written in Solidity, the programming language used for writing contracts on the Ethereum platform. This vulnerability arises when a contract uses a short address, or an address that has less than 20 bytes, as an input or output value. In this article, we will discuss the concept of short address attacks as an attack vector in Solidity, provide code examples, and discuss potential solutions to this problem.

### What is a short address attack?

In Ethereum, every account has a unique address, which is a 20-byte value that is used to identify the account on the blockchain. When a contract receives an address as an input value, it is important that the address is a valid, 20-byte value. However, if a contract uses a short address, or an address that is less than 20 bytes, it may be vulnerable to a short address attack.

For example, consider the following Solidity code:

```solidity
function transfer(address _to, uint256 _value) public {
    require(_to.length == 20);
    _to.transfer(_value);
}
```

This contract function is intended to transfer a specified value from the contract to the provided address. To ensure that the provided address is a valid, 20-byte value, the contract checks the length of the address using the `length` property. However, the `length` property is not a reliable way to check the length of an address, as it returns the number of bytes in the address, not the number of bits. This means that the contract is still vulnerable to a short address attack if an attacker provides a short address that is less than 20 bytes but has a length of 20 bytes or more.

To demonstrate how a short address attack can occur, consider the following Solidity code:

```solidity
function transfer(address _to, uint256 _value) public {
    require(_to.length == 20);
    _to.transfer(_value);
}

function testAttack() public {
    address shortAddress = 0x01;
    uint256 value = 100;
    transfer(shortAddress, value);
}
```

In this example, the `transfer` function is the same as the one in the previous example. However, the contract also includes a `testAttack` function, which calls the `transfer` function with a short address and a value of 100. Because the `transfer` function checks the length of the address using the `length` property, it believes that the provided address is a valid, 20-byte value, and proceeds to execute the `transfer` function. However, because the address is actually a short address, the `transfer` function will fail and the value will not be transferred.

**Solutions**

To mitigate the risk of short address attacks in Solidity, there are several potential solutions that developers can consider. One option is to use the `bytes20` data type for addresses, which ensures that the address is a fixed-length, 20-byte value. For example:

```solidity
function transfer(bytes20 _to, uint256 _value) public {
    _to.transfer(_value);
}
```

Another option is to use the `isContract` function to check if the provided address is a contract address. This can help to ensure that the address is a valid, 20-byte value, as contract addresses are always 20 bytes. For example:

```solidity
function transfer(address _to, uint256 _value) public {
    require(_to.isContract());
    _to.transfer(_value);
}
```

### Conclusion

In conclusion, short address attacks can be a significant threat to smart contracts written in Solidity. By understanding the risk of using short addresses and implementing proper security measures, such as using the `bytes20` data type or the `isContract` function, developers can help protect their contracts from this type of attack.