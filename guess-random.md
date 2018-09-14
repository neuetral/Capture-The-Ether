# Guess the random number

## Required Concepts

The complexity of randomness is out of the scope of this solution, but a good starting point is to understand that human's are a horrible source of randomness. Producing adequate levels of randomness can be a tricky task. The following puzzle attempts to provide adequate entropy. Entropy is a source of randomness used to increase cryptographic performance and security.

The entropy provided by the challenge's transaction block number and transaction's block time stamp are poor sources of randomness given that all of this data is public on the Ethereum network.

To learn more about the anatomy of a block, as well as a transaction, I highly recommend the [Ethereum Beige Paper](https://github.com/chronaeon/beigepaper/blob/master/beigepaper.pdf). It's a plain English version of the Ethereum Yellow Paper. 

## Challenge

Source: https://capturetheether.com/challenges/lotteries/guess-the-random-number/

>This time the number is generated based on a couple fairly random sources.

```
pragma solidity ^0.4.21;

contract GuessTheRandomNumberChallenge {
    uint8 answer;

    function GuessTheRandomNumberChallenge() public payable {
        require(msg.value == 1 ether);
        answer = uint8(keccak256(block.blockhash(block.number - 1), now));
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

Using javascript, the web.js library, and an infura.io API key we can inspect the ropsten test net blockchain:

```
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
```

We could find the ```uint8 guess``` value using javascript but for some practice here's a little bit of Solidity that does the same thing:

```
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
```

If you noticed that ```blockNow``` was hashed as ```bytes32``` in the example above while the challenge contract hashed ```now``` as a ```uint``` (the default ```now``` value type), the Solidity contract below demonstrates that they will both return the same result:

```
contract returnTest {
    function verification() public view returns(uint8[2]) {
        uint8 answer = uint8(keccak256(blockhash(block.number - 1), now));
        uint8 answerVariant = uint8(keccak256(blockhash(block.number - 1), bytes32(now)));
        return [answer, answerVariant];
    }
}
```
