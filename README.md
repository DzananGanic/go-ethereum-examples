# go-ethereum-examples
This repository contains the examples of using go-ethereum as a library.
I have found that there is very little documentation on this topic, and assembling a list of snippets and examples of using go-ethereum as a library might come in handy, and save a lot of time.
The list of examples might expand in the future, and if you have useful snippets, feel free to contribute.

## Table of Contents:

- [Creating a Client](https://github.com/DzananGanic/go-ethereum-examples#creating-a-client)
- [Creating an Account](https://github.com/DzananGanic/go-ethereum-examples#creating-an-account)
- [Private Key From String](https://github.com/DzananGanic/go-ethereum-examples#private-key-from-string)
- [Hex String to Address](https://github.com/DzananGanic/go-ethereum-examples#hex-string-to-address)
- [Getting Nonce (Pending/Current)](https://github.com/DzananGanic/go-ethereum-examples#getting-nonce-pendingcurrent)
- [Suggest Gas Price](https://github.com/DzananGanic/go-ethereum-examples#suggest-gas-price)
- [Creating, Signing and Sending a Raw Transaction](https://github.com/DzananGanic/go-ethereum-examples#creating-signing-and-sending-a-raw-transaction)
- [Creating a Smart Contract Wrapper](https://github.com/DzananGanic/go-ethereum-examples#creating-a-smart-contract-wrapper)
- [Initializing a Smart Contract](https://github.com/DzananGanic/go-ethereum-examples#initializing-a-smart-contract)
- [Calling Smart Contract Methods](https://github.com/DzananGanic/go-ethereum-examples#calling-smart-contract-methods)
- [Waiting for a Transaction to be Mined](https://github.com/DzananGanic/go-ethereum-examples#waiting-for-a-transaction-to-be-mined)
- [Cross-Compiling a Project](https://github.com/DzananGanic/go-ethereum-examples#cross-compiling-a-project)


## Creating a Client


```go
conn, err := ethclient.Dial(ipc)
if err != nil {
  return nil, fmt.Errorf("Failed to connect to the Ethereum client: %v", err)
}
return conn, nil
```


## Creating an Account


```go
key, err := crypto.GenerateKey()
if err != nil {
  panic(err)
}

// obtaining string address and private key
address := crypto.PubkeyToAddress(key.PublicKey).Hex()
privKey := hex.EncodeToString(key.D.Bytes())
```

## Private Key From String

```go
privKey, err := key.FromString(strPrivKey)

// or 

privKey, err := crypto.HexToECDSA(strPrivKey)

if err != nil {
  panic(err)
}
```

## Hex String to Address

```go
address := common.HexToAddress(strAddress)
```

## Getting Nonce (Pending/Current)

```go
// NonceAt returns the account nonce of the given account.
// nonce, err := client.NonceAt(ctx, address, nil)

// This is the nonce that should be used for the next transaction.
nonce, err := client.PendingNonceAt(ctx, address)
if err != nil {
  panic(err)
}
```

## Suggest Gas Price

```go
price, err := client.SuggestGasPrice(ctx)
if err != nil {
  panic(err)
}
```

## Creating, Signing and Sending a Raw Transaction

```go
// Create new transaction
tx := types.NewTransaction(
  nonce,
  toAddress,
  amount,
  gasLimit,
  gasPrice,
  data,
)

// Sign the transaction with private key
signTx, err := types.SignTx(tx, types.HomesteadSigner{}, privateKey) // See other signers in transaction_signing.go file in go-ethereum project
if err != nil {
  panic(err)
}

// Send the transaction
err = client.SendTransaction(ctx, signTx)
if err != nil {
  panic(err)
}

// Obtain transaction hash as a string
strHash := signTx.Hash().String()
```

## Creating a Smart Contract Wrapper

First we need to obtain contract ABI and store it in a file (e.g. contract_abi.abi).
Make sure that this file contains only contract ABI as a json.

go-ethereum includes a source code generator that converts ABI definitions into easy to use Go packages.
We can build the generator with:

```
$ cd $GOPATH/src/github.com/ethereum/go-ethereum
$ godep go install ./cmd/abigen
```

To generate a wrapper, we use:
```
abigen --abi contract_abi.abi --pkg packagename --type MyContract --out contract.go
```

This will take ABI from contract_abi.abi file, and generate a wrapper called MyContract which belongs to packagename package and output it in contract.go file.

--abi and --pkg flags are mandatory, whereas --type and --out flags are optional.

## Initializing a Smart Contract

```go
cont, err := packagename.NewMyContract(contractAddress, client)
if err != nil {
  panic(err)
}
```

## Calling Smart Contract Methods

This is an example of ERC20 contract Transfer method
```go

// generate options
opts := bind.NewKeyedTransactor(privateKey)

tx, err := cont.Transfer(
  opts,
  toAddress,
  amount,
)
if err != nil {
  panic(err)
}
```

Note: There is an error I encountered while calling/signing lots of transactions in short time. Error stated "replacement transaction underpriced". I have found that it occurs because the new transaction is trying to replace the existing pending transaction. Setting "Nonce" field on opts to nil resolved my issue (Correct Nonce was then generated automatically).

## Waiting for a Transaction to be Mined

```go
receipt, err := bind.WaitMined(context, client, tx)
if err != nil {
  panic(err)
}
```

## Cross-Compiling a Project

While trying to cross-compile my project which used go-ethereum, I used [xgo](https://github.com/karalabe/xgo) cross compiler. I needed to compile it for linux amd64 architecture, for which I have used the following line 

```
xgo --deps=https://gmplib.org/download/gmp/gmp-6.0.0a.tar.bz2 --targets=linux/amd64 .
```

You can find more on cross-compiling with xgo [here](https://github.com/karalabe/xgo).
