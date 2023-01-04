# Locked Ether
Contracts that accept Ether via payable functions but without withdrawal mechanisms will lock up that Ether, to solve this problem remove payable attribute or add withdraw function

## Code Example
```
pragma solidity 0.4.24;
contract Locked{
    function receive() payable public{
    }
}
```
