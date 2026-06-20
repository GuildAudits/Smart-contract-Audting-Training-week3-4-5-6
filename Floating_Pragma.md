# Floating Pragma

Contracts should be deployed with the same compiler version and flags that they have been tested the most with. Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors.


    // bad
    pragma solidity ^0.4.4;


    // good
    pragma solidity 0.4.4;
    
## Note:

A floating `pragma version (ie. ^0.4.25)` will compile fine with `0.4.26-nightly`., however nightly builds should never be used to compile code for production.


## Security Issues Pertaining to Floating Pragma

### 1. Shifting to an older compiler version

An older compiler version may contain disclosed public vulnerabilities of the latest security checks and fixed vulnerabilities. In the case of floating Pragma, shifting to an older one is a possibility because of the range of compilers compatible with the code. 

### 2. Using a very recent compiler version

As an older version can have missing security fixes, a new compiler version can be susceptible to undiscovered vulnerabilities. 

### 3. Use of multiple pragma versions across different files

Using different versions across different files can cause code inconsistency leading to unknown security issues.
