# Objectives of talk:

## Difficulties of multithreaded programming

### Deadlocks, Livelocks

Two threads, each waiting on each other to finish something, so neither are able to make any progress either by waiting or spinning.

### Data races

The worst problem you can run into---two threads concurrently running (without order guarantees) that manipulate or read the same data. Indeterminate result (in fact, in c++11 standard this means your program execution is undefined---anything could happen).

## Side note: GIL

Data races are less prevalent in python due to the GIL. CPython requires a global lock to execute python code, so all your python code (threads included) is happening in a linear order. By default something like 100 bytecode instructions between context switches---so you might still have data races!.  Makes it harder to run into multithreading issues in Python code---but still possible, and still valid for other languages with true multithreading (also jython). 

## Summary

Writing correct multithreaded code is hard---as a developer you have to consider all of these issues that are tricky to reason about and even harder to debug. 

## What is STM and what is it an attempt to solve?

Software Transactional Memory is an approach to allow the user to ignore race conditions and threaded conflicts. It pushes into the language level a way for user code to be multithreaded-safe. 

### STM in Clojure

Clojure has no variables. Clojure has no mutable data. We'll be talking about STM as implemented in clojure---a way to modify references to immutable data. Important because *once you have some data, it will never change under you*. The STM will control how *references* to this immutable data are allowed to change

    (def r (ref 0))
    (alter r inc)                  => error!
    (dosync (alter r inc))    => success!
    @r                              => @r is now 1
    

#### Transactions

* Atomic to outside observers
* Consistent view of the outside world
* Automatic retries on conflict

**diagram of multithreaded STM usage**

### Benefits

Don't have to think about locking. Just write multithreaded code that modifies refs in a transaction -> just works. Talk about the ants.clj example.

## How is a STM system implemented

** diagram of classes and relationships in human-readable form **

### Ref

* tvals (history chain)
* faults (atomic integer) => talk about atomics
* lock (read write mutex) => talk about mutex & read write locks & recursive locks
* tinfo => come  back to later

## Key hook for refs/STM

can always *get* value of a ref with currentValue()---just get newest in history chain
can only *set* a ref to something new with refSet/alter/commute---and they _check for a running transaction_

### LockingTransaction

* thread-local variables (what are they?) (show usage in LockingTransaction:get()/ensureGet())
* global ordering (count)
* go through key parts:
    * run (runs function, capturing changes to all refs, then tries to commit results)
    * _takeOwnership (associates a ref w/ this transaction---sets tinfo on the ref)
    * _doSet (show how it's very simple: takes ownership, saves value)

# Conclusion / extra questions

* can anyone tell me why clojure's immutable data is required for this STM system to work?
