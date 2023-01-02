
# Unchecked Call Return Value


The return value of a message `call` is not checked. Execution will `resume` even if the called `contract throws an exception`. If the call fails accidentally or an attacker forces the call to fail, this may cause `unexpected behaviour in the subsequent program logic`.



    contract MyConc{
        function my_func(address payable dst) public payable{
            dst.call.value(msg.value)("");
        }
    }
    
    
The return value of the `low-level call` is not checked, so if the call fails, the Ether will be locked in the `contract`. If the `low level` is used to prevent `blocking operations`, `consider logging failed calls`.

## Recommendation

Ensure that the return value of a low-level call is checked or logged.

    contract MyConc{
          function my_func(address payable dst) public payable{
              (bool success, )=dst.call.value(msg.value)("");
              require(success, "transaction failed");
          }
      }
    
