# Fifty Years

## Challenge
Source:https://capturetheether.com/challenges/math/fifty-years/
>This contract locks away ether. The initial ether is locked away until 50 years has passed, and subsequent contributions are locked until even later.
>
>All you have to do to complete this challenge is wait 50 years and withdraw the ether. If you’re not that patient, you’ll need to combine several techniques to hack this contract.
```
pragma solidity ^0.4.21;

contract FiftyYearsChallenge {
    struct Contribution {
        uint256 amount;
        uint256 unlockTimestamp;
    }
    Contribution[] queue;
    uint256 head;

    address owner;
    function FiftyYearsChallenge(address player) public payable {
        require(msg.value == 1 ether);

        owner = player;
        queue.push(Contribution(msg.value, now + 50 years));
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function upsert(uint256 index, uint256 timestamp) public payable {
        require(msg.sender == owner);

        if (index >= head && index < queue.length) {
            // Update existing contribution amount without updating timestamp.
            Contribution storage contribution = queue[index];
            contribution.amount += msg.value;
        } else {
            // Append a new contribution. Require that each contribution unlock
            // at least 1 day after the previous one.
            require(timestamp >= queue[queue.length - 1].unlockTimestamp + 1 days);

            contribution.amount = msg.value;
            contribution.unlockTimestamp = timestamp;
            queue.push(contribution);
        }
    }

    function withdraw(uint256 index) public {
        require(msg.sender == owner);
        require(now >= queue[index].unlockTimestamp);

        // Withdraw this and any earlier contributions.
        uint256 total = 0;
        for (uint256 i = head; i <= index; i++) {
            total += queue[i].amount;

            // Reclaim storage.
            delete queue[i];
        }

        // Move the head of the queue forward so we don't have to loop over
        // already-withdrawn contributions.
        head = index + 1;

        msg.sender.transfer(total);
    }
}
```

## Solution

Much like the ```Donation``` challenge, this challenge uses a struct type dynamic array. Because of the order in which the variables are declared, it is clear that the length of the array and the ```head``` variable could be manipulated if the smart contract uses these values as function arguments or if these values are altered indirectly through a function call.

Interestingly enough, this challenge requires very little actual Solidity smart contract programming. Instead, what it requires is a deeper understanding of Solidity's operations and how they can have unintended side effects if used in a series of unforeseen steps.

The solution begins with a closer look at the ```upsert``` function. This function provides the opportunity to manipulate the dynamic array. It also let's the contributor provide their own timestamp given that it's larger than the previous by atleast ```1 day```. What is ```1 day``` in Ethereum?
```
1 day = 86400 seconds
```
Take a closer look at the ```upsert``` function's require statement:
```
// Append a new contribution. Require that each contribution unlock
// at least 1 day after the previous one.
require(timestamp >= queue[queue.length - 1].unlockTimestamp + 1 days);
```
Rewriting this in simpler terms, we have:
```
require(a >= b + 86400)
```
Notice that ```timestamp``` is an unsigned 32 bit integer. Because this function let's us manipulate the index value and ```timestamp``` simultaneously, we can make two consecutive transactions that will overflow the timestamp and reset it to '0'. Remember:
```
uint256(2**256) == 0
```
Thus to reset our ```timestamp``` to zero we must first create a transaction that sets the next ```timestamp``` to:
```
(2**256) - 1 day
(2**256) - 86400 seconds
115792089237316195423570985008687907853269984665640564039457584007913129553536
```
The ```msg.value``` sent with our function call will have the ability to expand or contract the ```queue``` dynamic array's ```length```. To expand it we set the ```msg.value``` to any value greater than ```0```. To increment by 1, use ```1 wei``` not ```1 ether``` because passing a ```msg.value``` of ```1 ether``` will actually expand the array to ```1*10**18``` slots (which is far too many for our purposes). Therefore, ```msg.value``` alters the length of the array by changing the value found in slot ```1``` of contract storage. The ```queue.push()``` adds 1 more to ```queue.length``` after we reassign ```contribution.amount``` with ```msg.value```.

The solution requires three transactions...

### First tx:
msg.value: ```1 wei```  - this expands the array by 1, so that we don't overwrite ```queue[0]```. ```index == 1```, so that we by-pass the if statement, and go to the else statement. This happens because ```1``` is out of range of the array since it only has index ```0``` at this time:
```
upsert (1,"115792089237316195423570985008687907853269984665640564039457584007913129553536")
```
### Second tx:
msg.value: ```1 wei``` - this keeps the array at two values, so we don't overwrite anything slots. ```index == 2```, so that we by-pass the if statement again. We have to use two, because we've grown the length of the array by 1.
```
upsert (2,0)
```
### Third tx:
We index from 1 so that we empty both slots. The original, and the ```index==1``` that we created.
```
withdraw(1)
```
