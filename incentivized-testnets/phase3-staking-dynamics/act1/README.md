The phase3 "Staking Dynamics" testnet was launched on Thursday May 6 at noon
pacific. We ran a low-rate load generator, creating AMM trades once per hour.
It halted four days later, around 7am pacific. We call this iteration "act1",
to distinguish it from the subsequent restart ("act2").

The immediate cause of the halt was the LMDB kernel database exceeding its
pre-configured 2GiB size limit, which occurred during the commit of block
number 54569. In this version of the codebase, about 98% of the database is
filled with vat transcripts. https://github.com/Agoric/agoric-sdk/issues/3065
is our plan to move this into separate files, while
https://github.com/Agoric/agoric-sdk/issues/3073 is our plan to let the LMDB
database grow when necessary. Together these should fix the immediate issue.

The chain also suffered a fatal error during that same block, when the "zoe"
vat was killed by a metering fault. Every two hours, the chain issued rewards
to all clients, and one step in the issuance process sends a `combine()` and
a `splitMany()` to this vat. The crank which processed the bulk of these two
messages (triggered by a resolved-promise notification delivery) used
slightly more CPU than the pre-configured "10M computrons" limit, which
triggered the kernel's runaway-computation protection, so the vat was
terminated. Since zoe manages all contracts in the system, no further
contract activity could have taken place after the zoe vat was terminated,
even if the LMDB fault hadn't happened.

Analyzing the slogfile shows there are two cranks that consume a lot of CPU
meter: the zoe crank for `combine()/splitMany()`, and another sent to the
"bank" vat shortly afterwards that involves a method named
`depositMultiple()`. The workload given to both methods increases slightly
over the lifetime of the chain, as more clients are provisioned (causing
rewards to be paid out to more accounts), resulting in roughly 115 clients by
the end.

The zoe crank shows increasing CPU meter usage as well as wallclock time
spent over the four days of the chain. Initially, it used 1.4M computrons,
and about 200-400ms. By the end, it took 5 or 6 seconds, and reached the 10M
computron limit.

The bank crank shows a similar pattern, although the CPU meter usage grew
more slowly (reaching 2.6M and holding steady once the last new client was
provisioned), and the wallclock time showed much greater variation. Some
cranks in the middle and late phases of the chain took 15-20s, and two blocks
took nearly a full minute to compute this one crank. (Any block that takes
more than about 5 seconds will be noticeable, and blocks which take more than
20s are likely to cause problems for at least some of the validators).

The slow cranks also show more engine-level GC operations taking place,
suggesting a lot of internal activity (creating lots of short-lived or
long-lived objects).

Our suspicion is that the lack of kernel-level GC is causing the retention of
otherwise short-lived objects, increasing the size of the `harden` WeakMap,
and causing a general slowdown of all vat activity. We're working to figure
out what exactly might be slowed down by this, as well as trying to finish
kernel-level GC to alleviate the problem.
