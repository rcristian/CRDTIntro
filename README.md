# A gentle introduction to CRDTs

aka Conflict-Free Replicated Data Types

## Why consistency models are important

What is the consistency model of your newly designed system / component?

As an afterthought: 

"I wanted it to be 'strong' [..ly consistent], and I had to battle the 'weak' [consistency]".

So there's a scale: from 'none' via 'weak', to 'strong'.

NB: consistency models are built in the 3rd party services you love and advocate (e.g. 'transaction isolation').

When it comes to distributed systems & consistency, the sky is not the limit: the CAP Theorem is.

## Types of consistency

Strong - 'all parties see the same thing at all times' - lot of locking, slow (damn you Chronos, master of all time!) NB: *your* time is different than *mine*

In-between, Weak - 'sometimes, most of us know the same thing' - compromise on something (data model, time)

None - you'll never know what you're going to get - but it's fast!

## Eventual consistency

Eventual consistency = at some point in time, if no more updates come in, if all updates are received and terminate

Example: Vector Clocks to detect concurrent updates on replicas

![Vector clocks - conflict detection](vector-clocks.png)

Conflicts are possible! (deal with it manually, or LWW...)

*Semantic Resolution* - the business logic knows how to deal with conflicts, delegate to it to arbitrate.

Safest: stop the world! but - it's hard and makes things really slow.

Consistency needs formal proof - it's something that you need to 'rely on', not 'take care of it later'. You just can't debug 1000 threads and live for a while on coke and pizza.


## CRDTs - what's that?

Data structures crafted to achieve eventual strong consistency between replicas.

Replication is important for scalability:
* fault-tolerance
* decreased read latency


Strong eventual consistency = two replicas that received the same set of updates will be in the same state - no conflicts possible, no rollbacks needed

Goal: Eliminate rollbacks, reconciliation - offer a deterministic way to move forward, no matter the number of conflicts.

What do we sacrifice? "Specially designed data structures and special operations on them"

Claims to offer a solution to the CAP problem.

## Simplest example - Increment-only counter, state-based

Increment only = allow only 'increase' operations

State-based = replica update packages contain the full state of the node

Assume *n* replicas and a particular node with index *x* (thus *0 <= x <= n-1*)

On node *x*:

```
cluster_state = [0,0,... 0]

increment():
    cluster_state[x] += 1

query():
    return sum(cluster_state)

merge(other_cluster_state):
    for i in range(n): cluster_state[i] = max(cluster_state[i], other_cluster_state[i])

send_update():
    send cluster_state to replicas (1..n) - x
```

## State-based rules

Note how the state on node *x* increases monotonically, and;

the 'merge' function is:
* commutative: ab = ba
* associative: a(bc) = (ab)c
* idempotent: may apply the same update twice, without effect

(Tell me this is not going to slip into the 'formal proof' realm... well, its a *semilattice*)

![increment-only state-based counter - out-of-order updates](Increment-only-state-based-counter.png)

The math rules mentioned have to be in place to deal with network reality: out-of-order messages, messages lost that have to be sent again.

The state-based counter we showed is a CvRDT (Cv = 'Convergent')

## Increment-only counter, operation-based

Don't send all state (which might be huge) but rather the operation.

On node *x*:

```
value_on_x = 0

increment():
    value_on_x += 1

    send_update():
        send '+1' to replicas (1..n) - x

query():
    return value_on_x

```

The above data structure is a CmRDT (Cm = 'Commutative')

## Op-based rules

Operations must be commutative.

![Concurrent operations must be commutative](op-based-ops-must-commute.png)

Updates must be applied once and once only.

![Updates must be applied once and once only](op-based-once-and-only-once.png)

Need in-order delivery of messages for non-concurrent messages (per-node in-order).

![Need in-order delivery of messages](op-based-in-order-delivery.png)

## Observed-remove set, operation-based

A set to which we can add and also remove elements from it.

On node *x*:

```
set_on_x = ()

add(e):
    unique_op_id = unique()
    set_on_x += (unique_op_id, e)

    send_add():
        send '+ (unique_op_id, e)'

remove(e):
    if (unique_op_id, e) in set_on_x then set_on_x -= (unique_op_id, e)

    send_remove():
        send 'remove (unique_op_id, e)'

query(e):
    return true if (_, e) in set_on_x 
```

Why do we need the unique() thing? Because without it, concurrent add() and remove() or combinations won't commute.

## How about a counter that you can also decrement?

You can combine CRDTs to yield new data types.

For a state-based solution to get a counter that can also go down, keep two increment-only counters:

* one to account for increments
* one to account for decrements

When querying for the value, return the difference.

For an op-based solution it is straightforward.

## Known CRDTs

### Counters

* G(row only)
* P(ositive)N(egative)

### Registers

* L(ast)W(riter)W(ins) - uses timestamps to order operations
* M(ulti)V(value) - keep conflicting values together, 'will fix later'

### Sets 

* G(row only) - no remove, ever
* 2P(hase) - similar to PN Counter, keep lists for removed elements (tombstones)
* U(nique) - each element has a unique identity
* LWW-element - timestamp based
* PN - associate with each element a counter that goes up when an element is added, goes down when removed; the element is in the set if the counter is positive
* O(bserved)R(emove)

### Graphs 

* Add-only monotonic DAG
* Add-Remove Partial Order data type

..and Co-operative text editing

..and most important: Invent your own! Because now you can.

## Use cases

* Counters - ad serving counts, likes
* Sets - followers, likes (again)
* Shopping carts (OR-Set)

If your use case is simple enough, you can 'roll your own' or use existing libraries.

Some are built-in Riak.

## Downsides

They're still complex, especially graphs.

Garbage collection (data tends to grow) might require synchronization.

## References

[A comprehensive study of CRDTs](http://hal.upmc.fr/inria-00555588/document)

Full of alien math signs, but it has all the algorithms!

Basho (developers of Riak) has a good collection of articles about CRDTs and other interesting 'distributed data' stuff; you can start browsing [here](https://docs.basho.com/riak/kv/2.1.4/learn/concepts/crdts/)

[The gist of this talk @ github](https://github.com/rcristian/CRDTIntro)





 
