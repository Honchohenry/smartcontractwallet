# smartcontractwallet

// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;
contract SmartContractWallet {

    address payable owner;
    mapping(address => uint) public allowance;
    mapping(address => bool) public isAllowedToSend;

    mapping(address => bool) public guardians;  
    address payable nextOwner;
    uint guardiansResetCount;
    uint public constant confirmationfromGuardiansForReset = 3 ;

    constructor(){
        owner = payable(msg.sender);
  }

  function proposeNewOwner(address payable _newOwner) public {
    require(guardians[msg.sender], "You are not guardian of this wallet, aborting");
    if(_newOwner != nextOwner) {
        nextOwner = _newOwner;
        guardiansResetCount = 0;
    }

    guardiansResetCount++;

    if(guardiansResetCount >= confirmationfromGuardiansForReset){
        owner = nextOwner;
        nextOwner = payable (address(0));
    }
  }

    

    function setGuardian(address _guardian, bool _isGuardian) public {
        require(msg.sender == owner, "You are not the owner, aborting");
        guardians[_guardian] = _isGuardian;
    }

    function setAllowance(address _for, uint _amount) public {
    require(msg.sender == owner, "You are not the owner, aborting");
    allowance[msg.sender] = _amount;

    if (_amount > 0) {
        isAllowedToSend[_for] = true;
    } else {
        isAllowedToSend[_for] = false;
    }
  }

  function transfer(address payable _to, uint _amount, bytes memory _payload) public returns(bytes memory) {
   // require(msg.sender == owner, "You are not the owner, aborting");
    if(msg.sender != owner) {
        require(isAllowedToSend[msg.sender], "You are not allowed to send anything from this smart contract contract, aborting");
        require (allowance[msg.sender]  >= _amount, "You are trying to send more than you are allowed to, aborting");

    }
    (bool success, bytes memory returnData) =_to.call{value:_amount}(_payload);
    require(success, "Aborting, call was successful");
    return returnData;
  }
  receive() external payable{}
}
