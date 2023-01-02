

# DelegateCall or Storage Collision Attack on Smart Contracts

The `CALL` and `DELEGATECALL opcodes` are useful in allowing Ethereum developers to modularise their code. Standard external message calls to contracts are handled by the `CALL opcode` whereby code is run in the context of the external contract/function. The `DELEGATECALL opcode` is identical to the standard message call, except that the code executed at the targeted address is run in the context of the calling contract along with the fact that `msg.sender and msg.value remain unchanged`. 


It allows you to call the code of the callee contract from the caller with the storage context of the caller.

Let's start with some example contract

![image](https://substackcdn.com/image/fetch/w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fbd68e45d-6238-483a-bf8f-393db30ac39c_2650x1572.png)


if We call the 2 functions in Contract A, `setVarsDelegateCall`.

* let's assume , We will pass in the parameters `Contract B Address`, `a uint of 12` and a `Wei value of 1000000000000000000 (1 ETH)`.

### how to work delegate call in above contract?

1. An EOA address calls Contract A’s `setVarsDelegateCall` with `Contract B’s address`, uint `12` and `value 1000000000000000000 Wei`. This in turn makes a delegatecall to `Contract B’s setVars(uint256) function with uint 12`.

2. The `delegatecall` executes the `setVars(uint256)` code from Contract B but updates `Contract A’s storage`. The execution has the same storage, `msg.sender & msg.value` as its parent call `setVarsDelegateCall`.

3. The values are set in `Contract A’s storage`, `12 for num`, `0x5b38…c4 for sender (EOA Address)` & `1000000000000000000 for value`. Despite `setVars(uint256)` being called by Contract A with no value when we check `msg.sender` & `msg.value` we get the values from the original `setVarsDelegateCall`.

After the execution of this function we can check the `num, sender & value state` items of Contract A & B. We will see that none of the values are initialised in Contract B while all are `set in Contract A`


![image](https://user-images.githubusercontent.com/82324643/208247768-59eef9e3-1e2e-4a47-a2c4-76b14b8680b2.png)



Let's attacker deployed  `contract B` with different `storage variable at different slot` .

#### when change the order of storage variable of `Contract B`


![image](https://user-images.githubusercontent.com/82324643/208247777-c89fb1e5-2597-45ae-a0d7-a2682b885bff.png)

When we run `DELEGATECALL` the `“num”` value is going to be stored in `storage slot 2` for `Contract A` which maps to the `“value”` state variable. The same applies for when `“value”` is stored its going to update `slot 0` which maps to the `“num”` state variable.


## Remediation

Use `delegatecall` with caution and make sure to never call into `untrusted contracts`. If the `target address` is derived from user input ensure to check it against a whitelist of trusted contracts.

