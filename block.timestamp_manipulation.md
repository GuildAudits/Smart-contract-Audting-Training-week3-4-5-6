## block.timestamp manipulation

because miners generally are the ones to add a transaction into a block, they have access to manipulate the block.timestamp value thereby gaining the system.

and if a lottery system or a winning sytem makes use of block.timestamp as a wining criteria, then the miner can time and add in a transaction at or around the wining time to beat the system

```
pragma solidity 0.6.10

contract Roulette{
    function spin() external payable{
        require(msg.value >= 1 ether);

        if (block.timestamp % 7 == 0){
            (bool sent,) = msg.sender.call{value: address(this).balance}("");
            require(sent, "failed to send");
        }
    }
}
```

in the game above, the miner playing this game will try to find a timestamp that is divisible by 7 and if lucky he gets to mine the transaction into the next block and add in that timestamp thereby winning all the money in the contract

#### Mitigation

as much as possible, when huge amount of money is involved, avoid using block.timestamp as a source of truth or as criteria for cashing out. because if the money is huge then attackers would feel properly incentivized to want to want to manipulate the value
