---
layout: distill
title:  Recovery > Crash
date: 2023-01-07 15:57:30
description: This is the special case of the recovery when the system crashes.
tags: acs
categories: study ucph
Related_posts: true
img: assets/img/crash.png

authors:
  - name: Ying Liu
    affiliations:
      name: DIKU, UCPH
---

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/D0YTFXX.png" style="zoom: 80%" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

## Recovery: Crash

**RAM**

*What is the Transaction Table (XT)?*

*What is the Dirty Page Table (DPT)? => The first modification of this page that has not been written to disk (recLSN).*

*What is the `flushedLSN`? => It only refers to the log and recall that the log is also split between the memory and the disk. => Denotes which part lives in memory, and which part lives in the disk.* 

**DISK**

*What is the master record? => It is just a log sequence number and we need to store it a bit more securely to facilitate a speedy recovery. => Master record refers to the beginning of the last checkpoint. Because this will be the point from which we need to start the recovery.*

### Crash Recovery: Big Picture

***What happens after the crash or what should happen?***

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/aMm0Hww.png" alt="image-20230107155730658" style="zoom:33%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

<div align=center><i><b>We can crash at anytime including during the recovery phase!</b></i></div>

The recovery looks like and it is subdivided three phases, the `ARU` phases.

`A` => Analysis, `R` => Redo, `U` => Undo, and they are executed in the same order.

The arrows' direction is the direction in *which they go through the log in their own phase*.

It does not have to be the case that one is before the other same with the analysis phase. ***The length of the arrow is not indicative, in particular, the UNDO phase could stop earlier than the point where the REDO phase starts .***

<hr>


<font color=red>Analysis Phase</font>

What happens at the moment of the crash is that ***we'll lose our dirty page table and our transaction table***. That's some important pieces of information that we need. Also, this is the purpose of the ***Analysis*** phase to ***reconstruct these tables as closely as possible to the moment of the crash***. Once these are reconstructed, we can find out which transactions have actually committed and which have failed and should be aborted. => Task is to reconstruct these tables. ***The reconstruction will start from the last checkpoint as the checkpoint contains the snapshot of the tables that is valid at the moment of the `begin_checkpoint` which we know by the master record.*** Therefore, we go through the log in that direction and update our tables according to what we find there.

<font color=red>REDO Phase</font>

Then, the second phase *comes after we reconstructed those tables* and we redo **all** the actions. We are not looking whether a transaction succeeded or failed. Irrespective of that, we will be performing the actions that our log contains. 

***To find out where we need to start REDOing actions***, we look in the ***dirty page table*** and find the ***smallest*** `recLSN` which is the smallest timestamp that made some page dirty. We redo all actions until the current point or until the end of our log.

<font color=red><b>UNDO Phase</b></font>

Undo the transactions that have not been successfully committed yet. For that, we will proceed as in the normal operation in the normal abort of the transaction as last time. [Recovery | Normal](https://yingliu.bio/advanced-computer-system/recovernormal)

<hr>


### Recovery: The Analysis Phase

+ ***Reconstruct state at checkpoint***

  + via `end_checkpoint` record as it contains the tables (XT & DPT) which are valid at the moment of the `begin_checkpoint`.

+ Scan log forward from checkpoint => (`begin_checkpoint`), go through all the actions and perform the following actions. We are not actually executing any writes here but those are only modifications to the two tables that we have.

  When we encounter,

  + `End` record: Remove the transaction from the transaction table.
  + Other records: Add transaction to the transaction table, set `lastLSN = LSN`
  + `Commit` record, change the transaction status
  + `Update` record: If P not in Dirty Page Table, 
    + Add P to Dirty Page Table, set its `recLSN = LSN`

  Note that precisely those modifications to the tables happen also when we execute the system in the normal execution, but we are not performing any writes yet. It's just modifications of the two tables.

### Recovery: The REDO Phase

We ***repeat History*** to reconstruct state at crash:

+ Reapply all updates (even of aborted transactions!), redo CLRs. (CLRs which stand for compensation log for records are records that are written when we are in the process of undoing a transaction)

+ Scan forward from log record containing the ***smallest*** `recLSN` in Dirty Page Table. For each ***CLR*** or ***update*** log record with `LSN`, REDO the action **unless**:

  + Affected page is ***not in the Dirty Page Table***, or

    That's because if it is not in the DPT, it means it has ***already been flushed to the disk beforehand***, and ***that version is newer*** than what we are doing here.

  + Affected page is in ***Dirty Page Table, but has `recLSN > LSN`***, or ***`pageLSN (in DB) >= LSN`.***

    Either of these conditions means that we have newer data in the database. Somehow we are processing the log from top to bottom but it means that further late time, ***there will be another action that will modify the same page and this modification will have been propagated to the disk already***.

  If one of the two occurs, just skip => however, this is only the ***optimization***. We can definitely redo everything, but this is a very easy way to figure out whether this redo is needed or not.

+ To **REDO** an action:

  + Reapply logged action.
  + Set `pageLSN` to `LSN`. No additional logging except the end log record! (Why?) => The recovery can still fail.
  + Write `End` log record.

### Recovery: The UNDO Phase

<div align=center><b>ToUndo = { lsn | lsn a lastLSN of a "loser" Xact}</b></div>

Abort for all losers.

Loser: Those are the ones that are not in the active in the transaction table after REDOing phase.

**Repeat:**

+ Choose largest LSN among **ToUndo**.
+ If this LSN is a **CLR** and **undonextLSN == NULL** ==> There is no previous thing for this transaction to undo. We have undone all actions of the transaction.
  + Write an **End** record for this Xact
+ If this LSN is a **CLR**. and **undonextLSN != NULL** 
  + Add **undonextLSN** to **ToUndo**
+ Else this LSN is an **update**. Undo the update, write a CLR, add **prevLSN** to **ToUndo**.

Until **ToUndo** is empty.

According to the home assignment, the committed transactions before crash are winners.


<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/aMm0Hww.png" alt="image-20230107155730658" style="zoom:33%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>


<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/HOB9twW.png" alt="image-20230107203841645" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>

### Additional Crash Issues

***What happens if system crashes during ANALYSIS? During REDO?***

If exactly happens during ANALYSIS when we are in the process of reconstructing the table from the previous crash, we will ***reconstruct the tables again from the same information what we have.***

**REDO**

We are performing anything that is recorded in the log yet. We will redo all the actions again from scratch. But there is a little bit difference from the above. If there is something already propagated to disk, there is no need to redo it. But in principle, we assume everything should be REDOne.

***How do you limit the amount of the work in REDO?***

+ Flush asynchronously in the background. => Flush often do help!
+ Watch "hot spots"! => If there are a lot of transactions active at some time, flush them to disk often!

***How do you limit the amount of the work in UNDO?***

+ Avoid long-running transactions as we need to go back to the start of every loser transaction.



## A Short Summary of Before

Strong modularity is a kind of fault tolerant.

<div align=center>
  <div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
      <img src="https://i.imgur.com/oDBoceD.png" alt="image-20230107224641524" style="zoom:50%;" class="img-fluid rounded z-depth-1" />
    </div>
  </div>
</div>
