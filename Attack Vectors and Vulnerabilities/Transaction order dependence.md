# Transaction order dependence
In Ethereum race conditions can be forced on specific transactions by monitoring the mempool. The classic ERC20 approve() change can be front-run using thhis method for example, its advisable not to make assumptions about transaction order dependence.

## Code Example
```
pragma solidity ^0.4.16;

contract EthTxOrderDependenceMinimal {
    address public owner;
    bool public claimed;
    uint public reward;

    function EthTxOrderDependenceMinimal() public {
        owner = msg.sender;
    }

    function setReward() public payable {
        require (!claimed);

        require(msg.sender == owner);
        owner.transfer(reward);
        reward = msg.value;
    }

    function claimReward(uint256 submission) {
        require (!claimed);
        require(submission < 10);

        msg.sender.transfer(reward);
        claimed = true;
    }
}
```
