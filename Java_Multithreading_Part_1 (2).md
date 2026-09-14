# Java Multithreading — Part 1
## Thread Fundamentals, Race Conditions & `synchronized`

---

## 1. What Is a Process?

Suppose you open:

```text
Chrome
IntelliJ
Spotify
```

Each running application can have its own **process**.

A process has its own resources such as:

```text
Process
│
├── Memory
├── Heap
├── Files
└── Threads
```

A process can contain **multiple threads**.

---

## 2. What Is a Thread?

A **thread is a unit of execution inside a process**.

For example, imagine a backend application:

```text
Java Backend Process
        │
        ├── Thread 1 → Handle Request A
        ├── Thread 2 → Handle Request B
        ├── Thread 3 → Database work
        └── Thread 4 → Background task
```

Multiple threads of the same process **share resources**.

Most importantly:

```text
             Process
                │
        ┌──── Shared Heap ────┐
        │                     │
    Thread 1              Thread 2
    Stack 1               Stack 2
```

Each thread has its **own stack**, but threads can access objects in the **shared heap**.

⭐ This shared memory is the source of both the **power and problems** of multithreading.

---

## 3. Why Do We Need Multithreading?

Imagine your server receives:

```text
Request A → takes 2 seconds
Request B → takes 1 second
Request C → takes 3 seconds
```

If everything executes sequentially:

```text
A ──────> B ───> C ─────────>

Total ≈ 6 sec
```

With multiple threads, work can overlap:

```text
Thread 1: A ──────>
Thread 2: B ───>
Thread 3: C ─────────>
```

This improves **responsiveness and throughput**, especially when tasks spend time waiting for I/O.

### Interview Point

Don't say:

> Multithreading always makes applications faster.

❌ Not necessarily.

Thread creation, context switching, synchronization, contention, CPU availability, and workload type all matter.

---

## 4. Concurrency vs Parallelism

This is a common interview question.

### Concurrency

Multiple tasks **make progress during overlapping periods**.

Example with one CPU core:

```text
Time →

Thread A: ███     ███
Thread B:    ███     ███
```

The CPU switches between them.

### Parallelism

Multiple tasks are **literally executing at the same time**, generally using multiple CPU cores.

```text
Core 1 → Thread A █████████
Core 2 → Thread B █████████
```

### Remember

```text
Concurrency
    ↓
Dealing with multiple tasks at once

Parallelism
    ↓
Executing multiple tasks simultaneously
```

⭐ **Concurrency doesn't necessarily mean parallelism.**

---

## 5. Creating a Thread in Java

There are two fundamental approaches you should know.

### Approach 1 — Extend `Thread`

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Task running");
    }
}

public class Main {

    public static void main(String[] args) {

        MyThread t1 = new MyThread();

        t1.start();
    }
}
```

`run()` contains the work that the thread performs.

---

## 6. Approach 2 — Implement `Runnable`

```java
class MyTask implements Runnable {

    @Override
    public void run() {
        System.out.println("Task running");
    }
}

public class Main {

    public static void main(String[] args) {

        Thread t1 = new Thread(new MyTask());

        t1.start();
    }
}
```

Think:

```text
Runnable
   ↓
What work should be done?

Thread
   ↓
Who executes that work?
```

This separation is important.

---

## 7. `Thread` vs `Runnable`

Usually prefer **`Runnable`**.

Why?

Suppose:

```java
class PaymentService extends SomeParentClass {
}
```

Java doesn't support multiple class inheritance.

So you cannot do:

```java
class PaymentService
        extends SomeParentClass, Thread
```

❌ Not possible.

But you can:

```java
class PaymentService
        extends SomeParentClass
        implements Runnable {
}
```

Also, conceptually, `Runnable` separates the **task** from the **execution mechanism**.

⭐ This idea becomes very important when we reach `ExecutorService`.

---

## 8. The Most Important Beginner Trap — `start()` vs `run()`

Consider:

```java
Thread t1 = new Thread(() -> {
    System.out.println("Hello");
});
```

Now:

```java
t1.start();
```

and:

```java
t1.run();
```

are **not equivalent**.

### `start()`

```text
main thread
     │
     │ t1.start()
     ↓
JVM starts another thread
     │
     ↓
t1 executes run()
```

So there can be:

```text
main thread

t1 thread
```

### Calling `run()` directly

```java
t1.run();
```

This is just a normal method call.

```text
main thread
     │
     ↓
run()
     │
     ↓
returns
```

No new thread is started.

### ⭐ Interview Answer

```text
start()
   ↓
Starts a new thread
   ↓
New thread executes run()

run()
   ↓
Normal method call
   ↓
Current thread executes it
```

---

## 9. See It Yourself

```java
public class Main {

    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            System.out.println(
                "Running: " + Thread.currentThread().getName()
            );
        });

        t1.start();

        System.out.println(
            "Main: " + Thread.currentThread().getName()
        );
    }
}
```

You might get:

```text
Main: main
Running: Thread-0
```

Notice:

```text
main
Thread-0
```

Two different threads.

---

## 10. Can We Predict Which Thread Runs First?

Consider:

```java
Thread t1 = new Thread(() -> {
    System.out.println("A");
});

Thread t2 = new Thread(() -> {
    System.out.println("B");
});

t1.start();
t2.start();
```

Can we guarantee:

```text
A
B
```

?

**No.**

It could be:

```text
A
B
```

or:

```text
B
A
```

Thread scheduling is handled by the JVM/OS environment and should generally **not be assumed to follow `start()` order**.

⭐ Very important multithreading mindset:

> Don't depend on thread execution order unless you explicitly coordinate it.

---

## 11. Thread Lifecycle

For interviews, know these Java thread states:

```text
NEW
 │
 │ start()
 ↓
RUNNABLE
 │
 ├───────────────┐
 ↓               ↓
BLOCKED        WAITING
                 │
                 ↓
           TIMED_WAITING

Eventually
    ↓
TERMINATED
```

Java's `Thread.State` defines:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

Note that Java's **RUNNABLE** state covers a thread that is eligible/running from the JVM-state perspective; Java doesn't expose a separate `RUNNING` enum state.

---

## 12. `sleep()`

Suppose:

```java
Thread.sleep(2000);
```

It tells the **current thread** to pause for approximately 2 seconds.

Example:

```java
System.out.println("Start");

Thread.sleep(2000);

System.out.println("End");
```

Conceptually:

```text
Start
 ↓
sleep 2 sec
 ↓
End
```

It throws `InterruptedException`, so you must handle or declare it.

```java
try {
    Thread.sleep(2000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

---

## 13. Important `sleep()` Interview Trap

Does `sleep()` release a lock?

**No.**

Suppose:

```java
synchronized (lock) {

    Thread.sleep(5000);

}
```

The thread sleeps, but **continues owning that monitor**.

Another thread needing the same lock has to wait.

Remember:

```text
sleep()
   ↓
Pauses thread
   ↓
Does NOT release monitor lock
```

Later, when you learn `wait()`, you'll see an important difference.

---

## 14. `join()`

Suppose the main thread needs to wait for `t1` to finish.

```java
Thread t1 = new Thread(() -> {
    System.out.println("Processing...");
});

t1.start();

t1.join();

System.out.println("Finished");
```

Flow:

```text
main
 │
 ├── start t1
 │
 │
 │    t1 → Processing...
 │         finishes
 │
 └── main continues
      Finished
```

So:

```java
t1.join();
```

means:

> Current thread waits for `t1` to terminate.

---

## 15. `interrupt()`

Suppose a thread is doing some work:

```java
Thread t1 = new Thread(() -> {

    try {
        Thread.sleep(10000);
    } catch (InterruptedException e) {
        System.out.println("Interrupted");
    }

});

t1.start();

t1.interrupt();
```

`interrupt()` is a **cooperative cancellation signal**, not a "force kill this thread" operation.

If the thread is blocked in methods such as `sleep()`, `wait()`, or `join()`, interruption can cause:

```text
InterruptedException
```

For running code, you can check:

```java
Thread.currentThread().isInterrupted()
```

---

## 16. Now the REAL Multithreading Problem

Everything becomes interesting when threads share data.

Consider:

```java
class Counter {

    int count = 0;

    void increment() {
        count++;
    }
}
```

Now two threads repeatedly call:

```java
counter.increment();
```

You might think:

```text
Thread 1 → +1
Thread 2 → +1

Therefore total → +2
```

Not necessarily.

This is where **race conditions** start.

---

## 17. Why Is `count++` Dangerous?

This looks like one operation:

```java
count++;
```

But conceptually it's a **read-modify-write** operation:

```text
1. Read count
2. Add 1
3. Write count
```

Suppose:

```text
count = 5
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
5 + 1 = 6
write 6
```

Thread B:

```text
5 + 1 = 6
write 6
```

Final result:

```text
6
```

But expected:

```text
7
```

One increment was **lost**.

🔥 This is a classic **race condition / lost update**.

---

## 18. What Is a Race Condition?

A **race condition occurs when the correctness of a result depends on the timing/interleaving of multiple threads accessing shared mutable state without proper synchronization**.

Example:

```text
Thread A ──┐
           ├──→ shared count
Thread B ──┘
```

Both threads are racing to modify the same data.

The result can become unpredictable.

---

## 19. Critical Section

The part of code accessing shared mutable data that must be protected is called a **critical section**.

Here:

```java
void increment() {
    count++;
}
```

`count++` is our critical operation.

What we want:

```text
Thread A
   ↓
count++
   ↓
finish
   ↓
Thread B
   ↓
count++
```

Instead of both interfering with each other.

That's where:

```java
synchronized
```

comes in.

---

## 20. `synchronized`

We can write:

```java
class Counter {

    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

Now, for the same `Counter` instance, only one thread at a time can execute the synchronized instance method guarded by that object's monitor.

Conceptually:

```text
          Counter Object
               │
             LOCK
               │
       ┌───────┴───────┐
       │               │
   Thread A         Thread B
       │               │
   acquires           waits
     lock
       │
   count++
       │
   releases
     lock
                       │
                    acquires
                      lock
```

---

## 21. What Lock Does `synchronized` Use?

Every Java object can be used as an **intrinsic lock / monitor**.

Suppose:

```java
Counter counter = new Counter();
```

and:

```java
public synchronized void increment() {
    count++;
}
```

For an instance synchronized method, this is conceptually equivalent to:

```java
public void increment() {

    synchronized (this) {
        count++;
    }

}
```

The lock is:

```text
this
 ↓
current Counter object
```

---

## 22. Important: Lock Belongs to the Object

Suppose:

```java
Counter c1 = new Counter();
Counter c2 = new Counter();
```

Thread A:

```java
c1.increment();
```

Thread B:

```java
c2.increment();
```

They are locking **different objects**.

```text
c1
└── Lock 1

c2
└── Lock 2
```

Therefore they don't block each other just because the same method is declared `synchronized`.

🔥 Interview trap:

> `synchronized` instance methods lock the **object instance**, not the method itself.

---

## 23. Synchronized Block

Instead of synchronizing an entire method:

```java
public synchronized void process() {

    doSomething();

    count++;

    doSomethingElse();
}
```

we can protect only the critical section:

```java
public void process() {

    doSomething();

    synchronized (this) {
        count++;
    }

    doSomethingElse();
}
```

This can reduce unnecessary lock contention when only a small part actually requires mutual exclusion.

---

## 24. Custom Lock Object

You can also do:

```java
class Counter {

    private int count = 0;

    private final Object lock = new Object();

    public void increment() {

        synchronized (lock) {
            count++;
        }

    }
}
```

Now the monitor being acquired is:

```text
lock object
```

rather than `this`.

This is often useful because the locking mechanism is kept private and controlled.

---

## 25. Static `synchronized`

Now consider:

```java
public static synchronized void increment() {
}
```

What gets locked?

Not an instance.

It locks the corresponding **Class object**.

Conceptually:

```text
Counter.class
     ↓
   monitor
```

Equivalent idea:

```java
synchronized (Counter.class) {
    ...
}
```

---

## 26. Instance Lock vs Class Lock

This is very important.

```java
public synchronized void methodA() {
}
```

locks:

```text
this
```

Whereas:

```java
public static synchronized void methodB() {
}
```

locks:

```text
ClassName.class
```

Therefore:

```text
Instance synchronized
        ↓
Object-level lock

Static synchronized
        ↓
Class-level lock
```

Because they're different monitors, an instance synchronized method and a static synchronized method **do not automatically block one another**.

---

## 27. Does `synchronized` Only Solve Atomicity?

No.

This will connect directly to `volatile` in Part 2.

`synchronized` gives us important guarantees around:

### 1. Mutual Exclusion

```text
Only one thread enters protected section
for the same monitor at a time
```

### 2. Visibility / Memory Ordering

Changes made by a thread before releasing a monitor become visible, under Java's happens-before rules, to a thread that subsequently acquires that same monitor.

So `synchronized` isn't merely "prevent two threads entering."

---

## 28. Full Race Condition Example

```java
class Counter {

    private int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}

public class Main {

    public static void main(String[] args)
            throws InterruptedException {

        Counter counter = new Counter();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                counter.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                counter.increment();
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println(counter.getCount());
    }
}
```

Expected:

```text
200000
```

But without synchronization, you are **not guaranteed** to get `200000`.

---

## 29. Fix It

Change:

```java
public void increment() {
    count++;
}
```

to:

```java
public synchronized void increment() {
    count++;
}
```

Now:

```text
Thread 1
   ↓
acquire lock
   ↓
count++
   ↓
release lock

Thread 2
   ↓
acquire lock
   ↓
count++
```

The increments are protected.

---

## 30. Important Interview Question 🔥

### Is `count++` atomic?

**No.**

Because it involves a read-modify-write sequence conceptually:

```text
read
 ↓
increment
 ↓
write
```

Multiple threads can interleave these operations.

We'll later solve the same problem using:

```java
AtomicInteger
```

without writing a `synchronized` block ourselves.

---

## 31. Another Interview Trap

Suppose:

```java
synchronized void methodA() {
}

synchronized void methodB() {
}
```

Thread 1 executes:

```java
obj.methodA();
```

Can Thread 2 simultaneously execute:

```java
obj.methodB();
```

on the **same object**?

### No.

Because both synchronized instance methods acquire:

```text
obj's monitor
```

It doesn't matter that the methods are different.

```text
            obj monitor
              /    \
        methodA    methodB
```

Same monitor → mutual exclusion.

---

## 32. But What About Different Objects?

```java
MyClass a = new MyClass();
MyClass b = new MyClass();
```

Thread 1:

```java
a.methodA();
```

Thread 2:

```java
b.methodA();
```

They **can execute concurrently**.

Because:

```text
a → monitor A

b → monitor B
```

Different objects → different monitors.

---

## 33. `sleep()` Inside `synchronized`

Interview question:

```java
synchronized (lock) {

    Thread.sleep(5000);

}
```

Does another thread get `lock` while this one sleeps?

**No.**

```text
Thread A
 ↓
acquires lock
 ↓
sleep()
 ↓
still owns lock
 ↓
wakes
 ↓
leaves synchronized
 ↓
releases lock
```

This distinction will matter when we discuss `wait()`.

---

## 34. What You Should Understand Before Part 2

The whole story so far is:

```text
Process
   ↓
Multiple Threads
   ↓
Threads share heap objects
   ↓
Shared mutable data
   ↓
Concurrent modification
   ↓
Race Condition
   ↓
Critical Section
   ↓
Need synchronization
   ↓
synchronized / monitor lock
```

That flow is **fundamental**. If this is clear, `ReentrantLock`, `volatile`, atomics, and thread pools become much easier.

---

# Interview Questions — Part 1

## Q1. Process vs Thread?

**Process** is an independent running program with its own memory/resources.

A **thread** is an execution unit within a process; threads in the same process share heap/resources but have their own stacks.

---

## Q2. `start()` vs `run()`?

```text
start()
→ starts a new thread
→ new thread invokes run()

run()
→ normal method invocation
→ executes on current thread
```

---

## Q3. What is a race condition?

When multiple threads access shared mutable state concurrently and the result depends on their execution/interleaving.

---

## Q4. Why isn't `count++` thread-safe?

Because it is a **read-modify-write** operation and isn't atomic.

---

## Q5. What does `synchronized` do?

It provides **mutual exclusion** using an intrinsic monitor and also establishes important **memory visibility/order guarantees**.

---

## Q6. What does an instance synchronized method lock?

```java
public synchronized void method()
```

locks:

```text
this
```

—the current object.

---

## Q7. What does static synchronized lock?

```java
public static synchronized void method()
```

locks the corresponding:

```java
ClassName.class
```

monitor.

---

## Q8. Does `sleep()` release a synchronized lock?

**No.**

---

## Q9. Can two synchronized methods execute simultaneously on the same object?

Generally **no**, if both require that same object's monitor.

---

## Q10. Can the same synchronized instance method execute simultaneously on two different objects?

**Yes**, because the objects have different monitors.

---

# Predict the Output / Behaviour

## Question 1

```java
public static void main(String[] args) {

    Thread t = new Thread(() -> {
        System.out.println(
            Thread.currentThread().getName()
        );
    });

    t.run();
}
```

Which thread executes it?

**Answer:**

```text
main
```

Because `run()` was called directly.

---

## Question 2

```java
Thread t = new Thread(() -> {
    System.out.println("A");
});

t.start();
t.start();
```

Valid?

**No.**

A `Thread` instance cannot be started twice. The second `start()` results in:

```text
IllegalThreadStateException
```

---

## Question 3

```java
class Test {

    synchronized void a() {
        // ...
    }

    synchronized void b() {
        // ...
    }
}
```

Two threads call:

```java
obj.a();
obj.b();
```

Can both enter simultaneously?

**No.**

Same object → same monitor.

---

# ⭐ 30-Second Interview Answer

> **Java multithreading allows multiple threads to execute within the same process. Threads have their own stacks but can share heap objects, which makes shared mutable state a concurrency concern. Operations such as `count++` aren't atomic and can cause race conditions. We can protect critical sections using `synchronized`, which uses an object's intrinsic monitor to provide mutual exclusion and memory-visibility guarantees. Instance synchronized methods lock the current object, while static synchronized methods lock the Class object.**

---

# Quick Revision

```text
Thread
  ↓
Unit of execution

start()
  ↓
Starts new thread

run()
  ↓
Normal method if called directly

Shared mutable state
  ↓
Race condition

count++
  ↓
Read + Modify + Write
  ↓
Not atomic

synchronized
  ↓
Intrinsic / Monitor Lock
  ↓
Mutual exclusion + visibility

synchronized instance method
  ↓
Locks this

static synchronized method
  ↓
Locks ClassName.class

sleep()
  ↓
Pauses current thread
  ↓
Does NOT release monitor

join()
  ↓
Wait for another thread to finish

interrupt()
  ↓
Cooperative interruption signal
```

---

## ⭐ Part 1 Priority

Be completely comfortable with:

1. **Thread vs Process**
2. **Concurrency vs Parallelism**
3. **`start()` vs `run()`**
4. **Thread lifecycle**
5. **`sleep()`, `join()`, `interrupt()`**
6. **Shared memory**
7. **Race conditions**
8. **Why `count++` is not atomic**
9. **Critical sections**
10. **`synchronized`**
11. **Object-level vs class-level locking**
12. **Monitor / intrinsic locks**

These concepts are the foundation for **Part 2: `synchronized` vs `ReentrantLock` vs `ReadWriteLock`, `volatile`, Java Memory Model, happens-before, and `AtomicInteger`**.
