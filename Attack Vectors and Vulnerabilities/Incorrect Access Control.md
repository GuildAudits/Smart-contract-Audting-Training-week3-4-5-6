# Incorrect Access Control
Access control is a very important area in building smart contracts, when done wrong it can give attackers or users access to the funds in the smart contract. 
Contract functions executing critical logic and operations should have appropriate access control enforced via address checks (e.g. owner, controller etc.) typically in modifiers. Missing checks allow attackers to control critical logic.

## Code Example

function initContract() public {

	owner = msg.sender;
  
}
