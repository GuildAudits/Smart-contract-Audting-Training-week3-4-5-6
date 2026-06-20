# Controlled delegate call

This vulnerability occurs when delegatecall() or callcode() to an address is controlled by the user and allows execution of malicious contracts in the context of the caller’s state. Ensure trusted destination addresses for such calls. 

## Code Example
```
pragma solidity ^0.4.24;

contract Proxy {

  address owner;

  constructor() public {
    owner = msg.sender;  
  }

  function forward(address callee, bytes _data) public {
    require(callee.delegatecall(_data));
  }

}
```
