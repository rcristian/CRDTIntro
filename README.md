# A gentle introduction to CRDTs

aka Conflict-Free Replicated Data Types

## Agenda
---

A practical way (CRDTs) of reasoning about consistency

Goal: give you a key to (one) of the 1000's doors in the consistency kingdom; *a DYI formula* in the realm of consistency models

## Why consistency models are important
---

What is the consistency model of your newly designed system / component?

Most of you will shrug their shoulders. As an afterthought: 

"I wanted it to be 'strong' [..ly consistent], and I had to battle the 'weak' [consistency]".

So there's a scale: from 'none' via 'weak', to 'strong'.

NB: consistency models are built in the 3rd party services you love and advocate (e.g. 'transaction isolation').

## Types of consistency
---

Strong - 'all parties see the same thing at all times' - lot of locking, slow (damn you Chronos, master of all time!) NB: *your* time is different than *mine*

In-between, Weak - 'sometimes, most of us know the same thing' - compromise on something (data model, time)

None - you'll never know what you're going to get - but it's fast!

## CRDTs - what are they
---

Specially designed data structures to achieve eventual strong consistency between replicas.

Eventual = "at some point in time, not necessarily on the spot, if no more updates come in" - conflicts are possible! (deal with it manually, or LWW)

Strong = "two replicas that received the same set of updates will be in the same state" - no conflicts possible, no rollbacks needed

What do we sacrifice? "Specially designed data structures and special operations on them"

## Simplest example - Increment-only counter, state-based
---

We'll define the terms above a bit later

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
```

## What just happened?
---

No constraints on time (Ha! Told you, Chronos!)

The 'merge' function is:
* commutative - ab = ba
* associative a(bc) = (ab)c
* idempotent - may apply the same update twice, without effect

(Tell me this is not going to slip into the 'formal proof' realm...)

[increment-only state-based counter - out-of-order updates](Increment-only-state-based-counter.png)


 
