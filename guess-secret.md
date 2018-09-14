# Guess the secret number

An unsigned integer (uint) value type in Solidity only includes zero and positive integers up to a predefined max value. 'uint' is a short hand notation of 'uint256'. The suffix can be any 8 bit increment between uint8 and uint256. To determine the largest integer given a particular suffix:

uint256 = 2**256 - 1

uint8 = 2**8 - 1

This implies that the correct answer to this puzzle is a number between 0 and 255; inclusive.

The setup up for this puzzle is identical to the last, the only difference is that the passing condition validates the guess using a hashed bytes32 representation of the correct answer as opposed to an integer value.

A hashed value is a value returned from a hash function. Hash functions are designed to map data of arbitrary size to data of a fixed size. The Ethereum protocol uses a cryptographic hash function called 'Keccak256'. It is almost identical to the SHA3 hash function except for a few minor changes, but that which result in completely different hash values. In fact, the SHA3 nomenclature is now deprecated in Ethereum as of EIP-59 (https://github.com/ethereum/EIPs/issues/59). For more on the history and complications of the standard, you can check out this resource: https://ethereum.stackexchange.com/questions/550/which-cryptographic-hash-function-does-ethereum-use

Ethereum's keccak256 hash function is a one way hash function. This means that one can freely share a hashed value, because there's no way to reverse the encryption given just the hash. The issue with the puzzle at hand, is that we're given exploitable bits of information by way of the source file. Namely that the source file holds enough pieces of information to brute force our way to the 'answerHash'.

The first piece of the puzzle is the 'answerHash' which we use as a validator.
The second piece is the fact that we have a 1/255 chance of making a correct guess. The third piece of the puzzle is that this smart contract has locked in their guess at deployment, so it is a value that will not change from one guess to the next.

Using javascript, and the web3.js library, we can use a for loop to try every possible hash between 0 and 255. The only thing we have to do is make sure that all of our hashed guesses are converted to the uint8 padded hex value of each number, not the base ten value.

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

The brute force result is '170'. With this argument as our guess, we can solve the puzzle.

Guess the random number

The complexity of randomness is out of the scope of this solution, but a good starting point is to understand that human's are a horrible source of randomness. Producing adequate levels of randomness can be a tricky task. The following puzzle attempts to provide adequate entropy. Entropy is a source of randomness used to increase cryptographic performance and security.

The entropy provided by the challenge's transaction block number and transaction's block time stamp are poor sources of 'randomness' given that all of this data is public on the Ethereum network.

To learn more about the anatomy of a block, as well as a transaction, I highly recommend the Ethereum Beige Paper (a plain English version of the Ethereum Yellow Paper): https://github.com/chronaeon/beigepaper/blob/master/beigepaper.pdf

Using javascript, the web.js library, and an infura.io API key we can inspect the ropsten test net blockchain:

const Web3 = require('web3');

// this uses an api key from infura.io to access the ropsten test network
let web3 = new Web3(new Web3.providers.HttpProvider(
    'https://ropsten.infura.io/APIKEY'
));

// change 'challengeTx' hex value to your deployed challenge address
let challengeTx = '0xbcbf9a307c7cdb80a824b6320888d08948cbde64fc8d3e2008716bdceacdec1e';

// Promise chain retrieves challenge block number and timestamp
web3.eth.getTransaction(challengeTx).then(

  function(challengeTx) {
    console.log('Challenge Block Number: ' + challengeTx.blockNumber);
    return challengeTx.blockNumber;
  }

).then(function(blockNumber){

  web3.eth.getBlock(blockNumber).then(

    function(block) {
      console.log('Challenge Time Stamp: ' + block.timestamp);
    }

  );
});

We could find the uint8 'guess' value using javascript but for some practice here's a little bit of Solidity that does the same thing:

pragma solidity ^0.4.24;

contract GuessTheRandomNumberChallenge {

    function guess(uint256 _blockNumber, uint256 _timeStamp) public view returns (uint8) {
        // This is the challenge transaction's block number
        uint blockNumber = _blockNumber - 1;
        // this is the challenge transaction's block time stamp
        bytes32 blockNow = _timeStamp;
        return uint8(keccak256(blockhash(blockNumber), blockNow));
    }

}

If you noticed that 'blockNow' was hashed as bytes32 in the example above while the challenge contract hashed 'now' as a uint (the default 'now' value type), the Solidity contract below demonstrates that they will both return the same result:

contract returnTest {
    function verification() public view returns(uint8[2]) {
        uint8 answer = uint8(keccak256(blockhash(block.number - 1), now));
        uint8 answerVariant = uint8(keccak256(blockhash(block.number - 1), bytes32(now)));
        return [answer, answerVariant];
    }
}
