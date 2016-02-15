---
layout: post
title:  "Launching Android Studio from the Terminal"
date:   2016-2-15 12:00:00
comments: true
categories: etherium
---

## 1) Install the Etherium Go client, Geth

From the terminal:


```
bash <(curl https://install-geth.ethereum.org -L)
```

## 2) `cd` to a new blank directory

`mkdir ~/etherium_test && cd ~/etherium_test`

## 3) Create a directory for the test blockchain

`mkdir .testchain`

## 4) Create a test `genesis.json` file

_genesis.json_
```
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


## 2) Run a private test net

`geth --genesis genesis.json --datadir .testchain --networkid 9876 --nodiscover --maxpeers 0 console`

Only clients on the same network id are allowed to connect to each other, so if you put in a unique network id, this creates a private testnet.

## Create an account from the geth console

`personal.newAccount("password")`

## Check that your account was created

`eth.accounts[0]` will print your account address, and is your default account.

## 3) Mine a couple blocks so you have some Ether to work with. Ether is the currency of Etherium. The difficulty is set very low in `genesis.json` to allow for quick mining.

`> miner.start(10)`

## 4) Stop your miner

`> miner.stop()`

## 5) Check your balance from the Geth console

```javascript
var account = eth.accounts[0]
eth.getBalance(account)
```

Now that you have Ether, you can follow the example on the etherium website to set up a [greeter contract][greeter-contract]

[greeter-contract]:https://www.ethereum.org/greeter


