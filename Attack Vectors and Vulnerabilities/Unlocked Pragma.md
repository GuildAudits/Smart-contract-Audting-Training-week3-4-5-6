# Unlocked Pragma
This is a Best Practice to avoid errors, Contracts should be deployed using the same compiler version/flags with which they have been tested. Locking the pragma (for e.g. by not using ^ in pragma solidity 0.5.10) ensures that contracts do not accidentally get deployed using an older compiler version with unfixed bugs.

## Code Example
```
pragma solidity ^0.4.0;

contract PragmaNotLocked {
    uint public x = 1;
}

```
