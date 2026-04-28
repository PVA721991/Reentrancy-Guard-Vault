# Reentrancy-Guard-Vault
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract SecureVault {
    mapping(address => uint256) public balances;
    
    error InsufficientFunds();
    error TransferFailed();

    event Deposited(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);

    // Simple reentrancy lock
    bool private locked;
    modifier nonReentrant() {
        if (locked) revert("Reentrancy detected");
        locked = true;
        _;
        locked = false;
    }

    function deposit() public payable {
        balances[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    function withdraw(uint256 amount) public nonReentrant {
        if (balances[msg.sender] < amount) revert InsufficientFunds();

        // Effects first
        balances[msg.sender] -= amount;

        // Interaction last
        (bool success, ) = msg.sender.call{value: amount}("");
        if (!success) revert TransferFailed();

        emit Withdrawn(msg.sender, amount);
    }
}
