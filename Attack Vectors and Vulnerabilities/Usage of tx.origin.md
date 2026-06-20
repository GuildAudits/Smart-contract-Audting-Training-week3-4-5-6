# Incorrect usage of tx.origin
This is a best practice, in solidity the use of tx.origin for authorization may be abused by a MITM malicious contract forwarding calls from the legitimate user who interacts with it. Use msg.sender instead.

## Code Examples
```
pragma solidity 0.4.24;

contract MyContract {

    address owner;

    function MyContract() public {
        owner = msg.sender;
    }

    function sendTo(address receiver, uint amount) public {
        require(tx.origin == owner);
        receiver.transfer(amount);
    }

}
```
