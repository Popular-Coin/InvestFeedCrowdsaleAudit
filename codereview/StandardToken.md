# StandardToken

Source file [../contracts/StandardToken.sol](../contracts/StandardToken.sol)

<br />

<hr />

```javascript
// BK Ok
pragma solidity ^0.4.11;

// BK Next 2 Ok
import './ERC20.sol';
import './SafeMathLib.sol';


/**
 * Standard ERC20 token with Short Hand Attack and approve() race condition mitigation.
 *
 * Based on code by FirstBlood:
 * https://github.com/Firstbloodio/token/blob/master/smart_contract/FirstBloodToken.sol
 */
contract StandardToken is ERC20, SafeMathLib {
  /* Token supply got increased and a new owner received these tokens */
  // BK Ok
  event Minted(address receiver, uint amount);

  /* Actual balances of token holders */
  // BK Ok
  mapping(address => uint) balances;

  /* approve() allowances */
  // BK Ok
  mapping (address => mapping (address => uint)) allowed;

  // BK Ok - Returns true/false instead of throwing if cannot transfer
  function transfer(address _to, uint _value) returns (bool success) {
    // BK Ok - Account has balance to transfer
    if (balances[msg.sender] >= _value
        // BK Ok - Transferring something 
        && _value > 0 
        // BK Ok - Overflow check
        && balances[_to] + _value > balances[_to]
        ) {
      // BK Ok
      balances[msg.sender] = safeSub(balances[msg.sender],_value);
      // BK Ok
      balances[_to] = safeAdd(balances[_to],_value);
      // BK Ok - Log event
      Transfer(msg.sender, _to, _value);
      // BK Ok
      return true;
    }
    else{
      // BK Ok
      return false;
    }
    
  }

  // BK Ok - Returns true/false instead of throwing if cannot transfer
  function transferFrom(address _from, address _to, uint _value) returns (bool success) {
    // BK Ok
    uint _allowance = allowed[_from][msg.sender];

    // BK Ok - Account has balance to transfer
    if (balances[_from] >= _value   // From a/c has balance
        // BK Ok - Allowance greater or equal to the value being transferred
        && _allowance >= _value    // Transfer approved
        // BK Ok - Transferring something
        && _value > 0              // Non-zero transfer
        // BK Ok - Overflow check
        && balances[_to] + _value > balances[_to]  // Overflow check
        ){
    // BK Ok
    balances[_to] = safeAdd(balances[_to],_value);
    // BK Ok
    balances[_from] = safeSub(balances[_from],_value);
    // BK Ok
    allowed[_from][msg.sender] = safeSub(_allowance,_value);
    // BK Ok - Log event
    Transfer(_from, _to, _value);
    // BK Ok
    return true;
        }
    else {
      // BK Ok
      return false;
    }
  }

  // BK Ok
  function balanceOf(address _owner) constant returns (uint balance) {
    // BK Ok
    return balances[_owner];
  }

  // BK Ok
  function approve(address _spender, uint _value) returns (bool success) {

    // To change the approve amount you first have to reduce the addresses`
    //  allowance to zero by calling `approve(_spender, 0)` if it is not
    //  already 0 to mitigate the race condition described here:
    //  https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
    // BK NOTE - require(!((_value != 0) && (allowed[msg.sender][_spender] != 0)))
    //         - require((_value == 0) || (allowed[msg.sender][_spender] == 0))
    //         - Can only approve if new approval is 0 or current approval is 0
    // BK Ok
    require(!((_value != 0) && (allowed[msg.sender][_spender] != 0)));
    //if ((_value != 0) && (allowed[msg.sender][_spender] != 0)) throw;

    // BK Ok
    allowed[msg.sender][_spender] = _value;
    // BK Ok - Log event
    Approval(msg.sender, _spender, _value);
    // BK Ok
    return true;
  }

  // BK Ok
  function allowance(address _owner, address _spender) constant returns (uint remaining) {
    // BK Ok
    return allowed[_owner][_spender];
  }

} 
```