# Java Multithreading — Part 3
## Thread Pools, `ExecutorService`, `ThreadPoolExecutor`, `Callable`, `Future` & Concurrent Utilities

---

# 1. Why Do We Need Thread Pools?

In Part 1, we created threads manually:

```java
Thread t1 = new Thread(() -> {
    System.out.println("Task running");
});

t1.start();
```

This is fine for learning.

But imagine a backend server receives:

```text
10 requests
100 requests
10,000 requests
```

If we create one new thread for every task:

```text
Task 1 → new Thread()
Task 2 → new Thread()
Task 3 → new Thread()
...
Task 10000 → new Thread()
```

this can become very expensive.

Why?

Each thread consumes resources:

```text
Thread
├── Stack memory
├── JVM bookkeeping
├── OS scheduling
└── Context switching cost
```

Creating and destroying threads repeatedly is inefficient.

So instead of:

```text
Task arrives
   ↓
Create thread
   ↓
Execute
   ↓
Destroy thread
```

we create a **pool of reusable worker threads**.

```text
              Thread Pool

Tasks ───→   [Thread-1]
Tasks ───→   [Thread-2]
Tasks ───→   [Thread-3]
Tasks ───→   [Thread-4]
```

The threads are reused for multiple tasks.

⭐ This is one of the most important ideas in Java backend development.

---

# 2. What Is a Thread Pool?

A **thread pool is a collection of reusable worker threads that execute submitted tasks**.

Conceptually:

```text
             Task Queue
          ┌──────────────┐
          │ Task 1       │
          │ Task 2       │
          │ Task 3       │
          │ Task 4       │
          └──────┬───────┘
                 │
                 ↓
          Thread Pool
       ┌─────────────────┐
       │ Worker Thread 1 │
       │ Worker Thread 2 │
       │ Worker Thread 3 │
       └─────────────────┘
```

A worker:

```text
Take task
   ↓
Execute task
   ↓
Take next task
   ↓
Execute
```

The same thread can handle many tasks over its lifetime.

---

# 3. Why Are Thread Pools Better?

Main benefits:

### 1. Thread reuse

We avoid repeatedly creating and destroying threads.

### 2. Resource control

We can restrict how many threads run concurrently.

Example:

```text
1000 tasks

but only

10 worker threads
```

### 3. Better performance

Less thread creation and reduced scheduling overhead.

### 4. Task management

Java provides APIs for:

```text
submit
cancel
wait for result
shutdown
schedule
```

### 5. Prevent uncontrolled thread creation

Without a pool:

```text
10000 requests
   ↓
10000 threads
```

This can exhaust memory or overwhelm the system.

With a pool:

```text
10000 requests
   ↓
Queue
   ↓
20 worker threads
```

Much more controlled.

---

# 4. `Executor`

Java introduced an abstraction so that we don't have to directly manage threads.

At the base:

```java
Executor
```

It has:

```java
void execute(Runnable command);
```

Example:

```java
Executor executor = command -> new Thread(command).start();

executor.execute(() -> {
    System.out.println("Task running");
});
```

But in real applications, we commonly use:

```java
ExecutorService
```

---

# 5. What Is `ExecutorService`?

`ExecutorService` is a higher-level interface for managing asynchronous task execution.

It supports:

```text
execute()
submit()
shutdown()
shutdownNow()
invokeAll()
invokeAny()
```

Typical usage:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);
```

Then:

```java
executor.submit(() -> {
    System.out.println("Task running");
});
```

At the end:

```java
executor.shutdown();
```

---

# 6. Basic `ExecutorService` Example

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {

    public static void main(String[] args) {

        ExecutorService executor =
                Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 5; i++) {

            int taskId = i;

            executor.submit(() -> {

                System.out.println(
                    "Task " + taskId +
                    " executed by " +
                    Thread.currentThread().getName()
                );

            });
        }

        executor.shutdown();
    }
}
```

Possible output:

```text
Task 1 executed by pool-1-thread-1
Task 2 executed by pool-1-thread-2
Task 3 executed by pool-1-thread-3
Task 4 executed by pool-1-thread-1
Task 5 executed by pool-1-thread-2
```

Notice:

```text
5 tasks
3 threads
```

Threads are reused.

---

# 7. `newFixedThreadPool()`

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);
```

This creates a pool with a **fixed number of worker threads**.

Conceptually:

```text
             Task Queue

Task 4 ─┐
Task 5 ─┼── waits
Task 6 ─┘

            ↓

Thread 1 → Task 1
Thread 2 → Task 2
Thread 3 → Task 3
```

Once Thread 1 finishes:

```text
Thread 1 → Task 4
```

---

# 8. Fixed Thread Pool Internals

A fixed thread pool is conceptually configured as:

```text
corePoolSize = n
maximumPoolSize = n

Queue = LinkedBlockingQueue
```

So if you create:

```java
Executors.newFixedThreadPool(5);
```

you effectively have:

```text
core threads = 5
max threads  = 5
```

Additional tasks wait in the queue.

Important:

The queue used by the standard fixed thread pool is effectively **unbounded**.

That means:

```text
Tasks keep arriving
   ↓
All 5 threads busy
   ↓
Tasks keep entering queue
```

If tasks arrive faster than they are processed for a long time, the queue can grow substantially and consume memory.

🔥 Interview point:

> `newFixedThreadPool()` provides bounded thread count, but its default queue is unbounded.

---

# 9. When Should We Use Fixed Thread Pool?

Useful when:

- You want to **limit concurrency**.
- Tasks are relatively predictable.
- You want a stable number of worker threads.
- You don't want a new thread for every incoming task.

For CPU-intensive work, a pool roughly related to the number of CPU cores is often considered.

Example:

```java
Runtime.getRuntime().availableProcessors();
```

But real pool sizing depends on the workload.

---

# 10. CPU-Bound vs I/O-Bound Tasks

This matters a lot in interviews.

## CPU-bound

Examples:

```text
Image processing
Encryption
Compression
Heavy calculations
Sorting large datasets
```

The CPU is busy most of the time.

Too many threads can cause excessive:

```text
context switching
```

For CPU-heavy work, thread count is often kept relatively close to available CPU cores.

---

## I/O-bound

Examples:

```text
Database call
HTTP API call
File access
Network request
```

Threads spend significant time waiting.

So having more threads than CPU cores can sometimes improve throughput because while some threads wait:

```text
Thread A → waiting for DB
Thread B → waiting for API
Thread C → CPU work
```

⭐ There is no single perfect thread-pool size for every application.

---

# 11. `newCachedThreadPool()`

Now:

```java
ExecutorService executor =
        Executors.newCachedThreadPool();
```

A cached thread pool behaves very differently.

Conceptually:

```text
Task arrives
   ↓
Is an idle thread available?
   │
   ├── Yes → reuse it
   │
   └── No → create a new thread
```

So the number of threads can grow dynamically.

---

# 12. Cached Thread Pool Internals

Conceptually:

```text
corePoolSize = 0

maximumPoolSize = very large

Queue = SynchronousQueue
```

A `SynchronousQueue` does **not store tasks like a normal queue**.

It behaves more like a handoff mechanism:

```text
Producer gives task
        ↓
Worker must take it
```

If no worker is available, the pool may create another worker thread.

---

# 13. Fixed vs Cached Thread Pool

Very important interview comparison.

| Feature | FixedThreadPool | CachedThreadPool |
|---|---|---|
| Number of threads | Fixed | Dynamic |
| Queue | `LinkedBlockingQueue` | `SynchronousQueue` |
| Max threads | Same as core size | Very large |
| Reuses threads | Yes | Yes |
| Creates new threads aggressively | No | Yes |
| Best for | Controlled workload | Many short-lived asynchronous tasks |
| Main risk | Queue can grow | Thread count can grow |

Remember:

```text
FixedThreadPool
   ↓
Bound threads
   ↓
Potentially growing queue
```

```text
CachedThreadPool
   ↓
Very flexible thread count
   ↓
Potentially too many threads
```

---

# 14. Why Can Cached Thread Pool Be Dangerous?

Suppose:

```text
100,000 slow tasks arrive
```

If tasks cannot immediately be handed to available workers:

```text
new thread
new thread
new thread
...
```

The thread count can grow significantly.

This can cause:

```text
Memory pressure
Too much context switching
System overload
OutOfMemoryError
```

So don't say:

> CachedThreadPool is always faster.

❌ Incorrect.

---

# 15. `newSingleThreadExecutor()`

Another useful executor:

```java
ExecutorService executor =
        Executors.newSingleThreadExecutor();
```

It uses one worker thread.

Tasks execute sequentially:

```text
Task 1
  ↓
Task 2
  ↓
Task 3
  ↓
Task 4
```

Useful when:

- Tasks must execute in order.
- You want asynchronous execution but only one task at a time.

Example:

```java
ExecutorService executor =
        Executors.newSingleThreadExecutor();

executor.submit(() -> System.out.println("A"));
executor.submit(() -> System.out.println("B"));
executor.submit(() -> System.out.println("C"));

executor.shutdown();
```

The tasks are executed sequentially by a single worker.

---

# 16. Important: `Executors` Is a Factory Class

This class:

```java
Executors
```

contains factory methods such as:

```java
Executors.newFixedThreadPool(...)
Executors.newCachedThreadPool()
Executors.newSingleThreadExecutor()
```

But the actual worker pool implementation is commonly:

```java
ThreadPoolExecutor
```

This is very important.

Think:

```text
Executors
    ↓
Convenient factory methods
    ↓
creates/configures
    ↓
ThreadPoolExecutor
```

---

# 17. `ThreadPoolExecutor` — Core Internals

This is one of the biggest interview topics.

Constructor:

```java
ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue
)
```

There are additional parameters in other constructors, but these are the important core concepts.

Let's understand each.

---

# 18. `corePoolSize`

Example:

```text
corePoolSize = 3
```

The pool tries to maintain up to 3 core worker threads for task execution.

Conceptually:

```text
Thread 1
Thread 2
Thread 3
```

These are the base workers.

---

# 19. `maximumPoolSize`

Suppose:

```text
corePoolSize = 3
maximumPoolSize = 6
```

The pool can grow up to:

```text
6 threads
```

depending on the queue type and workload.

Conceptually:

```text
Core:
Thread 1
Thread 2
Thread 3

Extra:
Thread 4
Thread 5
Thread 6
```

---

# 20. `workQueue`

Tasks that cannot immediately be executed may wait here.

Example:

```text
Thread Pool
   ↓
All core workers busy
   ↓
Task Queue
```

Common queues:

```java
LinkedBlockingQueue
ArrayBlockingQueue
SynchronousQueue
```

Different queues create very different thread-pool behavior.

---

# 21. `keepAliveTime`

Suppose:

```text
core = 3
max  = 6
```

Threads 4, 5, and 6 may be temporary extra workers.

If they stay idle longer than:

```text
keepAliveTime
```

they can be terminated.

Conceptually:

```text
Extra thread
    ↓
Idle
    ↓
wait keepAliveTime
    ↓
terminate
```

This allows the pool to shrink after temporary spikes.

---

# 22. Most Important ThreadPoolExecutor Flow 🔥

Suppose:

```text
corePoolSize = 2
maximumPoolSize = 4

queue capacity = 2
```

Now tasks arrive.

### Task 1

```text
Current threads < corePoolSize
```

Create Thread 1.

```text
Thread 1 → Task 1
```

### Task 2

Create Thread 2.

```text
Thread 1 → Task 1
Thread 2 → Task 2
```

### Task 3

Core threads are busy.

Put task into queue.

```text
Queue → Task 3
```

### Task 4

Queue still has capacity.

```text
Queue → Task 3, Task 4
```

### Task 5

Queue is full.

But:

```text
current threads < maximumPoolSize
```

Create Thread 3.

### Task 6

Create Thread 4.

### Task 7

Now:

```text
Core full
Queue full
Maximum threads reached
```

Task must be **rejected**.

---

# 23. Golden ThreadPoolExecutor Rule

Memorize this flow:

```text
New Task
   ↓

Threads < corePoolSize?
   │
   ├── Yes → Create worker
   │
   └── No
        ↓
Can task enter queue?
   │
   ├── Yes → Queue task
   │
   └── No
        ↓
Threads < maximumPoolSize?
   │
   ├── Yes → Create extra worker
   │
   └── No
        ↓
Reject task
```

🔥 This is a very common interview question.

---

# 24. Important Interview Trap

Many people think:

```text
core threads full
   ↓
immediately create threads until maximumPoolSize
```

That is **not always true**.

Normally, after core workers are occupied, the executor first attempts to:

```text
queue the task
```

Only when the queue cannot accept the task does it consider creating threads beyond the core size.

That's why the queue choice matters so much.

---

# 25. Rejected Execution

When the pool cannot accept a task:

```text
all allowed threads busy
+
queue full
```

a rejection policy is used.

Java provides policies such as:

```java
ThreadPoolExecutor.AbortPolicy
ThreadPoolExecutor.CallerRunsPolicy
ThreadPoolExecutor.DiscardPolicy
ThreadPoolExecutor.DiscardOldestPolicy
```

---

# 26. `AbortPolicy`

This is the default.

It throws:

```text
RejectedExecutionException
```

when the task cannot be accepted.

---

# 27. `CallerRunsPolicy`

Interesting and useful.

The task is executed by the **thread that submitted it**.

Example:

```text
main thread submits task
        ↓
pool overloaded
        ↓
main thread executes task itself
```

This can naturally slow down the producer.

This is a form of **backpressure**.

---

# 28. `DiscardPolicy`

The rejected task is simply discarded.

```text
Task
 ↓
Rejected
 ↓
Dropped
```

No exception.

Potentially dangerous if you need guaranteed task execution.

---

# 29. `DiscardOldestPolicy`

The oldest queued task is removed, and the new task is retried.

Conceptually:

```text
Queue

Old Task  ← removed
Task 2
Task 3

New Task  ← inserted
```

---

# 30. `execute()` vs `submit()`

Very common interview question.

## `execute()`

```java
executor.execute(() -> {
    System.out.println("Task");
});
```

Defined originally by:

```java
Executor
```

It accepts:

```java
Runnable
```

and returns:

```text
void
```

---

## `submit()`

```java
Future<?> future = executor.submit(() -> {
    System.out.println("Task");
});
```

It belongs to:

```java
ExecutorService
```

and returns:

```java
Future
```

It can accept:

```text
Runnable
Callable
```

---

# 31. Quick Comparison — `execute()` vs `submit()`

| Feature | `execute()` | `submit()` |
|---|---|---|
| Defined in | `Executor` | `ExecutorService` |
| Accepts Runnable | Yes | Yes |
| Accepts Callable | No | Yes |
| Returns value | No | `Future` |
| Can track result | No | Yes |
| Can wait for completion | Not directly | Yes |

---

# 32. Exception Difference — `execute()` vs `submit()`

This is a tricky interview point.

Suppose a task throws an exception.

With:

```java
execute()
```

an unchecked exception can reach the worker thread's uncaught-exception handling.

With:

```java
submit()
```

the exception is captured inside the returned:

```java
Future
```

You typically observe it when calling:

```java
future.get();
```

wrapped as:

```text
ExecutionException
```

This distinction is frequently asked.

---

# 33. What Is `Callable`?

`Runnable`:

```java
public interface Runnable {
    void run();
}
```

It:

```text
returns nothing
```

But sometimes we want a background task to calculate and return a result.

For that:

```java
Callable<V>
```

Conceptually:

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

So:

```text
Runnable → no result

Callable → returns result
```

---

# 34. `Callable` Example

```java
import java.util.concurrent.*;

public class Main {

    public static void main(String[] args)
            throws Exception {

        ExecutorService executor =
                Executors.newFixedThreadPool(2);

        Callable<Integer> task = () -> {

            Thread.sleep(1000);

            return 10 + 20;
        };

        Future<Integer> future =
                executor.submit(task);

        Integer result = future.get();

        System.out.println(result);

        executor.shutdown();
    }
}
```

Output:

```text
30
```

---

# 35. What Is `Future`?

A `Future` represents a result that may become available later.

```text
Submit task
   ↓
Future returned immediately
   ↓
Task executes asynchronously
   ↓
Result becomes available
```

Example:

```java
Future<Integer> future =
        executor.submit(task);
```

The task may still be running.

Later:

```java
Integer result = future.get();
```

---

# 36. Important: `future.get()` Is Blocking

Suppose:

```java
Future<Integer> future = executor.submit(task);

System.out.println("Before get");

Integer value = future.get();

System.out.println("After get");
```

If task hasn't finished:

```text
main thread
   ↓
future.get()
   ↓
WAIT
   ↓
task completes
   ↓
return result
```

So although the task itself is asynchronous:

```java
future.get()
```

can block the calling thread.

🔥 Important interview point.

---

# 37. Useful `Future` Methods

```java
future.get();
future.isDone();
future.cancel(true);
future.isCancelled();
```

Meaning:

```text
get()
→ wait for result

isDone()
→ task completed?

cancel(...)
→ attempt cancellation

isCancelled()
→ cancellation succeeded?
```

---

# 38. `Future.get()` With Timeout

Instead of waiting forever:

```java
future.get(2, TimeUnit.SECONDS);
```

If no result is available within 2 seconds:

```text
TimeoutException
```

This is useful in backend systems where you need bounded waiting time.

---

# 39. `shutdown()` vs `shutdownNow()`

Very important.

## `shutdown()`

```java
executor.shutdown();
```

Means:

```text
Stop accepting new tasks

BUT

finish already submitted tasks
```

Conceptually:

```text
Existing tasks → complete

New tasks → rejected
```

---

## `shutdownNow()`

```java
executor.shutdownNow();
```

Attempts to:

```text
stop actively running tasks
+
return queued tasks that never started
```

It typically interrupts worker threads.

But remember:

> Interruption is cooperative.

So `shutdownNow()` **cannot guarantee that every running task instantly stops**.

---

# 40. Executor Lifecycle

Think:

```text
RUNNING
   ↓
shutdown()
   ↓
SHUTDOWN
   ↓
tasks complete
   ↓
TERMINATED
```

or:

```text
RUNNING
   ↓
shutdownNow()
   ↓
STOP
   ↓
TERMINATED
```

Useful methods:

```java
executor.isShutdown();
executor.isTerminated();
executor.awaitTermination(...);
```

---

# 41. Proper Shutdown Example

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);

try {

    executor.submit(() -> {
        System.out.println("Task");
    });

} finally {

    executor.shutdown();
}
```

Often:

```java
executor.awaitTermination(
    10,
    TimeUnit.SECONDS
);
```

is used when the calling code needs to wait for orderly shutdown.

---

# 42. Why Not Forget `shutdown()`?

Executor worker threads are usually non-daemon threads.

If the executor is not shut down, your application may remain alive because those worker threads are still running/waiting.

So this:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);
```

should generally have a corresponding shutdown strategy.

---

# 43. Concurrent Collections

Now let's connect thread pools to shared data.

Normal collections such as:

```java
ArrayList
HashMap
HashSet
```

are not automatically thread-safe for arbitrary concurrent mutation.

Java provides concurrent alternatives such as:

```java
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
```

---

# 44. `ConcurrentHashMap`

Suppose multiple threads access:

```java
Map<String, Integer>
```

Instead of manually synchronizing a normal `HashMap`, we can use:

```java
ConcurrentHashMap<String, Integer> map =
        new ConcurrentHashMap<>();
```

Example:

```java
map.put("Java", 10);

Integer value = map.get("Java");
```

It is designed for concurrent access and typically allows much better concurrency than synchronizing an entire `HashMap`.

---

# 45. Important `ConcurrentHashMap` Point

Don't think:

> ConcurrentHashMap locks the entire map for every operation.

❌ Oversimplification.

Modern implementations use finer-grained synchronization / atomic techniques internally so many operations can proceed concurrently.

Interview takeaway:

```text
Collections.synchronizedMap(...)
        ↓
coarser synchronization

ConcurrentHashMap
        ↓
designed specifically for concurrent access
```

---

# 46. Atomic Compound Operations

Suppose:

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

Even with a concurrent map, composing separate operations like this may introduce race conditions.

Better use atomic APIs such as:

```java
map.putIfAbsent(key, value);
```

or:

```java
map.computeIfAbsent(key, k -> createValue());
```

🔥 Important interview lesson:

> Thread-safe individual operations do not automatically make a sequence of operations atomic.

---

# 47. `CopyOnWriteArrayList`

Useful when:

```text
Reads are frequent
Writes are rare
```

On modification, a new underlying array copy is created.

Conceptually:

```text
Old Array
[A, B, C]

Add D
   ↓

New Array
[A, B, C, D]
```

Readers can safely continue using existing snapshots.

Good for:

```text
Mostly-read configuration/listener lists
```

Not ideal for heavy writes because copying is expensive.

---

# 48. `BlockingQueue`

A very important concurrency abstraction.

Examples:

```java
ArrayBlockingQueue
LinkedBlockingQueue
PriorityBlockingQueue
```

A `BlockingQueue` supports operations that can wait.

For example:

```java
queue.put(value);
```

may wait when the queue is full.

And:

```java
queue.take();
```

may wait when the queue is empty.

This is perfect for:

```text
Producer
   ↓
Queue
   ↓
Consumer
```

---

# 49. Producer-Consumer Pattern

One of the most important multithreading patterns.

```text
Producer Threads
      ↓
      ↓
 BlockingQueue
      ↓
      ↓
Consumer Threads
```

Producer:

```java
queue.put(task);
```

Consumer:

```java
Task task = queue.take();
```

If queue is empty:

```text
consumer waits
```

If bounded queue is full:

```text
producer waits
```

This naturally coordinates producers and consumers.

---

# 50. `ArrayBlockingQueue`

Example:

```java
BlockingQueue<Integer> queue =
        new ArrayBlockingQueue<>(5);
```

Capacity:

```text
5
```

So:

```text
Producer
   ↓
Queue full?
   │
   ├── No → insert
   │
   └── Yes → put() waits
```

This gives us **bounded buffering**.

Very useful when you want controlled memory usage.

---

# 51. `add()` vs `offer()` vs `put()`

Important queue interview topic.

If queue is full:

### `add()`

```java
queue.add(value);
```

throws:

```text
IllegalStateException
```

### `offer()`

```java
queue.offer(value);
```

returns:

```text
false
```

### `put()`

```java
queue.put(value);
```

waits until space becomes available.

Remember:

```text
add()
→ exception

offer()
→ false

put()
→ wait
```

---

# 52. `remove()` vs `poll()` vs `take()`

If queue is empty:

### `remove()`

Throws exception.

### `poll()`

Returns:

```text
null
```

### `take()`

Waits until an item becomes available.

Remember:

```text
remove()
→ exception

poll()
→ null

take()
→ wait
```

---

# 53. `CountDownLatch`

Suppose main thread must wait for 3 worker tasks.

```text
Worker 1 ─┐
Worker 2 ─┼── finish
Worker 3 ─┘

Main waits
```

Use:

```java
CountDownLatch latch =
        new CountDownLatch(3);
```

Each worker:

```java
latch.countDown();
```

Main:

```java
latch.await();
```

When count becomes:

```text
0
```

main continues.

---

# 54. `CountDownLatch` Example

```java
import java.util.concurrent.CountDownLatch;

public class Main {

    public static void main(String[] args)
            throws InterruptedException {

        CountDownLatch latch =
                new CountDownLatch(3);

        for (int i = 1; i <= 3; i++) {

            int workerId = i;

            new Thread(() -> {

                System.out.println(
                    "Worker " + workerId + " done"
                );

                latch.countDown();

            }).start();
        }

        latch.await();

        System.out.println(
            "All workers completed"
        );
    }
}
```

---

# 55. Important `CountDownLatch` Property

A latch is basically:

```text
one-use
```

Once count reaches:

```text
0
```

it cannot be reset.

If you need a reusable synchronization barrier, Java has other tools such as:

```java
CyclicBarrier
```

---

# 56. `Semaphore`

A `Semaphore` controls how many threads may access a resource concurrently.

Example:

```java
Semaphore semaphore =
        new Semaphore(3);
```

Means:

```text
Only 3 permits
```

At most 3 threads can hold permits at once.

Thread:

```java
semaphore.acquire();
```

After work:

```java
semaphore.release();
```

---

# 57. Semaphore Example — Database Connections

Suppose only 3 operations should access a limited resource simultaneously.

```text
Thread 1 → permit
Thread 2 → permit
Thread 3 → permit

Thread 4 → waits
Thread 5 → waits
```

Once Thread 1 releases:

```text
Thread 4 → permit
```

This is useful for protecting **limited resources**, not just a single critical section.

---

# 58. `Semaphore` vs `synchronized`

`synchronized` commonly allows:

```text
1 thread
```

inside a critical section for the same lock.

A semaphore can allow:

```text
N threads
```

Example:

```java
new Semaphore(5);
```

allows up to 5 concurrent permit holders.

Think:

```text
synchronized
→ mutual exclusion

Semaphore
→ concurrency limit
```

---

# 59. `CountDownLatch` vs `Semaphore`

| Feature | CountDownLatch | Semaphore |
|---|---|---|
| Main use | Wait for tasks/events | Limit concurrent access |
| Counter changes | Counts down | Acquire/release permits |
| Reusable | No | Yes |
| Example | Wait for 3 services | Allow 10 DB operations |

---

# 60. Thread Pool + Blocking Queue Connection

Now the bigger picture becomes clear.

Internally, a typical thread pool looks like:

```text
            Submitted Tasks
                  ↓
             Work Queue
                  ↓
        ┌─────────────────┐
        │ Worker Thread 1 │
        │ Worker Thread 2 │
        │ Worker Thread 3 │
        └─────────────────┘
```

The queue controls:

```text
How tasks wait
```

The pool controls:

```text
How many tasks execute concurrently
```

This is why understanding both:

```text
ThreadPoolExecutor
+
BlockingQueue
```

is important.

---

# 61. Backend Example

Suppose your service must process uploaded files.

You receive:

```text
1000 upload-processing tasks
```

Bad design:

```java
new Thread(task).start();
```

for every request.

Potential result:

```text
1000 new threads
```

Better:

```java
ThreadPoolExecutor executor =
    new ThreadPoolExecutor(
        10,
        20,
        60,
        TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(100)
    );
```

Now you control:

```text
Base workers = 10
Maximum      = 20
Queue        = 100
```

This gives predictable resource limits.

---

# 62. Why Explicit `ThreadPoolExecutor` Can Be Better

Factory methods are convenient:

```java
Executors.newFixedThreadPool(...)
```

But in production systems, explicit configuration gives control over:

```text
corePoolSize
maximumPoolSize
queue capacity
keepAliveTime
thread factory
rejection policy
```

That's important because uncontrolled queues or thread counts can cause production issues.

---

# 63. Thread Factory

`ThreadPoolExecutor` can use a:

```java
ThreadFactory
```

to create threads.

This allows customization like:

```text
Thread names
Daemon status
Priority
Uncaught exception handling
```

Example idea:

```java
ThreadFactory factory = runnable -> {

    Thread t = new Thread(runnable);

    t.setName("payment-worker");

    return t;
};
```

Good thread names make debugging much easier.

---

# 64. Interview Scenario

### Question

You have:

```text
corePoolSize = 2
maximumPoolSize = 4
queue capacity = 2
```

All tasks take a long time.

What happens when 7 tasks arrive?

### Answer

```text
Task 1 → Thread 1
Task 2 → Thread 2

Task 3 → Queue
Task 4 → Queue

Task 5 → Thread 3
Task 6 → Thread 4

Task 7 → Rejected
```

Assuming all earlier tasks are still active and the queue remains full.

🔥 You should be able to answer this instantly.

---

# 65. Another Interview Scenario

### Question

Why does `maximumPoolSize` often have no practical effect in a standard `newFixedThreadPool()`?

Because:

```text
corePoolSize == maximumPoolSize
```

So no additional threads beyond the fixed count can be created.

Also, its queue can keep accepting tasks.

---

# 66. Another Important Interview Question

### What happens when you submit 100 tasks to:

```java
Executors.newFixedThreadPool(10);
```

Roughly:

```text
10 tasks → run concurrently

remaining tasks → wait in queue

as workers finish
   ↓
they pick next tasks
```

The executor does not create 100 threads.

---

# 67. Another Important Interview Question

### What happens when you submit many tasks to `newCachedThreadPool()`?

The executor:

```text
reuses idle workers when possible
```

and if none are available:

```text
creates more threads
```

So it can scale thread count aggressively.

---

# 68. Common Mistake — Blocking Inside Small Pool

Suppose:

```text
Pool size = 2
```

Task A waits for Task C.

Task B waits for Task D.

But C and D are also submitted to the same pool.

```text
Thread 1 → Task A → waiting
Thread 2 → Task B → waiting

Queue:
Task C
Task D
```

No thread is available to run C or D.

This can cause a form of:

```text
thread starvation / pool deadlock
```

🔥 Interview insight:

> Thread pools can deadlock even without the classic two-lock deadlock pattern.

---

# 69. Don't Hold Threads Unnecessarily

In backend applications, think carefully before blocking worker threads for long periods.

Examples:

```text
long network waits
slow DB calls
future.get()
locks
sleep()
```

If all pool workers become blocked:

```text
Throughput collapses
```

This is why thread-pool sizing and task behavior must be considered together.

---

# 70. Quick Architecture Understanding

Modern backend request flow can conceptually look like:

```text
HTTP Requests
      ↓
Application Thread Pool
      ↓
Business Logic
      ↓
DB / API / Kafka
```

If thread pool capacity is exhausted:

```text
requests wait
or
requests are rejected
```

So thread pools directly affect:

```text
Latency
Throughput
Resource usage
Stability
```

---

# Interview Questions — Part 3

## Q1. Why use thread pools?

To reuse worker threads, control concurrency, reduce thread-creation overhead, and avoid uncontrolled thread creation.

---

## Q2. What is `ExecutorService`?

A high-level interface for submitting, managing, and shutting down asynchronous tasks.

---

## Q3. FixedThreadPool vs CachedThreadPool?

```text
FixedThreadPool
→ fixed workers
→ queued tasks
→ controlled thread count

CachedThreadPool
→ dynamic workers
→ creates threads when needed
→ potentially very large thread count
```

---

## Q4. What queue does `newFixedThreadPool()` use?

Conceptually, an unbounded:

```java
LinkedBlockingQueue
```

---

## Q5. What queue does `newCachedThreadPool()` use?

```java
SynchronousQueue
```

---

## Q6. Explain `ThreadPoolExecutor` task flow.

```text
1. If workers < corePoolSize
   → create worker

2. Otherwise try queue

3. If queue full and workers < maxPoolSize
   → create extra worker

4. Otherwise
   → reject task
```

---

## Q7. `execute()` vs `submit()`?

```text
execute()
→ Runnable
→ no Future

submit()
→ Runnable or Callable
→ returns Future
```

---

## Q8. Runnable vs Callable?

```text
Runnable
→ run()
→ no result

Callable
→ call()
→ returns result
→ can throw checked exception
```

---

## Q9. What is `Future`?

An object representing the eventual completion/result of an asynchronous task.

---

## Q10. Is `future.get()` blocking?

**Yes.**

If the result isn't ready, the calling thread waits.

---

## Q11. `shutdown()` vs `shutdownNow()`?

```text
shutdown()
→ reject new tasks
→ finish submitted tasks

shutdownNow()
→ attempt to interrupt running tasks
→ return queued tasks
```

---

## Q12. Why can `newFixedThreadPool()` be dangerous?

Because its task queue is effectively unbounded and can grow if tasks arrive faster than they are processed.

---

## Q13. Why can `newCachedThreadPool()` be dangerous?

Because the number of threads can grow very large under sustained load.

---

## Q14. What is a rejection policy?

Defines what happens when the executor cannot accept another task.

---

## Q15. What does `CallerRunsPolicy` do?

The thread submitting the task executes the rejected task itself.

---

## Q16. What is a `BlockingQueue`?

A thread-safe queue that can block producers or consumers when the queue is full or empty.

---

## Q17. `add()` vs `offer()` vs `put()`?

```text
add()
→ exception if full

offer()
→ false if full

put()
→ waits if full
```

---

## Q18. `remove()` vs `poll()` vs `take()`?

```text
remove()
→ exception if empty

poll()
→ null if empty

take()
→ waits if empty
```

---

## Q19. What is `CountDownLatch`?

Allows one or more threads to wait until a specified number of events/tasks complete.

---

## Q20. What is a Semaphore?

Controls the number of threads that can access a limited resource concurrently.

---

# 🔥 Predict the Behaviour

## Question 1

```java
ExecutorService executor =
        Executors.newFixedThreadPool(2);

for (int i = 0; i < 10; i++) {
    executor.submit(task);
}
```

How many worker threads are normally used concurrently?

**Answer:**

```text
2
```

Remaining tasks wait in the queue.

---

## Question 2

```java
Future<Integer> future =
        executor.submit(() -> {
            Thread.sleep(5000);
            return 10;
        });

System.out.println(future.get());
```

Does `get()` return immediately?

**No.**

The calling thread waits until the task completes.

---

## Question 3

```java
executor.shutdown();

executor.submit(task);
```

What happens?

The new task is rejected, typically with:

```text
RejectedExecutionException
```

---

# ⭐ 30-Second Interview Answer

> **Java uses the Executor framework to separate task submission from thread management. `ExecutorService` lets us submit tasks and manage their lifecycle, while `ThreadPoolExecutor` provides the underlying configurable thread-pool behavior. A task is first assigned to a core worker if available; otherwise it is queued, then additional workers up to `maximumPoolSize` may be created if the queue is full, and finally the task is rejected if no capacity remains. `FixedThreadPool` uses a fixed number of workers with an effectively unbounded queue, while `CachedThreadPool` dynamically creates threads and uses a `SynchronousQueue`. `submit()` returns a `Future`, allowing results, waiting, and cancellation.**

---

# Quick Revision

```text
Thread Pool
   ↓
Reusable worker threads

Executor
   ↓
execute(Runnable)

ExecutorService
   ↓
submit()
shutdown()
Future

FixedThreadPool
   ↓
Fixed worker count
   ↓
LinkedBlockingQueue
   ↓
Queue may grow

CachedThreadPool
   ↓
Dynamic threads
   ↓
SynchronousQueue
   ↓
Thread count may grow

ThreadPoolExecutor
   ↓
corePoolSize
maximumPoolSize
workQueue
keepAliveTime
rejection policy

Task arrives
   ↓
workers < core?
   ↓ yes
create worker

otherwise
   ↓
queue task

queue full?
   ↓
workers < maximum?
   ↓ yes
create extra worker

otherwise
   ↓
reject

execute()
   ↓
Runnable
   ↓
void

submit()
   ↓
Runnable / Callable
   ↓
Future

Callable
   ↓
Returns result

Future.get()
   ↓
Blocking wait

shutdown()
   ↓
No new tasks
   ↓
finish old tasks

BlockingQueue
   ↓
Producer/Consumer coordination

CountDownLatch
   ↓
Wait for N events

Semaphore
   ↓
Limit N concurrent users
```

---

# ⭐ Part 3 Priority

Master these for interviews:

1. **Why thread pools are needed**
2. **`Executor` vs `ExecutorService`**
3. **`FixedThreadPool` internals**
4. **`CachedThreadPool` internals**
5. **`FixedThreadPool` vs `CachedThreadPool`**
6. **`ThreadPoolExecutor`**
7. **`corePoolSize`**
8. **`maximumPoolSize`**
9. **`workQueue`**
10. **`keepAliveTime`**
11. **Exact task-allocation flow**
12. **Rejection policies**
13. **`execute()` vs `submit()`**
14. **`Runnable` vs `Callable`**
15. **`Future` and why `get()` blocks**
16. **`shutdown()` vs `shutdownNow()`**
17. **`BlockingQueue`**
18. **`ConcurrentHashMap`**
19. **`CountDownLatch`**
20. **`Semaphore`**
21. **Thread-pool starvation/deadlock scenario**

These concepts prepare you for **Part 4: deadlocks, deadlock prevention, lock ordering, `tryLock()`, starvation, livelock, thread-safe Singleton, double-checked locking, and final multithreading interview questions**.
