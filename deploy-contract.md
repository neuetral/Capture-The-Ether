# Deploy a Contract

## Required Concepts

Ethereum is a general purpose blockchain and 'Turing complete' singleton state machine first authored by Vitalik Buterin, Gavin Wood, and Joseph Lubin. That sentence is a lot to take in, so let's break it down. A blockchain is a decentralized, or distributed, immutable ledger of cryptographically secured blocks. A ledger, in accounting, is a double-entry method of book keeping that keeps track of every transaction performed by a target entity. Double-entry book-keeping is the act of journaling every debit or credit from one account to another. Each block on the Ethereum blockchain is a bundled set of validated transactions, or journal entries. These blocks are stacked on-top of each other, hence a chain. These transactions are processed by the Ethereum Virtual Machine, which validates each transaction according to a set of rules to achieve consensus amongst all participants. The Ethereum homepage can be found here: https://www.ethereum.org/

Ethereum is decentralized over a peer-to-peer network that connects and shares this ledger amongst its participants. Since it's immutable, each participant retains a copy of the entire ledger. That means every participant has a copy of every transaction that's ever happened on the blockchain. Participants are incentivized to verify the validity of each new transaction through a proof-of-work algorithm. This algorithm is a game theoretically sound process which allows for trustless interaction. Proof-of-work is a calculation that requires a considerable expenditure of computer processing power. This synchronized blockchain is also considered Ethereum's singleton state.

Turing completeness implies that a system is capable of being programmed to find any solution, given that it also has access to infinite time and resources. It's basically one giant open-source computer. That's what makes Ethereum so cool, and why it's so exciting. Ethereum's infrastructure allows for the deployment of executable programs, or smart contracts, onto the blockchain. These smart contracts are executed by the Ethereum Virtual Machine and have the ability to make state changes on the Ethereum blockchain given a human (or external to the blockchain) initiated interaction.

Ether is the cryptocurrency used within the Ethereum network. It is involved in every transaction on the distributed ledger. This is because Ether acts as a metering device to limit usage of the network. Each execution has a 'gas' cost. 'Gas' fees are expressed in Ether but 'gas' itself isn't Ether (more on this later). Participants on the Ethereum network are incentivized to participate because validating blocks for the blockchain will earn them Ether. This process is called mining.

Smart contracts are developed using a number of different nascent high-level programming languages, but the most commonly used language is called [Solidity](https://github.com/ethereum/solidity). All Capture the Ether challenges involve Solidity. Solidity was created by Gavin Wood, one of the creators of Ethereum. It is a procedural, compiled language that walks and talks a lot like C++ and Javascript. The Solidity docs can be found at: https://solidity.readthedocs.io/

Smart contract development doesn't follow the standard development life-cycle for a number of reasons. Once smart contract code is deployed onto the blockchain... it's there forever and it's going to do exactly what it's been programmed to do. This includes all of the unintended side-effects! These mistakes have already resulted in millions of dollars in losses.

Smart contract development is also about keeping the code as concise and computationally efficient as possible. Smart contracts should be concise for easier auditing. They should also be as computationally efficient as possible given that each computation costs real world value to execute.

This was the main reason why I chose Capture the Ether as my first introduction to smart contract development. By understanding anti-patterns and learning to audit smart contracts first, I feel better equipped to begin developing my own contracts with a conservative mindset that puts security and efficiency before anything else.

Ethereum clients are used to communicate over the Ethereum peer-to-peer network. Clients are open-source software that follow the specification outlined in [Ethereum's Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf). With Ethereum, you have the option of using a client that will run a full node, like [Parity](https://paritytech.io/), or a remote client by way of a web-browser plugin like [MetaMask](https://metamask.io/). A full node client allows for independent verification of all transactions and supports the decentralized mission of a blockchain network, while a remote client hands off that responsibility to someone else. Remote clients don't store a local copy of the blockchain or validate any new blocks. For this reason, they are less secure than running a full node and do not guarantee any form of anonymity.

For solving these types of puzzles or performing any entry level smart contract development, it isn't necessary to run your own full node. Other options are available to developers. These include running a local private Ethereum blockchain, or connecting to a test net using a remote client like MetaMask. A test net is an Ethereum blockchain that functions in the same way as the Main net, except the ether used in transactions has no value.

Yes, you read that right. Test net contract deployment and function execution on test nets is free. Make as many mistakes as you want. Free use of the test net implies using it within reason. Test nets have been abused in that past and that just ruins it for everyone. That's why most test nets have public faucets that let participants 'drip' a limited number of test ether over a specified time frame. This ether is used in the exact same way you'd use it over the main net.

## Problem

Source: https://capturetheether.com/challenges/warmup/deploy/
```
pragma solidity ^0.4.21;

contract DeployChallenge {
    // This tells the CaptureTheFlag contract that the challenge is complete.
    function isComplete() public pure returns (bool) {
        return true;
    }
}
```

## Solution

We will need to install a client to get some test ether in order to complete this challenge. The challenge asks us to use the [MetaMask remote client](https://metamask.io/). MetaMask is a browser plugin that injects the web3.js javascript library into each page you visit. This library and its functions act as a bridge between your browser and the blockchain. MetaMask can connect to a bunch of different Ethereum networks, including the Ropsten test net needed for this challenge.
The web3.js docs can be found at: https://web3js.readthedocs.io/

MetaMask connects to Ropsten and other test nets using [Infura](https://infura.io/). You can download and install MetaMask [here](https://metamask.io/). I suggest installing the Chrome plug-in or the Brave plug-in. Once the plugin is installed, you will be prompted to create an account. You will be told to write down a mnemonic phrase that can be used to regenerate your public and private keys, should you need it in the future. Write this mnemonic phrase down a long with your password onto a piece of paper... twice... and don't share it with anyone! This method of storing information is called 'cold storage' and while it may be old school, it is a pretty secure method of storage. And I know you probably won't be using this account for any serious storage of value, but it's good to get into the habit of keeping your account information safe.

Now that you've put down your pencil, pick up the keyboard again, and let's get back to the challenge at hand. According to the instructions on the [Capture The Ether website](https://capturetheether.com/challenges/warmup/deploy/), this puzzle requires contract deployment to the ropsten testnet.
Open the MetaMask plug-in and look in the top right hand corner. You should see a drop down menu that shows your current Ethereum network connection. Switch from the main net to the ropsten test net. Then click 'Buy' to get some test ether. This will bring you to a web page with access to a faucet. Go ahead and grab some test ETH.

The reason we need some test ether is because deploying a contract to the Ethereum network requires an expenditure of gas from the participant deploying the contract. MetaMask prompts you to specify a gas limit and a gas price.

'Gas limit' is the maximum amount of gas the participant initiating the transaction is willing to buy. 'Gas price' is the maximum price a participant is willing to pay for the gas he's willing to use. The price is denoted in 'wei', the smallest denomination of the ether cryptocurrency (```1 wei = 1 / 10**18 ether```). Gas expenditures from Ethereum transactions are distributed to the miners who verify and mine the block. The more a participant is willing to pay per unit of gas, the sooner a miner will accept and verify the transaction. Gas limits are for metering the use of the Ethereum network. Because the Ethereum Network is Turing complete, and could run indefinitely, the gas system ensures that a bad actor cannot hang the network by way of an infinite loop or recursive function. Once a transaction is accepted, any unused gas is returned to the participant that sent the transaction. But beware, it's important to note that if a transaction runs out of gas, all states will be reverted but any gas expended up to that point will not be returned. These values are required inputs prior to executing any transaction on the Ethereum network. Gas costs fluctuate according to the activity of the network.

Now that you've got MetaMask installed, an account set up, and some test ether mined you should have enough of an understanding to be able to deploy the contract. When you start the challenge, MetaMask will open a pop-up window prompting you to input a gas limit and gas price prior to accepting the transaction.
