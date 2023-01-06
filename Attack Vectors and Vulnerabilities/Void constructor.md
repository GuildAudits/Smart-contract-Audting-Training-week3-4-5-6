# Void Constructor
This occurs when calls to base contract constructors that are unimplemented leads to misplaced assumptions. To avoid this issue, check if the constructor is implemented or remove call if not.

## Code Example
```
contract A{}
contract B is A{
    constructor() public A(){}
}
```
