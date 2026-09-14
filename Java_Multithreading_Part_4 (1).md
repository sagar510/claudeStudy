# Java Multithreading — Part 4
## Deadlocks, Prevention, Starvation, Livelock, Thread-Safe Singleton & Interview Patterns

---

# 1. What Is a Deadlock?

A **deadlock** happens when two or more threads are permanently waiting for each other to release resources.

Simple example:

```text
Thread 1 owns Lock A
Thread 2 owns Lock B

Thread 1 waits for Lock B
Thread 2 waits for Lock A
```

Neither thread can continue.

```text
Thread 1                     Thread 2
   │                            │
   ↓                            ↓
Lock A acquired             Lock B acquired
   │                            │
   ↓                            ↓
Wait for Lock B            Wait for Lock A
   │                            │
   └──────── WAIT ──────────────┘
```

Both wait forever.

🔥 This is the classic deadlock situation.

---

# 2. Classic Java Deadlock Example

```java
public class Main {

    private static final Object lockA = new Object();
    private static final Object lockB = new Object();

    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {

            synchronized (lockA) {

                System.out.println("T1 acquired Lock A");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }

                synchronized (lockB) {
                    System.out.println("T1 acquired Lock B");
                }
            }
        });

        Thread t2 = new Thread(() -> {

            synchronized (lockB) {

                System.out.println("T2 acquired Lock B");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }

                synchronized (lockA) {
                    System.out.println("T2 acquired Lock A");
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

Possible situation:

```text
T1 acquires A
T2 acquires B

T1 wants B
T2 wants A
```

Now:

```text
T1 waits for T2
T2 waits for T1
```

Deadlock.

---

# 3. Why Did the Deadlock Happen?

Look at lock acquisition order.

Thread 1:

```text
A → B
```

Thread 2:

```text
B → A
```

The order is inconsistent.

This creates a circular dependency:

```text
Thread 1
   ↓
owns A
   ↓
needs B
   ↓
owned by Thread 2
   ↓
needs A
   ↓
owned by Thread 1
```

Cycle created.

---

# 4. Four Necessary Conditions for Deadlock

A classic interview question.

Deadlock requires these four conditions.

## 1. Mutual Exclusion

A resource can be held by only one thread at a time.

```text
Lock A
   ↓
Thread 1 only
```

---

## 2. Hold and Wait

A thread holds one resource while waiting for another.

```text
Thread 1

holds A
+
waits for B
```

---

## 3. No Preemption

A lock cannot simply be forcibly taken away from another thread.

The owning thread must release it.

---

## 4. Circular Wait

A cycle exists.

```text
T1 waits for T2
T2 waits for T1
```

or:

```text
T1 → T2 → T3 → T1
```

⭐ Deadlock can be prevented by breaking at least one of these conditions.

---

# 5. Most Important Prevention Technique — Consistent Lock Ordering

Suppose every thread follows:

```text
Always acquire A first
Then acquire B
```

Thread 1:

```text
A → B
```

Thread 2:

```text
A → B
```

Now Thread 1 may acquire A first.

Thread 2 waits for A.

But Thread 2 does **not** acquire B while waiting.

Flow:

```text
T1:
A acquired
   ↓
B acquired
   ↓
work
   ↓
release B
release A

T2:
waiting for A
   ↓
A acquired
   ↓
B acquired
```

No circular wait.

---

# 6. Fixed Deadlock Example

Instead of:

```java
// Thread 1
synchronized (lockA) {
    synchronized (lockB) {
    }
}

// Thread 2
synchronized (lockB) {
    synchronized (lockA) {
    }
}
```

use the same ordering everywhere:

```java
// Thread 1
synchronized (lockA) {
    synchronized (lockB) {
    }
}

// Thread 2
synchronized (lockA) {
    synchronized (lockB) {
    }
}
```

Both use:

```text
A → B
```

🔥 **Consistent lock ordering is one of the best practical deadlock-prevention techniques.**

---

# 7. Real Backend Example — Bank Transfer

Suppose:

```java
transfer(accountA, accountB, amount);
```

To safely transfer money, you may need to lock both accounts.

Thread 1:

```text
Transfer A → B
```

Thread 2:

```text
Transfer B → A
```

If Thread 1 locks A first and Thread 2 locks B first:

```text
T1 owns A → waits B
T2 owns B → waits A
```

Deadlock.

---

# 8. Ordering Locks Using IDs

A common solution is to establish deterministic ordering.

Example:

```java
Account first;
Account second;

if (from.getId() < to.getId()) {
    first = from;
    second = to;
} else {
    first = to;
    second = from;
}

synchronized (first) {

    synchronized (second) {

        // perform transfer

    }
}
```

Now every thread acquires accounts based on:

```text
smaller ID
   ↓
larger ID
```

rather than transfer direction.

So both:

```text
A → B
B → A
```

still acquire locks in the same global order.

---

# 9. Deadlock Prevention Using `tryLock()`

With:

```java
synchronized
```

a thread waits until the monitor becomes available.

But:

```java
ReentrantLock
```

provides:

```java
tryLock()
```

Example:

```java
if (lock.tryLock()) {

    try {

        // critical section

    } finally {

        lock.unlock();

    }
}
```

If the lock is unavailable:

```text
tryLock()
   ↓
returns false
```

instead of waiting forever.

---

# 10. Two Locks With `tryLock()`

Example:

```java
Lock lockA = new ReentrantLock();
Lock lockB = new ReentrantLock();

boolean acquiredA = false;
boolean acquiredB = false;

try {

    acquiredA = lockA.tryLock();

    if (acquiredA) {

        acquiredB = lockB.tryLock();

        if (acquiredB) {

            // work requiring both locks

        }
    }

} finally {

    if (acquiredB) {
        lockB.unlock();
    }

    if (acquiredA) {
        lockA.unlock();
    }
}
```

If the second lock cannot be acquired:

```text
release first lock
   ↓
retry later
```

This can avoid indefinite circular waiting.

---

# 11. `tryLock()` With Timeout

Even better:

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
Try acquiring lock
   ↓
Wait at most 2 seconds
   ↓
Success → work
Failure → continue alternative path
```

This provides bounded waiting.

---

# 12. Important `tryLock()` Interview Point

Don't say:

> `tryLock()` automatically solves every deadlock.

❌ Incorrect.

It **gives you the ability to avoid indefinite waiting**, but your code must correctly:

```text
detect failure
release already acquired locks
retry / back off / fail gracefully
```

Bad locking logic can still create problems.

---

# 13. Deadlock Detection

Suppose a production Java application hangs.

Possible symptoms:

```text
Requests stop completing
CPU may be low
Threads remain blocked
Application still running
```

One important debugging technique is a:

```text
thread dump
```

A thread dump shows:

```text
Thread states
Stack traces
Locks owned
Locks being waited for
```

---

# 14. Using `jstack`

A common JVM diagnostic tool:

```bash
jstack <pid>
```

Example:

```bash
jstack 12345
```

The output can show blocked threads and may explicitly report a detected Java-level deadlock.

Conceptually:

```text
Thread-1
  waiting to lock B
  owns A

Thread-2
  waiting to lock A
  owns B
```

That immediately reveals the cycle.

---

# 15. Detecting Deadlocks Programmatically

Java also provides:

```java
ThreadMXBean
```

from:

```java
java.lang.management
```

Example:

```java
ThreadMXBean bean =
        ManagementFactory.getThreadMXBean();

long[] deadlockedThreads =
        bean.findDeadlockedThreads();
```

If:

```java
deadlockedThreads != null
```

a deadlock has been detected among supported synchronizers/monitors.

This is more advanced, but useful interview knowledge.

---

# 16. Deadlock Prevention Checklist

In real applications:

```text
1. Keep lock scope small
2. Avoid unnecessary nested locks
3. Acquire multiple locks in consistent order
4. Use tryLock() where appropriate
5. Avoid calling unknown/external code while holding locks
6. Avoid long blocking operations while holding locks
7. Use higher-level concurrent utilities when possible
```

---

# 17. Why Avoid External Calls While Holding a Lock?

Bad example:

```java
synchronized (lock) {

    paymentApi.call();

}
```

Suppose the API takes:

```text
10 seconds
```

Your lock remains held for the entire call.

Other threads needing the same lock:

```text
wait
wait
wait
```

This increases contention and can contribute to liveness problems.

Better principle:

```text
Lock only the shared-state operation that actually requires protection.
```

---

# 18. Deadlock vs Starvation vs Livelock

These three are frequently confused.

```text
Deadlock
Starvation
Livelock
```

Let's separate them.

---

# 19. Starvation

**Starvation** means a thread keeps waiting because other threads repeatedly get access to the resource.

Example:

```text
Thread A → gets resource
Thread B → waiting

Thread C → gets resource
Thread B → waiting

Thread D → gets resource
Thread B → waiting
```

Thread B may theoretically be able to run, but it keeps losing access.

Think:

```text
Starvation
   ↓
Thread gets insufficient opportunity to progress
```

---

# 20. Starvation Example

Suppose many high-priority tasks continuously execute while another task receives little opportunity to run.

Or a non-fair lock repeatedly happens to be acquired by other threads.

```text
T1 → lock
T3 → lock
T4 → lock
T1 → lock
...
T2 keeps waiting
```

T2 is starved.

---

# 21. Fair `ReentrantLock`

You can create:

```java
ReentrantLock lock =
        new ReentrantLock(true);
```

`true` requests a **fair** lock policy.

Conceptually:

```text
Threads waiting

T1
T2
T3

Lock tends to grant access
in waiting order
```

This can reduce starvation.

But fairness can reduce throughput.

So:

```text
Fair lock
   ↓
More predictable waiting
   ↓
Potential performance cost
```

---

# 22. What Is Livelock?

In a deadlock:

```text
Threads are stuck waiting.
```

In a livelock:

```text
Threads are active
but still make no useful progress.
```

Example:

Two people meet in a hallway.

```text
Person A moves left
Person B moves left

Both blocked

A moves right
B moves right

Both blocked

repeat...
```

They are moving, but neither passes.

---

# 23. Livelock in Threads

Imagine:

```text
Thread A notices B needs resource
→ A releases it

Thread B notices A needs resource
→ B releases it

Both retry

Same thing happens repeatedly
```

The threads aren't blocked.

They are continuously reacting to each other.

But:

```text
No useful work completes
```

That's livelock.

---

# 24. Preventing Livelock

Possible strategies:

```text
Randomized retry delay
Backoff
Priority rules
Limit retry count
Deterministic ownership rules
```

Example:

Instead of both retrying immediately:

```text
Thread A → wait random 20 ms
Thread B → wait random 80 ms
```

One may then acquire the resources first.

---

# 25. Quick Comparison

| Problem | Meaning |
|---|---|
| Deadlock | Threads wait forever for each other |
| Starvation | A thread rarely/never gets needed resources |
| Livelock | Threads remain active but make no progress |

Remember:

```text
Deadlock
→ no movement

Livelock
→ movement, no progress

Starvation
→ one thread keeps losing opportunity
```

---

# 26. Thread-Safe Singleton — Interview Question 🔥

A very common interview question:

> How would you implement a thread-safe Singleton in Java?

First understand the normal Singleton.

---

# 27. Basic Singleton

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

This ensures callers cannot directly do:

```java
new Singleton();
```

because the constructor is private.

But this implementation is **not thread-safe**.

---

# 28. Why Is the Basic Singleton Not Thread-Safe?

Suppose:

```text
instance = null
```

Thread 1:

```text
checks instance == null
→ true
```

Before it creates the object, Thread 2 runs:

```text
checks instance == null
→ true
```

Now both execute:

```java
new Singleton();
```

Result:

```text
Thread 1 → Object A
Thread 2 → Object B
```

Two Singleton objects can be created.

🔥 Race condition.

---

# 29. Solution 1 — Synchronized Method

```java
class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {

        if (instance == null) {
            instance = new Singleton();
        }

        return instance;
    }
}
```

Now only one thread can execute `getInstance()` at a time.

This is thread-safe.

---

# 30. Problem With Synchronizing Entire Method

After the Singleton has already been created:

```text
instance != null
```

every call still acquires the class monitor.

```text
Thread 1 → lock → return instance
Thread 2 → wait → lock → return instance
Thread 3 → wait...
```

The synchronization is unnecessary after initialization.

This motivates:

```text
Double-Checked Locking
```

---

# 31. Double-Checked Locking

```java
class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {

            synchronized (Singleton.class) {

                if (instance == null) {

                    instance = new Singleton();

                }
            }
        }

        return instance;
    }
}
```

Notice:

```text
check 1
   ↓
synchronized
   ↓
check 2
   ↓
create
```

Hence:

```text
Double-Checked Locking
```

---

# 32. Why Do We Check Twice?

Suppose Thread 1 and Thread 2 both reach:

```java
if (instance == null)
```

Both see:

```text
null
```

Thread 1 acquires lock.

Thread 2 waits.

Thread 1 creates Singleton:

```text
instance = Singleton object
```

Then releases lock.

Thread 2 acquires the lock.

If there were no second check:

```java
instance = new Singleton();
```

Thread 2 would create another object.

So inside the lock we check again:

```java
if (instance == null)
```

Now Thread 2 sees the existing object and does not create another.

---

# 33. Why Is `volatile` Required?

This connects directly to Part 2.

We declare:

```java
private static volatile Singleton instance;
```

Why?

Object construction and reference publication must be safely visible to other threads.

Conceptually, creation involves steps such as:

```text
1. Allocate memory
2. Initialize object
3. Publish reference
```

Without the required memory-ordering guarantees, another thread must not be allowed to observe an improperly published / incompletely initialized object through `instance`.

`volatile` provides the required visibility and ordering guarantees for the singleton reference.

🔥 Interview answer:

> `volatile` is required in double-checked locking to ensure safe publication and prevent problematic reordering/visibility around object initialization.

---

# 34. Is `volatile` Making Singleton Creation Atomic?

No.

Important.

```text
volatile
→ visibility + ordering

synchronized
→ mutual exclusion during creation
```

The combination makes double-checked locking work correctly.

Don't say:

> volatile makes `new Singleton()` atomic.

❌ Wrong.

---

# 35. Singleton Using Eager Initialization

A simpler thread-safe approach:

```java
class Singleton {

    private static final Singleton INSTANCE =
            new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

Class initialization is handled safely by the JVM.

Advantages:

```text
Simple
Thread-safe
No explicit synchronization
```

Disadvantage:

```text
Object created eagerly
```

even if it is never used.

---

# 36. Initialization-on-Demand Holder Idiom

Excellent interview solution.

```java
class Singleton {

    private Singleton() {
    }

    private static class Holder {

        private static final Singleton INSTANCE =
                new Singleton();
    }

    public static Singleton getInstance() {

        return Holder.INSTANCE;

    }
}
```

Why is this good?

```text
Thread-safe
Lazy initialization
No synchronized method on every access
Uses JVM class initialization guarantees
```

The nested `Holder` class is initialized when it is first actively used.

---

# 37. Enum Singleton

Often considered one of the simplest robust Singleton approaches.

```java
enum Singleton {

    INSTANCE;

    public void doSomething() {

        System.out.println("Working");

    }
}
```

Usage:

```java
Singleton.INSTANCE.doSomething();
```

Benefits include:

```text
Thread-safe initialization
Simple
Serialization handled safely
Strong protection against normal reflective construction
```

---

# 38. Which Singleton Should You Give in an Interview?

If interviewer asks:

> Implement a thread-safe Singleton.

A strong answer is:

```text
Option 1:
Initialization-on-demand holder

Option 2:
Double-checked locking + volatile

Option 3:
Enum Singleton
```

If they specifically want to test multithreading:

🔥 **Double-checked locking + `volatile`** is important because it tests:

```text
synchronized
volatile
race condition
memory visibility
safe publication
```

---

# 39. Full Double-Checked Singleton

```java
public final class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {

            synchronized (Singleton.class) {

                if (instance == null) {

                    instance = new Singleton();

                }
            }
        }

        return instance;
    }
}
```

Memorize the reasoning, not just the code.

---

# 40. Interview Explanation of Double-Checked Locking

Explain it like this:

```text
First null check
   ↓
Avoid locking after object exists

synchronized block
   ↓
Only one thread initializes

Second null check
   ↓
Another thread may have initialized
while current thread waited

volatile
   ↓
Safe visibility/publication
and ordering
```

---

# 41. Thread Safety Strategies — Big Picture

By now you've seen several ways to make code thread-safe.

## 1. Immutability

Example:

```java
String
```

Immutable state cannot be modified after creation.

This greatly simplifies thread safety.

---

## 2. `synchronized`

```java
synchronized (lock) {
    // critical section
}
```

Provides:

```text
Mutual exclusion
Visibility
Ordering
```

---

## 3. Explicit Locks

```java
ReentrantLock
ReadWriteLock
```

Provide more flexible locking.

---

## 4. Atomic Variables

```java
AtomicInteger
AtomicLong
AtomicReference
```

Useful for atomic operations without traditional explicit locking in many cases.

---

## 5. Concurrent Collections

```java
ConcurrentHashMap
BlockingQueue
CopyOnWriteArrayList
```

Prefer high-level thread-safe structures when they fit the problem.

---

## 6. Avoid Shared Mutable State

Often the best concurrency strategy.

Instead of:

```text
Many threads
   ↓
same mutable object
```

prefer designs where tasks own independent state or communicate through safe abstractions.

---

# 42. Thread Confinement

If data is used by only one thread:

```text
Thread A
   ↓
private data
```

other threads cannot race on it.

Local variables are naturally useful here:

```java
void process() {

    int localCount = 0;

}
```

Each invocation/thread has its own stack-local variable.

No shared mutable state:

```text
No synchronization needed for that local variable.
```

---

# 43. Immutable Objects

Suppose:

```java
final class User {

    private final String name;

    User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

If properly constructed and safely published, immutable objects are much easier to share across threads because their state doesn't change.

Think:

```text
No mutation
   ↓
No conflicting writes
   ↓
Much easier concurrency
```

---

# 44. `synchronized` vs `ReentrantLock` vs `ReadWriteLock`

Must-cover interview comparison.

| Feature | `synchronized` | `ReentrantLock` | `ReadWriteLock` |
|---|---|---|---|
| Basic mutual exclusion | Yes | Yes | Yes |
| Automatic unlock | Yes | No | No |
| `tryLock()` | No | Yes | Via underlying locks |
| Interruptible lock acquisition | Limited compared with explicit lock APIs | Yes | Depends on lock |
| Fairness option | No explicit fairness policy | Yes | Implementation-dependent; `ReentrantReadWriteLock` supports fairness |
| Separate read/write locking | No | No | Yes |
| Simplicity | Highest | Medium | More complex |
| Typical use | General locking | Advanced lock control | Read-heavy shared data |

---

# 45. When to Use `synchronized`

Prefer `synchronized` when:

```text
Locking requirement is simple
Critical section is small
No timed lock attempt needed
No explicit fairness requirement
```

Example:

```java
synchronized (lock) {
    count++;
}
```

Simple and difficult to forget unlocking because the JVM releases the monitor when the block exits.

---

# 46. When to Use `ReentrantLock`

Useful when you need:

```text
tryLock()
Timed lock attempts
Interruptible lock acquisition
Fairness option
Multiple Condition objects
More explicit lock control
```

Pattern:

```java
lock.lock();

try {

    // critical section

} finally {

    lock.unlock();

}
```

🔥 Always use `unlock()` in `finally`.

---

# 47. When to Use `ReadWriteLock`

Useful when:

```text
Many readers
Few writers
```

Example:

```java
ReadWriteLock rwLock =
        new ReentrantReadWriteLock();

Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();
```

Multiple readers may hold the read lock concurrently:

```text
Reader 1 ─┐
Reader 2 ─┼── allowed together
Reader 3 ─┘
```

Writer requires exclusive access:

```text
Writer
   ↓
No readers/writers concurrently holding conflicting locks
```

---

# 48. Read-Heavy Example

Imagine a cache:

```text
95% reads
5% writes
```

Using one exclusive lock:

```text
Reader 1
Reader 2 → waits
Reader 3 → waits
```

With read/write locking:

```text
Reader 1 ─┐
Reader 2 ─┼→ read concurrently
Reader 3 ─┘
```

When a writer comes:

```text
Writer
   ↓
exclusive write lock
```

This can improve concurrency for appropriate read-heavy workloads.

---

# 49. Don't Automatically Use `ReadWriteLock`

More complex locking is not automatically faster.

For:

```text
very small critical sections
few threads
many writes
low contention
```

the extra complexity may provide little benefit.

Interview principle:

> Choose the simplest synchronization mechanism that correctly solves the concurrency requirement.

---

# 50. `volatile` Final Revision

Must-cover.

Suppose:

```java
private volatile boolean running = true;
```

Thread 1:

```java
while (running) {
    // work
}
```

Thread 2:

```java
running = false;
```

`volatile` helps ensure updates to `running` become visible across threads according to Java Memory Model guarantees.

---

# 51. `volatile` Does NOT Provide General Atomicity

This is one of the biggest interview traps.

```java
volatile int count = 0;
```

Then:

```java
count++;
```

is still not atomic.

Because:

```text
Read
Modify
Write
```

Multiple threads can interleave.

So:

```text
volatile count
+
count++
```

❌ does not make increment thread-safe.

---

# 52. Visibility vs Atomicity

Memorize:

```text
volatile
   ↓
Visibility + ordering guarantees

NOT
   ↓
General mutual exclusion / compound-operation atomicity
```

For atomic increments:

```java
AtomicInteger count =
        new AtomicInteger(0);

count.incrementAndGet();
```

or use appropriate locking.

---

# 53. `volatile` vs `synchronized`

| Feature | `volatile` | `synchronized` |
|---|---|---|
| Visibility | Yes | Yes |
| Ordering guarantees | Yes | Yes |
| Mutual exclusion | No | Yes |
| Protect compound operations | No | Yes |
| Thread blocking for lock | No | Possible |
| Good for status flags | Yes | Usually unnecessary |

---

# 54. Common `volatile` Use Case

A status flag:

```java
class Worker {

    private volatile boolean running = true;

    public void run() {

        while (running) {

            // work

        }
    }

    public void stop() {

        running = false;

    }
}
```

Here the operation is simple:

```text
write boolean
read boolean
```

We mainly need visibility.

---

# 55. Happens-Before — Interview-Level Understanding

You don't need to recite the entire Java Memory Model.

Understand the idea:

> A happens-before relationship provides ordering and visibility guarantees between actions in different threads.

Important examples:

```text
Unlock monitor
happens-before
later lock of same monitor
```

```text
volatile write
happens-before
later volatile read of same variable
```

Also, actions before calling `Thread.start()` happen-before actions in the started thread, and actions in a thread happen-before another thread successfully returns from `join()` on it.

These guarantees explain why correctly synchronized programs see expected values.

---

# 56. Atomic Classes

Examples:

```java
AtomicInteger
AtomicLong
AtomicBoolean
AtomicReference
```

Example:

```java
AtomicInteger counter =
        new AtomicInteger(0);

counter.incrementAndGet();
```

This provides an atomic increment without writing:

```java
synchronized
```

yourself.

---

# 57. `AtomicInteger` Example

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

Multiple threads can safely increment the counter.

---

# 58. CAS — Compare And Set

Atomic classes commonly rely on operations based around **Compare-And-Set / Compare-And-Swap (CAS)**.

Conceptually:

```text
Current value = 5

Expected = 5
New value = 6

Is current still 5?
   │
   ├── Yes → change to 6
   │
   └── No → operation failed/retry
```

This avoids traditional lock acquisition for many atomic operations.

---

# 59. `synchronized` vs AtomicInteger

For:

```text
simple counter
```

`AtomicInteger` is often convenient.

But for:

```text
multiple variables
complex invariants
several operations that must happen together
```

you may still need locking.

Example:

```text
check balance
deduct balance
update transaction state
```

Those operations may need to be protected as one critical section.

---

# 60. Important Interview Pattern — Check Then Act

Bad:

```java
if (balance >= amount) {

    balance -= amount;

}
```

With multiple threads:

```text
T1 checks balance → enough
T2 checks balance → enough

T1 deducts
T2 deducts
```

The compound operation isn't atomic.

This is called a:

```text
check-then-act race
```

Fix by protecting the complete invariant:

```java
synchronized (lock) {

    if (balance >= amount) {

        balance -= amount;

    }
}
```

---

# 61. Another Pattern — Read Modify Write

Example:

```java
count++;
```

Pattern:

```text
Read
   ↓
Modify
   ↓
Write
```

Fix using:

```text
synchronized
ReentrantLock
AtomicInteger
```

depending on the requirement.

---

# 62. Another Pattern — Lazy Initialization Race

Example:

```java
if (instance == null) {
    instance = new Object();
}
```

Two threads may both initialize.

This is the exact race we saw in Singleton.

Solutions include:

```text
synchronization
holder idiom
safe class initialization
double-checked locking + volatile
```

---

# 63. Interview Scenario — Two Locks

Question:

```java
synchronized (lockA) {

    synchronized (lockB) {

        // work

    }
}
```

Is this always a deadlock?

**No.**

Nested locks do not automatically mean deadlock.

Deadlock requires a circular dependency.

If all threads acquire:

```text
A → B
```

there is no lock-order cycle.

The danger occurs when another thread does:

```text
B → A
```

---

# 64. Interview Scenario — `tryLock()`

Question:

> How can `tryLock()` help prevent deadlock?

Answer:

Instead of waiting indefinitely, a thread can attempt lock acquisition and, if it fails:

```text
release already acquired locks
   ↓
back off
   ↓
retry later
```

This can break hold-and-wait/circular-wait behavior.

---

# 65. Interview Scenario — `volatile`

Question:

```java
volatile int count = 0;

count++;
```

Is it thread-safe?

**No.**

`volatile` provides visibility and ordering, but `count++` is a compound read-modify-write operation.

---

# 66. Interview Scenario — Singleton

Question:

> Why is the first check outside `synchronized` in double-checked locking?

Because after initialization:

```text
instance != null
```

most calls can return immediately without acquiring the lock.

This avoids unnecessary synchronization overhead.

---

# 67. Interview Scenario — Why Second Check?

Because multiple threads may pass the first check before one acquires the lock.

The second check ensures only the first thread actually creates the instance.

---

# 68. Interview Scenario — Why `volatile`?

To provide the visibility and ordering needed for safe publication of the initialized Singleton reference.

---

# 69. Full Multithreading Mental Model

Everything from all four parts connects like this:

```text
Threads
   ↓
Share heap state
   ↓
Race conditions
   ↓
Need thread safety
   ↓
─────────────────────────────
│ synchronized              │
│ ReentrantLock             │
│ ReadWriteLock             │
│ volatile                  │
│ Atomic classes            │
│ Concurrent collections    │
─────────────────────────────
   ↓
Need task management
   ↓
ExecutorService
   ↓
ThreadPoolExecutor
   ↓
Worker Threads + Queue
   ↓
Need safe liveness
   ↓
Avoid deadlock
Avoid starvation
Avoid livelock
```

---

# Interview Questions — Part 4

## Q1. What is deadlock?

A situation where threads wait indefinitely for resources held by each other, creating a circular dependency.

---

## Q2. What are the four deadlock conditions?

```text
1. Mutual exclusion
2. Hold and wait
3. No preemption
4. Circular wait
```

---

## Q3. How can deadlocks be prevented?

Common techniques:

```text
Consistent lock ordering
Avoid unnecessary nested locks
Keep lock scope small
tryLock() with timeout/backoff
Avoid slow/external operations while holding locks
```

---

## Q4. How does consistent lock ordering prevent deadlock?

All threads acquire multiple locks in the same global order, preventing circular wait.

---

## Q5. How can `tryLock()` help?

It allows a thread to fail or time out instead of waiting indefinitely, so it can release other locks and retry later.

---

## Q6. How do you detect a Java deadlock?

Use tools such as:

```text
Thread dumps
jstack
JVM monitoring tools
ThreadMXBean
```

Look for threads waiting on locks held by each other.

---

## Q7. Deadlock vs starvation?

```text
Deadlock
→ threads wait for each other indefinitely

Starvation
→ a thread continually fails to get enough access/resources
```

---

## Q8. Deadlock vs livelock?

```text
Deadlock
→ threads are blocked

Livelock
→ threads remain active but make no useful progress
```

---

## Q9. How do you implement a thread-safe Singleton?

Strong approaches include:

```text
Eager initialization
Initialization-on-demand holder
Enum Singleton
Double-checked locking + volatile
```

---

## Q10. Why is normal lazy Singleton not thread-safe?

Two threads can simultaneously observe:

```text
instance == null
```

and both create objects.

---

## Q11. Why use `volatile` in double-checked locking?

To ensure visibility and ordering required for safe publication of the Singleton instance.

---

## Q12. Does `volatile` make `count++` atomic?

**No.**

---

## Q13. `synchronized` vs `ReentrantLock`?

Use `synchronized` for simpler locking.

Use `ReentrantLock` when you need features such as:

```text
tryLock()
timed lock acquisition
interruptible acquisition
fairness
Condition
```

---

## Q14. When is `ReadWriteLock` useful?

When a shared resource has:

```text
many reads
few writes
```

because multiple readers can access concurrently.

---

## Q15. Why must `unlock()` be inside `finally`?

Because if an exception occurs:

```text
without finally
→ lock may never be released
```

which can block other threads indefinitely.

---

## Q16. What is CAS?

Compare-And-Set checks whether a value still equals an expected value and updates it atomically if so.

---

## Q17. When would you use `AtomicInteger`?

For simple atomic counter/state operations where locking a larger critical section isn't required.

---

## Q18. Can atomic variables replace all locks?

**No.**

Complex invariants involving multiple variables or operations may still require locking or another coordination mechanism.

---

# 🔥 Coding Question — Thread-Safe Singleton

```java
public final class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (instance == null) {

            synchronized (Singleton.class) {

                if (instance == null) {

                    instance = new Singleton();

                }
            }
        }

        return instance;
    }
}
```

### Explanation

```text
volatile
   ↓
visibility + safe publication/order

first null check
   ↓
avoid unnecessary synchronization

synchronized
   ↓
one initializer at a time

second null check
   ↓
prevent second creation after waiting
```

---

# 🔥 Coding Question — Deadlock Prevention With Ordering

```java
class Account {

    private final int id;

    Account(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }
}

public class TransferService {

    public void transfer(
            Account from,
            Account to) {

        Account first;
        Account second;

        if (from.getId() < to.getId()) {

            first = from;
            second = to;

        } else {

            first = to;
            second = from;

        }

        synchronized (first) {

            synchronized (second) {

                // perform transfer

            }
        }
    }
}
```

The important idea is:

```text
Every thread
   ↓
same deterministic lock order
   ↓
no circular wait
```

---

# ⭐ 30-Second Interview Answer

> **Deadlock occurs when threads wait indefinitely for locks held by one another, creating a circular dependency. The four necessary conditions are mutual exclusion, hold-and-wait, no preemption, and circular wait. In practice, I prevent deadlocks by using consistent lock ordering, keeping critical sections small, avoiding unnecessary nested locks, and using `ReentrantLock.tryLock()` with timeout or backoff where appropriate. For a thread-safe lazy Singleton, I can use double-checked locking with a `volatile` instance: synchronization ensures only one thread initializes it, while `volatile` provides the visibility and ordering needed for safe publication.**

---

# Quick Revision

```text
Deadlock
   ↓
Threads wait forever

T1
owns A
needs B

T2
owns B
needs A

Deadlock conditions
   ↓
Mutual exclusion
Hold and wait
No preemption
Circular wait

Prevention
   ↓
Consistent lock ordering
Small lock scope
tryLock()
Timeout/backoff

Detection
   ↓
Thread dump
jstack
ThreadMXBean

Starvation
   ↓
Thread keeps losing access

Livelock
   ↓
Threads active
but no progress

Thread-safe Singleton
   ↓
Double-checked locking
+
volatile

volatile
   ↓
Visibility + ordering
NOT general atomicity

count++
   ↓
Not atomic

synchronized
   ↓
Simple mutual exclusion

ReentrantLock
   ↓
Advanced lock control
tryLock()
fairness
interruptible locking

ReadWriteLock
   ↓
Many readers
Few writers

AtomicInteger
   ↓
Atomic counter operations

Thread safety
   ↓
Prefer simplest correct solution
Avoid shared mutable state when possible
```

---

# ⭐ Part 4 Priority

Master these for interviews:

1. **What deadlock is**
2. **Four necessary deadlock conditions**
3. **Classic two-lock deadlock**
4. **Consistent lock ordering**
5. **Bank-transfer lock-ordering example**
6. **Deadlock prevention using `tryLock()`**
7. **`tryLock()` timeout/backoff**
8. **Deadlock detection with thread dumps / `jstack`**
9. **Deadlock vs starvation vs livelock**
10. **Fair `ReentrantLock`**
11. **Thread-safe Singleton**
12. **Why basic lazy Singleton fails**
13. **Synchronized Singleton**
14. **Double-checked locking**
15. **Why the second null check exists**
16. **Why `volatile` is required**
17. **Initialization-on-demand holder**
18. **Enum Singleton**
19. **`synchronized` vs `ReentrantLock` vs `ReadWriteLock`**
20. **`volatile`: visibility, not general atomicity**
21. **`AtomicInteger` and CAS**
22. **Check-then-act race**
23. **Read-modify-write race**
24. **Avoiding shared mutable state**

---

# Final 4-Part Multithreading Revision Flow

```text
PART 1
Thread Fundamentals
   ↓
Thread vs Process
start() vs run()
Race Conditions
synchronized

PART 2
Java Memory Model & Locks
   ↓
volatile
Atomicity
ReentrantLock
ReadWriteLock
Atomic Classes

PART 3
Thread Pools
   ↓
ExecutorService
FixedThreadPool
CachedThreadPool
ThreadPoolExecutor
Callable / Future
Concurrent Utilities

PART 4
Liveness & Interview Patterns
   ↓
Deadlock
Lock Ordering
tryLock()
Starvation
Livelock
Thread-Safe Singleton
```

If you can explain this entire flow clearly, you have covered the **core Java multithreading topics expected in many backend/SDE interviews**.
