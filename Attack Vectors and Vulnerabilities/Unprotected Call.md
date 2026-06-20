# Unprotected Call
A common attack vector and means through which a contract can be exploited are unprotected calls. For example, A user/attacker can mistakenly/intentionally kill the contract. Protect access to such functions.

## Code Example
```
pragma solidity ^0.4.23;

contract SuicideMultiTxFeasible {
    uint256 private initialized = 0;
    uint256 public count = 1;

    function init() public {
        initialized = 1;
    }

    function run(uint256 input) {
        if (initialized == 0) {
            return;
        }

        selfdestruct(msg.sender);
    }
}

```
