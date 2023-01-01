# Incorrect Modifier
The use of Incorrect Modifiers in a smart contract can be exploited, If a modifier does not execute _ or revert, the function using that modifier will return the default value causing unexpected behavior.

## Code Examples
```
 modidfier myModif(){
        if(..){
           _;
        }
    }
    function get() myModif returns(uint){

    }
```
