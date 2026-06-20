# SOLIDITY ATTACK VECTORS #1 : Unprotected Function
The use of appropriate access control checks when writing a smart contract is critical to avoiding contract vulnerability. Smart contract functions that execute critical logic should have appropriate access control enforced through address checks (e.g., owner, controller, etc.), which are typically in modifiers.

Inadequate checks allow attackers to take control of critical logic. The functionality of a contract is typically accessed via its public or external functions. While insecure visibility settings provide attackers with easy access to a contract's private values or logic, access control bypasses can be more subtle.

## Code Example

A smart contract has an initialization function that should be called by only the contract deployer or owner which then grants the caller a DEFAULT_ADMIN_ROLE and a REVIEWER_ROLE to be able to perform specific actions on the contract like grant roles to account and withdraw the contract’s funds.

Unfortunately in the following code snippet, the contract's initialization function sets the caller of the function as its admin and reviewer which means that the function can be called by anyone even after it has already been called. Allowing anyone to become the admin of the contract, take its funds and be able to grant access roles to any account. However, the logic is detached from the contract's constructor, and it does not keep track of the fact that it has already been called.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract AccessControl{
    event GrantRole(bytes32 role, address account);
    event RevokeRole(bytes32 role, address account);

    address immutable tokenAddress;
    
    mapping(bytes32 => mapping(address => bool)) roles;
    mapping(address => uint256) balances;


    bytes32 public constant DEFAULT_ADMIN_ROLE =   keccak256(abi.encodePacked("ADMIN"));
    bytes32 public constant REVIEWER_ROLE = keccak256(abi.encodePacked("REVIEWER"));


    constructor(address _tokenAddress){
        tokenAddress = _tokenAddress;
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }

    //This function should be protected
    function initialize() public {
        //Anyone can call the init function 
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(REVIEWER_ROLE, msg.sender);
    }

    modifier onlyRole(bytes32 role){
        require(roles[role][msg.sender]);
        _;
    }

    function deposit() public payable{
        require(msg.value >= 1 ether, "Not enough ether");
        balances[msg.sender] += msg.value;
    }

    //This function should be protected
    function withdraw() public{
        payable(msg.sender).transfer(address(this).balance);
    }

    function _grantRole(bytes32 _role, address _account) internal{
        roles[_role][_account] = true;
    }

    function grantRole(bytes32 _role, address _account) external onlyRole(DEFAULT_ADMIN_ROLE){
        _grantRole(_role, _account);
        emit GrantRole(_role, _account);
    }

    function revokeRole(bytes32 role, address _account) external onlyRole(DEFAULT_ADMIN_ROLE){
        roles[role][_account] = false;
        emit RevokeRole(role, _account);
    }
}
```

## Preventive Measures

The initialize function should have a modifier that restricts the caller to only the DEFAULT_ADMIN_ROLE like this;

```
function initialize() public onlyRole(DEFAULT_ADMIN_ROLE) {
        //Anyone can call the init function 
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(REVIEWER_ROLE, msg.sender);
    }
```

The withdraw function should also be protected with a function modifier that restricts the caller to only the REVIEWER_ROLE to avoid stolen funds by attackers.

```
function withdraw() public onlyRole(REVIEWER_ROLE){
        payable(msg.sender).transfer(address(this).balance);
    }
```

## CONCLUSION

Access controls that are missing or insufficient allow attackers to control critical logic in a smart contract. The solution is to implement controls so that withdrawals can only be triggered by authorized parties or according to the specifications of the smart contract system.
<br/>
I believe you learned something new from this article. Watch out for my next article!