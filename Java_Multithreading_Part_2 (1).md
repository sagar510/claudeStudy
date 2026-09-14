# Java Multithreading — Part 2
## `synchronized` vs `ReentrantLock` vs `ReadWriteLock`, `volatile`, Java Memory Model & Atomic Variables

---

# 1. Where Part 1 Left Us

In Part 1, we saw this problem:

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }
}
```

`synchronized` protects the critical section.

Conceptually:

```text
Thread A
   ↓
Acquire monitor
   ↓
Critical section
   ↓
Release monitor

Thread B waits for the same monitor
```

This gives us two very important things:

```text
synchronized
    ↓
Mutual Exclusion
    +
Memory Visibility / Ordering
```

But Java provides more advanced locking mechanisms when we need greater control.

That brings us to:

```text
synchronized
ReentrantLock
ReadWriteLock
```

---

# 2. First Understand: What Is a Lock?

Suppose two threads want to modify the same bank account:

```text
             Account
                │
          balance = 1000
                │
        ┌───────┴───────┐
        │               │
    Thread A         Thread B
    withdraw()       deposit()
```

We don't want both threads to modify the shared state in an unsafe way.

So we protect the critical section using a **lock**.

```text
Thread A → acquire lock → modify → release lock

Thread B → wait → acquire lock → modify → release lock
```

A lock essentially controls **who can enter a protected section at a given time**.

---

# 3. `synchronized` Recap

We already know:

```java
public synchronized void increment() {
    count++;
}
```

For an instance method, the monitor is:

```text
this
```

Equivalent conceptually:

```java
public void increment() {
    synchronized (this) {
        count++;
    }
}
```

The JVM automatically handles acquiring and releasing the monitor.

---

# 4. The Nice Thing About `synchronized`

Consider:

```java
synchronized (lock) {
    count++;
}
```

You don't manually write:

```text
lock
...
unlock
```

The monitor is released when execution leaves the synchronized block, including when an exception propagates out of it.

So `synchronized` is simple and difficult to accidentally leave locked.

---

# 5. Then Why Do We Need `ReentrantLock`?

Sometimes we need more control than intrinsic monitors provide.

For example:

```text
Can I attempt to acquire the lock without waiting forever?

Can I wait only 2 seconds?

Can waiting for the lock be interruptible?

Can I request a fair lock policy?

Can I have multiple Condition queues?
```

This is where:

```java
ReentrantLock
```

becomes useful.

It is part of:

```java
java.util.concurrent.locks
```

---

# 6. Basic `ReentrantLock`

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Counter {

    private int count = 0;

    private final Lock lock = new ReentrantLock();

    public void increment() {

        lock.lock();

        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

Flow:

```text
Thread
  ↓
lock.lock()
  ↓
Acquire lock
  ↓
Critical section
  ↓
lock.unlock()
```

---

# 7. Why `try-finally` Is Mandatory Practice

Never casually write:

```java
lock.lock();

count++;

lock.unlock();
```

Why?

Suppose something inside the protected code throws an exception:

```java
lock.lock();

process(); // exception

lock.unlock();
```

`unlock()` might never execute.

Then another thread may keep waiting for that lock.

Correct pattern:

```java
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

⭐ **Interview + production rule:** If you explicitly acquire a `Lock`, release it in `finally`.

---

# 8. What Does "Reentrant" Mean?

This is important.

Suppose a thread already owns a lock.

Can that same thread acquire the same lock again?

With a reentrant lock: **Yes.**

Example:

```java
class Service {

    private final ReentrantLock lock = new ReentrantLock();

    void methodA() {
        lock.lock();
        try {
            methodB();
        } finally {
            lock.unlock();
        }
    }

    void methodB() {
        lock.lock();
        try {
            System.out.println("Working");
        } finally {
            lock.unlock();
        }
    }
}
```

Thread T enters `methodA()`:

```text
T acquires lock
    ↓
methodA()
    ↓
methodB()
    ↓
T acquires SAME lock again
```

This is allowed because the lock is **reentrant**.

Internally, the lock tracks ownership and a hold count.

```text
First lock()  → hold count 1
Second lock() → hold count 2
unlock()      → hold count 1
unlock()      → hold count 0 → actually available
```

Important: Java's intrinsic `synchronized` monitors are also reentrant.

---

# 9. `synchronized` vs `ReentrantLock`

Core comparison:

| Feature | `synchronized` | `ReentrantLock` |
|---|---|---|
| Lock management | JVM / lexical | Explicit |
| Release | Automatic on block exit | Must call `unlock()` |
| `tryLock()` | No | Yes |
| Timed lock attempt | No | Yes |
| Interruptible lock acquisition | Not like `lockInterruptibly()` | Yes |
| Fairness option | No configurable fairness | Yes |
| Multiple conditions | One monitor wait-set | Multiple `Condition`s |
| Reentrant | Yes | Yes |

### Interview answer

> Use `synchronized` when simple mutual exclusion is enough. Use `ReentrantLock` when you need capabilities such as `tryLock()`, timed or interruptible lock acquisition, configurable fairness, or multiple conditions.

Don't say:

> `ReentrantLock` is always faster than `synchronized`.

❌ That is not a good general rule. Modern JVMs optimize synchronization heavily, and performance depends on workload and contention.

---

# 10. `tryLock()` — Very Important

Normal:

```java
lock.lock();
```

If another thread owns the lock, the current thread may wait until it becomes available.

With:

```java
lock.tryLock();
```

Java attempts to acquire the lock **immediately**.

It returns:

```text
true  → lock acquired
false → lock currently unavailable
```

Example:

```java
if (lock.tryLock()) {
    try {
        // critical section
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Could not acquire lock");
}
```

This is useful when you don't want a thread to wait indefinitely.

---

# 11. Timed `tryLock()`

We can also wait for a limited time:

```java
if (lock.tryLock(2, TimeUnit.SECONDS)) {
    try {
        // work
    } finally {
        lock.unlock();
    }
}
```

Meaning:

```text
Try to get lock
      ↓
Wait at most 2 seconds
      ↓
Acquired? → work
Not acquired? → take alternative action
```

This becomes especially useful in **deadlock-avoidance strategies**, which we'll cover deeply in Part 4.

---

# 12. `lockInterruptibly()`

Normal lock acquisition:

```java
lock.lock();
```

`ReentrantLock` also supports:

```java
lock.lockInterruptibly();
```

This lets a thread waiting to acquire the lock respond to interruption.

Example:

```java
try {
    lock.lockInterruptibly();
    try {
        // critical section
    } finally {
        lock.unlock();
    }
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

This can be useful in cancellation-sensitive concurrent systems.

---

# 13. Fair vs Unfair `ReentrantLock`

Default:

```java
new ReentrantLock();
```

is normally **non-fair**.

You can request fairness:

```java
new ReentrantLock(true);
```

Conceptually, fairness attempts to favor threads that have been waiting longer.

```text
T1 waiting
T2 waiting
T3 waiting

Fair policy → roughly queue-oriented acquisition
```

But fairness can reduce throughput.

⭐ Interview point:

> Fairness can reduce starvation risk, but may cost performance. It does not mean thread scheduling becomes perfectly predictable.

---

# 14. The Next Problem: Many Readers, Few Writers

Imagine a cache:

```java
Map<String, String> cache
```

Suppose:

```text
100 threads → reading
2 threads   → writing
```

If we use one exclusive lock:

```text
Reader 1 acquires lock
Reader 2 waits
Reader 3 waits
Reader 4 waits
...
```

But multiple readers aren't modifying the data.

Why prevent safe readers from reading concurrently?

This leads to:

```java
ReadWriteLock
```

---

# 15. `ReadWriteLock`

Java provides:

```java
ReadWriteLock
```

A common implementation is:

```java
ReentrantReadWriteLock
```

It conceptually provides two locks:

```text
ReadWriteLock
     │
     ├── Read Lock
     │
     └── Write Lock
```

---

# 16. Read Lock

Multiple readers can generally hold the **read lock simultaneously**, provided no writer owns the write lock.

```text
Reader 1 ─┐
Reader 2 ─┼──→ Shared Data
Reader 3 ─┘
```

All can read concurrently.

---

# 17. Write Lock

A writer needs exclusive access.

```text
          Writer
             │
        WRITE LOCK
             │
        Shared Data
```

While a writer holds the write lock:

```text
Readers → wait
Other writers → wait
```

---

# 18. `ReadWriteLock` Example

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class Cache {

    private int value;

    private final ReadWriteLock rwLock =
            new ReentrantReadWriteLock();

    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();

    public int read() {
        readLock.lock();
        try {
            return value;
        } finally {
            readLock.unlock();
        }
    }

    public void write(int newValue) {
        writeLock.lock();
        try {
            value = newValue;
        } finally {
            writeLock.unlock();
        }
    }
}
```

---

# 19. Read/Write Compatibility

Remember this table:

| Existing Lock | New Reader | New Writer |
|---|---:|---:|
| Read | Usually allowed | Wait |
| Write | Wait | Wait |

Simplified:

```text
Read + Read   → YES
Read + Write  → NO
Write + Read  → NO
Write + Write → NO
```

This is the key idea behind read-write locks.

---

# 20. When Is `ReadWriteLock` Useful?

Best suited when:

```text
Reads >> Writes
```

For example:

```text
Configuration store
Cache-like shared state
Read-heavy in-memory structures
```

But don't automatically use it everywhere.

Read-write locking introduces extra complexity and overhead. For small critical sections or frequent writes, a simple lock may be better.

---

# 21. `synchronized` vs `ReentrantLock` vs `ReadWriteLock`

This is one of your must-know interview comparisons.

| Mechanism | Main Idea |
|---|---|
| `synchronized` | Simple intrinsic mutual exclusion |
| `ReentrantLock` | Explicit exclusive lock with advanced controls |
| `ReadWriteLock` | Separate shared read and exclusive write locking |

Think:

```text
Simple critical section
        ↓
   synchronized

Need tryLock / timeout / fairness / interruptibility
        ↓
   ReentrantLock

Many reads + few writes
        ↓
   ReadWriteLock
```

---

# 22. Now Understand the Second Big Problem: Visibility

So far we focused mainly on multiple threads modifying the same value.

But concurrency has another important problem:

```text
Thread A changes a value

Will Thread B definitely observe that change correctly?
```

This is the **visibility problem**.

---

# 23. Simple Visibility Example

Consider:

```java
class Worker {

    boolean running = true;

    void work() {
        while (running) {
            // do work
        }
    }

    void stop() {
        running = false;
    }
}
```

Thread A executes:

```java
worker.work();
```

Thread B executes:

```java
worker.stop();
```

You may assume:

```text
Thread B sets running = false
              ↓
Thread A immediately sees false
              ↓
loop stops
```

Without proper synchronization, that observation is **not something you should rely on** under the Java Memory Model.

---

# 24. Why Can Visibility Be a Problem?

Modern systems involve multiple optimization layers:

```text
CPU registers
CPU caches
Compiler/JIT optimizations
Main memory
```

Conceptually, one thread may keep working with a value without being forced to observe another thread's write in the way your source-code intuition expects.

The Java Memory Model defines the rules for when writes by one thread are guaranteed to become visible to another.

---

# 25. `volatile`

We can declare:

```java
private volatile boolean running = true;
```

Now:

```java
void stop() {
    running = false;
}
```

and:

```java
while (running) {
    // work
}
```

have the required volatile memory semantics.

A write to a volatile variable **happens-before** a subsequent read of that same volatile variable.

For interview understanding:

```text
Thread A writes volatile variable
          ↓
Thread B subsequently reads it
          ↓
Thread B sees the write according to JMM guarantees
```

---

# 26. What Does `volatile` Guarantee?

The two important ideas are:

### 1. Visibility

Changes to the volatile variable are properly published between threads according to Java Memory Model rules.

### 2. Ordering

Volatile reads/writes establish memory-ordering constraints and a happens-before relationship.

For most interviews, remember:

```text
volatile
   ↓
Visibility + ordering
```

---

# 27. The Biggest `volatile` Interview Trap

Does `volatile` make:

```java
count++;
```

atomic?

### **NO.**

Even if:

```java
volatile int count = 0;
```

this:

```java
count++;
```

is still conceptually:

```text
READ
 ↓
ADD 1
 ↓
WRITE
```

Two threads can still interleave.

---

# 28. Example: `volatile` Does NOT Fix Lost Updates

Suppose:

```java
volatile int count = 5;
```

Thread A:

```text
read 5
```

Thread B:

```text
read 5
```

Thread A:

```text
write 6
```

Thread B:

```text
write 6
```

Final:

```text
6
```

Expected:

```text
7
```

So:

```text
volatile ≠ atomicity for compound operations
```

🔥 This is one of the most frequently asked Java concurrency interview points.

---

# 29. `volatile` vs `synchronized`

Simplified comparison:

| Feature | `volatile` | `synchronized` |
|---|---:|---:|
| Visibility | Yes | Yes |
| Ordering guarantees | Yes | Yes |
| Mutual exclusion | No | Yes |
| Makes `count++` atomic | No | Protected inside lock: Yes |
| Thread blocking for lock | No | Can block |

Think:

```text
Need threads to see latest state / publish a simple flag
        ↓
volatile may be enough

Need read-modify-write critical section
        ↓
synchronization / lock / atomic class
```

---

# 30. Good Use Case for `volatile`

A shutdown/status flag is a classic example:

```java
class Worker implements Runnable {

    private volatile boolean running = true;

    @Override
    public void run() {
        while (running) {
            // process work
        }
    }

    public void stop() {
        running = false;
    }
}
```

Why can this work?

Because we're primarily communicating a state change through a volatile variable rather than performing a compound update like:

```java
count++;
```

---

# 31. What Is Atomicity?

An operation is **atomic** when other threads cannot observe/interleave it as partially completed from the perspective of that operation.

Conceptually:

```text
Atomic operation

START ─────────────> END

Other thread cannot interleave inside that logical operation
```

For example, a compound increment requires an atomic read-modify-write if multiple threads are updating it concurrently.

---

# 32. Atomic Classes

Java provides atomic classes under:

```java
java.util.concurrent.atomic
```

Examples:

```text
AtomicInteger
AtomicLong
AtomicBoolean
AtomicReference
```

For our counter:

```java
AtomicInteger count = new AtomicInteger(0);
```

Then:

```java
count.incrementAndGet();
```

performs an atomic increment operation.

---

# 33. `AtomicInteger` Example

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {

    private final AtomicInteger count =
            new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();
    }

    public int getCount() {
        return count.get();
    }
}
```

Now two threads can call:

```java
counter.increment();
```

without wrapping that increment in `synchronized`.

---

# 34. Common `AtomicInteger` Methods

```java
get()
set(value)
incrementAndGet()
getAndIncrement()
decrementAndGet()
addAndGet(value)
compareAndSet(expected, update)
```

Difference:

```java
incrementAndGet()
```

increments first and returns the **new value**.

```java
getAndIncrement()
```

returns the **old value**, then increments.

Example:

```java
AtomicInteger x = new AtomicInteger(5);

System.out.println(x.incrementAndGet());
```

Output:

```text
6
```

Whereas:

```java
AtomicInteger x = new AtomicInteger(5);

System.out.println(x.getAndIncrement());
```

Output:

```text
5
```

Afterward, `x` contains `6`.

---

# 35. How Do Atomic Classes Work?

A key idea is **CAS**:

```text
Compare-And-Set / Compare-And-Swap
```

Conceptually:

```text
Current value = 5

I expect = 5
I want   = 6

Is current still 5?
       ↓
YES → update to 6
NO  → operation fails/retries as appropriate
```

This allows many atomic operations to avoid a traditional exclusive lock around each update.

---

# 36. CAS Example

```java
AtomicInteger count = new AtomicInteger(5);

boolean updated = count.compareAndSet(5, 10);
```

If current value is `5`:

```text
updated = true
count = 10
```

If current value is not `5`:

```text
updated = false
count remains unchanged
```

---

# 37. `volatile` vs `AtomicInteger`

Suppose:

```java
volatile int count = 0;
```

and:

```java
count++;
```

❌ Not safe for concurrent increments.

But:

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();
```

provides an atomic increment.

Remember:

```text
volatile
   ↓
Visibility / ordering

AtomicInteger
   ↓
Visibility semantics + atomic operations
```

---

# 38. Java Memory Model — Interview-Level Understanding

You do not need to start by memorizing CPU architecture.

Understand the problem:

```text
Thread A
   ↓
Writes shared data

Thread B
   ↓
Reads shared data
```

Question:

> Under what conditions is Thread B guaranteed to observe Thread A's actions correctly?

The **Java Memory Model (JMM)** defines rules for visibility, ordering, and synchronization between threads.

---

# 39. What Is "Happens-Before"?

This term is extremely important in interviews.

If action A **happens-before** action B, the JMM guarantees the required visibility and ordering of A's effects to B.

Don't interpret it merely as:

```text
A happened earlier according to wall-clock time
```

It is a **memory-ordering relationship** defined by the Java Memory Model.

---

# 40. Important Happens-Before Relationships

You should know these.

### 1. Monitor unlock → subsequent lock

```text
Thread A
synchronized(lock) {
    data = 10;
}

        ↓ monitor release/acquire relationship

Thread B
synchronized(lock) {
    read data;
}
```

An unlock on a monitor happens-before every subsequent lock on that same monitor.

---

### 2. Volatile write → subsequent volatile read

```java
volatile boolean ready;
```

Thread A:

```java
ready = true;
```

Thread B subsequently reads:

```java
if (ready) {
    ...
}
```

A write to a volatile field happens-before every subsequent read of that same field.

---

### 3. `Thread.start()`

Actions in the thread that calls:

```java
t.start();
```

before the `start()` call happen-before actions in the started thread.

---

### 4. Thread termination / `join()`

All actions performed by a thread happen-before another thread successfully returns from `join()` on that thread.

This is one reason `join()` is not merely about execution order—it also has memory-visibility semantics.

---

# 41. Very Important Example

```java
class Data {
    int value = 0;
    volatile boolean ready = false;
}
```

Thread A:

```java
data.value = 100;
data.ready = true;
```

Thread B:

```java
if (data.ready) {
    System.out.println(data.value);
}
```

If Thread B observes the volatile write by reading `ready == true`, the happens-before relationship makes the preceding write to `value` visible as required.

This demonstrates why `volatile` is about more than simply "read from main memory."

---

# 42. Don't Explain `volatile` as Only "Main Memory"

A common beginner answer is:

> volatile forces every read/write directly to main memory.

This is an oversimplification and not the best interview explanation.

Better:

> `volatile` provides visibility and ordering guarantees under the Java Memory Model. A write to a volatile variable happens-before a subsequent read of that variable, but volatile does not provide mutual exclusion or make compound operations such as `count++` atomic.

⭐ Much stronger interview answer.

---

# 43. `synchronized` vs `ReentrantLock` vs `ReadWriteLock` vs `volatile` vs Atomic

Use this mental model:

```text
Need simple mutual exclusion?
        ↓
 synchronized

Need advanced exclusive locking?
        ↓
 ReentrantLock

Read-heavy shared structure?
        ↓
 ReadWriteLock

Need visibility of simple state / flag?
        ↓
 volatile

Need atomic single-variable operations?
        ↓
 AtomicInteger / AtomicLong / etc.
```

---

# 44. Interview Scenario 1

### "I have a boolean `shutdown` flag. One thread changes it, another checks it. What would you use?"

A common answer:

```java
volatile boolean shutdown;
```

because the main concern is visibility of a simple state transition.

---

# 45. Interview Scenario 2

### "100 threads increment the same counter. Can I use `volatile int`?"

**No.**

Because:

```java
count++
```

is a compound read-modify-write operation.

Use something such as:

```java
AtomicInteger
```

or synchronization, depending on the broader operation.

---

# 46. Interview Scenario 3

### "I need to update multiple related fields together. AtomicInteger?"

Suppose:

```java
balance -= amount;
transactionCount++;
lastUpdated = now;
```

These multiple operations may need to maintain one shared invariant.

An `AtomicInteger` only makes its own supported operations atomic; it does not automatically make a larger multi-field transaction atomic.

You may need:

```text
synchronized
or
Lock
```

to protect the complete critical section.

---

# 47. Interview Scenario 4

### "I want to acquire two locks but don't want to wait forever."

This points toward:

```java
ReentrantLock.tryLock()
```

Potential pattern:

```text
Try Lock A
   ↓
Try Lock B
   ↓
If B unavailable
   ↓
Release A
   ↓
Retry / back off / fail
```

We'll use this when discussing deadlock prevention in Part 4.

---

# 48. Interview Scenario 5

### "My application has a shared cache with extremely frequent reads and rare writes."

A possible option:

```java
ReadWriteLock
```

because multiple readers may proceed concurrently while writers get exclusive access.

But in a real system, also consider whether a concurrent collection such as `ConcurrentHashMap` better matches the use case. We'll cover concurrent utilities later.

---

# 49. Common Interview Traps 🔥

### Trap 1

> `volatile` makes every operation thread-safe.

❌ False.

---

### Trap 2

> `count++` becomes atomic if `count` is volatile.

❌ False.

---

### Trap 3

> `ReentrantLock` must always be preferred over `synchronized`.

❌ False.

---

### Trap 4

> `ReadWriteLock` allows one writer and readers simultaneously.

❌ Normally the write lock is exclusive against readers and other writers.

---

### Trap 5

> `synchronized` is not reentrant.

❌ False. Java intrinsic monitors are reentrant.

---

### Trap 6

> Calling `lock()` automatically unlocks when the method ends.

❌ False for explicit `Lock`. You must call `unlock()`, normally in `finally`.

---

# 50. Interview Questions — Part 2

## Q1. What is `ReentrantLock`?

A lock implementation providing explicit locking with advanced features such as `tryLock()`, timed and interruptible acquisition, fairness configuration, and `Condition`s.

---

## Q2. Why is it called reentrant?

Because the thread that already owns the lock can acquire that same lock again. The lock maintains a hold count and becomes available only after matching unlocks.

---

## Q3. Is `synchronized` reentrant?

**Yes.**

---

## Q4. Why use `try-finally` with `ReentrantLock`?

To ensure the lock is released even if the protected code throws an exception.

---

## Q5. `synchronized` vs `ReentrantLock`?

`synchronized` is simpler and automatically releases its monitor. `ReentrantLock` gives explicit control and features such as `tryLock()`, timed/interruptible acquisition, fairness, and multiple conditions.

---

## Q6. What is `ReadWriteLock`?

It separates locking into a shared read lock and an exclusive write lock, which can improve concurrency for read-heavy workloads.

---

## Q7. Can two readers hold a read lock simultaneously?

**Yes**, normally, when no conflicting writer owns the write lock.

---

## Q8. Can two writers hold the write lock simultaneously?

**No.**

---

## Q9. What does `volatile` guarantee?

Primarily **visibility and ordering guarantees** under the Java Memory Model.

---

## Q10. Does `volatile` guarantee atomicity?

Not for compound operations such as:

```java
count++;
```

---

## Q11. What is `AtomicInteger`?

A class providing thread-safe atomic operations on an integer, such as increment, decrement, add, and compare-and-set.

---

## Q12. What is CAS?

**Compare-And-Set**: update a value only if it still equals an expected value.

---

## Q13. What is the Java Memory Model?

It defines rules governing how threads interact through memory, including visibility, ordering, and synchronization guarantees.

---

## Q14. What is happens-before?

A Java Memory Model relationship that guarantees the required ordering and visibility of one action's effects to another action.

---

# 51. Predict the Behaviour 🔥

## Question 1

```java
volatile int count = 0;

// 100 threads
count++;
```

Is the final value guaranteed?

### Answer

**No.**

`volatile` doesn't make the compound increment atomic.

---

## Question 2

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();
```

Is the increment atomic?

### Answer

**Yes.**

---

## Question 3

```java
Lock lock = new ReentrantLock();

lock.lock();
try {
    process();
} finally {
    lock.unlock();
}
```

Why `finally`?

### Answer

So the lock is released even when `process()` throws an exception.

---

## Question 4

```java
ReadWriteLock rw = new ReentrantReadWriteLock();

Lock r1 = rw.readLock();
Lock r2 = rw.readLock();
```

Can two threads generally acquire the read lock together?

### Answer

**Yes**, assuming there is no conflicting writer.

---

## Question 5

```java
ReentrantLock lock = new ReentrantLock();

if (lock.tryLock()) {
    try {
        // work
    } finally {
        lock.unlock();
    }
}
```

What happens if another thread owns the lock?

### Answer

`tryLock()` returns `false` immediately instead of waiting indefinitely.

---

# ⭐ 30-Second Interview Answer

> Java provides several concurrency mechanisms depending on the requirement. `synchronized` is useful for simple mutual exclusion and also provides visibility guarantees. `ReentrantLock` provides explicit locking with advanced capabilities such as `tryLock()`, timed or interruptible acquisition, fairness, and conditions. `ReadWriteLock` allows concurrent readers while keeping writes exclusive, making it useful for read-heavy workloads. `volatile` provides visibility and ordering guarantees but not mutual exclusion, so it does not make compound operations like `count++` atomic. For atomic single-variable operations, Java provides classes such as `AtomicInteger`, commonly implemented using compare-and-set techniques.

---

# Quick Revision

```text
synchronized
    ↓
Intrinsic monitor
    ↓
Mutual exclusion + visibility
    ↓
Simple and automatic release

ReentrantLock
    ↓
Explicit lock
    ↓
lock() / unlock()
    ↓
tryLock()
Timed acquisition
Interruptible acquisition
Fairness option
Conditions

ReadWriteLock
    ↓
Read Lock + Write Lock
    ↓
Read + Read   → allowed
Read + Write  → blocked
Write + Read  → blocked
Write + Write → blocked

volatile
    ↓
Visibility + ordering
    ↓
NO mutual exclusion
    ↓
count++ still NOT atomic

AtomicInteger
    ↓
Atomic single-variable operations
    ↓
CAS

Java Memory Model
    ↓
Visibility + Ordering + Synchronization rules

Happens-Before
    ↓
Memory-ordering / visibility guarantee
```

---

# ⭐ Part 2 Priority

Master these for interviews:

1. **`synchronized` vs `ReentrantLock`**
2. **Why explicit locks need `try-finally`**
3. **What reentrant means**
4. **`tryLock()` and timed `tryLock()`**
5. **Fair vs unfair lock basics**
6. **`ReadWriteLock` and read-heavy use cases**
7. **`volatile` = visibility/order, not compound-operation atomicity**
8. **Why `volatile count++` is unsafe**
9. **`AtomicInteger` and CAS**
10. **Java Memory Model basics**
11. **Happens-before relationships**
12. **Choosing the correct concurrency mechanism in an interview scenario**

Next: **Part 3 — `ExecutorService`, FixedThreadPool, CachedThreadPool, `ThreadPoolExecutor` internals, task queues, `Callable`, `Future`, and concurrent utilities.**
