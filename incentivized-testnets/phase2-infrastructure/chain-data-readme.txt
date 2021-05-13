This contains the (compressed) contents of ~/.ag-chain-cosmos/ from a
monitoring node that was attached to the "phase 2" (infrastructure)
of the incentivized testnet. This node was running agoric-sdk from
the "warner-phase2-loadgen" branch (git commit-id
57fe8c758effb7add44a37170e0db5419e956b93).

This *ought* to be sufficient to restart a node, by doing:

* git clone https://github.com/Agoric/agoric-sdk
* cd agoric-sdk
* git checkout 57fe8c758effb7add44a37170e0db5419e956b93
* yarn && yarn build && make -C packages/cosmic-swingset
* ag-chain-cosmos start
  (the 'ag-chain-cosmos' binary will be in your Go binary directory,
   perhaps ~/go/bin/)

(however this process has not actually been tested)

Once running, you should be able to connect to the RPC port and query for
chain status.


