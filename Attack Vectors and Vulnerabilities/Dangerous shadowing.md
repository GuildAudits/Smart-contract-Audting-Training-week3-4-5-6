# Dangerous shadowing
This is a best practice and occurs when local variables, state variables, functions, modifiers, or events with names that shadow (i.e override) built in solidity symbols, which can lead to several unexpected usages, errors and behaviour.

## Code Example
```
pragma solidity ^0.4.24;

contract Bug {
    uint now; // Overshadows current time stamp.

    function assert(bool condition) public {
        // Overshadows built-in symbol for providing assertions.
    }

    function get_next_expiration(uint earlier_time) private returns (uint) {
        return now + 259200; // References overshadowed timestamp.
    }
}
```
