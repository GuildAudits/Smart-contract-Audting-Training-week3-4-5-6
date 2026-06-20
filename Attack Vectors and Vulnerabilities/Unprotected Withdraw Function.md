# Unprotected withdraw function

Unprotected withdraw funcions can serve as an attack vector for hackers. For example, Unprotected(external/public) functions which send to or retrieve Ether from user-controlled addressed can allow users/attackers to withdraw unauthorised funds.

## Code Example

```
contract TokenSaleChallenge {

    mapping(address => uint256) public balanceOf;
    
    uint256 constant PRICE_PER_TOKEN = 1 ether;
    
    function TokenSaleChallenge(address _player) public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance < 1 ether;
    }

    function buy(uint256 numTokens) public payable {
        require(msg.value == numTokens * PRICE_PER_TOKEN);

        balanceOf[msg.sender] += numTokens;
    }

    function sell(uint256 numTokens) public {
        require(balanceOf[msg.sender] >= numTokens);

        balanceOf[msg.sender] -= numTokens;
        msg.sender.transfer(numTokens * PRICE_PER_TOKEN);
    }
}
```
