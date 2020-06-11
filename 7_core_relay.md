# Configure block-producing and relay nodes
Let's stop that single node now and do something more interesting

As stake pool operator, you will have two types of nodes, **block producing nodes** and **relay nodes**. Each block producing node must be accompagnied by several relay nodes.

To be clear: Both types of nodes run exactly the same program, **cardano-node**. The difference between the two types lies in how they are configured and how they are connected to each other:

* A **block producing** node will be configured with various key-pairs needed for block generation (cold keys, KES hot keys and VRF hot keys). It will only be connected to its relay nodes.

* A **relay node** will not be in possession of any keys and will therefore be unable to produce blocks. It will be connected to its block producing node, other relays and external nodes.

Each node should run on a dedicated server, and the block producing node's server's firewall should be configured to only allow incoming connections from its relays.

In this tutorial, we will simplify matters by having a block producing node (It won't produce blocks yet), and by using a single relay. For now, we will run both nodes on the same server.

We have explained how to run a single node, and now you have suitable configuration files `ff-config.json`,`ff-topology.json` and `ff-genesis.json` available.

Both our nodes must use the same `ff-genesis.json`, they can use the same `ff-config.json` (but don't have to), and they need different `ff-topology.json` files.

Let us create separate folders for the two nodes and simply copying the configuration files we have:

    cd cardano-node
    mkdir block-producing
    mkdir relay
    cp ff-config.json ff-genesis.json ff-topology.json block-producing/
    cp ff-config.json ff-genesis.json ff-topology.json relay/


We will run our block-producing node on port 3000 (make sure it is opened) and our relay on port 3001
(you can of course use different ports if you like)

We must modify the block-producer's `ff-topology.json` to only "talk" to the relay:

Navigate to `/block-producing` and open `topology.json` with your favorite text editor:

    {
      "Producers": [
        {
          "addr": "x.x.x.x", # Replace with your public IP
          "port": 3001,
          "valency": 1
        }
      ]
    }

In the  `relay/topology.json` we instruct the node to "talk" to the block-producer *and* an external node as before:


	{
	   "Producers": [
	     {
	       "addr": "x.x.x.x",
	       "port": 3000,
	       "valency": 1
	     },
	     {
	       "addr": "relays-new.ff.dev.cardano.org",
	       "port": 3001,
	       "valency": 2
	     }
	   ]
	 }

To start your nodes on our AWS instance, a terminal multiplexer like [`tmux`](https://github.com/tmux/tmux/wiki) is useful, because it allows us to open different panes in a single terminal window.

We have already installed `tmux` when we installed dependencies.

You can find a short overview of available commands [here](https://tmuxcheatsheet.com/).

You start `tmux` with

    tmux new

Then you can split the screen with `Ctrl`-`b`-`%` and navigate between the two panes with `Ctrl`-`b`-`→` and `Ctrl`-`b`-`←`.




From one `tmux`-panel we start the block-producing node with the following command. Under `host-addr` replace the x.x.x.x with your public ip

    cardano-node run \
    --topology block-producing/ff-topology.json \
    --database-path block-producing/db \
    --socket-path block-producing/db/node.socket \
    --host-addr x.x.x.x --port 3000 \
    --config block-producing/ff-config.json

The node will start, but it won't receive any data, because we have configured it to only "talk" to the relay node, and we haven't yet started the relay.

We switch to the other `tmux`-panel with `Ctrl`-`b`-`→` and start the relay node with a similar command. Under `host-addr` replace the x.x.x.x with your public ip

    cardano-node run \
     --topology relay/ff-topology.json \
     --database-path relay/db \
     --socket-path relay/db/node.socket \
     --host-addr x.x.x.x \
     --port 3001 \
     --config relay/ff-config.json

After a few seconds, both nodes should receive data.

Cool!! we have put a couple of nodes to work! But this nodes can't do anything more than read from the blockchain. To setup a stake pool and being able to produce blocks we will need a set of keys, addresses, and other things. Let's create some keys first.
