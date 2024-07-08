---
layout: distill
title: Recovery > Normal
date: 2023-01-07
description: This is the special part for recovery and normal.
tags: acs
categories: study ucph
related_posts: true
giscus_comments: true
thumbnail: assets/img/normal.png

authors:
  - name: Ying Liu
    url: "https://di.ku.dk/Ansatte/forskere/?pure=da/persons/762476"
    affiliations:
      name: DIKU, UCPH
---

<div align=center>
  <div class="row mt-3 mb-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/NMeVaGS.png" alt="image-20230104213954735" style="zoom:80%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

## Recovery

### Do-it-yourself-recap: The many faces of atomicity

- **Atomicity** is strong modularity mechanism!
  - Hides that one high-level action is actually made of many sub-actions
- **Before-or-after** atomicity
  - == Isolation (AC<b>I</b>D)
  - Cannot have effects that would only arise by interleaving of parts of transactions
- What was the meaning of **all-or-nothing atomicity**?
  - == Atomicity (<b>A</b>CID) => Cannot be partially executed
  - == Duration (ACI<b>D</b>) => Once been committed, be visible forever durable.

### Objectives Today

- Explain the concepts of **volatile**, **nonvolatile**, and **stable storage** as well as the **main assumptions underlying database recovery**
- Predict how **force/no-force and steal/no-steal strategies** for writes and buffer management influence the need for **redo** and **undo**
- Explain the **notion of logging** and the **concept of write-ahead logging**
- Predict **what portions of the log and database are necessary for recovery** based on the **recovery equations**
- Explain **how write-ahead logging is achieved in the ARIES protocol**
- Explain the **functions of recovery metadata** such as the transaction table and the dirty page table
- Interpret th**e contents of the log resulting from ARIES** normal operation

## Recovery: Basic Concepts

For this part, please review [Experi | Recovery](https://liuying-1.github.io/blog/2023/recover/).

## Recovery: ARIES, normal operation

### Recovery Equations

**_Let's talk about the algorithm, we need to support all REDO and UNDOs. What do we log and how do we log it?_**

#### WAL & the Log

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/9pIoUVM.png" alt="image-20230106162212065" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

What are the things that we need to store in the log to do our accounting, accounting is what happening in the system.

First of all, we need IDs for the records that we store in the log and we will call **log sequence number** or just LSN. The log is a sequential thing, and LSNs always increasing. We will use this fact to compare to sequence numbers and to figure out what has happened before what.

- Each log record has a unique **Log Sequence Number (LSN)**
  - LSNs always increasing

Then, when we look at the **data pages** that are stored everywhere in the system both in memory and on disk, we will extend the data page beyond the actual data, also containing **Log Sequence Number which is the most recent update to this page**.

- Each **<u>data page</u>** contains a **pageLSN**.
  - The LSN of the <u>most recent log</u> record for an update to that page.

Moreover, the system will keep track of other special log sequence numbers. What does this special log sequence number (**flushedLSN**)? The log itself should live in stable storage, maybe the largest part of it. But still, when we extend it, we first need to go through RAM. The log itself will be split into a part that lives on the disk, and the part that lives on the RAM. And the **flushedLSN** is a kind of pointer to where the split happens between the two.

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/yFVU6XI.png" alt="image-20230106164519107" style="zoom: 33%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

- System keeps track of **flushedLSN**.
  - The **max** LSN flushed so far.

**_When a crash happens, we will lose the "Log tail" since it lives on the RAM._** So this is not so perfect, this part will be lost and his will be our only information about actions that were happening in this period of time. => 1. We want to flush the log frequently. We want to make this time where entries log entries leave in the RAM as short as possible. 2. We are not aiming for perfection here, we will lose some information here. However, we don't want to lose those entries.

What does the **write-ahead logging** means? This means, whatever we write a page to the disk, we need to make sure that pageLSN which is the most recent record modifying the page is below what has been flushed. And what has been flushed means that the log record corresponding to this modification is already in the "blue" part.

- WAL: Before a page is written,
  - pageLSN <= flushedLSN ----> 否则，大于的部分无法恢复了
  - https://zhuanlan.zhihu.com/p/482491814

#### Log Records

**LogRecord fields**

| prevLSN | XID | type | pageID | length | offset | old data | new data |
| :-----: | :-: | :--: | :----: | :----: | :----: | :------: | :------: |

- `prevLSN` is a pointer to the **previous log entry by the same transaction**. It's kind of in addition to just having a sequential view on all the actions. We also **storing this linked lists that correspond to the actions performed by a single transaction**. So, when we maybe **undoing** things of a single transaction, we can focus more effectively on just this transaction don't need to traverse the entire log. Just have easy access to the actions of a single transaction.

- `type` -> Possible log record types

  | Update | Commit | Abort | End | Compensation Log Records (CLRs) |
  | :----: | :----: | :---: | :-: | :-----------------------------: |

  `Update` -> `old data` & `new data`

  `Commit` -> signifies the start of `commit`

  `End` -> As `commit` or `abort` also take some certain time, this signifies **end** of `commit` or `abort`.

  `Compensation Log Records (CLRs)` -> they will be recording the undo actions

#### Other Log-Related State

**Transaction Table** and **Dirty Page Table** are the **data structures maintained in memory**. This was kind of the local state of the algorithm that there's responsible for recovery, but it will be **completely lost on the crash** and we will **be using the log to reconstruct the state of these data structures** at the moment of the crash.

- **Transaction Table (XT)**

  - One entry per active transaction.

  - Contains **XID**, **status** (running/commited/aborted), and **lastLSN**.

    | XID | Status | lastLSN |
    | :-: | :----: | :-----: |

    So, what is the `lastLSN`? It is the last operation performed by a transaction. Kind of the entry point to the backwards pointing linked list associated with transaction. => <font color=red>Not fully understood</font>

- **Dirty Page Table (DPT)**

  - One entry per dirty page in buffer pool.

  - Contains `recLSN` -- the LSN of the log record which <u><b><i>first</i></b></u> caused the page to be dirty.

    | PID | recLSN |
    | :-: | :----: |

### Normal Execution of a Transaction

<div align=center>It must be OK to crash <b>at any time -> repeat history</b>!</div>

_What do we do during normal execution of a transaction?_

Firstly, what is a transaction? A transaction is a series of `READ`s and `WRITE`s at least from our point of view followed by a `Commit` or `Abort` at the end. We have assumed that _<u>writes to disk have no duration</u>_ as opposed to commits or aborts. But in practice, we need to handle the fact that writes also take time.

- Series of `READ`s & `WRITE`s, followed by `commit` or `abort`.

  - We will assume that write is atomic on disk.
    - In practice, additional details to deal with non-atomic `WRITE`s.

- S2PL -> concurrency is correctly handled => Isolated Correctly
- `STEAL`, `NO-FORCE` buffer management, with `Write-Ahead Logging`.

### The Big Picture: What's Stored Where

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/m77xouw.png" alt="image-20230106175408043" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

We have the log that somehow sits in the hypothetical stable storage. It contains all these things, but maybe not all entries, for example, the `pageID`, `length`, `offset`, `old data`, and `new data` are only for the `Update` entries. However, these things are always there.

Then, we have the database, and the databases have been extended by the pointer of `pageLSN` => last modification to this page. We also have the master record.

`flushedLSN` indicates how the log is split between the parts that still in the RAM and the part that lives in the stable storage. Additionally, the log tail should also in the RAM. The two data structures, transaction table and dirty page table sit at memory.

**_Right now, we have to deal with the case where the RAM part is completely gone._**

### Checkpointing

Before I tell you how we deal with the normal operation handle crashes. Here are the two extra log records. They can be seen as an optimization. **_What we want to avoid is that the recovery after crash takes a long time._**

This happens **_when the transaction table and the dirty page table that we are losing_**. If you need to start from the beginning of time when reconstructing those, this will be a problematic. => So we want to **_make this point in time as late as possible from where we need to start working construction the transaction table and the dirty page table._** And that's we will take chekpoints of those and store them also in the log.

_We will initiate a checkpoint and it consists of the transaction table and the dirty page table. Initiating a checkpoint means starting writing these things to the log and once we are done, we create the end checkpoint record which contains the current state of the two tables (XT & DPT)._

_Then, when we recover, we can start from this state as opposed the empty table. So, this is called `fuzzy checkpoint` as it takes the time to write these tables. And the point is fuzziness._

_The other transcations continue to run. What is stored in the `end_checkpoint` record is the snapshot at the moment of the corresponding `begin_checkpoint` record._

_This is the reliable information. From that point (`begin_checkpoint`), we can rely on what is stored at this slightly later point in time and we will be recreating the state of our table starting from the earlier point._

Since this is an optimization trying to bring the time as close to the present from which we need to start, it doesn't actually matter that much that there is a certain operation that happened between the two.

<hr>

**Official Version**

- Periodically, the DBMS creates a <u>checkpoint</u>, to minimize the time taken to recover in the event of a system crash. Write to log:

  - **`begin_checkpoint`** record: Indicates when checkpoint began.

  - **`end_checkpoint`** record: Contains current Xact table and dirty page table. This is a **`fuzzy checkpoint`**:

    - Other Xacts continue to run; so these tables accurate only as of time of the **`begin_checkpoint`** record.

    - **No attempt to force dirty pages to disk**; effectiveness of checkpoint limited by oldest unwritten change to a dirty page, `minDirtyPagesLSN`.

      We are not trting to force dirty pages to disk at any point. For the checkpoint, it is just about recording these tables, nothing about the actual data and the actual data is handled by the background process.

    - Use **background process** to flush dirty pages to disk!

  - Store LSN of `begin_checkpoint` record in a safe place (**_master_** record). => This is the point from which we need to start our recovery.

<hr>

### Transaction Commit

_What should happen at when the transaction commits?_

1. We want to write `commit` record to the log and we want to write this as quickly as possible.
2. Moreover, we want to flush all the records up to this `commit` record to the disk. And it guarantees us that `flushedLSN` will be greater than the `lastLSN` transaction that has committed. It will make sure that the user can rely on the information about the commit made it to the stable storage.
3. These are sequentials and synchronous writes to disk. The log is sequential everywhere.
4. We will try to have multiple of records per single log page. When we are writing a single log page, we are writing a lot of log reference.

After we flush the log, then `Commit()` can return. Then the user gets the notification that "Your transaction has been committed". You can rely that it's durable.

**_Here I did not say anything about the actual data written to disk, this happens in background process._**

Then, we need to update the local state (XT and DPT) of algorithm. After that, you will write the `end` record to the log.

<hr>

**Official Version**

- Write `commit` record to log.
- All log records up to Xact's `lastLSN` are flushed.
  - Guarantees that `flushedLSN >= lastLSN`.
  - Note that log flushes are sequential, synchronous writes to disk.
  - Many log records per log page.
- `Commit()` returns.
- Update the local state.
- Write `end` record to log. => We don't need to flush it immediately.

The transaction is considered as committed as soon as the commit record makes it to the stable storage which is the last entry that's flushed.

<hr>

### Simple Transaction Abort

- For now, consider an explicit abort of a transaction.
  - No crash involved. => **We did not lose our RAM yet.**

What we want to do is UNDOing the effects of the transaction that happens so far. And we essentially will play back the log backwards to UNDO the latest action by the transaction, then the second, and so on, until the very start.

- We want to "play back" the log in **reverse** order, **UNDOing** updates.

**_So, where do we get the lastLSN of the transaction?_** We looked up in the **_transaction table_**.

- Get `lastLSN` of Xact from **XT**.

We look it up to get what the latest update was. **_If it was a standard `update`, we UNDO it by taking the old data entry from the log record_**, and write it to memory. Flush to disk at some time.

Once we've done that, we will follow our linked list backwards => looking at the `prevLSN` of the last log entry that we've UNDOne.

- Can follow chain of log records backward via the `prevLSN` field.

Before we starting doing backwards traverse, we write an `Abort` log record. And this is important when we have a crash during this UNDO phase. Remeber that we wanted to abort this transaction.

- Before starting UNDO, write an **Abort log record**.
  - For recovering from crash during UNDO.

### Abort, cont.

First of all, we will be writing pages, and this is a running transaction. To perform UNDO, we must have a lock on data. But magically, we have the lock on the data because we are using S2PL. We only releasing a lock once the transaction commits. => It will not work if using C2PL as it releases locks gradually.

- To perform UNDO, must have a lock on data!

  - S2PL enforces this

- Before restoring old value of a page, write a CLR log record during the UNDO:

  - You continue logging while you UNDO!!

  - CLR has on extra filed: **`undonextLSN`** => another sequential number

    - Points to the next LSN to undo (i.e., the **`prevLSN`** of the record we're currently undoing).

    - CLRs **_never_** Undone (but they might be Redone when repeating history: guarantees Atomicity!)

      They are never undone like. During Undo phase when we are performing UNDO records, we simply skip over them. But they might need to be redone when we redo these things again.

- At end of UNDO, write an `end` log record. => We reached the first action of the transaction and we will be using like a `NULL` pointer instead of the concrete pointer in the `prevLSN` for any first action performed by the transaction.

### Example

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/od2vjF5.png" alt="image-20230106194011808" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

Solution: This is my RAM below to record what we have in the data structures as we go.

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/2rI6gKZ.png" alt="image-20230107004543282" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

### A Longer Example

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/S1ZctOH.jpeg" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

<font color=red>**_`flushedLSN` has not effect on the tables._**</font>

## Summary

Today is all about the normal execution of a transaction, including transaction commit, and transaction aborts. The next lecture, we are going to talk about **_Crash Recovery_**.
