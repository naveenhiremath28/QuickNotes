



```
Orchestrated comprehensive overview of fault tolerance techniques

Fault tolerance is a system's ability to keep operating correctly even when some of its components fail. It's worth distinguishing from a couple of neighbors people often blur together: _reliability_ is about how rarely a system fails at all, _availability_ is the fraction of time it's usable, and _fault tolerance_ is specifically about surviving failures gracefully rather than collapsing. A fault-tolerant system assumes failure is inevitable—disks die, networks partition, processes crash, whole data centers go dark—and is engineered so that no single failure takes the whole thing down.


The main techniques you'll see across systems:

**Redundancy** is the foundation. You run more than one of everything. There are two flavors: _active-active_, where all replicas serve traffic simultaneously (like the two app servers above, which also gives you load distribution for free), and _active-passive_, where a standby sits idle until the primary fails (like the database replica). Active-active uses your hardware better but is harder to keep consistent; active-passive is simpler but wastes the standby's capacity until it's needed.

**Failover** is the act of detecting a failure and switching to a backup. The detection usually comes from health checks—the load balancer pings each app server, and the moment one stops responding, it's pulled from rotation and traffic flows only to the healthy ones. The quality of a failover system lives in two numbers: how fast it detects failure, and how fast it recovers.

**Replication** keeps redundant copies of data in sync so a backup is actually usable. The key trade-off here is _synchronous_vs _asynchronous_. Synchronous replication confirms the write on the replica before acknowledging it, so you lose nothing on failover, but every write pays the round-trip latency. Asynchronous is fast but risks losing the last few writes if the primary dies before they propagate. This is a direct expression of the CAP theorem—when the network partitions, you must choose between consistency and availability.


The thing to keep in mind underneath all of this: fault tolerance is never free. Every technique trades cost, latency, or complexity for resilience, and adding redundancy adds coordination problems of its own (split-brain, replication lag, consensus overhead). The engineering judgment is deciding _how much_ fault tolerance a given system actually warrants—a payments ledger and a meme-image cache sit at very different points on that curve.
```


SQL NoSQL
Master Slave architecture database
