# Signature Malleability

The implementation of a cryptographic signature system in Ethereum contracts often assumes that the signature is unique, but signatures can be altered without the possession of the private key and still be valid and Valid signatures could be created by a malicious user to replay previously signed messages.

## how the `transaction malleability` attack works?

* suppose **Alice** sends `1 ETH` with expected `txid A`.
* but **Bob** modifies the transaction so the `new txid is B`.
*  The modified **transaction B** then gets added to the blockchain, which implicitly invalidates **A**.
*   Alice's client software keeps checking for `txid A`, but never sees it.
*   Alice's wallet software will **debit 1 ETH** from her account once the `modified transaction is confirmed`, since the modified transaction still sent **1 ETH** from her account. 
* But if Alice isn't paying close attention, she might eventually give up and think the `transaction failed for some reason`, and she could `retry the transaction`.
*  If she does retry the transaction, she'll send another `1 ETH to the same address`.
* `In essence, Bob has tricked Alice into double paying`.


## How Transaction Malleability Actually Works?

`Bob` can't change where `Alice's money comes from`, `where it goes`, or `how much is sent`. These parameters are all `cryptographically signed by Alice`, using her `private key`. Bob can really just change the actual txid shown to humans.

`Smart contracts` on Ethereum have access to the built-in `ECDSA signature verification algorithm` through the system method `ecrecover`.

* An `ECDSA signature` consists of pair of numbers `(r,s).`
* The `elliptic curve` itself has integer order `n`.
* if `(r,s)` is a `valid signature`, then  `(r,−s mod n)` is the `complementary signature` of original signature.
* Given a `signature (r,s)` it's possible to calculate the complementary signature without knowing the `ECDSA private keys`.
* And the `complementary signature` has a `different hash`, so using the complementary signature will result in a `new transaction Id`.
* So an attacker can change a `txid by broadcasting a variation of the transaction` that uses the `complementary ECDSA signature`.

## how to prevent this attack?

 both `signature values(signature and it's complement)` are calculated, but only the signature with the smaller `S-value` is `considered valid`. That is, the correct representation is the form with the `smaller unsigned integer representation`.


     function tryRecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) internal pure returns (address, RecoverError) {
     
            // EIP-2 still allows signature malleability for ecrecover(). Remove this possibility and make the signature
            // unique. Appendix F in the Ethereum Yellow paper (https://ethereum.github.io/yellowpaper/paper.pdf), defines
            // the valid range for s in (301): 0 < s < secp256k1n ÷ 2 + 1, and for v in (302): v ∈ {27, 28}. Most
            // signatures from current libraries generate a unique signature with an s-value in the lower half order.
            //
            // If your library generates malleable signatures, such as s-values in the upper range, calculate a new s-value
            // with 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 - s1 and flip v from 27 to 28 or
            // vice versa. If your library also generates signatures with 0/1 for v instead 27/28, add 27 to v to accept
            // these malleable signatures as well.
            
            
            if (uint256(s) > 0x7FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF5D576E7357A4501DDFE92F46681B20A0) {
                return (address(0), RecoverError.InvalidSignatureS);
            }

            // If the signature is valid (and not malleable), return the signer address
            
            
            address signer = ecrecover(hash, v, r, s);
            if (signer == address(0)) {
                return (address(0), RecoverError.InvalidSignature);
            }

            return (signer, RecoverError.NoError);
        }
