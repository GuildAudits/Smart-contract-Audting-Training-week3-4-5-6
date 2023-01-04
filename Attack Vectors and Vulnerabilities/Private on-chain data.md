# Prvate on-chain data
Marking variables private does not mean that they cannot be read on-chain. Private data should not be stored unencrypted in contract code or state but instead stored encrypted or off-chain.

## Code Example
```
pragma solidity ^0.5.0;

contract OddEven {
    struct Player {
        address addr;
        uint number;
    }

    Player[2] private players;
    uint count = 0;

    function play(uint number) public payable {
            require(msg.value == 1 ether, 'msg.value must be 1 eth');
            players[count] = Player(msg.sender, number);
            count++;
            if (count == 2) selectWinner();
    }

    function selectWinner() private {
            uint n = players[0].number + players[1].number;
            (bool success, ) = players[n%2].addr.call.value(address(this).balance)("");
            require(success, 'transfer failed');
            delete players;
            count = 0;
    }
}
```
