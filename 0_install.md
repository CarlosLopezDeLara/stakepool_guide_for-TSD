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
For the FF-testnet, we will use tag `1.13.0`, which we can check out as follows:

    git fetch --all --tags
    git tag
    git checkout tags/1.13.0


## Build and install the node

Now we build and install the node with ``cabal``,
which will take a couple of minutes the first time you do a build. Later builds will be much faster, because everything that does not change will be cached.

    cabal install cardano-node cardano-cli --overwrite-policy=always

   This will build and install `cardano-node` and `cardano-cli` in folder `~/.cabal/bin` by default, so the remark about your `PATH` from above
   applies here as well: Make sure folder `~/.cabal/bin` is in your path or copy the executables to a folder that is.

If you ever want to update the code to a newer version, go to the ``cardano-node`` directory, pull the latest code with ``git`` and rebuild.
This will be much faster than the initial build:

    cd cardano-node
    git fetch --all --tags
    git tag
    git checkout tags/<the-tag-you-want>
    cabal install cardano-node cardano-cli --overwrite-policy=always

__Note:__ You need to delete the `db`-folder (the database-folder) before running an updated version of the node.
