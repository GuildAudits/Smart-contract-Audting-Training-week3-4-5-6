# Gas optimization for Constant variable

State variables that are never going to change should be declared as constants to save gas.

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

contract Test {
    uint constant decimal = 18;
    
    //This function cost 21210 gas
    function getDecimal() public pure returns(uint){
        return decimal;
    }   
}


contract Test2 {
    uint decimal = 18;

    //This function cost 23310 gas
    function getDecimal() public view returns(uint){
        return decimal;
    }  
}
```
We saved 2100 gas because we save the variable with constant