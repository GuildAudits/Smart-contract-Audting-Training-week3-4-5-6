# Attack Vectors in Solidity #09: Bad randomness

### Introduction

Bad randomness, also known as the "nothing is secret" attack, is a type of vulnerability that can occur in smart contracts written in Solidity, the programming language used for writing contracts on the Ethereum platform. This vulnerability arises when a contract relies on randomness to generate a value or make a decision, but the source of the randomness is not truly random or can be predicted by an attacker. In this article, we will discuss the concept of bad randomness as an attack vector in Solidity, provide code examples, and discuss potential solutions to this problem.

### Vulnerability

One common example of bad randomness in Solidity is when a contract uses the block hash as a source of randomness. The block hash is a value that is derived from the contents of a block on the Ethereum blockchain, and it is considered to be unpredictable because it is based on the transactions and other data contained in the block. However, the block hash is not truly random because it is determined by the contents of the block, which may be known or predictable to an attacker. For example, consider the following Solidity code:

```solidity
function randomNumber() public view returns (uint) {
    return uint(keccak256(abi.encodePacked(block.difficulty))) % 10;
}
```

This contract function generates a random number between 0 and 9 by taking the keccak256 hash of the block difficulty and modulo 10. While the block difficulty may be unpredictable, it is still determined by the miners who are adding blocks to the blockchain, and an attacker may be able to influence the difficulty and predict the output of this function.

Another example of bad randomness in Solidity is when a contract uses the current block timestamp as a source of randomness. The block timestamp is the time at which a block was added to the blockchain, and it is considered to be unpredictable because it is determined by the miner who adds the block. However, the block timestamp is not truly random because it can be influenced by an attacker who controls a miner or has the ability to manipulate the time on their own machine. For example, consider the following Solidity code:

```solidity
function randomNumber() public view returns (uint) {
    return uint(keccak256(abi.encodePacked(block.timestamp))) % 10;
}
```

This contract function generates a random number between 0 and 9 by taking the keccak256 hash of the block timestamp and modulo 10. While the block timestamp may be unpredictable, it is still determined by the miner who adds the block, and an attacker may be able to influence the timestamp and predict the output of this function.

### Solution

To address the issue of bad randomness in Solidity, there are several potential solutions that developers can consider. One option is to use a hardware random number generator (RNG) to generate truly random values that cannot be predicted by an attacker. Another option is to use a decentralized randomness beacon, such as Chainlink's VRF, which is a decentralized oracle service that provides secure and verifiable randomness to smart contracts.

### Conclusion

In conclusion, bad randomness is a significant attack vector in Solidity, as it can allow an attacker to predict or influence the output of contract functions that rely on randomness. To mitigate this risk, developers should use secure sources of randomness, such as hardware RNGs or decentralized randomness beacons, to ensure that their contracts are resistant to this type of attack.

