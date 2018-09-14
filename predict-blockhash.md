# Predict the block hash

## Challenge

Source: https://capturetheether.com/challenges/lotteries/predict-the-block-hash/

>Guessing an 8-bit number is apparently too easy. This time, you need to predict the entire 256-bit block hash for a future block.

```
pragma solidity ^0.4.21;

contract PredictTheBlockHashChallenge {
    address guesser;
    bytes32 guess;
    uint256 settlementBlockNumber;

    function PredictTheBlockHashChallenge() public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function lockInGuess(bytes32 hash) public payable {
        require(guesser == 0);
        require(msg.value == 1 ether);

        guesser = msg.sender;
        guess = hash;
        settlementBlockNumber = block.number + 1;
    }

    function settle() public {
        require(msg.sender == guesser);
        require(block.number > settlementBlockNumber);

        bytes32 answer = block.blockhash(settlementBlockNumber);

        guesser = 0;
        if (guess == answer) {
            msg.sender.transfer(2 ether);
        }
    }
}
```

## Solution

The exploit to this contract is, again, in its failure to use a proper source of entropy to sufficiently randomize the answer. The assumption is that the ```blockhash``` of every block within the Ethereum network is retrievable using Solidity. According to the [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) and the [Solidity docs](https://solidity.readthedocs.io/en/v0.4.24/units-and-global-variables.html#block-and-transaction-properties), this blockhash function information is only available for the last 256 most recent complete blocks. Any blocks older than this will return a value of zero in Solidity, or in our case a ```bytes32``` value of:
```
0x0000000000000000000000000000000000000000000000000000000000000000
```
The above 64 hex value will be the guess we lock in. The secret here is patience. To verify this assertion, we can use the following Solidity contract:

```
pragma solidity ^0.4.21;

// deploy this contract to ropsten to see the above assumptions hold true
contract PredictTheBlockHashChallenge {
    function myGuess() public view returns(bytes32){
        return blockhash(block.number - 257);
    }
}
```

Once the ```lockInGuess``` transaction is called. We must now wait for 256 blocks to be mined before we call ```settle```. We can observe the block count with the javascript below:

```
const webThree = require('web3');

// this uses an api key from infura.io to access the ropsten test network
let web3 = new webThree(new webThree.providers.HttpProvider(
   'https://ropsten.infura.io/APIKEY'
));

// transaction address of our 'lockInGuess' function call retrieved
// from the remix console
let challengeTx = '0x8091fa284ce483e1d0c134032a7a2621068cfbca551e8ca2d29e55b338a079f7';

web3.eth.getTransaction(challengeTx).then(
  function(challengeTx) {
    web3.eth.getBlockNumber().then(
      function(latestBlockNumber) {

        // Calculates the number of blocks that have been mined since
        // the block that includes our 'lockInGuess' transaction
        let mined = latestBlockNumber - challengeTx.blockNumber;

        console.log(mined);
      }
    )
  }
);
```

When the above value is greater than 256, we can now call the ```settle``` function and solve the challenge.
