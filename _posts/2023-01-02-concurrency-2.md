---
layout: distill
title: Concourrency 2
date: 2023-01-02
description: This is the chapter relating to concurrency control - Part 2.
tags: acs
categories: study ucph
related_posts: true
gist_comments: true
thumbnail: assets/img/concurrency-2.png

toc:
  sidebar: left

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH
---

<div align=center>
  <div class="row mt-3 mb-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/NBdawZO.png" alt="Group_8_YhdE3vh" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

## Do-it-yourself-recap

**Locking Solutions for Isolation in ACID Transactions**

- Why two phases?

  _<u>Acquiring phase</u>_ and _<u>releasing phase</u>_. What does this give us? The claim from the last lecture is that this will give us serializability. So only schedules that are produced that can be produced by such a mechanism will be serializable.

- What about the problems with these different strategies? (Solution 4 & Solution 5) How to acquire and release locks? How to structure the two phases?

  C2PL => Cascading Aborts, S2PL => Deadlocks

## Is S2PL correct? (Assuming the database is not dynamic)

Strict 2PL can however deadlock => We will see how to handle deadlock automatically

## Schedules -> Recap

What is a schedule?

A schedule is a sequence of `READ / WRITE` actions that these performed by different transactions.

Interleaving is one possible execution of these actions and we allow the transactions to interleave their actions. This is what will give us concurrency but in the schedule, all actions are linearized. **So there are no concurrent actions in a schedule.** **_!IMPORTANT_**

## Scheduling Transactions -> Recap

**Serial schedule**: Schedule that does not interleave the actions of different transactions.

**Equivalent schedules**: See [Concurrency Control 1](https://liuying-1.github.io/advanced-computer-system/concurrency/)

- Moreover, we can abstract a way of what the transactions actually do here the actual computational operations just see the `READ`s and `WRITE`s. So we can not somehow assume anything about what these transactions might do to the state of the system so because we need to require that all the values read by the transactions will be the same in the two transactions. So this encodes the assumption that we abstract away the logic of two of the actual transaction.

**Serializable schedule**: [Concurrency Control 1](https://liuying-1.github.io/advanced-computer-system/concurrency/)

So if we have a serializable schedule, this is the kind of core property that helps us achieve why we are interested in serializable schedules. It helps us to reason about consistency. Consistency is not the task of the database management system to provide it somehow application specific but active if your schedules are not serializable, it becomes very hard for the use of a DBMS to think about consistency.

**_Note: If each transaction preserves consistency, every serializable schedule preserves consistency._**

Whereas for a serializable schedule, the user can pretend that there is no concurrence in the database system. So if there is a transaction that is consistent without any concurrency, if it's consistent as if it was executed as the only transaction in the system then also the serializable will preserve consistency.

Summary => We want serializable property that it can preserve consistency.

## Anomalies with Interleaved Execution

These three are the most typical three.

**Dirty Reads (WR Conflicts)** => Read uncommitted Data

The first transaction would write the value of A and then the second transaction would execute a Read of A and much later the first transaction with the action aborted for some reason. Then, transaction 2 has a problem because that value should have never been written in the first transaction when the transaction has been aborted, we don't want to see any of its actions take any effect on the database state.

**Unrepeatable Reads (RW Conflicts)**

Here the transaction reads a value and transaction 2 modifies that value and transaction 1 reads that value again. Well, it will be a potentially different value because it has been modified. **Thus this is a bit problematic like the transaction should rely on that when it reads if it is the only transaction running in the system, whatever it reads will not be modified by anyone else. That's the isolation view that we want to enforce.**

**Overwriting Uncommitted Data (WW Conflicts)**

Blind Write without reading any values. The last written value for B is the one from transaction 1, and the last written value for A is the one from transaction 2. There is simply no serial execution of this.

[WW-Conflicts, Wikipedia](https://en.wikipedia.org/wiki/Write%E2%80%93write_conflict)

## Conflict Serializable Schedules

Now comes the refinement of this notion of serializability. And this is a very different kind of notion I would say. So while serializability is a semantic notion, it talks about how database state changes across the execution of the schedule. This one is very syntactic. It just looks at which actions are performed and which actions can be easily swapped.

We call the schedule conflict serializable if it is conflict equivalent to some serial schedule and now the important part is that conflict equivalent is a different thing. This is not the same as equivalent, this is where the difference between the two definitions lies. => Introduce the definition of the **_Conflict_ _Equivalent_**

<hr>

Two schedules are conflict equivalent if they have the same actions of the same transaction.

_Tip: In equivalent schedules, the serializability doesn't care about the sequence of the actions, while it does matters in the Conflict Equivalent_.

Moreover, every pair of conflicting actions should be ordered in the same way. First, what are conflicting actions? Those are the kind of actions that cause troubles => Conflicts mentioned before. So, two actions are conflicting if they access the same memory location and one of the two is the `Write`.

For every pair of actions in two schedules, if conflicting actions to schedules, they must occur in the same order in both schedules. So if a `READ` to A occurred before a `WRITE` to A in one schedule, then it must do so in the second schedule as well.

<hr>

**Official Version**

Two schedules are conflict equivalent if:

- Involve _<u>the same actions of the same transactions</u>_
- Every pair of conflicting actions _<u>is ordered the same way</u>_
- Two actions are called conflicting _<u>if they access the same object and one of them is a write</u>_

Schedule S is <u>_conflict serializable_</u> if S _<u>is conflict equivalent to some serial schedule</u>_.

## Example that is not conflict serializable

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/a2hsLI3.png" alt="image-20230104145627828" style="zoom:33%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

We will build **_<u>Precedence Graph</u>_** to see if there is a cycle. If there is like here for example, then the schedule will not be conflict serializable. If there is not, then the schedule will be conflict serializable. There is an arrow if there is a pair of conflicting operations between the two transactions, and the earlier part of the pair comes from the source of the arrow and the later parties and the target of the arrow.

**Analysis of the example**

Conflicts

1. The very first `READ` conflicts with the `WRITE` => `T1: R(A)` -> `T2: W(A)`. And because it occurs in the schedule before the `WRITE`, that's the reason why there is an arrow from `T1` to `T2`.
2. `T1: W(A)`, `T2: R(A)` is the case of dirty reads, and also `T1: W(A)` conflicts with `T2: W(A)`. And shares the same arrow with conflict 1 as they are all caused by `T1`.
3. `T2: R(B)` conflicts with `T1: W(B)`, and `T2: W(B)` conflicts with `T1: R(B)`. Hence, there is a back edge arrow.

<font face="宋体" color=red>在这个例子里，我不太理解为什么第一个冲突会是RW冲突，因为T1并没有继续重复读取A的值（已解决，看Recap中关于RW标粗的定义）。同时我也不太能理解为什么第二个里面会有WW冲突，因为和上面的WW冲突的定义不符合；</font>

## Precedence Graph

One node per transaction; edge from `Ti` to `Tj` if an operation in `Tj` conflicts with an earlier operation in `Ti`.

<div align=center><i><b>Theorem: Schedule is conflict serializable if and only if its precedence graph is acyclic. (1)</b></i></div>

<div align=center><i><b>Theorem: Strict 2PL only results in conflict serializable schedules. (2)</b></i></div>

Hence, S2PL only gives us conflict serializable schedules.

## Exercises

Are the following schedules conflict serializable?

| T1  | R(A) |      |      | W(B) |      |      |      |  C  |     |     |
| :-: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :-: | :-: | :-: |
| T2  |      | R(B) |      |      | R(A) |      | R(C) |     |  C  |     |
| T3  |      |      | R(B) |      |      | W(C) |      |     |     |  C  |

Yes. This is conflict serializable.

| T1  | R(A) |      | W(B) |      |      |      |      |  C  |     |     |
| :-: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :-: | :-: | :-: |
| T2  |      | R(B) |      |      | R(A) |      | R(C) |     |  C  |     |
| T3  |      |      |      | R(B) |      | W(C) |      |     |     |  C  |

No, as there is a cycle in the Precedence Graph.

**_The above are my own solutions, I am not sure whether they are correct right now._**

<font color=red>We need to pay attention to the direction of the graph. And there is no need to think about commit action.</font>

## Returning to Definition of Serializability

A schedule S is serializable if there exists a serial order SO such that:

- The **state of the database** after S is the **same** as the state of the database after SO
- The **values read** by each transaction in S are the **same** as that returned by each transaction in SO
  - Database does not know anything about the internal structure of the transaction programs

**Under this definition, certain serializable executions are not conflict serializable!**

## Example to show the scope for conflict serializable is tighter

| T1  | R(A) |      | W(A) |      |
| :-: | :--: | :--: | :--: | :--: |
| T2  |      | W(A) |      |      |
| T3  |      |      |      | W(A) |

Is this schedule serializable? <font color=red>How to judge whether the schedule is serializable?</font>

Yes, the equivalent schedule is below.

| T1  | R(A) | W(A) |      |      |
| :-: | :--: | :--: | :--: | :--: |
| T2  |      |      | W(A) |      |
| T3  |      |      |      | W(A) |

However, is this schedule conflict serializable?

No, as there is a cycle between `T1` and `T2`.

_Hence, it is serializable but not conflict serializable._

## View Serializability

There is kind of an alternative that sits between conflicts serializable and serializable, view serializable.

A schedule is _view serializable_ if it is view equivalent to a serial schedule. Here is a new term, view equivalent.

<hr>

Schedules S1 and S2 are **view equivalent** if:

- If `Ti` reads initial value of `A` in `S1`, then `Ti` also reads initial value of `A` in `S2`. => Initial Read

- If `Ti` reads value of `A` written by `Tj` in `S1`, then `Ti` also reads value of `A` written by `Tj` in `S2` => Any Read happens Later

- If `Ti` writes final value of `A` in `S1`, then `Ti` also writes final value of `A` in `S2.`

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
     <img src="https://i.imgur.com/hU9XTKV.png" alt="image-20230104164137815" style="zoom:33%;" />
    </div>
  </div>
</div>

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/cGIZt1C.png" alt="image-20230104174522681" style="zoom:33%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

[Reference about View Serializable](https://hw311.me/zh/study-notes/database/2019/02/24/transaction-consistency-serializability/)

## Deadlocks

When we use S2PL, we can get into a deadlocks, and what is the deadlock?

**_Deadlock: Cycle of transactions waiting for locks to be released by each other. Hold the locks that are required for the other transaction to proceed._**

We have two ways dealing with deadlocks.

One is **_Deadlock Prevention_** that will _ensure that no cycle will happen_. When taking locks, we will decide what to do with transactions and we will abort those that would cause a cycle to arise. (Before)

The second is **_Deadlock Detection_**, in this strategy, we don't do the same as the first one. We only always monitor who is waiting for whom, looking if there are cycles. And only when the cycle is there, then, we start aborting transactions. (Arise)

[<font face=songti>死锁预防｜死锁检测</font>](https://github.com/CyC2018/CS-Notes/blob/master/notes/计算机操作系统%20-%20死锁.md#死锁预防)

## Deadlock Prevention

How to prevent deadlocks? The basic idea is to assign certain priorities or weights to our transactions and one good strategy is to use **_timestamps_**.

The idea is that the oldest transaction should get the higher priority somehow the transaction that needs to wait longer to execute should get priority over more recent transactions.

Hence, we,

- Assign priorities based on **_<u>timestamps</u>_**.

- Lower timestamps get higher priority, i.e., older transactions get prioritized.

- Assume `Ti` wants a lock that `Tj` holds. Two policies are possible (alternatives).

  - Wait-Die: **If `Ti` has higher priority**, **`Ti` waits for `Tj`; otherwise `Ti` aborts**. => `Ti` should wait that it has higher priority but it still needs to wait because the lock has been held by `Tj` and we want to wait for `Tj` to finish its execution, but we also do not want to abort `Ti` because it has higher priority. Hence, that is the decision to let `Ti` wait. If `Ti` does not have higher priority, we abort with `Ti`.

    => 如果A的优先权为1，B的优先权为2。此时，如果A向B要锁，A的权限更高，然后A就等，因为A的生存时间更久；如果此时B也正好要A的锁，则B abort，否则造成死锁。

  - Wound-wait: **If `Ti` has higher priority**, **`Tj` aborts; otherwise `Ti` waits**.

    => 同样的例子，优先级高的永远在要锁的时候能抢到锁，优先级低的直接abort。

- If a transaction re-starts, make sure it has its original timestamp.

## Deadlock Detection

When this graph has a cycle, then we start getting active and start killing transactions.

- Create a **waits-for graph**:
  - Nodes are transactions
  - There is an edge from `Ti` to `Tj` if `Ti` is waiting for `Tj` to release a lock => Different from the **_Precedence Graph_**
- Periodically check for cycles in the waits-for graph => Maybe we don't need to detect the deadlocks as soon as they arise, but at some point, we would like to detect them.

## Example of Deadlock Detection

| T1  | S(A) | R(A) |      |      | S(B) |      |      |      |      |                             |
| :-: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :-------------------------: |
| T2  |      |      | X(B) | W(B) |      |      |      | X(C) |      |                             |
| T3  |      |      |      |      |      | S(C) | R(C) |      |      | <font color=red>X(A)</font> |
| T4  |      |      |      |      |      |      |      |      | X(B) |                             |

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/h6gORNH.png" alt="image-20230104182841732" style="zoom:33%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

In the example above, when `T1` would like to `R(B)`, it first gets a shared lock for B, however, it needs to wait for T2 to release the exclusive lock. Hence, `T1` to `T2`. The others are the same.

The right part for `X(A)` is in red because this is the operation close the cycle and eventually detects that there is a cycle and decides which transaction to abort. Once we have a cycle, we need to abort one of the involved transactions. => Abort 1, 2, 3 哪个都行

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/nj9nRGn.png" alt="image-20230104184008200" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

[REFERENCE](https://www.inlighting.org/archives/database-concurrency-control)

## Practices Waits-for

| T1  | S(A) |      |      | X(D) |     | X(C) |  C  |      |      |
| :-: | :--: | :--: | :--: | :--: | :-: | :--: | :-: | :--: | :--: |
| T2  |      |      | X(A) |      |     |      |     | X(B) |      |
| T3  |      | S(B) |      |      |     |      |     |      | S(C) |

Note: We only show locking operations for brevity! C alone denotes commit, **<u>all locks released</u>**

There is no deadlock.

**_Notice: We need to delete the arrow from T2 to T1 as soon as the locks are released by T1._**

| T1  |      |      |      | S(C) | S(A) |      | X(D) |
| :-: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| T2  | S(B) |      |      |      |      | X(C) |      |
| T3  |      | S(D) | X(B) |      |      |      |      |

There is a deadlock.

**_Here the contents above are something about lockful approaches for concurrency control. Let's talk about lockless approaches._**

## The Problems with Locking

Locking is a **pessimistic** (Prepare yourself for the rest to happen) approach in which conflicts are prevented. Disadvantages:

- Lock management overhead => There is a certain overhead coming for managing the locks. We need to deal with deadlocks.
- Deadlock detection/resolution
- Lock contention for heavily used objects => if you have objects that are very heavily accessed read or written, there will be some lock contention for the objects.

But what we will be looking right now is a more **optimistic** approach. Just let things happen and if bad things happen, you try to resolve them without using locks.

However, even if we were to give up locking, we still wanted to enforce some form of serializibility without requiring the entire schedule to be completely serial, so without destroying concurrency.

We will look at how to **_repair violations_** when they happen by aborting transactions. Locking is used to prevent violations. While **_aborts_** is the only method to **_fix violations_**.

_How can we design a protocol based on aborts instead of locks?_

## Optimistic Concurrency Control: Kung-Robinson Model

The basic idea is, there are no locks, so completely different from before, but the transactions has been structured into 3 different phases.

`READ` => We have a `READ` phase when you read things from the database. We can also write things but not to the database. (You are writing things you are making changes to copies of the objects you read from the database).

`VALIDATE` => We will talk in detail what the validate phase needs to do to ensure in the end serializability. => Check for conflicts.

`WRITE` phase where we write out all the local changes that we accumulated, but we are not reading anymore in this phase. We only access database in the `WRITE` phase => Make local copies of changes public.

## Validation Phase

Test conditions that are **sufficient** to ensure that no conflict occured.

Each transaction is assigned a numeric id and just use a **timestamp**.

Transaction IDs are assigned at end of `READ` phase, just before validation begins. When the transaction is finished with reading then we record that timestamp and say this is the id of the transaction.

And then, we need to consider the set of objects read by the given transaction and modified by the transaction. And we call these sets `ReadSet(Ti)` and `WriteSet(Ti)`.

Official => `ReadSet(Ti)`: Set of objects read by transaction `Ti`, `WriteSet(Ti)`: Set of objects modified by `Ti`

The validation phase proceed the tests and there are **_three test conditions_**. If the first test fails, we still have a chance to succeed with the second test and so on.

## Test 1

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/YsSDoDX.png" alt="image-20230104213954735" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

<u>For all i and j such that `Ti` < `Tj`, check that `Ti` completes before `Tj` begins.</u> Ti and Tj are the id of the transactions and also the timestamp after the end of the `READ` phase. But **_if additionlly `Ti` has completed doing everything what it was supposed to do before `Tj` even started, that's ok. So then we passed the first test_** and there is no problem between these two transactions, this is kind of they do not overlap in time. => Transactions are not executed concurrently.

## Test 2

For all i and j such that `Ti` < `Tj`, check that:

- `Ti` completes before `Tj` begins its `WRITE` phase

- `WriteSet(Ti)` $\and$ `ReadSet(Tj)` is empty. => `Tj` does not read dirty data that has been modified by `Ti`.

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/iltElnk.png" alt="image-20230104214924137" style="zoom: 50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

## Test 3

For all i and j such that `Ti` < `Tj`, check that:

- Ti completes `READ` phase before `Tj` does

- `WriteSet(Ti)` $\and$ `ReadSet(Tj)` is empty => whatever `Tj` reads is not touched by `Ti`

- `WriteSet(Ti)` $\and$ `WriteSet(Tj)` is empty => `Ti` will not overwrite `Tj`'s writes.

  <div align=center>
    <div class="row mt-3">
      <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/tyKYj3r.png" alt="image-20230104215433507" style="zoom:50%;" />
      </div>
    </div>
  </div>

## Validation Example

Predict whether T3 will be allowed to commit, given the transaction below.

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/XwaG6Fe.png" alt="image-20230104215705886" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

This is the Optimistic Concurrency Control.

## Overheads in Optimistic CC

**_So, there is a question comes out. Why were we developing this complex algorithms for the 2 phase locking if you can do without locks?_**

There is some overhead that also comes in this way of achieving serializability.

- Must record read/write acitivity in `ReadSet` and `WriteSet` per transaction. => Must create and destory these sets as needed.
- Must check for conflicts during validation, and must make validated writes "global".
  - Critical section (The validation phase is also something that comes with its own overhead and is not allowed to continue executing the transaction writing the parts of the transaction that need to be persisted before all the validation checks have passed) reduces concurrency. => Even though the validation checks are not computationally very expensive, but it's still some overhead.
  - Scheme for making writes global can reduce clustering of objects
- Optimistic CC restarts Xacts that fail validation as work done so far is wasted; Requires clean-up for the local section for each transaction.

Still, optimistic techniques widely used in software transactional memory (STM), main-memory databases.

## Snapshot Isolation

- Often databases implement properties that are weaker than serializability, snapshot isolation
- Snapshot:
  - Transactions see snapshot as of beginning of their execution
  - First Committer Wins: Conflicting writes to same item lead to aborts
- May lead to **write skew**
  - Databse must have at least one doctor on call
  - Two doctors on call concurrently examine snapshot and see exactly each other on call
  - Doctors update their own records to being on leave
    - No write-write conflicts: different records!
- After commits, database has no doctors on call

## Examples of Transaction Support in Databases by using Snapshot

## What should we learn today?

- Discuss the definition of serializability and the notion of anomalies
- Apply the conflict-serializability test using a precedence graph to transaction schedules
- Discuss the difference between conflict-serializability and view-serializability
- Explain deadlock prevention and detection techniques
- Apply deadlock detection using a waits-for graph to transaction schedules
- Explain the optimistic concurrency control model
- Predict validation decisions under optimistic concurrency control
