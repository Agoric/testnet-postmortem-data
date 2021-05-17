The first iteration of the phase3 "Staking Dynamics" testnet halted on Monday
May 10th (around 7am pacific). We re-launched it about 1pm on Tuesday May
11th, with a few bug fixes and without the AMM load generator. This second
iteration of the chain is known as "act 2". Six hours later, this chain
suffered a fatal error (but did not halt) at 7:02pm during the execution of
block 65084, when the xsnap worker process hosting the "bank" vat suffered a
SIGSEGV.

The fact that the chain proceeded past the bank vat's termination means that
all validators observed the worker exit at the same time, indicating the
segfault was deterministic. Many chain operations continue to work without
the bank vat, however all token transfers between the cosmos-sdk Bank module
and JavaScript-side Purses depend upon a functional bank vat.

We do not yet know why the worker process was killed. The `deposit()` message
it was processing at the time did not appear unusual in any way. Our best
guess is a memory usage problem that was not caught by the metering code, or
some other XS engine problem. We are working to replay the vat in question to
reproduce the crash under a debugger.

https://github.com/Agoric/agoric-sdk/issues/2958 is our plan to have each
validator node exit when a worker process dies unexpectedly, rather than
terminating the vat. As described in the ticket, we expect deterministic
worker failures to be caught by metering, and other forms of worker exit may
be due to unusual local circumstances, like a machine-wide shutdown that
happens to kill the worker process before the kernel process.

## Data

The monitoring node is a non-voting non-staking fullnode, launched shortly
after the chain started, and run until after the phase was declared complete.
We use a monitoring node to collect data about the chain (including all
swingset operations), and as an RPC server for a load-generating client. This
particular "act 2" phase did not include any load generation, so no client
was started.

The monitoring node was running agoric-sdk at commit
855aaca64f2854af2f011acfe785190de47b34c7, which is
[agorictest-12](https://github.com/Agoric/agoric-sdk/tree/agorictest-12)
(commit
[d8f34ff1881b4f7e627f3fdab6888e9ef711bab6](https://github.com/Agoric/agoric-sdk/tree/d8f34ff1881b4f7e627f3fdab6888e9ef711bab6))
plus two minor patches:

* fix(swingset): expose writeSlogObject to host application
* fix(cosmic-swingset): slog begin/end-block and input events

When the monitoring node was shut down (10:20pm PT sunday may 16th, block
137458), the kernel and vat worker process sizes were as follows:

```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
warner     59522 26.7 27.1 15876036 1095880 pts/1 Sl  May12 1994:39 node /home/warner/stuff/agoric/agoric-sdk/packages/cosmic-swingset/bin/../src/chain-entrypoint.cjs start --log_level=warn
warner     59546  0.0  1.0  59452 41024 pts/1    S    May12   0:08 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v2:board -l 10000000
warner     59547  0.0  1.3  71272 55476 pts/1    S    May12   0:14 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v12:bootstrap -l 10000000
warner     59548  0.0  1.3  63436 54664 pts/1    S    May12   0:05 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v3:distributeFees -l 10000000
warner     59549  0.0  1.1  59452 44776 pts/1    S    May12   0:02 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v4:ibc -l 10000000
warner     59550  0.0  1.1  59452 45024 pts/1    S    May12   0:26 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v5:mints -l 10000000
warner     59551  0.0  1.3  63436 54340 pts/1    S    May12   0:04 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v6:network -l 10000000
warner     59552  0.0  1.3  63436 54552 pts/1    S    May12   0:18 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v7:priceAuthority -l 10000000
warner     59553  0.0  1.1  59452 44824 pts/1    S    May12   0:02 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v8:provisioning -l 10000000
warner     59554  0.0  0.0  59452   920 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v9:registrar -l 10000000
warner     59555  0.0  0.0  59452   972 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v10:sharing -l 10000000
warner     59556  0.0  0.8  59452 32928 pts/1    S    May12   0:02 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v16:timer -l 10000000
warner     59557  0.0  0.0  59452  1216 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v13:vatAdmin -l 10000000
warner     59558  0.0  2.3 108492 96488 pts/1    S    May12   1:39 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v11:zoe -l 10000000
warner     59559  0.0  3.4 145356 138136 pts/1   S    May12   1:20 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v17:undefined -l 10000000
warner     59560  0.0  0.0  59452  1040 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v18:undefined -l 10000000
warner     59561  0.0  1.6  75724 65992 pts/1    S    May12   0:27 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v19:undefined -l 10000000
warner     59562  0.0  0.0  59452  1048 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v20:undefined -l 10000000
warner     59563  0.0  0.0  59452  1048 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v21:undefined -l 10000000
warner     59564  0.0  0.0  59452   980 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v22:undefined -l 10000000
warner     59565  0.0  0.7  59452 30244 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v23:undefined -l 10000000
warner     59566  0.0  1.1  59452 47556 pts/1    S    May12   0:00 /home/warner/stuff/agoric/agoric-sdk/packages/xsnap/build/bin/lin/release/xsnap v24:undefined -l 10000000
```

The `data/` directory contains data from the monitoring node:

* `chain.out.gz` is stdout/stderr, which includes some swingset error messages
* `chain.slog.gz` is the slogfile, which holds a detailed record of all swingset operations
* `ag-cosmos-chain-state/` contains the `data.mdb` (compressed) and `lock.mdb` which make up the swingset "KernelDB". This can be used to recreate any vat.
* `dot-ag-chain-cosmos.tar.bz2` contains the complete `~/.ag-chain-cosmos/` directory, which includes the full cosmos state as well as the swingset kerneldb. This can be used to recreate the monitoring node.

