# Missing Inheritance
A contract might appear (based on name or functions implemented) to inherit from another interface or abstract contract without actually doing so.

## Code example
```
interface ISomething {
    function f1() external returns(uint);
}

contract Something {
    function f1() external returns(uint){
        return 42;
    }
}
```
