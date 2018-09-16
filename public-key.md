# Public key

## Challenge
Source: https://capturetheether.com/challenges/accounts/public-key/
>Recall that an address is the last 20 bytes of the keccak-256 hash of the address’s public key.
>
>To complete this challenge, find the public key for the owner’s account.

```
pragma solidity ^0.4.21;

contract PublicKeyChallenge {
    address owner = 0x92b28647ae1f3264661f72fb2eb9625a89d88a31;
    bool public isComplete;

    function authenticate(bytes publicKey) public {
        require(address(keccak256(publicKey)) == owner);

        isComplete = true;
    }
}

```
## Solution

Using javascript:

```
const webThree = require('web3');
const ethTx = require('ethereumjs-tx');

let web3 = new webThree(new webThree.providers.HttpProvider(
    // input your infura api key here
    'https://ropsten.infura.io/APIKEY'
));

/*
to save ourselves the hassle of searching the entire blockchain for an external
out-going transaction by this account, we can take a shortcut and find it
using etherscan. The following link shows one outgoing transaction:
https://ropsten.etherscan.io/address/0x92b28647ae1f3264661f72fb2eb9625a89d88a31

Copy and paste the transaction(tx) hash into 'ownerTx' below
*/
let ownerTx = '0xabc467bedd1d17462fcc7942d0af7874d6f8bdefee2b299c9168a216d3ff0edb';

web3.eth.getTransaction(ownerTx).then(

  // promise returns our tx and passes it to the callback function below
  function(message){
    /*
    'ethereumjs-tx' stores the following 2 values as strings
    which caused a lot of headaches on my end before noticing. The reason they
    are strings is because they are stored as 'big numbers' which javascript
    isn't very good at processing on its own. Since the values in our tx aren't
    that big, we can convert them back to integers.
    */
    message['gasPrice'] = parseInt(message['gasPrice']);
    message['value'] = parseInt(message['value']);

    // create a transaction
    let tx = new ethTx(message);

    // retrieve public key from the tx hash, v, r, and s values
    let publicKey = tx.getSenderPublicKey()

    // convert to hex
    publicKey = publicKey.toString('hex');

    // spit out the public key to solve the challenge
    console.log('publicKey: ' + publicKey);

  }
);
```
