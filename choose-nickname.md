# Choose a nickname

While the title to this challenge is pretty self-explanatory, the solution isn't as straight forward as it may seem. For this puzzle you have to pass an argument in the ```setNickname``` function call. But passing a ```string``` to register a nickname won't work, because the function's argument value type is ```bytes32```. In Solidity, ```bytes32``` is a fixed size array of 32 bytes in length; hex encoded. The reason a string can't be passed as an argument for this challenge is because in Solidity a ```string```, much like ```bytes```(note there is no number beside bytes), are actually not a value type, but are both dynamic arrays.

All variables, arguments, and return values in Solidity must declare a value type. You can read more about the various value types in Solidity here: https://solidity.readthedocs.io/en/v0.4.24/types.html

It's probably been a while since you really thought about bits, bytes, and hexadecimal values so let's do a quick review. A bit is a portmanteau for binary digit and can represent one of two states (on or off). It's the most basic unit in computing science and is generally displayed using a base 2 numeral system. Hexadecimal, on the other hand, is a base 16 numeral system. Hexadecimal values are more convenient and are easier to manage. We'll be using hexadecimal values a lot from here on out so it's best to familiarize yourself with them now. The relationships are defined below:

* 1 bit  = 1 ```(base 2)```
* 2 bits = 10 ```(base 2)```
* 4 bits = 100 ```(base 2)```
* 8 bits = 1000 ```(base 2)```

* 1 hexadecimal digit  = 4 bits
* 2 hexadecimal digits = 8 bits

* 1 byte = 8 bits
* 32 bytes = 64 hexadecimal digits

The programming languages (```Python```, ```Javascript```, and ```Solidity```) that we'll be using for these puzzles denote hexadecimal values with an ```0x``` prefix.

Therefore:

* ```0xF``` in base 16 = ```15``` in base 10 = ```1111``` in base 2
* ```0xA``` in base 16 = ```10``` in base 10 = ```1010``` in base 2

To solve this puzzle, we'll use the web3.js library to convert our nickname string into a bytes32 padded fixed size array. Use node package manager to install the module into your desired directory by typing ```node npm web3``` into terminal.

Then convert your nickname with the javascript snippet below by substituting ```YourNickname``` with your desired nickname:

```
const web3 = require('web3');

/*
'YourNickname' example output:
0x596f75724e69636b6e616d65
*/
nickname = web3.utils.fromAscii('YourNickname');

/*
Padded 'YourNickname' example output:
0x596f75724e69636b6e616d650000000000000000000000000000000000000000
*/
nickname = web3.utils.padRight(nickname,64);

console.log(nickname);
```

Pass this 32 byte hex as the argument to the ```setNickname``` function. Be advised that all hex values must be enclosed in a set of quotation marks for the Remix IDE to consider it a valid argument. Using our example you would pass:

```
"0x596f75724e69636b6e616d650000000000000000000000000000000000000000"
```

This function argument would set your nickname to ```YourNickname```.
