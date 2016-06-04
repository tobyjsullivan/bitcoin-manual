# An outdated guide to manually issuing a Bitcoin transaction to the network

**WARNING CAUTION BEWARE NOTICE** If you follow this method, and do not know 
the full implications of what you are doing (even if you think you do), you 
will definitely [lose all of your 
money](http://hackingdistributed.com/2016/04/29/bitcoins-137k-jackpot/). 
I _will not_ be held responsible for any reason. 
**Don't fuck around** with balances of any significance.

Most of this is just taken from http://bitcoinjs.org/.

## Creating a wallet

```sh
npm install bitcoinjs-lib
node
```

In the REPL:

```js
var bitcoin = require('bitcoinjs-lib')
var keyPair = bitcoin.ECPair.makeRandom()
console.log("Your private key is "+keyPair.toWIF())
console.log("Your public wallet address is " + keyPair.getAddress())
```

## Submitting a transaction 

Once coins have been loaded into a wallet, you can issue a transaction
to the network to move those coins to a different wallet.

**WARNING CAUTION BEWAAAAAAAARE** Any balances from input transactions 
which are not allocated to any outputs will be considered "fee" for 
processing the transaction. For example, if your input transaction sent
291 BTC to your wallet, and you decide to send 0.0001 BTC to another wallet
in a transaction but fail to account for the remaining balance with other
outputs, the entire remaining balance of 290.9999 BTC (~US$164,636.10 at time
of writing) will be lost permanently. Technically, it will be given to the
bitcoin miner that processes your transaction. 
When [this happens](http://hackingdistributed.com/2016/04/29/bitcoins-137k-jackpot/),
you done **fucked up**.

**Note:** For security reasons, it is recommended to move all coins out
of a wallet during any outbound transaction and never reuse the address.
You can do this by specifying multiple outputs in the transaction. The
first being the destination you want to send *some* coins to and the 
second output being a new wallet which will hold your balance.

In a node REPL:

```node
var bitcoin = require('bitcoinjs-lib')
var tx = new bitcoin.TransactionBuilder()

// You need to find the transaction ID that loaded coins into the source wallet.
// Transaction histories are effectively just singly-linked lists so this is
// the reference to the parent.
var txnId = '65cd547815ebe67da25aa6b44f0c82673a49909dcad1ce1acdcb2f4eb4ced45a'
// Additionally, any transaction can have multiple outputs so you need to
// specify the exact output which we are drawing from.
var txnIndex = 0

// You can add multiple inputs to aggregate coins from several wallets.
tx.addInput(txnId, txnIndex)

// You only need the public wallet address of the transfer destination.
// This means you can send coins to anybody without needing any knowledge of 
// their private key.
var destWallet = '1Gj1VK5yNwmYF2tVtQVpR3zBcjnb5oK9r4' 
// Feel free to send "thank you" coins to this address if you found this guide
// helpful.

// The exact amount to transfer is in Satoshis.
// A Satoshi is one hundred millionth of a single bitcoin (0.00000001 BTC).
var amount = 99000

// Add the output to the transaction.
// Again, you can add multiple outputs to a single transaction so long as
// the total of amounts to transfer does not exceed the cumulative balance 
// of all the inputs.
// WARNING: Again, be absolutely aware that any balance from the input 
// transactions which is not accounted for in any output is considered a
// fee by the Bitcoin network and will be lost permanently. Please don't
// fuck this up.
tx.addOutput(destWallet, amount)

// Initialize a private key. You will need one private key for each input.
// The private key must match the wallet address which was the target of 
// the input transaction-index pair.
var srcPrivateKeyWIF = 'Kz3Q8xBhaEefaztHKhNKAsYgU5f7KyVf7Mq97JLQCkFBTtxRA7Ta'
var keyPair = bitcoin.ECPair.fromWIF(srcPrivateKeyWIF)

// Sign the input of the transaction. 
// The input index identifies which input we are signing.
var inputIdx = 0
tx.sign(inputIdx, keyPair)

// Get the transaction serialized as hex
// NOTE: The transaction has not been submitted to the network yet
console.log(tx.build().toHex())
// > 01000000015ad4ceb44e2fcbcd1aced1ca9d90493a67820c4fb4a65aa27de6eb157854cd6
//   5000000006b483045022100c959d71157d9735d39b0206ce0f3a77dd639f4d234623afd27
//   3892bdae36505002201f141d7c526bf2790be4501579709af123e3b2768e6f4d2922f927d
//   49d75d882012103e91352e9a550480e2b9a96a5cf2948e5c84446054702b3de50e2c72520
//   cec0deffffffff01b8820100000000001976a914ac7bc3be8c65e6dd1d26718c6ea501576
//   bc40be088ac00000000

// To issue the transaction to the Bitcoin network, submit it over at
// https://blockchain.info/pushtx
// Note: transactions for amounts less than 0.01 BTC tend to require some fee.
// The minumum fee in this case, as best I can tell, is 1000 satoshis.
```

I followed this method to submit [this transaction](https://blockchain.info/tx/2abdca091859214e5df75990bd3bef043ddf0c0d824354d8581cf99d0217c55f)
to the Bitcoin network. For reference, the values above represent the actual values
used in the transaction.
I was required to include a fee of 1000 Satoshis due to the small amount involved.

