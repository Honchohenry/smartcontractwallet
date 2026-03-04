# smartcontractwallet

// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;
contract SmartWallet {

    address payable owner;

    mapping(address => uint) public allowance;
    mapping(address => bool) public isAllowedToSend;

    constructor(){
        owner = payable(msg.sender);
    }

    function setAllowance(address _for, uint _amount) public {
        require(msg.sender == owner, "You are not the owner, aborting");
        allowance[msg.sender] = _amount;

        if(_amount > 0) {
            isAllowedToSend[_for] = true;
        }else {
            isAllowedToSend[_for] = false;
        }
    }

    function transfer(address payable _to, uint _amount, bytes memory _payload) public returns(bytes memory) {
        require(msg.sender == owner, "You are not the owner, aborting");
        if(msg.sender != owner) {
            require(isAllowedToSend[msg.sender], "You are not allowed to send anything from this smart contract, aborting");
            require(allowance[msg.sender] >= _amount, "You are allowed to send anything from this smart contract, aborting");
            allowance[msg.sender] -= _amount;
        }
        


        (bool success, bytes memory returnData) = _to.call{value:_amount}(_payload);
        require(success, "Aborting, call was not successful");
        return returnData;
    }
    receive() external payable {}  
}
