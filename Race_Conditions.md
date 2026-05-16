## Race Conditions

Imagine in a car racing situation where each race car has a monitoring device embedded to monitor and control certain functionalities, and during the competition, a car owner that is losing the race suddenly decides to hack the monitoring device of all other cars in other to give his a speed advantage thereby wining the race. this is similar idea to front running: a system in which transactions are ordered so that a later transaction can be sent ahead of others.

#### Scenario

Imagine a contract that provides a problem where users have to find the value that corresponds to a given hash, and the winner wins lets say 10 eth. Alice solves the challenge and sends the transaction, within that time, Bob sees alice solution from the transaction pool and using the answer, sends his own transaction with a higher gas fee. because both transactions lands in the mempool of miners, the miners are conditioned to add the transaction with a higher gas fee into a block thereby processing it first. so given that bob provided a higher gas fee, bob now has the opportunity of winning the challenge thereby getting the price and Alice is left cheated.

```
pragma  solidity 0.6.10;

contract FindThisHash{
    bytes32 constant public hash = 0x5678ddddd555500000333333344444555eeddrrr2133bbbrfty;

    constructor() public payable{}

    function solve(string memory solution) public {
        require(hash == keccak256(abi.encodePacked(solution)), "incorrect answer");

        (bool sent,) = msg.sender.call{value: address(this).balance}("");
        require(sent, "failed to send ether");
    }
}

```

#### mitigation

A common proposed way to help with this is to conceal transaction then reveal by sending another transaction after the transaction has been mined.

Another is to work on the maximum amount of gas that could be sent with a transaction thereby restricting bad players from sending transactions with higher gas values
