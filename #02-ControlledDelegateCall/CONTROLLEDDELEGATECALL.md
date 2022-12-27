# **Attack Vectors in Solidity #2: Controlled DeligateCall**

### **Introduction**

Solidity is a programming language for writing smart contracts on the Ethereum platform. One of the features of Solidity is the ability to use the delegatecall function, which allows a contract to execute code from a different contract in its own context. While this can be useful for dynamically loading code at runtime, it can also pose a security risk if the code being executed is malicious. In this article, we will discuss the attack vector of controlled delegatecall and how it can be exploited to compromise the security of a contract.

### Controlled Delegatecall Attack

Controlled delegatecall is an attack vector that allows an attacker to execute arbitrary code in the context of another contract. This is done by calling the delegatecall function and passing in the address of a contract with malicious code as an argument.

Here is an example of a vulnerable controlled delegatecall:

```solidity
pragma solidity ^0.4.24;

contract Victim {
  function execute(address _contract) public {
    // Vulnerable to controlled delegatecall attack
    _contract.delegatecall(abi.encodeWithSignature("doAttack()"));
  }
}

contract Attacker {
  function doAttack() public {
    // Construct malicious code that transfers all of Victim's funds to Attacker
    bytes memory maliciousCode = "6080604052348015600f57600080fd5b5060358060206000396000f3006080604052600080fd00a165627a7a7230582089f4040e20d9813d1c3b3e2173dd24f744ee9d2d2c1d33ca10f59c4d8e4e4c380029";
    // Call execute function in Victim contract with malicious code
    Victim v = new Victim();
    v.execute(address(this));
  }
}
```

In this example, the Attacker contract calls the execute function in the Victim contract with the address of itself as an argument. The Victim contract then calls the delegatecall function and passes in the address of the Attacker contract and the function doAttack(). This allows the Attacker contract to execute its malicious code in the context of the Victim contract, potentially transferring all of the victim's funds to the Attacker.

To prevent this attack vector, we can use a whitelist of allowed contracts and only allow those contracts to be called using delegatecall. Here is an example of this solution:

```solidity
pragma solidity ^0.4.24;

contract Victim {
  address[] public whitelist;

  function execute(address _contract) public {
    require(_contract.call(abi.encodeWithSignature("isWhitelisted()")));
    _contract.delegatecall(abi.encodeWithSignature("doAttack()"));
  }

  function addToWhitelist(address _contract) public {
    require(whitelist.length == 0 || whitelist[whitelist.length - 1] != _contract);
    whitelist.push(_contract);
  }
}

contract Attacker {
  function doAttack() public {
    // Construct malicious code that transfers all of Victim's funds to Attacker
    bytes memory maliciousCode = "6080604052348015600f57600080fd5b5060358060206000396000f3006080604052600080fd00a165627a7a7230582089f4040e20d9813d1c3b3e2173dd24f744ee9d2d2c1d33ca10f59c4d8e4e4c380029";
    // Call execute function in Victim contract with malicious code
    Victim v = new Victim();
    v.execute(address(this));
  }

  function isWhitelisted() public view returns (bool) {
    return true;
  }
}
```

In this solution, the Attacker contract must first be added to the whitelist before it can be called using delegatecall. This prevents unauthorized contracts from being called and executing arbitrary code in the context of the Victim contract.

### Conclusion

***Controlled delegatecall*** is a feature in Solidity that allows a contract to execute code from another contract in its own context. While this can be useful, it can also be dangerous if the code being executed is malicious. To prevent attacks through controlled delegatecall, it is important to carefully review and test any code being executed and to implement a whitelist of allowed contracts. By following these best practices, we can ensure the security of our contracts and protect against potential vulnerabilities.

