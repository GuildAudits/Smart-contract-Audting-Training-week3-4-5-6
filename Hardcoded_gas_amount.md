# Message call with hardcoded gas amount

The `transfer()` and `send()` functions forward a fixed amount of `2300 gas`. Historically, it has often been recommended to use these functions for value transfers to guard against `reentrancy attacks`. However, the gas cost of `EVM instructions` may change significantly during `hard forks` which may break already deployed contract systems that make fixed assumptions about gas costs. 


####  `EIP 1884` broke several existing smart contracts due to a cost increase of sevral instruction.

* The `SLOAD (0x54)` operation changes from `200 to 800 gas`.
* The `BALANCE (0x31)` operation changes from `400 to 700 gas`.
* The `EXTCODEHASH (0x3F)` operation changes from `400 to 700` gas.
* A new opcode, `SELFBALANCE` is introduced at `0x47`.
* `SELFBALANCE` is priced as GasFastStep, at `5 gas`.

for more info reach out [EIP-1884](https://eips.ethereum.org/EIPS/eip-1884#sload) and [this](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/)


some smart contract broke because their `fallback functions` used to consume less than `2300 gas`, and they’ll now consume more. The amount of gas a contract’s `fallback function` receives if it’s called via Solidity’s `transfer()` or `send()` methods is `2300`.


    contract Vulnerable {
        function withdraw(uint256 amount) external {
            // This forwards 2300 gas, which may not be enough if the recipient
            // is a contract and gas costs change.
            msg.sender.transfer(amount);
        }
    }

    contract Fixed {
        function withdraw(uint256 amount) external {
            // This forwards all available gas. Be sure to check the return value!
            (bool success, ) = msg.sender.call.value(amount)("");
            require(success, "Transfer failed.");
        }
    }
