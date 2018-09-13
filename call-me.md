# Call me

## Required Concepts

Smart contracts are deterministic and immutable programs. They are not legal contracts and have nothing to do with machine learning. The term was first coined by [Nick Szabo](http://unenumerated.blogspot.com/) back in the nineties.

In the context of these challenges, smart contracts are written in the [Solidity](https://solidity.readthedocs.io/) programming language. As a compiled language, Solidity's solc compiler converts the programs into Ethereum Virtual Machine readable bytecode. Once the contract is converted and deployed, we are then able to use the smart contract's corresponding Application Binary Interface (ABI) to encode contract calls into machine code and then read out data from a transaction. ABI's define a smart contract's functions, corresponding arguments, and how to return the results.

The web based [Remix Integrated Development Environment (IDE)](https://github.com/ethereum/remix-ide) will be our one stop shop for Solidity smart contract development and interaction for these puzzles. You can access it here: https://remix.ethereum.org. Remix has resources like a source code editor, a debugger, several different run-time environments, and a compiler. In addition to these great features, under the 'run' tab, it also lets participants access and interact with deployed contracts via the contract's respective ABI. With that said, even though all contracts are visible to everyone that uses the Ethereum network, it doesn't mean that they are accessible. Without the ABI, or the smart contract's source code to reproduce the ABI, you generally won't be able to interact with a smart contract.

## Challenge

Source:https://capturetheether.com/challenges/warmup/call-me/
>To complete this challenge, all you need to do is call a function.
>
>The “Begin Challenge” button will deploy the following contract:
```
pragma solidity ^0.4.21;

contract CallMeChallenge {
    bool public isComplete = false;

    function callme() public {
        isComplete = true;
    }
}
```
>Call the function named callme and then click the “Check Solution” button.

## Solution

To complete this challenge, first deploy the contract. Once the transaction has been mined, copy the smart contract's source code into Remix. Compile the code and then select the 'Run' tab. Make sure the ```CallMeChallenge``` is loaded into the deployment selector and then copy and paste the challenge's address into the 'load contract from address' box. Click 'At Address'. If all went according to plan, you should see the ```CallMeChallenge``` under 'Deployed Contracts'. Call the ```callme``` function. This will prompt MetaMask to create a transaction. Complete the transaction to solve the puzzle.
