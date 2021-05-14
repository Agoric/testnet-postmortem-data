The first iteration of the phase3 "Staking Dynamics" testnet halted on Monday
May 10th (around 7am pacific). We re-launched it about 1pm on Tuesday May
11th, with a few bug fixes and without the AMM load generator. This second
iteration of the chain is known as "act 2". Six hours later, this chain
suffered a fatal error (but did not halt) at 7:02pm during the execution of
block 65084, when the xsnap worker process hosting the "bank" vat suffered a
SIGSEGV.

The fact that the chain proceeded means that all validators observed the
worker exit at the same time, indicating the segfault was deterministic. Many
chain operations continue to work without the bank vat, however all token
transfers between the cosmos-sdk Bank module and JavaScript-side Purses
depend upon a functional bank vat.

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


The `most-of/` directory contains the monitoring node's stdout
(`chain.out.gz`) and the slogfile trace up until thursday. These will be
replaced by the complete trace, as well as the kernel DB, when the phase
officially finishes at noon on friday.
