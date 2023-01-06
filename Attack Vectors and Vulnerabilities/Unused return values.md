# Unused return values
Unused return values of function calls are indicative of programmer errors which may have unexpected behavior.

## Code Example
```
contract MyConc{
    using SafeMath for uint;   
    function my_func(uint a, uint b) public{
        a.add(b);
    }
}
```
