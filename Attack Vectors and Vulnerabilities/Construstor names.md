# Constructor names
In solidity before solc 0.4.22, constructor names had to be the same name as the contract class containing it. Misnaming it wouldn’t make it a constructor which has security implications. Solc 0.4.22 introduced the constructor keyword. Until solc 0.5.0, contracts could have both old-style and new-style constructor names with the first defined one taking precedence over the second if both existed, which also led to security issues.

# Code Example
```
contract A {
    uint x;
    constructor() public {
        x = 0;
    }
    function A() public {
        x = 1;
    }
    
    function test() public returns(uint) {
        return x;
    }
}
```
