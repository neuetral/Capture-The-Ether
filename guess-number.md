# Guess the number

## Challenge

Source: https://capturetheether.com/challenges/lotteries/guess-the-number/

>I’m thinking of a number. All you have to do is guess it.

```
pragma solidity ^0.4.21;

contract GuessTheNumberChallenge {
    uint8 answer = 42;

    function GuessTheNumberChallenge() public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);

        if (n == answer) {
            msg.sender.transfer(2 ether);
        }
    }
}
```

## Solution

Since Solidity smart contracts have an address, this means they have the ability to send, receive, and hold Ether just like any other account on the Ethereum Network. This puzzle introduces the concept of an Ether bearing account that requires Ether at contract deployment, and every time a participant makes a function call to the contract's ```guess``` function. When the correct guess is made, the contract will then transfer ```2 ether``` to the address that made the guess.

Let's break down this contract to its most basic elements. On line one, we have the version ```pragma```. This informs us of what compiler to use. Any compiler greater than the version listed(in our case ```0.4.21```) will be rejected by the source file. This is to ensure that the compiler does not compile our contract(s) using deprecated or incompatible changes that have been introduced in later versions. The ```^``` in front of the Solidity version denotes that we can use any version of Solidity from ```0.4.0``` up to, but not including ```0.5.0```.

Contracts in Solidity are easier to understand once you realize their similarities to classes in other object oriented programming languages. Refer to this list for a full list of declarations that can be found within Solidity contracts: https://solidity.readthedocs.io/en/v0.4.24/structure-of-a-contract.html?highlight=contract

The contract declares a state variable ```answer```. This variable is permanently stored in contract storage and can be used through the contract.

Line 4 defines the constructor function ```GuessTheNumberChallenge```. A constructor function is called once at contract deployment, and then can no longer be called after the function has been deployed. The convention of using the contract's name, ```GuessTheNumberChallenge``` to denote the constructor function has been deprecated in future versions of solidity. The current convention to declare a constructor function is to use ```constructor()``` in place of ```function GeussTheNumberChallenge()```.

This constructor function has two modifiers: ```public```, and ```payable```. ```public``` is the default visibility specifier that allows for external or internal function calls. The ```payable``` modifier indicates that this function can process ```ether```.

The ```require()``` function is used to check and make sure conditions are fulfilled. It will throw an exception if the check returns ```false```.

```msg.sender``` is a special variable that is a member of the ```msg``` variables which exist in the global namespace. It is slightly confusing at first, but ```msg.sender``` is the sender of the transaction (also called a message). This means ```msg.sender``` can be different depending on who makes the external function call. More information on global namespace variables can be found here:
https://solidity.readthedocs.io/en/develop/units-and-global-variables.html#block-and-transaction-properties

The next function ```isComplete``` defines and returns a boolean ```true``` value if the ether balance at ```address(this)```, which refers to the contract's own address, is equal to zero.

The last function ```guess(uint8 n)``` passes one argument. This argument must be an 8 bit unsigned integer. This integer is your guess. To make a guess, you have to include ```1 ether``` as payment along with your guess. If you guess right, the contract will transfer ```2 ether``` to your account. If you guess wrong, nothing else happens.

The exploit to this contract is revealed inside the conditions of the ```guess``` function's if statement:
```
if (n == answer)
```
Since we have the source file to this challenge, we can see that ```answer``` is equal to ```42```. To solve this puzzle, pass the value ```42``` as your ```n``` argument in the ```guess``` function.

*the number 42, is also in reference to [Hitchhiker's Guide to the Galaxy](https://www.youtube.com/watch?v=aboZctrHfK8).
