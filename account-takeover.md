# Account takeover

## Challenge
Source: https://capturetheether.com/challenges/accounts/account-takeover/
>To complete this challenge, send a transaction from the owner’s account.
```
pragma solidity ^0.4.21;

contract AccountTakeoverChallenge {
    address owner = 0x6B477781b0e68031109f21887e6B5afEAaEB002b;
    bool public isComplete;

    function authenticate() public {
        require(msg.sender == owner);

        isComplete = true;
    }
}
```

## Solution

This one took a lot of soul searching to figure out. [Nils Schneider](http://www.nilsschneider.net) was a huge help on this one. I referenced and transposed [his solution](http://www.nilsschneider.net/2013/01/28/recovering-bitcoin-private-keys.html) for a similar issue regarding weak bitcoin signatures.

Let's begin our journey by having a look at the [transaction history](https://ropsten.etherscan.io/txs?a=0x6b477781b0e68031109f21887e6b5afeaaeb002b&p=2) of the ```owner``` account on etherscan. Here's the address:
```
address owner = 0x6B477781b0e68031109f21887e6B5afEAaEB002b;
```

Using the weak bitcoin signatures case study as an example, we can go all the way to the first transaction sent by the ```owner``` externally owned account. Lucky for us, the first two transactions are both out-going transactions to the same address. This is our first clue. To dig a little deeper we will use this short javascript code that will provide us with the ```r```, ```s```, and transaction hash ```z``` values of the two transactions of interest:

```
const webThree = require('web3');
const ethTx = require('ethereumjs-tx');
const readline = require('readline');

// replace this value with the address of each transaction
let tx = '0xd79fc80e7b787802602f3317b7fe67765c14a7d40c3e0dcb266e63657f881396';

// input your infura api key here
let web3 = new webThree(new webThree.providers.HttpProvider(
    'https://ropsten.infura.io/APIKEY'
));

web3.eth.getTransaction(tx).then(
  function(transaction) {
    transaction['gasPrice'] = parseInt(transaction['gasPrice']);
    transaction['value'] = parseInt(transaction['value']);
    let msg = new ethTx(transaction);

    /*
    hash parameter includeSignature [Boolean]
    whether or not to inculde the signature
    (optional, default true) - we want false
    */
    let msgHash = msg.hash(false);
    msgHash = msgHash.toString('hex');
    console.log(
      'r: ' + transaction['r'] + "\n" +
      's: ' + transaction['s'] + "\n" +
      'z: 0x' + msgHash
    );
  }
);
```

output for transaction 1 @  ```0xd79fc80e7b787802602f3317b7fe67765c14a7d40c3e0dcb266e63657f881396```:
```
r: 0x69a726edfb4b802cbf267d5fd1dabcea39d3d7b4bf62b9eeaeba387606167166
s: 0x7724cedeb923f374bef4e05c97426a918123cc4fec7b07903839f12517e1b3c8
z: 0x350f3ee8007d817fbd7349c477507f923c4682b3e69bd1df5fbb93b39beb1e04
```

output for transaction 2 @ ```0x061bf0b4b5fdb64ac475795e9bc5a3978f985919ce6747ce2cfbbcaccaf51009```:
```
r: 0x69a726edfb4b802cbf267d5fd1dabcea39d3d7b4bf62b9eeaeba387606167166
s: 0x2bbd9c2a6285c2b43e728b17bda36a81653dd5f4612a2e0aefdb48043c5108de
z: 0x4f6a8370a435a27724bbc163419042d71b6dcbeb61c060cc6816cda93f57860c
```

Interestingly enough, the ```r``` value of each transaction is exactly the same.
This is good news for us, but bad news for the ```owner``` of our target account.

From here it's pretty straight forward. Since Ethereum shares a lot of similarities with bitcoin we can follow Nils' example. In this instance, Ethereum even uses the same prime number p as bitcoin.

To complete the rest of the puzzle and find the private key, we'll need [sage math](http://www.sagemath.org/). Download it, and then calculate the snippet below (side note: GF(p) is a gallois field of a prime number):
```
r = 0x69a726edfb4b802cbf267d5fd1dabcea39d3d7b4bf62b9eeaeba387606167166
s1 = 0x7724cedeb923f374bef4e05c97426a918123cc4fec7b07903839f12517e1b3c8
s2 = 0x2bbd9c2a6285c2b43e728b17bda36a81653dd5f4612a2e0aefdb48043c5108de
z1 = 0x350f3ee8007d817fbd7349c477507f923c4682b3e69bd1df5fbb93b39beb1e04
z2 = 0x4f6a8370a435a27724bbc163419042d71b6dcbeb61c060cc6816cda93f57860c
p = 0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141

K = GF(p)

K((z1*s2 - z2*s1)/(r*(s1-s2)))
```

sage output:
```
9329076073068204043669332202639454059668260830422379264204062814073008711324
```

Convert it to hex with python:
```
sage = 9329076073068204043669332202639454059668260830422379264204062814073008711324
hex(sage)
```
The python output is the ```owner```'s ```privateKey```:
```
0x614f5e36cd55ddab0947d1723693fef5456e5bee24738ba90bd33c0c6e68e269
```

Input this ```privateKey``` into MetaMask and call the ```authenticate``` function from that account.
The call will provide the correct msg.sender:
```
0x6B477781b0e68031109f21887e6B5afEAaEB002b
```
