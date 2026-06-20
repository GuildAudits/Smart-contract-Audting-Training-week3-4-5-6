## Attack Vectors in Solidity #6:Unexpected Ether( Incorrect Use of this.balance)

## Introduction:

The use of the this.balance variable in Ethereum smart contracts can be a powerful tool for storing and accessing funds. However, if used incorrectly, it can also be a vulnerability that attackers can exploit. In this article, we will discuss the concept of unexpected ether, the vulnerability it presents with code examples, and preventative techniques that can be implemented to mitigate the risk.

## What is Unexpected Ether?

Unexpected ether refers to a situation where an attacker is able to send an unexpected amount of ether to a smart contract without the contract's intended logic being triggered. This can occur when the contract's code incorrectly uses the `this.balance` variable to determine the amount of ether that is available for a given action.

For example, consider the following contract that allows users to deposit and withdraw ether:

```solidity
pragma solidity ^0.5.0;

contract MyContract {
    function deposit() public payable {
        // code to deposit ether
    }
    
    function withdraw(uint256 _amount) public {
        if (this.balance >= _amount) {
            // code to withdraw ether
        }
    }
}
```

In this contract, the withdraw function uses the this.balance variable to determine if there is enough ether available for the withdrawal. However, if the contract's code does not properly handle the case where the `this.balance` variable is greater than the intended amount, an attacker could send a large amount of ether to the contract and then withdraw more than they deposited.

## Vulnerability

The vulnerability presented by unexpected ether is that it allows attackers to bypass the intended logic of a contract and gain access to funds that they should not be able to access. This can result in significant financial losses for the contract's owner and users.

Code Example:

To demonstrate the vulnerability of unexpected ether, consider the following scenario:

```solidity
pragma solidity ^0.5.0;

contract MyContract {
    function deposit() public payable {
        // code to deposit ether
    }
    
    function withdraw(uint256 _amount) public {
        if (this.balance >= _amount) {
            // code to withdraw ether
        }
    }
}

contract Attacker {
    function attack(address _contract) public payable {
        MyContract contract = MyContract(_contract);
        contract.deposit.value(1 ether)();
        contract.withdraw(2 ether);
    }
}
```

In this scenario, the Attacker contract sends 1 ether to the MyContract contract and then calls the withdraw function with an `_amount` of 2 ether. Since the `this.balance` variable is greater than 2 ether, the withdrawal is allowed to proceed, and the attacker is able to withdraw 2 ether, even though they only deposited 1 ether.

This scenario illustrates how an attacker can exploit the vulnerability of unexpected ether to gain access to more funds than they should be able to access.

## Preventative Techniques

There are several techniques that can be implemented to prevent unexpected ether attacks:

1. Use SafeMath library: One of the most effective preventative measures is to use the SafeMath library, which provides functions for performing arithmetic operations in a way that prevents overflows and underflows. By using `SafeMath`, a contract can ensure that any arithmetic operations involving this.balance are performed in a safe and secure manner.
    

For example, the MyContract contract could be modified to use the SafeMath library as follows:

```solidity
pragma solidity ^0.5.0;

import "https://github.com/OpenZeppelin/openzeppelin-solidity/contracts/math/SafeMath.sol";

contract MyContract {
    using SafeMath for uint256;
    
    function deposit() public payable {
        // code to deposit ether
    }
    
    function withdraw(uint256 _amount) public {
        require(this.balance.sub(_amount) >= 0, "Insufficient funds");
        // code to withdraw ether
    }
}
```

In this modified version of the contract, the withdraw function uses the `SafeMath` library's sub function to subtract the `_amount` from `this.balance` and ensure that the result is greater than or equal to 0. This prevents an attacker from being able to withdraw more than they deposited.

1. Use `require()` statements: Another preventative technique is to use `require()` statements to enforce conditions that must be met before certain actions can be taken. For example, a contract could use a `require()` statement to ensure that the this.balance variable is equal to or greater than the intended amount before allowing a withdrawal to proceed.
    

For example, the MyContract contract could be modified to use a `require()` statement as follows:

```solidity
pragma solidity ^0.5.0;

contract MyContract {
    function deposit() public payable {
        // code to deposit ether
    }
    
    function withdraw(uint256 _amount) public {
        require(this.balance >= _amount, "Insufficient funds");
        // code to withdraw ether
    }
}
```

In this modified version of the contract, the withdraw function uses a `require()` statement to ensure that the `this.balance` variable is equal to or greater than the `_amount` before allowing the withdrawal to proceed. This prevents an attacker from being able to withdraw more than they deposited.

1. Properly handle `this.balance`: It is important to carefully consider how the `this.balance` variable is used in a contract's code. If `this.balance` is used to determine the amount of ether available for a given action, it is important to properly handle the case where `this.balance` is greater than the intended amount.
    

For example, the MyContract contract could be modified to properly handle the this.balance variable as follows:

```solidity
pragma solidity ^0.5.0;

contract MyContract {
    mapping(address => uint256) public balances;
    
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }
    
    function withdraw(uint256 _amount) public {
        require(balances[msg.sender] >= _amount, "Insufficient funds");
        balances[msg.sender] -= _amount;
        // code to withdraw ether
    }
}
```

In this modified version of the contract, the `balances` mapping is used to track the amount of ether that each user has deposited. The withdraw function uses the `balances` mapping to determine if there is enough ether available for the withdrawal, rather than using the `this.balance` variable. This ensures that the contract's logic is properly triggered and prevents an attacker from being able to withdraw more than they deposited.

## Conclusion

Unexpected ether attacks can be a significant vulnerability in Ethereum smart contracts. By implementing preventative techniques such as using the `SafeMath` library and `require()` statements, and properly handling the `this.balance` variable, contract developers can mitigate the risk of unexpected ether attacks and protect the funds of their contracts and users.
