## Front running

Front-running a smart contract transaction is an attack where an attacker intercepts a transaction before it is mined and executes a similar transaction with a higher gas price, thus ensuring that their transaction is mined first. This can occur when a contract does not properly handle transactions that are executed one after another in a short period of time.

An example of a contract that is vulnerable to front-running is a contract that allows users to place orders to buy or sell tokens at a specific price:

pragma solidity ^0.8.0; 
contract Vulnerable { 
    mapping(address => uint) public balances;
    mapping(address => uint) public orders; 
    function placeOrder(uint _price) public {
    require(balances[msg.sender] >= _price)
    balances[msg.sender] -= _price;
    orders[msg.sender] = _price; 
    } }
In this example, the contract's placeOrder function allows users to place orders to buy or sell tokens at a specific price, but does not have any mechanism to prevent front-running. An attacker can intercept a transaction, and place a similar order with a higher gas price, ensuring that their transaction is mined first and they can buy the tokens at a lower price.

To prevent front-running attacks, it's important to ensure that contracts properly handle transactions that are executed one after another in a short period of time. One way to do this is to use a Commit-Reveal mechanism. Here is an example of how to prevent front-running using this mechanism:

pragma solidity ^0.8.0; 
contract Safe { mapping(address => uint) public balances; 
mapping(address => uint) public orders;
mapping(address => bytes32) public commitments;
function placeOrder(bytes32 _commitment, uint _price) public { 
require(keccak256(abi.encodePacked(_price)) == _commitment);
require(balances[msg.sender] >= _price);
require(commitments[msg.sender] == 0); 
commitments[msg.sender] = _commitment; 
}

function reveal(uint _price) public { 
    require(keccak256(abi.encodePacked(_price)) == commitments[msg.sender]); 
    balances[msg.sender] -= _price; 
    orders[msg.sender] = _price; 
    commitments[msg.sender] = 0; } }

In this example, the contract's placeOrder function allows users to place orders to buy or sell tokens at a specific price but now, it requires a commitment (a hash of the price) and saves it, ensuring that the user is committed to a specific price without revealing it.

To reveal the price, the user must call the reveal function that will validate the commitment and execute the order. This way, an attacker cannot know the price of the order and cannot place a similar order with a higher gas price.
Another way to prevent front-running is to use a pre-commit mechanism, where the user sends two transactions, the first is a commitment transaction, and the second is a reveal transaction.
