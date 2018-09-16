# Assume Ownership

## Challenge
Source: https://capturetheether.com/challenges/miscellaneous/assume-ownership/
>To complete this challenge, become the owner.
```
pragma solidity ^0.4.21;

contract AssumeOwnershipChallenge {
    address owner;
    bool public isComplete;

    function AssumeOwmershipChallenge() public {
        owner = msg.sender;
    }

    function authenticate() public {
        require(msg.sender == owner);

        isComplete = true;
    }
}
```

## Solution

The exploit in this challenge had to do with old Solidity syntax conventions which are no longer an issue in the current release. Solidity used to have a convention where the constructor function was denoted by the same name as the contract's name. In this case, the constructor function for the ```AssumeOwnershipChallenge``` contract is ```AssumeOwmershipChallenge```... except that there's a typo. I'm sure you can spot it. Normally, a constructor function is only called once at contract deployment. Given the typo, this function isn't recognized as a constructor function. Since it's a public function that means it can be called by anyone.

To solve this challenge, call the ```AssumeOwmershipChallenge``` function to re-assign ```owner```. Then call ```authenticate```.
