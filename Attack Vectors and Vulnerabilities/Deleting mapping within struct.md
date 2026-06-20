# Deleting a mapping within a struct
In Smart Contracts deleting a struct that contains a mapping will not delete the mapping contents which may lead to unintended consequences.

## Code Example
```
    struct BalancesStruct{
        address owner;
        mapping(address => uint) balances;
    }
    mapping(address => BalancesStruct) public stackBalance;

    function remove() internal{
         delete stackBalance[msg.sender];
    }
```
