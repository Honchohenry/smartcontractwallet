# smartcontractwallet

// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;
contract SmartWallet {

    address payable owner;

    constructor(){
        owner = payable(msg.sender);
    }

    function transfer(address payable _to, uint _amount, bytes memory _payload) public returns(bytes memory) {
        require(msg.sender == owner, "You are not the owner, aborting");
        


        (bool success, bytes memory returnData) = _to.call{value:_amount}(_payload);
        return returnData;
    }
    receive() external payable {}  
}
