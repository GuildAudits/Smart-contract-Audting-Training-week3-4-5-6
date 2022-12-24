# Incorrect modifier

### Function Modifiers:

They can be used to change the behaviour of functions in a declarative way. For example, you can use a modifier to automatically check a condition prior to executing the function. The function’s control flow continues after the `_;` in the `preceding modifier`. Multiple modifiers are applied to a function by specifying them in a whitespace-separated list and are evaluated in the order presented. The modifier can choose `not to execute the function body` at all and in that case the `return variables are set to their default values` just as if the function had an empty body. The ` _; ` symbol can appear in the modifier multiple times. 

If a modifier does not `execute _;` or `revert`, the function using that modifier will return the `default value` causing `unexpected behavior`. 

    pragma solidity 0.8.10;

    contract test {

        // Assume other required functionality is correctly implemented

        mapping (uint256 => address) addresses;
        bool check;

        modifier onlyIf() {
            if (check) {
                _;
            }
        }

        function setAddress(uint id, address addr) public {
            addresses[id] = addr;
        }

        function getAddress(uint id) public onlyIf returns (address) {
            return addresses[id];
        }
    }

In `above contract` if we call `getAddress() ` function it will returns the `expected addresses` if check is `true` otherwise it will return `zero address` (default value) if check is `false`.


* Note: Be sure to use `require()`, `assert()`, and `revert()` as needed in your logic and to implement strong modifiers with `proper reasoning`.
