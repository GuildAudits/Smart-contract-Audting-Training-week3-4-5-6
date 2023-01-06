# Constant state variables
This is a best practice when writing smart contracts, variables with similar names could be confused for each other and therefore should be avoided.

## Code Example
```
contract MyContract{
    uint 1_ether = 10000000000000000000; 
    uint one_ether = 1000000
    uint first_ether = 100000
}
```
