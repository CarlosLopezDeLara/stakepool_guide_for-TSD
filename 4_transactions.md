# Create a simple transaction

In this tutorial we want to create and submit a simple transaction.
We assume that you have created an _paynmet key pair_ and the corresponding _payment address_ and that you have
some funds in your payment.addr

You will need a new __payment address__ (payment2.addr) so that you can send funds to it. Let's get a new payment key pair for that:

    cardano-cli shelley address key-gen \
    --verification-key-file payment2.vkey \
    --signing-key-file payment2.skey

This has created two new keys: __payment2.vkey__ and __payment2.skey__

But to generate the address we will use the same stake key pair that we already have, so we do NOT need a new stake key pair. Let's generate the new address __payment2.addr__

    cardano-cli shelley address build \
    --payment-verification-key-file payment2.vkey \
    --stake-verification-key-file stake.vkey \
    --out-file payment2.addr

Let us assume that you want to send 100 ada from `payment.addr` to `payment2.addr`

Creating a transaction in cardano-node is a process that requires various steps:

We first want to calculate the necessary fees for this transaction. To do this we need the _protocol parameters_, which we can save to file `protocol.json` with:

       cardano-cli shelley query protocol-parameters \
       --testnet-magic 42 \
       --out-file protocol.json

We need another piece of information before we create our transaction. We need the CURRENT TIP of the blockchain, this is, the height of the last block produced. We are looking for the value of `unSlotNo`

    cardano-cli shelley query tip --testnet-magic 42

    > Tip (SlotNo {unSlotNo = 795346}) (ShelleyHash {unShelleyHash = HashHeader {unHashHeader =        6a28a1c9fac321d7f4c8df8de68ee17f1967695460b5b422c93e6faaeeaf5cf2}}) (BlockNo {unBlockNo = 33088})

So at this moment the tip is on block 795346. This is important because when creating the transaction we need to specify the TTL (Time to live), this is the block height limit for our transaction to be included in a block, if it is not in a block by that time it will be cancelled.

For example, we know that we have 1 slot per second, lets say that it will take us 10 minutes to build the transaction, and that we want to give it another 10 minutes window to be included in a block, otherwise we want it to be cancelled. So we need 20 minutes. This is 1200 slots. So will want to add 1200 to the current tip. 795346 + 1200 = 796546 So our TTL will be 796546.

Our transaction will have one input (from `payment.addr`) and two outputs, 100 ada to `payment2.addr` and the change back to `payment.addr`. We can calculate the fees with:

       cardano-cli shelley transaction calculate-min-fee \
       --tx-in-count 1 \
       --tx-out-count 2 \
       --ttl 796546 \
       --testnet-magic 42 \
       --signing-key-file payment.skey \
       --protocol-params-file protocol.json

       > 167965

(The `--testnet-magic 42` identifies the FF-testnet. Other testnets will use other numbers, and mainnet uses `--mainnet` instead.)

So we need to pay 167965 lovelaces fee to create this transaction.

Assuming we want to spend an original utxo containing 1,000,000 ada (1,000,000,000,000 lovelace), now we need to calculate how much is the __change__ that we want to send back to __payment.addr__

    expr 1000000000000 - 100000000 - 167965

    > 999899832035


We need the transaction hash and index of the utxo we want to spend, which we can find outas follows:

        cardano-cli shelley query utxo \
            --address $(cat payment.addr) \
            --testnet-magic 42

        >                            TxHash                                 TxIx        Lovelace
        > ----------------------------------------------------------------------------------------
        > 4e3a6e7fdcb0d0efa17bf79c13aed2b4cb9baf37fb1aa2e39553d5bd720c5c99     4     1000000000000



**RAW TRANSACTION**
Now have all the information we need to create the transaction (using a "time to live" of slot 796546, after which the transaction will become invalid). We will write our transaction in a file, we will name it `tx001.raw`.

    cardano-cli shelley transaction build-raw \
    --tx-in 4e3a6e7fdcb0d0efa17bf79c13aed2b4cb9baf37fb1aa2e39553d5bd720c5c99#4 \
    --tx-out $(cat payment2.addr)+100000000 \
    --tx-out $(cat payment.addr)+999899832035 \
    --ttl 796546 \
    --fee 167965 \
    --out-file tx001.raw

**SIGN THE TRANSACTION**
We need to sign the transaction with the signing key __payment.skey__ and save the signed transaction in __tx001.signed__

    cardano-cli shelley transaction sign \
    --tx-body-file tx001.raw \
    --signing-key-file payment.skey \
    --testnet-magic 42 \
    --out-file tx001.signed

**SUBMIT TRANSACTION**
Finnally, we can submit the transaction with:

    export CARDANO_NODE_SOCKET_PATH=~/cardano-node/relay/db/node.socket

    cardano-cli shelley transaction submit \
            --tx-file tx001.signed \
            --testnet-magic 42

**CHECK THE BALANCES**

We must give it some time to get incorporated into the blockchain, but eventually, we will see the effect:

        cardano-cli shelley query utxo \
            --address $(cat payment.addr) \
            --testnet-magic 42

        >                            TxHash                                 TxIx        Lovelace
        > ----------------------------------------------------------------------------------------
        > b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee     1      999899832035

        cardano-cli shelley query utxo \
            --address $(cat payment2.addr) \
            --testnet-magic 42

        >                            TxHash                                 TxIx        Lovelace
        > ----------------------------------------------------------------------------------------
        > b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee     0         100000000
	
