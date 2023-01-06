# Reentrancy Attack
A Reentrancy Attack withdraws funds from a smart contract until the funds are exausted, Untrusted external contract calls could callback leading to unexpected results such as multiple withdrawals or out-of-order events. Using check-effects-interactions pattern or reentrancy guards can be used to prevent this attack. 

## Code Example
```
contract ModifierEntrancy {

  mapping (address => uint) public tokenBalance;
  string constant name = "Nu Token";
  Bank bank;
  
  constructor() public{
      bank = new Bank();
  }

  //If a contract has a zero balance and supports the token give them some token
  function airDrop() hasNoBalance supportsToken  public{
    tokenBalance[msg.sender] += 20;
  }
  
  //Checks that the contract responds the way we want
  modifier supportsToken() {
    require(keccak256(abi.encodePacked("Nu Token")) == bank.supportsToken());
    _;
  }
  
  //Checks that the caller has a zero balance
  modifier hasNoBalance {
      require(tokenBalance[msg.sender] == 0);
      _;
  }
}

contract Bank{

    function supportsToken() external returns(bytes32) {
        return keccak256(abi.encodePacked("Nu Token"));
    }

}
```
