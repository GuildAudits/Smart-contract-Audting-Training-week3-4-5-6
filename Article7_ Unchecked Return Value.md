#  Unchecked Return Value vulnerability 

In Solidity an "unchecked return value" vulnerability is a sort of problem that happens when a function or external call returns  a value but the code invoking the function does not properly check or handle the returned value before using it. This can result to security vulnerabilities or contract issues.

# An example of a return value vulnerability
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GrantBank {

    function fee() public payable returns (bool) {
        if(msg.value > 1 ether){
            return true;
        } else{
            return false;
        }  
    }
}
```

If the contract invoking the fee() function does not properly check the returned value, it may cause vulnerability. For example:

```
pragma solidity ^0.8.0;

import "./GrantBank.sol"

contract Test {
    mapping(address => uint) public balance;

    GrantBank grantbank = new GrantBank();

    function callFee() public {
        // no-check for the return value of the fee
        grantbank.fee();

        balance[msg.sender] = 10;
    }
}
```
here, we didn't check for the return value of fee function so the balance of the msg.sender is increased despite not making payment.

## solution 
Always check for the return value of any function or external call if it passes/meets the criteria specified

an example:
```
pragma solidity ^0.8.0;

import "./GrantBank.sol"

contract Testwithcheck {

    mapping(address => uint) public balance;

    GrantBank grantbank = new GrantBank();

    function callFee() public{
       bool res =  grantbank.fee();

        require(res, "You are not eligible for grant");

        balance[msg.sender] = 10;

    }

}

```
here, we check if the fee payment was successful and we are able to know that the callFee function is reverting so the payment wasn't successfull and the balance isn't increase.
