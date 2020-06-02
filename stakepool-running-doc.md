# Installing and Running a Node


##PREREQUISITES

Set up your platform:

You will need:

* An x86 host (AMD or Intel), Virtual Machine or AWS instance with at least 2 cores, 4GB of RAM and at least 10GB of free disk space;
* A recent version of Linux, not Windows or MacOS – this will help us isolate any issues that arise;
* Make sure you are on a network that is not firewalled. In particular, we will be using TCP/IP port 3000 and 3001 by default to establish connections with other nodes, so this will need to be open.
* You can follow this [SERVER TUTORIAL](https://github.com/input-output-hk/cardano-tutorials/blob/master/node-setup/AWS.md) to get the server up and running. Alternatively, and only for learning purposes you can try this guide on VirtualBox [UBUNTU IMAGE FOR VIRTUAL BOX](https://linuxhint.com/install_ubuntu_virtualbox_2004/) 

## Install dependencies

We need the following packages and tools on our Linux system to download the source code and build it:
    - the version control system ``git``,
    - the ``gcc`` C-compiler,
    - C++ support for ``gcc``,
    - developer libraries for the the arbitrary precision library ``gmp``,
    - developer libraries for the compression library ``zlib``,
    - developer libraries for ``systemd``,
    - developer libraries for ``ncurses``,
    - ``ncurses`` compatibility libraries,
    - the Haskell build tool ``cabal``,
    - the GHC Haskell compiler.

If we are using an AWS instance running Amazon Linux AMI 2 (see the [AWS walk-through](AWS.md) for how to get such an instance up and running)or another CentOS/RHEL based system, we can install these dependencies as follows:

    sudo yum update -y
    sudo yum install git gcc gcc-c++ tmux gmp-devel make tar wget zlib-devel -y
    sudo yum install systemd-devel ncurses-devel ncurses-compat-libs -y

For Debian/Ubuntu use the following instead:
   
        
    sudo apt-get update -y
    sudo apt-get -y install build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5 -y
   
If you are using a different flavor of Linux, you will need to use the package manager suitable for your platform instead of `yum` or `apt-get`, and the names of the packages you need to install might differ.

Download, unpack, install and update Cabal:

    wget https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
    tar -xf cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
    rm cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz cabal.sig
    mkdir -p ~/.local/bin
    mv cabal ~/.local/bin/
    cabal update

This will work on a fresh [AWS instance](AWS.md) and assumes that folder `~/.local/bin` is in your `PATH`.
On other systems, you must either move the executable to a folder that is in your `PATH` or modify your `PATH` by adding the line

    export PATH="~/.local/bin:$PATH"

to your `.bashrc`-file.

Above instructions install Cabal version `3.2.0.0`. You can check the version by typing

   cabal --version

Finally we download and install GHC:

    wget https://downloads.haskell.org/~ghc/8.6.5/ghc-8.6.5-x86_64-deb9-linux.tar.xz
    tar -xf ghc-8.6.5-x86_64-deb9-linux.tar.xz
    rm ghc-8.6.5-x86_64-deb9-linux.tar.xz
    cd ghc-8.6.5
    ./configure
    sudo make install
    cd ..

## Download the source code for cardano-node

To download the source code, we use git:
    
    git clone https://github.com/input-output-hk/cardano-node.git
    

This should create a folder ``cardano-node``, then download the latest source code from git into it.

After the download has finished, we can check its content by
  
    ls cardano-node

Note that the content of your ``cardano-node``-folder can slightly differ from this!

We change our working directory to the downloaded source code folder:

    cd cardano-node

For reproducible builds, we should check out a specific release, a specific "tag". 
For the FF-testnet, we will use tag `pioneer`, which we can check out as follows:

    git fetch --all --tags
    git tag
    git checkout tags/pioneer-#latest-tag


## Build and install the node

Now we build and install the node with ``cabal``, 
which will take a couple of minutes the first time you do a build. Later builds will be much faster, because everything that does not change will be cached.

    cabal install cardano-node cardano-cli

   This will build and install `cardano-node` and `cardano-cli` in folder `~/.cabal/bin` by default, so the remark about your `PATH` from above
   applies here as well: Make sure folder `~/.cabal/bin` is in your path or copy the executables to a folder that is.
    
   Alternatively, you can use:  

    cabal build all
    cp -p dist-newstyle/build/x86_64-linux/ghc-8.6.5/cardano-node-1.12.0/x/cardano-node/build/cardano-node/cardano-node ~/.cabal/bin/
    cp -p dist-newstyle/build/x86_64-linux/ghc-8.6.5/cardano-cli-1.12.0/x/cardano-cli/build/cardano-cli/cardano-cli ~/.cabal/bin/

  If you have old versions of `cardano-node` installed on your system, make sure that the new one will be picked! You can check by typing

    which cardano-node
    > ~/.cabal/bin/cardano-node

If you ever want to update the code to a newer version, go to the ``cardano-node`` directory, pull the latest code with ``git`` and rebuild. 
This will be much faster than the initial build:

    cd cardano-node
    git fetch --all --tags
    git tag
    git checkout tags/<the-tag-you-want>
    cabal install cardano-node cardano-cli

Note: You need to delete the `db`-folder (the database-folder) before running an updated version of the node.


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

## Create key pair and an address

# Creating keys and addresses

We need to create two sets of keys and addresses: One set to control our funds (make and receive payments) and one set to control our stake (to participate in the protocol delegating our stake) 

Let's produce our cryptographic keys first, as we will need them to later create our addresses:

### Payment key pair
1. First we need to generate our _payment key pair_:

        cardano-cli shelley address key-gen \
            --verification-key-file payment.vkey \
            --signing-key-file payment.skey

   This will create two files (here named `payment.vkey` and `payment.skey`),
   one containing the _public verification key_, one the _private signing key_.

   The files are in plain-text format and human readable:

        cat payment.vkey

        > type: VerificationKeyShelley
        > title: Free form text
        > cbor-hex:
        >  18af58...

   The first line describes the file type and should not be changed.
   The second line is a free form text that we could change if we so wished.
   The key itself is the cbor-encoded byte-string in the fourth line.
   
### Stake key pair
2. Now let us create our _stake key pair_ : 

		cardano-cli shelley stake-address key-gen \
		--verification-key-file stake.vkey \
		--signing-key-file stake.skey

### Payment address
3. We then use `payment.vkey` and `stake.vkey` to create our `payment address`: 

		cardano-cli shelley address build \
		--payment-verification-key-file payment.vkey \
		--stake-verification-key-file stake.vkey \
		--out-file payment.addr  
     
This created the file payment.addr, let's check its content: 

		cat payment.addr 
		
		> 01ed8...


4. In order to query your address (see the utxo's at that address),
   you first need to set environment variable `CARDANO_NODE_SOCKET_PATH`
   to the socket-path specified in your node configuration. In this example we will use
   the block-producing node created in the previous steps:

        export CARDANO_NODE_SOCKET_PATH=~/cardano-node/relay/db/node.socket

   and make sure that your node is running.  Then use

        cardano-cli shelley query utxo \
            --address <YOURADDRESS> \
            --testnet-magic 42
   
   you should see something like this:

                              TxHash                                 TxIx        Lovelace
    ----------------------------------------------------------------------------------------

   (The `--testnet-magic 42` is specific to the FF-testnet, for mainnet we would use `--mainnet` instead.)
   
   
### Stake address
5. Finnaly, we can create our stake address. This address __CAN'T__ receive payments but will receive the rewards from participating in the protocol. We will save this address in the file `addr.stake`

		cardano-cli shelley stake-address build \
		--staking-verification-key-file stake.vkey \
		--out-file stake.addr 

This created the file stake.addr, let's check its content: 

		cat stake.addr 
		
		> 820058... 
		
## Get funds: The Faucet 

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

    
## Create a simple transaction 

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
	
## Register stake address : //TO DO//


## Configure block-producing and relay nodes
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
   



Cool, we have put a couple of nodes to work! But this nodes can't do anything more than read from the blockchain. To setup a stake pool and being able to produce blocks we will need a set of keys, addresses, and other things. Let's create some keys first.




# Understanding your configuration files and how to use them:




## The topology.json file

Tells your node to which nodes in the network it should talk to. A minimal version of this file looks like this: 


	{
	  "Producers": [
	    {
	      "addr": "x.x.x.x",
	      "port": 3001,
	      "valency": 1
	    }
	  ]
	}

* This means that your node will contact `relays-new.ff.dev.cardano.org` on `port 3001`. 

* `valency` tells the node how many connections your node should have. It only has an effect for dns addresses. If a dns asdress is given, valency governs to how many resolved ip addresses should we maintain acctive (hot) connection; for ip addresses, valency is used as a boolean value, where `0` means to ignore the address.

Your __block-producing__ node must __ONLY__ talk to your __relay nodes__, and the relay node should talk to other relay nodes in the network. Go to our telegram channel to find out IP addresses and ports of peers. 


## The genesis.json file

The genesis file is generated with the `cardano-cli` by reading a `genesis.spec.json` file, which is out of scope for this document. 
But it is important because it is used to set:

* `genDelegs`, a mapping from genesis keys to genesis delegates.
* `initialFunds`, a mapping from the initial addresses to the initial values at those address. 
* `MaxLovelaceSupply`, the total amount of lovelaces in the blockchain.  
* `startTime`, the time of slot zero.

The `genesis.json` file looks like the one below.

	{
	  "activeSlotsCoeff": 0.05,
	  "protocolParams": {
	    "poolDecayRate": 0,               
	    "poolDeposit": 500000000,         
	    "protocolVersion": {
	      "minor": 0,
	      "major": 0
	    },
	    "decentralisationParam": 0.5,
	    "maxTxSize": 16384,               
	    "minFeeA": 44,                                  
	    "maxBlockBodySize": 65536,        
	    "keyMinRefund": 0,                
	    "minFeeB": 155381,                 
	    "eMax": 1,                         
	    "extraEntropy": {
	      "tag": "NeutralNonce"
	    },
	    "maxBlockHeaderSize": 1400,
	    "keyDeposit": 400000,              
	    "keyDecayRate": 0,                 
	    "nOpt": 50,                        
	    "rho": 0.00178650067,              
	    "poolMinRefund": 0,                
	    "tau": 0.1,                        
	    "a0": 0.1                          
	  },
	  "protocolMagicId": 42,
	  "startTime": "2020-05-12T20:15:00.000000000Z", # Time of slot 0                                   
	  "genDelegs": {                       
	    "c1f35a67ff923bf2637a3ce75413bec24e97b4fdacb7f654ec248c3a23a2b1a3": "067696862f671a83fb64490938826466e530b8bc340e937d4931b4051020f58e",
	    "18c6ff0bd626e4728c3c1d2b171d8610109e74404e857f5cffc112784d74642c": "1d847f1e3d6ed31430435597802cd79e6566b64a29c857e42a3c3b8717986a22",
	    "1e5a4f62ccbad10b0a004717cb3f099ef43e8ca3a7554a9b71afa839606bdf20": "1f5f0e1aea3a320cd443d82959531fa7a8b1f4c6fe966018154e129dbbd57ff1"
	  },
	  "updateQuorum": 3,                                              
	  "maxMajorPV": 0,                    
	  "initialFunds": {                    
	 

	  },
	  "maxLovelaceSupply": 45000000000000000, 	                                        
	  "networkMagic": 42,
	  "epochLength": 21600,             
	  "staking": null,
	  "slotsPerKESPeriod": 86400,       
	  "slotLength": 1,
	  "maxKESEvolutions": 14,                                              
	  "securityParam": 108
    }

Here is a brief description of each parameter. You can learn more in the [spec](https://github.com/input-output-hk/cardano-ledger-specs/tree/master/shelley/chain-and-ledger/executable-spec)


| PARAMETER | MEANING | 
|----------| --------- |
| activeSlotsCoeff | The proportion of slots in which blocks should be issued. | 
| poolDecayRate | Decay rate for pool deposits |
| poolDeposit | The amount of a pool registration deposit |
| protocolVersion| Accepted protocol versions |
| decentralisationParam | Percentage of blocks produced by stake pools |
| maxTxSize | Maximal transaction size | 
| minFeeA | The linear factor for the minimum fee calculation | 
| maxBlockBodySize | Maximal block body size |
| keyMinRefund | The minimum percent refund guarantee |
| minFeeB | The constant factor for the minimum fee calculation |
| maxBlockBodySize | Maximal block body size | 
| keyMinRefund | The minimum percent refund guarantee |
| minFeeB | The constant factor for the minimum fee calculation | 
| eMax | Epoch bound on pool retirement | 
| extraEntropy | Well, extra entropy =) |
| maxBlockHeaderSize | | 
| keyDeposit | The amount of a key registration deposit |
| keyDecayRate | The deposit decay rate | 
| nOpt | Desired number of pools | 
| rho | Treasury expansion | 
|	poolMinRefund | The minimum percent pool refund | 
|	tau | Monetary expansion | 
|	a0 | Pool influence | 
| protocolMagicId | | 
| startTime | Time of slot 0 |
| genDelegs | Mapping from genesis keys to genesis delegate |                
| updateQuorum | Determines the quorum needed for votes on the protocol parameter updates |
| maxMajorPV | Provides a mechanism for halting outdated nodes |
| initialFunds | Mapping address to values | 
| maxLovelaceSupply | The total number of lovelace in the system, used in the reward calculation. |
| networkMagic | | 
| epochLength | Number of slots in an epoch. |
| staking | | 
| slotsPerKESPeriod | Number of slots in an KES period |
| slotLength | | 
| maxKESEvolutions | The maximum number of time a KES key can be evolved before a pool operator must create a new operational certificate |
| securityParam | Security parameter k |


## The config.json file 

The default `config.json` file that we downloaded is shown below.

This file has __4__ sections that allow you to have full control on what your node does and how the informtion is presented. 

__NOTE Due to how the config.json file is generated, fields on the real file are shown in a different (less coherent) order. Here we present them in a more structured way__

### 1 Basic Node Configuration.

First section relates the basic node configuration parameters. Make sure you have to `TPraos`as the protocol, the correct path to the `ff-genesis.json` file, `RequiresMagic`for its use in a testnet. 
Note that in this example we are using the SimpleView. This will send the output to `stdout`. Other option is `LiveView` which uses a terminal multiplexer to generate a fancy view. We will cover this topic later. 

	{
	  "Protocol": "TPraos",
	  "GenesisFile": "ff-genesis.json",
	  "RequiresNetworkMagic": "RequiresMagic",

### 2 Update parameteres
 
This protocol version number gets used by block producing nodes as part of the system for agreeing on and synchronising protocol updates.You just need to be aware of the latest version supported by the network. You dont need to change anything here. 
 
	  "ApplicationName": "cardano-sl",
	  "ApplicationVersion": 0,
	  "LastKnownBlockVersion-Alt": 0,
	  "LastKnownBlockVersion-Major": 0,
	  "LastKnownBlockVersion-Minor": 0,


### 3 Tracing

`Tracers` tell your node what information you are interested in when logging. Like switches that you can turn ON or OFF according the type and quantity of information that you are interesetd in. This provides fairly coarse grained control, but it is relatively efficient at filtering out unwanted trace output.

The node can run in either the `SimpleView` or `LiveView`. The `SimpleView` just uses standard output, optionally with log output. The `LiveView` is a text console with a live view of various node metrics.

`TurnOnLogging`: Enbles or disables logging overall.

`TurnOnLogMetrics`: Enable the collection of various OS metrics such as memory and CPU use. These metrics can be directed to the logs or monitoring backends.

`setupBackends`, `defaultBackends`, `hasEKG`and `hasPrometheus`: The system supports a number of backends for logging and monitoring. This settings list the the backends available to use in the configuration. The logging backend is called `Katip`. 
Also enable the EKG backend if you want to use the EKG or Prometheus monitoring interfaces. 

`setupScribes` and `defaultScribes`: For the Katip logging backend we must set up outputs (called scribes) The available types of scribe are:

* FileSK: for files
* StdoutSK/StdoutSK: for stdout/stderr
* JournalSK: for systemd's journal system
* DevNullSK
* The scribe output format can be ScText or ScJson. 

`rotation` The default file rotation settings for katip scribes, unless overridden in the setupScribes above for specific scribes.


	  "TurnOnLogging": true,
	  "TurnOnLogMetrics": true,
	  "ViewMode": "SimpleView",
	  "TracingVerbosity": "NormalVerbosity",
	  "minSeverity": "Debug",
	  "TraceBlockFetchClient": false,
	  "TraceBlockFetchDecisions": false,
	  "TraceBlockFetchProtocol": false,
	  "TraceBlockFetchProtocolSerialised": false,
	  "TraceBlockFetchServer": false,
	  "TraceChainDb": true,
	  "TraceChainSyncBlockServer": false,
	  "TraceChainSyncClient": false,
	  "TraceChainSyncHeaderServer": false,
	  "TraceChainSyncProtocol": false,
	  "TraceDNSResolver": true,
	  "TraceDNSSubscription": true,
	  "TraceErrorPolicy": true,
	  "TraceForge": true,
	  "TraceHandshake": false,
	  "TraceIpSubscription": true,
	  "TraceLocalChainSyncProtocol": false,
	  "TraceLocalErrorPolicy": true,
	  "TraceLocalHandshake": false,
	  "TraceLocalTxSubmissionProtocol": false,
	  "TraceLocalTxSubmissionServer": false,
	  "TraceMempool": true,
	  "TraceMux": false,
	  "TraceTxInbound": false,
	  "TraceTxOutbound": false,
	  "TraceTxSubmissionProtocol": false,
	  "setupBackends": [
	    "KatipBK"
	  ],
	  "defaultBackends": [
	    "KatipBK"
	  ],
	  "hasEKG": 12788,
	  "hasPrometheus": [
	    "127.0.0.1",
	    12798
	  ],
	  "setupScribes": [
	    {
	      "scFormat": "ScText",
	      "scKind": "StdoutSK",
	      "scName": "stdout",
	      "scRotation": null
	    }
	  ],
	  "defaultScribes": [
	    [
	      "StdoutSK",
	      "stdout"
	    ]
	  ],
	  "rotation": {
	    "rpKeepFilesNum": 10,
	    "rpLogLimitBytes": 5000000,
	    "rpMaxAgeHours": 24
	    },	  

### 4 Fine grained logging control

It is also possible to have more fine grained control over filtering of trace output, and to match and route trace output to particular backends. This is less efficient than the coarse trace filters above but provides much more precise control. `options`:

`mapBackends`This routes metrics matching specific names to particular backends. This overrides the defaultBackends listed above. And note that it is an **override** and not an extension so anything matched here will not go to the default backend, only to the explicitly listed backends.

`mapSubtrace` This section is more expressive, we are working on its documentation. 
	    
 
	  "options": {
	    "mapBackends": {
	      "cardano.node-metrics": [
	        "EKGViewBK",
	        {
	          "kind": "UserDefinedBK",
	          "name": "LiveViewBackend"
	        }
	      ],
	      "cardano.node.BlockFetchDecision.peers": [
	        "EKGViewBK",
	        {
	          "kind": "UserDefinedBK",
	          "name": "LiveViewBackend"
	        }
	      ],
	      "cardano.node.ChainDB.metrics": [
	        "EKGViewBK",
	        {
	          "kind": "UserDefinedBK",
	          "name": "LiveViewBackend"
	        }
	      ],
	      "cardano.node.metrics": [
	        "EKGViewBK",
	        {
	          "kind": "UserDefinedBK",
	          "name": "LiveViewBackend"
	        }
	      ]
	    },
	    "mapSubtrace": {
	      "benchmark": {
	        "contents": [
	          "GhcRtsStats",
	          "MonotonicClock"
	        ],
	        "subtrace": "ObservableTrace"
	      },
	      "#ekgview": {
	        "contents": [
	          [
	            {
	              "contents": "cardano.epoch-validation.benchmark",
	              "tag": "Contains"
	            },
	            [
	              {
	                "contents": ".monoclock.basic.",
	                "tag": "Contains"
	              }
	            ]
	          ],
	          [
	            {
	              "contents": "cardano.epoch-validation.benchmark",
	              "tag": "Contains"
	            },
	            [
	              {
	                "contents": "diff.RTS.cpuNs.timed.",
	                "tag": "Contains"
	              }
	            ]
	          ],
	          [
	            {
	              "contents": "#ekgview.#aggregation.cardano.epoch-validation.benchmark",
	              "tag": "StartsWith"
	            },
	            [
	              {
	                "contents": "diff.RTS.gcNum.timed.",
	                "tag": "Contains"
	              }
	            ]
	          ]
	        ],
	        "subtrace": "FilterTrace"
	      },

	      "cardano.epoch-validation.utxo-stats": {
	        "subtrace": "NoTrace"
	      },
	      "cardano.node-metrics": {
	        "subtrace": "Neutral"
	      },
	      "cardano.node.metrics": {
	        "subtrace": "Neutral"
	      }
	    }
	  }
	}

