# Chapter 16 Transactions

- Failure model in Concurrency
- Transactions & Nested transactions
- Solutions to handle conflict
  - Locks
  - Optimistic concurrency control
  - Timestamp ordering

## Failure model in Concurrency

### Lost update problem

| Transaction X                      | Transaction Y                    |
| ---------------------------------- | :------------------------------- |
| Get_balance:$200                   |                                  |
|                                    | Get_balance:      $200           |
| Deposit:$20    =>     balance:$220 | <u>`Lost update`</u>             |
|                                    | Deposit:$20   =>    balance:$220 |

final balance should be $200 + $20 + $20 = $240

### Inconsistent retrivals problem

Initial_account_balance: A: $200 B: $200

Transaction X: transfer $100 from account A to account B

| Transaction X                      | Transaction Y                              |
| ---------------------------------- | ------------------------------------------ |
| A.Withdraw($100) => A.balance=$100 |                                            |
| <u>`B.balance = $200`</u>          | A.balance + B.balance = $100 + $200 = $300 |
| B.deposit($100)  => B.balance=$300 |                                            |

Transaction Y retrives an intermediate state. Total balance should always be $400, no matter $200 + $200 or $100 + $300.

### Serial equivalence

...

## Locks

...

## Optimistic concurrency control

***Working phase*** -> ***Validation phase*** -> ***Update phase***

each transaction is assigned a **transaction number** when it **enters the validation phase** (that is, when the client issues a *closeTransaction*).

If the transaction is validated and completes successfully, it retains this number; if it fails the validation checks and is aborted, or if the transaction is read only, the number is released for reassignment.

a transaction always finishes its **working phase** after all transactions with lower numbers.

a transaction with the number *Ti* always **precedes** a transaction with the number *Tj* if ***i* < *j***.

### Valication rules of transactions

1. *Tv*: write *Ti*: read. 
2. *Tv*: read *Ti*: write.
3. *Tv*: write *Ti*: write.

When no two transactions may overlap in the update phase, **rule 3 is satisfied**.

### Backward validation

Backward validation checks the transaction undergoing validation with other preceding overlapping transactions – those that entered the validation phase before it. 

**As all the read operations of earlier overlapping transactions were performed before the validation of Tv started, they cannot be affected by the writes of the current transaction (and rule 1 is satisfied).**

*Tp*(preceding overlapping transactions) work phase end point - *Tpwe*

*Tv* (the validation transaction) work phase end point - *Tvwe*

Read by *Tp*  -> *Tpwe* -> *Tvwe* -> Update Phase *Tv*

​                                   -> Update Phase *Tp*

if Update Phases of *Tv* and *Tp* are mutexed, Write will not overlapped. (**rule 3 is satisfied**)

check if read set overlaps with any write set of preceding tansactions, **rule 2**. 

```
boolean valid = true;
for (int Ti = startTn+1; Ti <= finishTn; Ti++){
	if (read set of Tv intersects write set of Ti) valid = false; 
}
```

### Afterward validation

...

## Timestamp ordering

each operation in a transaction is validated when ***it is carried out***

A transaction’s request to write an object is valid only if that object was last read and written by earlier transactions. A transaction’s request to read an object is valid only if that object was last written by an earlier transaction.

Each transaction is assigned a unique timestamp value when ***it starts***.

#### Timestamp set

Each object has a a ***write timestamp*** and a set of ***tentative*** versions with a ***corresponding write timestamp*** 

and a set of ***read timestamps***.

The ***write timestamp*** of a commited object is earlier than any ***tentative write timestamp***

the set of ***read timestamps*** can be represented by its *maximum* member.

#### When set a timestamp

Whenever a transaction’s *write* operation on an object is accepted, the server creates a ***new tentative version*** of the object with its write timestamp set to the ***transaction timestamp***.

A transaction’s ***read operation*** is directed to the version with the ***maximum write timestamp less than*** the transaction timestamp.

Whenever a transaction’s *read* operation on an object is accepted, the timestamp of the transaction is added to its set of ***read timestamp***s. 

### Conflict models

1. Tc write, Ti read.  if Ti > Tc, must not write. Requires Tc ≥ max read timestamp(no read after Tc)
2. Tc write, Ti write. if Ti > Tc, must not write. Requires Tc > max write timestamp of committed object
3. Tc read, Ti write. if Ti > Tc, must not read. Requires Tc > max write timestamp of committed object

### Timestamp ordering write rule

```
if (Tc > maximum read timestamp on D &&
    Tc > write timestamp on committed version of D)
  perform write operation on tentative version of D with write timestamp Tc 
else /* write is too late */
  Abort transaction Tc
```

### Timestamp ordering read rule

```

if ( Tc > write timestamp on committed version of D) {
  let Dselected be the version of D with the maximum write timestamp ð Tc 
  if (Dselected is committed)
     perform read operation on the version Dselected 
  else
     wait until the transaction that made version Dselected commits or aborts then reapply 
     the read rule } 
else
  Abort transaction Tc
```

