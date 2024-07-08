---
layout: distill
title: Concourrency 1
date: 2022-12-30 10:43:00
description: This is the chapter relating to concurrency control.
tags: acs
categories: study ucph
related_posts: true
giscus_comments: true
thumbnail: assets/img/concurrency.png
toc:
  sidebar: left

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH
---



<div align=right><I>Reference: This note is based on the Course Advanced Computer Systems from UCPH</I></div>

## Do-it-yourself Recap

**Techniques for Performance**

_What is the meaning of the following performance metrics: throughput, latency, overhead, utilization, and capacity?_

_Why can concurrency improve throughput and latency? How is that related to modern hardware characteristics?_

Separate the processing of signal requests, and put a process into different requests in parallel as well => Improve throughput

We reduce the number of things that need to be executed sequentially. We may not reduce the latency of a single request, but on average, it improves the latency.

## Objectives Today

1. Identify the multiple interpretations of the property of atomicity. => Talk about atomicity as a _desirable_ property of a system that uses concurrency to optimize for performance.
2. Implement methods to _ensure before-or-after atomicity_, and argue for their correctness. => When talking before or after atomicity, 2 basic strategies for achieving it are considered, _lock_ and _lockless_.
3. Explain the variants of the two-phase locking (2PL) protocol, particularly the widely-used Strict 2PL.
4. Explain situations where predicate locking is required.
5. Discuss the definition of serializability and the notion of anomalies (that do not happen for serializable executions).

## Read-Write Systems

In general, we will look at Read-Write Systems when we could say we are **_operating against the memory abstraction or we are trying to speed up the memory abstraction by concurrency_** and we want to still **_have some guarantees about the computations_**. We do not want to leave the actions of the concurrent to execute arbitrarily but we want to put some restrictions on those. And the core idea is to **_group the individual actions_** that we want to interleave or not interleave in the transactions. So that is **_On-Line Transaction Processing_** (OLTP). We have processing not only read and write actions but groups of those and grouped into transactions. Such systems do these operations and group them into transactions that arise all over the place.

**Process multiple, but relatively simple, application functions**

**Examples** Distributed

Order processing, e.g., Amazon

Item buy/sell in computer games, e.g., EVE online

High-performance trading

Updates on social networks, e.g., Facebook

## Transaction

<b>In this course, we would like to focus more on the relationship between Abstractions and Performance, and to improve Performance, we can use concurrency which has ensured the property of Atomicity.</b>

**Transaction** is a group of certain smaller actions but from a high level, it is a reliable unit of work against memory abstraction. Particularly, this is a unit of work that does some reads and rights.

In this course, we consider memory abstraction with a database, as it is the original thing to have these four properties. In reality, memory abstraction goes beyond the databases, namely, the **ACID** properties go beyond just the databases.

**ACID Properties**

- **Atomicity**: transactions are **all-or-nothing**
- **Consistency**: a transaction takes the database **from one consistent state to another**
- **Isolation**: transaction executes as if it were the only one in the system (aka before-or-after atomicity)
- **Durability**: once the transaction is done ("committed"), results are persistent in the database

## Examples of Transactions in SQL (Bank Transfers)

Under the hood, we know it all translates to calls to `READ` and `WRITE`.

## Conceptual Model - Version Histories

Conceptually, we think of a system that executes transactions as having certain versions. For example, if we do a transaction of transfer, then, we might have one version of the database.

## The many faces of atomicity

The definition of **_atomicity_** is a strong modularity mechanism, it can hide that one high-level action is actually made of many sub-actions. For example, if you have a lot of sub-actions and make them together by using atomicity, then, even though you talk to these sub-actions, it is equivalent to talking to the high-level one action because of the **_atomicity_**.

However, the above is not what we are talking about here. We prefer to talk to the following two.

**Before-or-after** atomicity == isolation: => Cannot have effects that would only arise by an interleaving of parts of transactions.

**All-or-nothing** atomicity == atomicity + durability: => Must be fully executed and once executed and committed, they are durable or they should be rolled back or aborted.

<I>Official Statement: Cannot have partially executed transactions; once executed and confirmed, transaction effects are visible and not forgotten</I>

Today, we are going to focus on <b>Before-or-after</b> atomicity.

## Scale Up

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/wVp3U3t.png" alt="image-20221230154333333" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

This is the simplistic view of the machine, we need to use concurrency to scale it up. Before we talk about concurrency, we can capable to extend it to multiple disks which provide a consistent data state (RAID systems). We will not talk about it here, just a blue picture. Then, we extend the CPU into multiple CPUs with more concurrent resources. And we need to ensure that the **ACID** properties are still there and this is the task of concurrency control.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/S4ISHW1.png" alt="image-20221230155236297" style="zoom:33%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

The problem we are solving is we want to automatically make sure despite running and executing the transactions concurrently that the **_before-or-after atomicity is ensured_**.

## Goal of Concurrency Control

Transactions should be executed so that it is as though they are executed in some serial order. Also called **Isolation** or **Serializability** or **Before-or-after** atomicity. However, this is not easy to achieve and comes with a certain overhead. So, there are some weaker variants. Weaker variants are also possible, they are about a lower "**degree of isolation**". Below we are going to talk about several different degrees of isolation.

**Bank Example**

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/GuWyG6f.png" alt="image-20221228164315658" style="zoom: 33%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

What we mean by serial execution is that T1 should occur with T2 or vice versa. Should occur before T1 so we want to execute these transactions and we don't care how we execute how we split them out for the interleave of the transactions, but the overall net effect of executing these transactions should be as if either T1 executed first, then T2 executed or the opposite direction T2 get first and T1.

<i>Official Statement: If submitted concurrently, net effect should be equivalent to Xacts running in some serial order. No guarantee that T1 logically occurs before T2 (or vice-versa) - but one of them is true.</i>

## Solution 1

The first solution is not use concurrency. Just get exclusive lock on entire database => Execute the entire transaction and then release exclusive lock. Hence, only one transaction can access the database at the time. Serializability guarantee because execution is serial!

The problem is obviously no concurrency and it cannot achieve our goal of executing the transactions quickly.

## Solution 2

Get exclusive locks on _accessed_ data items. Execute the transaction and release exclusive locks. For example, some transaction will access accounts of A and B, so it will get locked on these accounts. Then execute the transaction and then release the exclusive locks. This is greater concurrency. So, if the two transactions do not access the same data item, they can interact execute concurrently.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/jwM83yW.png" alt="image-20221230164306540" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

In this solution, we grab all the locks we will need, and then, in the end, we release them at once.

However, it does not work for our mentioned example. If two transactions access the same data item, there will be no concurrency here, they will be executed serially.s

## Solution 3 CS2PL

Let's make it be more relax.

Get exclusive locks on data items that are **_modified_** and get shared locks on data items that are only **_read_**. Execute the transaction, and then, release all locks.

Multiple people can have a shared lock and simultaneously read from the same object that the lock. If you have an exclusive lock and someone tries to get a shared lock. This will not be allowed.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/WFlGt7F.png" alt="image-20221230164902936" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

We refine slightly the kind of locks that we are using and in fact, it's already not too far from practice. => Conservative strict two-phase locking.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/jwM83yW.png" alt="image-20221230164306540" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

The problem is that it can still not be concurrent with our example as in our example, there are two modification actions in two transactions.

## Solution 4 C2PL

Get exclusive locks on data items that are modified and get shared locks on data items that are read.

Execute transactions and release locks on objects no longer needed during execution. => _Release locks during execution_.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/BW1nljD.png" alt="image-20221230165613638" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

We are not releasing the locks only at the end of the transaction but releasing that gradually. => More concurrency as now other transactions can grab those locks that have been released earlier.

**Problem here**

It is very hard to roll back. Think of multiple transactions and suppose one transaction runs then it releases a lock for an object that has modified. So another transaction can start accessing this lock. Now, if the first transaction for some reason needs to abort and we would like to undo the changes it performed. We also need to abort the second transaction because the second transaction has already seen the modification by the first transaction that will not be part of. => **_Cascading_**. The abort of one transaction will cause potentially false aborts of the other transactions.

Conservative refers to the left side of this picture where we at the beginning grab all locks at once.

## Solution 5 S2PL

Get exclusive locks on data items that are modified and get shared locks on data items that are read, but do this during the execution of the transaction (as needed). Release all locks.

We don't get all the locks at the beginning, we rather start executing and when we need to access something, we try to get a lock. But we do release all the locks at once in the end. => **_Strict Two Phase Locking (S2PL)_**

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/oOrO61h.png" alt="image-20221230171004331" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

The problem is if T1 requests for A's lock, while T2 requests for B's lock, and then, they just keep waiting and block each other to get the locks. => **_Deadlock_**.

## Solution 6 2PL

Get exclusive locks on data items that are modified and get shared locks on data items that are read, but do this during the execution of the transaction (as needed).

Release locks on objects no longer needed during the execution of the transaction.

**_Cannot acquire locks once any lock has been released, hence two-phase (acquiring phase and releasing phase)_**

Gives more concurrency however we also suffer more problems, it has more problems with deadlocks and the cascading.

## Summary of Alternatives

**CS2PL** No deadlocks, no cascading aborts, but need to know objects a priori, **least concurrency**

**C2PL** No deadlocks, more concurrency than **CS2PL**, but need to know objects a priori, when to release locks **cascading aborts**

**S2PL** No cascading aborts, <u>no need to know objects a priori or when to release locks</u>, more concurrency than **CS2PL**, but **deadlocks**

**2PL** Most concurrency, no need to know objects a priori, but need to know when to release locks, cascading aborts, deadlocks => 2PL has all the kind of worst problems

According to the summary, we need to pick one of them to be the main approach to ensure atomicity. From the perspective of the realistic meaning, CS2PL is not a suitable approach as it is the least concurrency. Additionally, during the execution of the transactions, it is not easy to know the objects you need in advance and this will be a downside. Hence, we select **S2PL** to focus on and address the problems.

## Reason for choice S2PL

Cannot know objects a priori, so no Conservative options => only if you would know something about the application!

Thus only 2PL and S2PL left

2PL needs to know when to release locks (main problem), and has cascading aborts

Hence S2PL

**Thus, what we need to do is deal with deadlocks.** => S2PL's problem

## Lock Management

Typically our system has a lock manager and we do not technically implement it, we just request an exclusive/shared lock. _Lock/Unlock requests handled by the lock manager._

**Lock table** entry:

1. Number of transactions currently holding a lock => Which transaction currently holds locks
2. Type of lock held (shared or exclusive)
3. Pointer to the queue of lock requests

Locking and unlocking have to be atomic operations => Have to be in composable units.

Lock **upgrade**: a transaction that holds a shared lock can be upgraded to hold an exclusive lock.

## Dynamic Databases - Locking the objects that exist now in the database is not enough

The protocols (except **_Solution 1_**) which are mentioned above have the limitations that the databases are fixed, for example, the number of records should be fixed at least.

If we relax the assumption that the DB is a fixed collection of objects, even Strict 2PL will not work correctly。

The fundamental assumption for all these project protocols is that we **are not extending the objects**.

## The Problem

To address the problem mentioned above, we need some mechanism to enforce the assumption and that is called **index locking** or **predicate locking**.

The example shows that correctness is guaranteed for locking on individual objects **only if the set of objects is fixed**.

## Index Locking

If data is accessed by an **index** on the rating field, T1 should **lock the index page** containing the data entries with rating = 1. If there are no records with rating = 1, T1 must lock the index page where such a data entry would be, if it existed.

If there is a lock on the index, this will mean that the index cannot be extended so transaction 2 will not be able simply not be able to add a new sailor (example) = wait for the lock to continue.

If there is no suitable index, we need to lock all the pages and also the entire table to prevent new records can be added to this table.

Hence, this is called index locking. _Predicate locking is a small generalization of index locking._

## Multiple-Granularity Locks

We have talked about locking the entire database and this is another refinement of the protocols that we looked at you can take locks at all these different levels and this will give you more fined gradient access to the data items that you need.

In fact, the refinement consists of not only using the shared or exclusive locks for these different levels but also refining the kinds of locks that we are using. So the idea somehow is we don't want to decide precisely what to lock. We want to lock the containers as we go from the outer layer. Data containers are nested.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/8bIF0Mt.png" alt="image-20230101143547555" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

## Solution - New Lock Modes, Protocol

This is a new lock model that is used to extend the standard mode of excessive entry blocks with so-called intention locks.

Allow transactions to lock at each level, but with a special protocol using new "intention" locks.

Before locking an item, the transaction must set "intention locks" on all its ancestors.

For unlock, go from specific to general (i.e., bottom-up).

SIX mode: Like S & IX at the same time.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/sStZ4yr.png" alt="image-20230101144336181" style="zoom:33%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/ixOtbse.png" alt="image-20230101151354368" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

Firstly, T1 would like to read the table Emp, and then, it locks the entire dabase with IS and lock the table with S. After that, T2 comes in, it would like to modify the data item, hence, T2 locks the entire database with IX and the table with IX as T2 would like to modify the data item. However, it is not allowed that table Emp can hold S and IX at the same time. T2 wait for forward.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/S7Yr9D3.png" alt="image-20230101153220566" style="zoom:50%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

## Is S2PL correct? (Assuming database is not dynamic)

We will formalize now and next class **serializability** and argue that S2PL is correct.

S2PL can however deadlock. We will see how to handle deadlocks automatically.

## Schedules

To talk about serializability, we need an notion of schedules.

<div align=center>
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/5GEzD1u.png" alt="image-20230101155626139" style="zoom:33%;" class="img-fluid rounded z-depth-1" data-zoomable>
    </div>
  </div>
</div>

_Each action has its own momentum time._

## Scheduling Transactions

1. **Serial schedule**: Schedule that does not interleave the actions of different transcations.

_The two schedules are equivalent if they hold the following two poinst._

2. **Equivalent schedules**: For any database state, => 输入输出完全一样，只不过操作顺序不一样

- The effect (on the set of objects in the database) of executing the schedules is the same.

- The values read by transactions is the same in the schedules
  - assume no knowledge of transaction logic.

3. **Serializable schedule**: A schedule that is equivalent to some serial execution of the transactions.

我的理解是如果一个调度没有交互事件的话，则是1，如果此时有另一个调度，每一时刻的输入和输出和最初的调度完全一样，那么新的调度和原先的调度等价且是serializable的。

(Note: If each transaction preserves consistency, every serializable schedule preserves consistency)

## Anomalies with Interleaved Execution 3

1. Write-Read Conflict

   The transaction reads data that is not committed. => Reading Uncommitted Data (WR Conflicts, "dirty reads")

   <div align=center>
   <div class="row mt-3">
       <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/yBFkzPH.png" alt="image-20221228164315658" style="zoom: 33%;" class="img-fluid rounded z-depth-1" data-zoomable>
       </div>
     </div>
   </div>

   但是如果abort是commit，那就没事，这个调度会是serializable schedule.

2. Read-Write Conflict

   <div align=center>
   <div class="row mt-3">
       <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/uEtwoSg.png" alt="image-20221228164315658" style="zoom: 33%;" class="img-fluid rounded z-depth-1" data-zoomable>
       </div>
     </div>
   </div>

   同一个事件里，读取了同一个数据A，但是由于T2读了A并且更改了A的值并且提交了，所以T1再读A的值变得不一样，所以T1就不知道该用哪个A。如果没有提交的话，就是脏读。

   Just observe two reads of the same object that results in different values like a transaction must be able to rely on the property that if it reads the same object multiple times without itself writing to it then it should see the same values.

3. Write-Write Conflict

   <div align=center>
   <div class="row mt-3">
       <div class="col-sm mt-3 mt-md-0">
        <img src="https://i.imgur.com/Px0NhvF.png" alt="image-20221228164315658" style="zoom: 33%;" class="img-fluid rounded z-depth-1" data-zoomable>
       </div>
     </div>
   </div>

   What is the state in the end? The value of A is the one written by T2 and the value of b is the one written by T1, and this is not equivalent to any serial execution.

## Final Summary

- Identify the multiple interpretations of the property of atomicity
- Implement methods to ensure before-or-after atomicity, and argue for their correctness.
- Explain the variants of the two-phase locking (2PL) protocol, in particular the widely-used S2PL
- Explain situations where predicate locking is required
- Discuss the definition of serializability and the notion of anomalies
