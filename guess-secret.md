# Guess the secret number

## Required Concepts

An unsigned integer ```uint``` value type in Solidity only includes zero and positive integers up to a predefined max value. ```uint``` is a short hand notation of ```uint256```. The suffix can be any 8 bit increment between ```uint8``` and ```uint256```. To determine the largest integer given a particular suffix:

```
uint256 = 2**256 - 1

uint8 = 2**8 - 1
```

This implies that the correct answer to this puzzle is a number between ```0``` and ```255```; inclusive.

The setup up for this puzzle is identical to the last, the only difference is that the passing condition validates the guess using a hashed ```bytes32``` representation of the correct answer as opposed to an integer value.

A hashed value is a value returned from a hash function. Hash functions are designed to map data of arbitrary size to data of a fixed size. The Ethereum protocol uses a cryptographic hash function called ```Keccak256```. It is almost identical to the ```SHA3``` hash function except for a few minor changes, but that which result in completely different hash values. In fact, the ```SHA3``` nomenclature is now deprecated in Ethereum as of [EIP-59](https://github.com/ethereum/EIPs/issues/59). For more on the history and complications of the standard, you can check out this resource: https://ethereum.stackexchange.com/questions/550/which-cryptographic-hash-function-does-ethereum-use

Ethereum's ```keccak256``` hash function is a one way hash function. This means that one can freely share a hashed value, because there's no way to reverse the encryption given just the hash. The issue with the puzzle at hand, is that we're given exploitable bits of information by way of the source file. Namely that the source file holds enough pieces of information to brute force our way to the ```answerHash```.

## Challenge

>Putting the answer in the code makes things a little too easy.
>
>This time I’ve only stored the hash of the number. Good luck reversing a cryptographic hash!

```
pragma solidity ^0.4.21;

contract GuessTheSecretNumberChallenge {
    bytes32 answerHash = 0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365;

    function GuessTheSecretNumberChallenge() public payable {
        require(msg.value == 1 ether);
    }
    
    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);

        if (keccak256(n) == answerHash) {
            msg.sender.transfer(2 ether);
        }
    }
}
```

## Solution

The first piece of the puzzle is the ```answerHash``` which we use as a validator. The second piece is the fact that we have a 1/255 chance of making a correct guess. The third piece of the puzzle is that this smart contract has locked in their guess at deployment, so it is a value that will not change from one guess to the next.

Using javascript, and the web3.js library, we can use a for loop to try every possible hash between 0 and 255. The only thing we have to do is make sure that all of our hashed guesses are converted to the ```uint8``` padded hex value of each number, not the base ten value.

```
const web3 = require('web3');
let answerHash = '0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365';
let guess;

for (let i=0; i<=256; i++) {

  // convert to padded base 16 value
  // padLeft([hex value], [8 bits = 2 hex values], [optional argument: default is to pad using zeros])
  guess = web3.utils.padLeft((i).toString(16), 2, 0);

  // include the 0x hex prefix
  guess = '0x'.concat(guess);

  // sha3 (keccak) hash function used by Solidity
  guess = web3.utils.soliditySha3(guess);

  /* above is a break down of this:
  guess = web3.utils.soliditySha3({type: 'uint8', value: i});
  */

  // if the hash is a match, log our guess to the console
  if (answerHash.match(guess)) {
    console.log('Guess number: ' + i);
    console.log('Guess Hash: ' + guess);
    break;
  }
}
```

The brute force result is ```170```. With this argument as our guess, we can solve the puzzle.
