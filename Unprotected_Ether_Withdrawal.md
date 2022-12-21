# Unprotected_Ether_Withdrawal

Due to `missing` or `insufficient access controls`, malicious parties can `withdraw` some or all Ether from the contract account.

This bug is sometimes caused by `unintentionally exposing initialization functions`. By wrongly naming a function intended to be a constructor, the constructor code ends up in the runtime byte code and can be called by anyone to re-initialize the contract.

  
    function withdraw(uint256 amount) public {
        msg.sender.transfer(amount);
        balances[msg.sender] -= amount;
    }
    

    // In an emergency the owner can migrate  allfunds to a different address.

    function migrateTo(address to) public {
        to.transfer(this.balance);
    }
    
    
In `withdraw()  function ` user can withdraw their money but there is no any restriction to withdraw ,  `Attacker` can pay in a small amount of Ether and call `withdraw()` repeatedly to empty the contract, if in this contract use older version of compiler then most possibility to happen `underflow` and attaker withdraw all fund.

In `migrate() function ` owner can migrate all contract fund to any address but this function is not protected , So anybody can `migrate` all fund to their account.



## Remediation

Implement controls so withdrawals can only be triggered by authorized parties or according to the specs of the smart contract system.


    function withdraw(uint256 amount) public {
        require(amount <= balances[msg.sender]);
        msg.sender.transfer(amount);
        balances[msg.sender] -= amount;
    }


    // In an emergency the owner can migrate  allfunds to a different address.

    function migrateTo(address to) public {
        require(creator == msg.sender);
        to.transfer(this.balance);
    }
 Now, Users can `withdraw `  User can withdraw Ether, but not more than they paid in and only `creator` of smart contract can `migrate` to all fund to diffrent address. so now all function are protected from attacker.   
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
