---
layout: post
title:  "Getting Started With Etherium"
date:   2016-2-14 12:00:00
comments: true
categories: etherium
---

[Etherium][etherium-homepage] is a decentralized 'world computer' that lives on the Internet and allows secure execution of programs called 'smart contracts' on the Etherium blockchain. In order to deploy programs to the blockchain, you need Ether to pay for the resources you're consuming. You can generate Ether by 'mining', which is the process of participating in and securing the decentralized Etherium network. Once you have Ether, you can send it to other users using their Etherium address, or use it pay for running programs on the Etherium blockchain.

This guide will teach you how to set up a private testing blockchain, and how to mine your first Ether. The [getting started guide][eth-get-started] on the Etherium website didn't explain a few crucial points, such as how to get the `genesis.json` file. This is a quick guide to rectify that. See [here][etherium-intro] for a more detailed explanation of what Etherium is and why it's awesome.

## 1) Install the Etherium Go client, Geth

From the terminal:

```
bash <(curl https://install-geth.ethereum.org -L)
```

## 2) `cd` to a new blank directory

`mkdir ~/etherium_test && cd ~/etherium_test`

## 3) Create an empty directory for the test blockchain

This is so our testnet doesn't clobber the main Etherium blockchain

`mkdir .testchain`

## 4) Create a test `genesis.json` file
```json
{
	"nonce": "0xcafebabecafebabe",
	"timestamp": "0x0",
	"parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	"extraData": "0x0",
	"gasLimit": "0x8000000",
	"difficulty": "0x800",
	"mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	"coinbase": "0x3333333333333333333333333333333333333333",
	"alloc": {
	}
}

```


## 5) Run a private test net

`geth --genesis genesis.json --datadir .testchain --networkid 9876 --nodiscover --maxpeers 0 console`

You can set the network id to whatever you want. Only clients on the same network id are allowed to connect to each other, so if you put in a unique network id, this creates a private testnet. `--nodiscover` and `--maxpeers 0` make it so that no one can discover you on your test network if they happen to have the same id, and so that you won't connect to anyone else either. The difficulty is set very low so that we can mine quickly.

## 6) Create an account from the Geth console

`personal.newAccount("password")`

## 7) Check that your account was created

`eth.accounts[0]` will print your account address, and is your default account.

## 8) Mine some blocks for that sweet Ether

`miner.start(10)`

## 9) Stop your miner

`miner.stop()`

## 10) Check your balance from the Geth console

```javascript
var account = eth.accounts[0]
eth.getBalance(account)
```

Now that you have Ether, you can follow the example on the etherium website to set up a [greeter contract][greeter-contract]

[eth-get-started]:https://www.ethereum.org/cli
[greeter-contract]:https://www.ethereum.org/greeter
[etherium-intro]:https://ethereum.gitbooks.io/frontier-guide/content/ethereum.html
[etherium-homepage]:https://www.ethereum.org/


