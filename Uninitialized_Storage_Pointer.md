# Uninitialized Storage Pointer

Local variables in solidity functions default to `storage` or `memory` depending on their type. `Uninitialized local storage variables` can point to unexpected 
storage locations in the contract, leading to `intentional or unintentional vulnerabilities`.


Let's consider the following, relatively simple name registrar contract:

    // A Locked Name Registrar
    1    contract NameRegistrar {
    2
    3       bool public unlocked = false;  // registrar locked, no name updates
    4
    5       struct NameRecord { // map hashes to addresses
    6           bytes32 name;
    7           address mappedAddress;
    8       }
    9        mapping(address => NameRecord) public registeredNameRecord; // records who registered names
    10        mapping(bytes32 => address) public resolve; // resolves hashes to addresses
    11
    12        function register(bytes32 _name, address _mappedAddress) public {
    13            // set up the new NameRecord
    14            NameRecord newRecord;
    15            newRecord.name = _name;
    16            newRecord.mappedAddress = _mappedAddress;
    17
    18            resolve[_name] = _mappedAddress;
    19           registeredNameRecord[msg.sender] = newRecord;
    20
    21           require(unlocked); // only allow registrations if contract is unlocked
    22        }
    23   }

    
    
This straightforward name `registrar(bytes32,_mappedAddress) function`  serves a single purpose. Anyone can register a name (as a bytes32 hash) and map it
to an address once the contract is `unlocked`. Regrettably, the require prevents  `register()`  from adding name records 
because this registrar is initially locked.
There is however a `vulnerability ` in this contract, that allows `name registration` regardless of the `unlocked variable`.
    
    
### To discuss this vulnerability, first we need to understand how storage works in Solidity.


* `Storage` only stores `variables`, not `constant`.
* Each slot can store up to `256 bits (32 bytes)`.
* If the size of the variable exceeds the `remaining size of the slot`, this variable will be passed to the `new slot`.
* `Struct` creates a `new slot`, the struct elements are put into the slot in the same way as above.
* `Fixed size array` creates a `new slot`, struct elements are put into the slot in the same way as above.
* `Dynamic size array` creates a new slot, this slot only stores the `length of the array`, while the values in the array will be stored at `other locations`.
* `Mapping` always creates a `new slot to hold a place`, the `values` in the array will be stored in other locations.
* `String` creates a `new slot`, this slot stores `both data & data length`.


Return back to our topic, thus storage variable in above contract store in sequential order

|slot number|storage variable|
|---|---|
|0|unlocked|
|1|registeredNameRecord|
|2|resolve|


Thus, `unlocked` exists in `slot 0`, `registeredNameRecord` exists in `slot 1` and `resolve` in `slot 2` etc. Each of these slots are of `byte size 32 
`(there are added complexities with mappings which we ignore for now). The boolean unlocked will look like  `0x000...0 (64 0's, excluding the 0x) 
for false` or `0x000...1(63 0's) for true`.

Therefore, `newRecord` on line [14] defaults to `storage`. The vulnerability is caused by the fact that `newRecord` is not initialised.
Because it defaults to storage, it becomes a `pointer to storage` and because it is `uninitialised`, `it points to slot 0` (i.e. where `unlocked is stored`). 
Notice that on lines [15] and [16] we then set `nameRecord.name to _name` and `nameRecord.mappedAddress to _mappedAddress`, this in effect changes
the `storage location of slot 0 and slot 1` which modifies both `unlocked and the storage slot associated with registeredNameRecord`.


## Preventative Techniques

Uninitialized storage variables are flagged as warnings by the Solidity compiler, so developers should take special care to heed these alerts 
when creating smart contracts. When working with complex types, it's best to use the `memory or storage` keywords explicitly to make sure they behave as you'd expect.

**`Since Solidity version 0.5.0, memory and storage usage are required`.**
