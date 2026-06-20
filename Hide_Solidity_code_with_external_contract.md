# Hide Solidity code with an external contract

Any address can be cast into a particular contract in Solidity, even if the contract at the address is not the one being cast.


Let's see example-

1. Eve deploys `Trap contract`
2. Eve deploys `Foo` with the address of `Trap`





        contract Foo {
            Bar bar;

            constructor(address _bar) {
                bar = Bar(_bar);
            }

            function callBar() public {
                bar.log();
            }
        }

        contract Bar {
            event Log(string message);

            function log() public {
                emit Log("Bar was called");
            }
        }




Let's say Alice can see the code of `Foo` and `Bar` but not `trap` contract.
It is obvious to Alice that `Foo.callBar()` executes the code inside `Bar.log()`.


3. Alice calls `Foo.callBar()` after reading the code and judging that it is safe to call.




        // This code is hidden in a separate file
        contract Trap {
            event Log(string message);

            // fallback () external {
            //     emit Log("you are traped");
            // }

            // Actually we can execute the same exploit even if this function does
            // not exist by using the fallback
            function log() public {
                emit Log("Trap contract are called");
            }
        }
        
        
        
4. Although Alice expected `Bar.log()` to be execute, but `Trap.log()` was executed and Alice is trap in `Trap contract`


### Remediation:
* Initialize a new contract inside the `constructor`.
* Make the address of external contract public so that the code of the external contract can be reviewed.


      contract Foo {
          Bar public bar;

          constructor(address _bar) {
              bar=new Bar();
          }

          function callBar() public {
              bar.log();
          }
      }
