# Get funds: The Faucet

Before we can create transactions and do fun stuff in the blockchain, we need some funds to work with. We have a faucet to which we can request some testnet tokens.


    curl -v -XPOST "https://faucet.ff.dev.cardano.org/send-money/<YOURPAYMENTADDR>?apiKey=<API KEY>"

 Now check the balance of your address to see if you have got the funds:

    export CARDANO_NODE_SOCKET_PATH=~/cardano-node/relay/db/node.socket

Make sure that your node is running.  Then use

    cardano-cli shelley query utxo \
    --address <YOURADDRESS> \
    --testnet-magic 42

You should see something like this:

                              TxHash                                 TxIx        Lovelace
    ----------------------------------------------------------------------------------------
    65e99578e91dbf400c42989b5b5ae6dde877510900074f4afd8ff472639da6b3     0     1000000000000
