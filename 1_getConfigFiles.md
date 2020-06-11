## Get genesis, configutarion, topology files, and start the node

Let us create a new directory inside `cardano-node`to store the configuration files that we will need no start the node.

    cd cardano-node
    mkdir relay
    cd relay

Now, to start your node and connect it to F&F testnet you will need three important files: `ff-config.json` `ff-genesis.json` and `ff-topology.json`. We will download them from <https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html>

    wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/ff-config.json
    wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/ff-genesis.json
    wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/ff-topology.json

Starting the the node uses the command `cardano-node run` and a set of options.

You can get the complete list of available options with `cardano-node run --help`  

	--topology FILEPATH             The path to a file describing the topology.
  	--database-path FILEPATH        Directory where the state is stored.
  	--socket-path FILEPATH          Path to a cardano-node socket
  	--host-addr HOST-NAME           Optionally limit node to one ipv6 or ipv4 address
  	--port PORT                     The port number
  	--config NODE-CONFIGURATION     Configuration file for the cardano-node
  	--validate-db                   Validate all on-disk database files
  	--shutdown-ipc FD               Shut down the process when this inherited FD reaches EOF
  	--shutdown-on-slot-synced SLOT  Shut down the process after ChainDB is synced up to the
  	                                specified slot
   -h,--help                       Show this help text

Now you can start the node, double check that port 3001 is open. In the `cardano-node` directory run:

    cardano-node run \
       --topology relay/ff-topology.json \
       --database-path relay/db \
       --socket-path relay/db/node.socket \
       --host-addr x.x.x.x \
       --port 3001 \
       --config relay/ff-config.json

__NOTE__ you need to replace x.x.x.x with your public ip. In a VPS server, it is the one you use connect to the server. If you are using virtual box, you can find your public searching `My public ip` in google.  
