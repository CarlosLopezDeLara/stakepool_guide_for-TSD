# Delegate your stake to your own pool

### Create a delegation certificate, deleg.cert.

Assuming we have a stake pool verification key file pool.vkey with the following content:

type: Node operator verification key
title: Stake pool operator key
cbor-hex:
 58200a9a89fe46bbc3b58998ab0d58da862194b51f1ce48ae319076bc1cf725e6108
Generate the certificate:

cardano-cli shelley stake-address delegation-certificate \
    --staking-verification-key-file stake.vkey \
    --stake-pool-verification-key-file IOHK.vkey \
    --out-file deleg.cert
Then build, sign and submit a transaction as before.

cardano-cli shelley transaction calculate-min-fee \
    --tx-in-count 1 \
    --tx-out-count 1 \
    --ttl 360000 \
    --testnet-magic 42 \
    --signing-key-file pay.skey \
    --signing-key-file stake.skey \
    --certificate deleg.cert \
    --protocol-params-file params.json

> runTxCalculateMinFee: 172805

cardano-cli shelley query utxo \
    --address $(cat pay) \
    --testnet-magic 42

                 TxHash                       TxIx        Lovelace
--------------------------------------------------------------------
0cba01...                                        0       99999428691

expr 99999428691 - 172805
> 99999255886

cardano-cli shelley transaction build-raw \
    --tx-in 0cba01...#0 \
    --tx-out $(cat pay)+99999255886 \
    --ttl 360000 \
    --fee 172805 \
    --out-file tx.raw \
    --certificate deleg.cert

cardano-cli shelley transaction sign \
    --tx-body-file tx.raw \
    --signing-key-file pay.skey \
    --signing-key-file stake.skey \
    --testnet-magic 42 \
    --tx-file tx.signed

cardano-cli shelley transaction submit \
    --tx-file tx.signed \
    --testnet-magic 42
