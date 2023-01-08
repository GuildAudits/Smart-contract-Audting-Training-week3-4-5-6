## Gas optimization with function types

Function types can be categorized into four categories: internal, external, public, and private functions.

1. Internal Functions: It can only be called inside the current contract, or by another contract that inherits from that contract.

2. External Functions: It can only be called from outside.

3. Public Functions: It can be called from anywhere.

4. Private Functions: It can only be called from inside the contract, It is not visible or accessible to external contracts or users.

*public functions* that are never called by the contract should be declared as external

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

contract Test {

    string name = "dev";

    //This functioncost 24269 gas
    function namePublic() public view returns(string memory){
        return name;
    }

    //This function cost 24247 gas
    function nameExternal() external view returns(string memory){
        return name;
    }
}
```
By using the right function type, we were able to save 22 gas.

