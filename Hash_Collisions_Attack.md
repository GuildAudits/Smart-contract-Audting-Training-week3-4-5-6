# Hash Collisions With Multiple Variable Length Arguments



Using `abi.encodePacked()` with multiple variable length arguments can, in certain situations, lead to a `hash collision`. Since `abi.encodePacked()` packs all elements in order regardless of whether they're part of an array, you can move elements between arrays and, so long as all elements are in the same order, it will return the `same encoding`. In a signature verification situation, an attacker could exploit this by modifying the position of elements in a previous function call to effectively `bypass authorization`.






    contract AccessControl {
        mapping(address => bool) isAdmin;
        mapping(address => bool) isRegularUser;
        // Add admins and regular users.
        
        
        function addUsers(
            address[] calldata admins,
            address[] calldata regularUsers,
            bytes calldata signature
        )
            external
        {
            if (!isAdmin[msg.sender]) {
                // Allow calls to be relayed with an admin's signature.
                
                bytes32 hash = keccak256(abi.encodePacked(admins, regularUsers));
                
                address signer = hash.toEthSignedMessageHash().recover(signature);
                }
               }
              }
              
              
As we can see in the contract, if the `addUsers` function is called by an admin, `arrays of admins and regularUsers` are added to mappings of `isAdmin`. If the function is not called by an admin, it can be relayed with an admins signature.


## Vunlerability

*  The problem lies in the way that `abi.encodePacked()` manages its parameters. The following two statements return the same value, even though the parameters are unique.


        bytes32 hash = keccak256(abi.encodePacked([addr1, addr2,add3], [addr4, addr5,add6]));
        bytes32 hash = keccak256(abi.encodePacked([addr1, addr2, addr3,add4], [addr5,addr6]));



Given that different parameters can return the same value, an attacker could exploit this by modifying the position of elements in a previous function call to effectively bypass authorization. For example, 

if an attacker saw `addUser([addr1, addr2], [addr3, <attacker’s address>, addr4], sig)`,

they could call `addUser([addr1, addr2, addr3, <attacker’s address>], [addr4], sig)`. Since the return values are the same, the signature will still match, making the attacker `an admin. `


## prevent technique


* The first option is to not allow arrays as parameters, instead of passing a single value.
* Another option would be to only allow arrays of fixed lengths, so the positions cannot be modified. 

      function addUsers(
                  address[3] calldata admins,
                  address[4] calldata regularUsers,
                  bytes calldata signature
              )
                  external
              {
                  if (!isAdmin[msg.sender]) {
                      // Allow calls to be relayed with an admin's signature.

                      bytes32 hash = keccak256(abi.encodePacked(admins, regularUsers));

                      address signer = hash.toEthSignedMessageHash().recover(signature);
                      }
                     }


* To avoid this vulnerability by simply opting for the use of `abi.encode()` instead of `abi.encodePacked()`.


      function addUsers(
                  address[] calldata admins,
                  address[] calldata regularUsers,
                  bytes calldata signature
              )
                  external
              {
                  if (!isAdmin[msg.sender]) {
                      // Allow calls to be relayed with an admin's signature.

                      bytes32 hash = keccak256(abi.encode(admins, regularUsers));

                      address signer = hash.toEthSignedMessageHash().recover(signature);
                      }
                     }



## NOTES

`abi.encode` encode its parameters using the `ABI specs`. The ABI was designed to make calls to contracts. Parameters are padded to 32 bytes. If you are making calls to a contract you likely have to use `abi.encode`

`abi.encodePacked` encode its parameters using the minimal space required by the type. Encoding an `uint8 it will use 1 byte`. It is used when you want to save some space, and not calling a contract.

`abi.encodeWithSignature` same as encode but with the function signature as first parameter. Use when the signature is known and don't want to calculate the selector.

`abi.encodeWithSelector` same as encode but selector is the first parameter. It almost equal to encodeWithSignature.




