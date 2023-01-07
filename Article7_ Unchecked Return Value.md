#  Unchecked Return Value vulnerability 

In Solidity an "unchecked return value" vulnerability is a sort of problem that happens when a function or external call returns  a value but the code invoking the function does not properly check or handle the returned value before using it. This can result to security vulnerabilities or contract issues.

# An example of a return value vulnerability
```
pragma solidity ^0.8.0;

contract Test {
    function test() public returns (bool) {
        return true;
    }
}
```


