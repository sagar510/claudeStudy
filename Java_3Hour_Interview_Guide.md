# ☕ Java in 3 Hours — Generics · Exceptions · Threading
### Zero → medium-advanced · Interview tomorrow · Built to be un-forgettable

> **How to use:** Each hour = Understand with analogy (20 min) → Type the code yourself (25 min) → Interview drill (15 min).
> Every section ends with a **"Say this in the interview"** line — memorize those sentences word-for-word.

---

# ⏰ HOUR 1 — Exceptions: When Things Go Wrong
*(Easiest topic, fastest win — we start here to build momentum)*

## 1.1 The core idea

Your program is a **delivery boy on a route**. Most days, deliveries go fine. But sometimes:
- The address doesn't exist (bad input)
- The road is blocked (file missing, network down)
- The bike engine explodes (JVM out of memory)

An **exception** is Java's way of saying *"something went wrong at THIS step"* — and instead of the whole program dying silently, Java hands you a report (the exception object) and asks: **"How do you want to handle this?"**

## 1.2 The Exception Hierarchy (draw this once, remember forever)

```
                Throwable
               /         \
          Error           Exception
     (engine exploded)   /          \
     DON'T catch these  IOException,     RuntimeException
     OutOfMemoryError   SQLException...  (unchecked)
     StackOverflowError (CHECKED)        NullPointerException
                                         ArrayIndexOutOfBounds
                                         ArithmeticException
```

**The one distinction interviewers always ask — Checked vs Unchecked:**

| | Checked | Unchecked (RuntimeException) |
|---|---|---|
| Compiler forces you to handle? | ✅ YES — won't compile otherwise | ❌ No |
| Cause | **Outside world failing** (file, network, DB) | **Your bug** (null, bad index, ÷0) |
| Examples | IOException, SQLException | NullPointerException, ArithmeticException |
| Analogy | Road blocked — not your fault, but you MUST have a plan B | You forgot the address — fix your code |

🗣️ **Say this in the interview:** *"Checked exceptions represent recoverable external failures and the compiler forces handling; unchecked exceptions extend RuntimeException and usually indicate programming bugs."*

**Error vs Exception:** Errors (OutOfMemoryError, StackOverflowError) are JVM-level disasters you should NOT catch — the bike engine exploded; catching it won't help. Exceptions are situations you can plan for.

## 1.3 try / catch / finally — the safety net

```java
try {
    int result = 10 / 0;              // 💥 ArithmeticException thrown here
} catch (ArithmeticException e) {      // caught here — program survives
    System.out.println("Can't divide by zero: " + e.getMessage());
} finally {
    System.out.println("Runs ALWAYS — exception or not");
}
```

Rules interviewers probe:
- **Multiple catch blocks?** Yes — but order matters: **child before parent** (`ArithmeticException` before `Exception`), else compile error "already caught."
- **finally always runs?** Yes — even after a `return` in try. Only skipped by `System.exit()` or JVM crash. Use it for cleanup (closing files/connections).
- **Exception inside finally?** It **replaces** any exception from the try block — the original is lost. That's why heavy logic in finally is a bad practice.

## 1.4 throw vs throws (2-second answer)

```java
void readFile(String path) throws IOException {   // throws = WARNING on the door:
    if (path == null)                              //   "this method MAY throw, caller beware"
        throw new IllegalArgumentException("null path");  // throw = actually THROWING it, now
}
```

🗣️ *"`throw` is the action of raising one exception object; `throws` is the method-signature declaration warning callers which checked exceptions may escape."*

*Analogy: `throws` = the "⚠ Wet Floor" sign. `throw` = actually slipping.*

## 1.5 Exception propagation

If a method doesn't catch an exception, it **bubbles up** the call stack: `methodC → methodB → methodA → main → JVM kills program + prints stack trace`.

*Analogy: A complaint escalating — employee can't handle it → manager → director → CEO. If even the CEO (main) ignores it, the company (program) shuts down.*

## 1.6 try-with-resources (the modern answer — say this to look senior)

```java
// OLD: manual finally to close             // NEW (Java 7+): auto-close
try (BufferedReader br = new BufferedReader(new FileReader("a.txt"))) {
    System.out.println(br.readLine());
}   // br.close() called AUTOMATICALLY, even on exception
```
Anything implementing `AutoCloseable` gets closed for you — no leaked files/connections.

## 1.7 Custom exceptions + best practices

```java
class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String msg) { super(msg); }
}
// usage: throw new InsufficientBalanceException("Balance: 500, needed: 900");
```

**Best-practice checklist (from your LinkedIn saves — this Q was there!):**
1. Catch specific exceptions, never bare `catch (Exception e) {}` (swallowing = hiding bugs)
2. Never leave a catch block empty — at least log it
3. Use try-with-resources for anything closeable
4. Use custom exceptions for business rules (readable + precise)
5. In Spring: **@ControllerAdvice + @ExceptionHandler** = Global Exception Handling — one class catches exceptions from ALL controllers and returns clean error responses (your notes mentioned this — it's a common Spring follow-up).

## 🎤 Hour 1 drill — answer out loud, 30 seconds each
1. Checked vs unchecked? (table above)
2. throw vs throws? (wet-floor sign)
3. Does finally always run? (yes, except System.exit)
4. Can a constructor throw? (**Yes** — and it's good practice for invalid arguments)
5. Error vs Exception? (JVM disaster vs handleable situation)
6. What is exception propagation? (bubbles up the call stack until caught)

---

# ⏰ HOUR 2 — Generics: Type Safety Without Repetition

## 2.1 The problem Generics solve

Before Java 5, collections held plain `Object`s:

```java
List list = new ArrayList();       // raw list — accepts ANYTHING
list.add("hello");
list.add(42);                      // compiler is happy 😬
String s = (String) list.get(1);   // 💥 ClassCastException AT RUNTIME
```

The bug hides until the program is running — the worst time to find it.

```java
List<String> list = new ArrayList<>();   // generic list
list.add("hello");
list.add(42);                     // ❌ COMPILE ERROR — caught instantly
String s = list.get(0);           // no cast needed
```

*Analogy: A raw List is an unlabeled storage box — anyone can toss anything in, and you discover the mistake when you reach in and grab a snake instead of a rope. `List<String>` is a box labeled "ROPES ONLY" — the label is enforced at the door (compile time).*

🗣️ **Say this:** *"Generics move type errors from runtime to compile time and eliminate manual casting."*

## 2.2 Generic classes and methods — T is just a placeholder

```java
class Box<T> {                       // T = "some type, decided later"
    private T item;
    public void put(T item) { this.item = item; }
    public T get() { return item; }
}

Box<String> b1 = new Box<>();   b1.put("hi");     // T = String
Box<Integer> b2 = new Box<>();  b2.put(42);       // T = Integer
```

One class, works for every type, fully type-safe. Generic **method**:

```java
public static <T> T firstElement(List<T> list) {   // <T> before return type
    return list.get(0);
}
```

Convention letters (know these — interviewers notice): **T**=Type, **E**=Element, **K**=Key, **V**=Value (like `Map<K,V>`).

## 2.3 Bounded types — "any type, as long as it can do X"

```java
class Calculator<T extends Number> {        // T must BE-A Number
    double half(T value) { return value.doubleValue() / 2; }   // Number's methods available
}
Calculator<Integer> ok = new Calculator<>();     // ✅
Calculator<String>  no = new Calculator<>();     // ❌ compile error
```

*Analogy: A job posting. `<T>` = "anyone may apply." `<T extends Number>` = "must have a driving license" — now you can safely assign driving tasks (call Number's methods).*

## 2.4 Wildcards — ? , ? extends, ? super (the medium-advanced part)

The trap first: **`List<Dog>` is NOT a `List<Animal>`**, even though Dog extends Animal. Why? If it were allowed:

```java
List<Animal> animals = dogList;   // imagine this compiled
animals.add(new Cat());           // a CAT just entered the dog list! 💥
```

Wildcards fix this safely:

| Wildcard | Meaning | You can... | Use when |
|---|---|---|---|
| `List<?>` | list of unknown type | read as Object only | you only iterate/print |
| `List<? extends Animal>` | Animal or any subclass | **READ** as Animal; can't add | method **consumes/reads** data |
| `List<? super Dog>` | Dog or any superclass | **WRITE** Dogs; read as Object | method **produces/writes** data |

**The mnemonic that survives interview stress — PECS: "Producer Extends, Consumer Super."**
- The list *produces* values for you to read → `? extends`
- The list *consumes* values you put in → `? super`

```java
double sum(List<? extends Number> nums) { ... }   // reading numbers OUT → extends
void fillWithDogs(List<? super Dog> list) { list.add(new Dog()); }  // putting IN → super
```

## 2.5 Type Erasure — the "senior" answer

At runtime, generics **disappear**. The compiler checks everything, then erases: `List<String>` and `List<Integer>` both become plain `List` in bytecode.

Consequences interviewers fish for:
- `list instanceof List<String>` → ❌ illegal (type info gone at runtime)
- Can't do `new T()` or `new T[10]`
- `List<String>.class` doesn't exist — only `List.class`

*Analogy: Generics are scaffolding — essential while building (compiling), removed before the building opens (runtime). The safety was enforced during construction, so it's no longer needed.*

🗣️ **Say this:** *"Generics are compile-time only; the compiler enforces type safety and then erases type parameters — that's type erasure, kept for backward compatibility with pre-Java-5 code."*

## 🎤 Hour 2 drill
1. Why generics? → compile-time safety, no casts.
2. `List<Dog>` a subtype of `List<Animal>`? → No; use wildcards; explain the Cat-in-dog-list disaster.
3. PECS? → Producer Extends, Consumer Super.
4. What is type erasure? → generics removed at runtime; backward compatibility.
5. `<T extends Number>` means? → upper bound; T must be Number or subclass.

---

# ⏰ HOUR 3 — Threading: Doing Many Things at Once
*(Your 8-page LinkedIn Q&A carousel maps 1-to-1 onto this hour — I'll flag the matches)*

## 3.1 Thread vs Process, and why bother

- **Process** = a whole restaurant: own building, own kitchen, own stock (isolated memory).
- **Thread** = a waiter inside that restaurant: many waiters share the same kitchen and stock (shared memory), each serving different tables at once.

**Why multithreading?** One waiter serving 50 tables = customers wait forever. Multiple waiters = responsiveness + throughput. In code: handle many client requests, do I/O without freezing the app. *(Carousel p.1 Q1–Q2.)*

**Concurrency vs Parallelism** *(carousel Q3)*: Concurrency = ONE waiter juggling 5 tables by switching between them (interleaving on one core). Parallelism = 5 waiters literally working at the same instant (multiple cores). Concurrency is structure; parallelism is hardware.

**Context switching** *(Q5)*: the CPU pausing one thread, saving its state, loading another. Necessary, but too much switching = overhead — a waiter who switches tables every 3 seconds serves nobody well.

## 3.2 Creating threads *(carousel p.2)*

```java
// Way 1: extend Thread (rarely preferred — burns your only inheritance slot)
class MyThread extends Thread { public void run() { System.out.println("Hi"); } }
new MyThread().start();

// Way 2: implement Runnable (preferred)
Runnable task = () -> System.out.println("Hi from " + Thread.currentThread().getName());
new Thread(task).start();

// Way 3: Callable — like Runnable but RETURNS a value & can throw checked exceptions
Callable<Integer> job = () -> 40 + 2;
```

**The classic trap — start() vs run():** `start()` creates a real new thread (new call stack, JVM scheduler). Calling `run()` directly is just a normal method call **on the current thread** — no concurrency at all. Calling `start()` twice → `IllegalThreadStateException`. *(Q3–Q4.)*

**Thread lifecycle** *(p.1 Q4)*: `NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED`. Check with `Thread.getState()`.

**sleep() vs join() vs wait()** — the three-way classic:
| | What it does | Releases lock? |
|---|---|---|
| `sleep(ms)` | pause current thread for a time | ❌ No |
| `join()` | current thread waits for ANOTHER thread to finish | — |
| `wait()` | release the lock and wait until `notify()` | ✅ Yes (must be inside synchronized) |

🗣️ *"sleep pauses without releasing locks; wait releases the monitor and waits for notify; join waits for another thread's completion."*

## 3.3 The core problem: Race Conditions *(p.5 Q1)*

```java
class Counter { int count = 0; void increment() { count++; } }
// 2 threads × 10,000 increments each → final count is often < 20,000. WHY?
```

`count++` is secretly **3 steps**: read → add 1 → write back. Two threads can both read `5`, both write `6` — one update is **lost**.

*Analogy: A shared bank account with ₹5000. You and your spouse both check the balance (5000), both withdraw 3000 at the same moment at different ATMs, both machines write back "2000." The bank just lost ₹3000. That's a race condition — the outcome depends on unpredictable timing.*

## 3.4 synchronized — one at a time *(p.4, all 7 questions)*

```java
synchronized void increment() { count++; }        // method-level lock
// or finer:
void increment() { synchronized(this) { count++; } }   // block-level
```

- Every Java object has a hidden **monitor (intrinsic lock)**. Entering synchronized = acquiring that lock; everyone else queues.
- **Block-level > method-level** when only part of the method is critical — lock less, contend less.
- **Object-level vs class-level lock:** normal synchronized method locks `this` (per instance); `static synchronized` locks the `Class` object (shared by ALL instances).
- **Reentrant:** a thread already holding a lock can re-enter it (a synchronized method calling another synchronized method on the same object won't deadlock itself).

*Analogy: A single-key washroom. One key (monitor), whoever holds it is inside, everyone else waits at the door. Reentrant = the person inside can open the inner cabinet with the same key.*

## 3.5 volatile & Atomics *(p.4 Q5–6, p.6 Q1–2)*

**volatile** = "always read/write this variable from MAIN memory, never a thread's local CPU cache." Guarantees **visibility** (thread B instantly sees thread A's write) — but **NOT atomicity**: `volatile int i; i++` is still 3 steps and still races.

**AtomicInteger** fixes that with **CAS (Compare-And-Swap)**: a CPU instruction that says "update this value to 6 ONLY IF it's still 5 — atomically." No lock needed, faster under contention.

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();     // truly thread-safe, lock-free
```

🗣️ *"volatile gives visibility, not atomicity; for atomic counters use AtomicInteger, which relies on CAS instead of locking."*

## 3.6 Deadlock & friends *(p.5)*

**Deadlock:** Thread A holds Lock 1, wants Lock 2. Thread B holds Lock 2, wants Lock 1. Both wait forever.
*Analogy: Two people on a narrow bridge, each refusing to back up.*
**Prevention (memorize 3):** ① always acquire locks in the same global order, ② use `tryLock()` with timeout, ③ hold as few locks, as briefly, as possible.

- **Livelock:** both keep *reacting* but nobody progresses — two people in a corridor stepping side-to-side forever.
- **Starvation:** one thread never gets a turn because others always win priority.

**Thread-safe class** = behaves correctly under concurrent access — via synchronization, **immutability** (nothing to corrupt — final fields, no setters), or confinement (e.g., **ThreadLocal**: each thread gets its own private copy, like each waiter carrying their own notepad).

## 3.7 ExecutorService & Thread Pools *(p.7 — this makes you sound production-ready)*

Creating raw threads per task = hiring a new waiter for every order, firing him after. Insane. A **thread pool** = a fixed staff of waiters pulling tasks from a queue.

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
Future<Integer> future = pool.submit(() -> 40 + 2);   // Callable → Future
Integer answer = future.get();     // BLOCKS until result ready
pool.shutdown();                   // ⚠ ALWAYS — pool threads are non-daemon,
                                   //    an unshut pool keeps the JVM alive forever
```

- **Pool types:** `FixedThreadPool` (n threads), `CachedThreadPool` (grows/shrinks), `SingleThreadExecutor` (sequential), `ScheduledThreadPool` (timers/periodic).
- **Future** = a receipt for a result being cooked; `get()` = standing at the counter until it's ready.
- **CompletableFuture** = a smarter receipt: "when ready, THEN do this, THEN that" (`thenApply`, `thenCompose`), combine several, no blocking. The modern async answer.
- **ForkJoinPool** = divide-and-conquer specialist with work-stealing (idle waiters grab tasks from busy waiters' queues).

**Bonus vocabulary:** ConcurrentHashMap (fine-grained internal locking — way better than locking a whole HashMap), CopyOnWriteArrayList (copies array per write — perfect for read-heavy/write-rare), BlockingQueue (producer waits when full, consumer waits when empty — natural producer-consumer). *(p.6.)*

**Daemon thread** *(p.1 Q6)*: background helper (e.g., garbage collector) that does NOT keep the JVM alive — when only daemons remain, JVM exits.

## 🎤 Hour 3 drill — rapid fire
1. Thread vs process? → shared memory vs isolated program.
2. start() vs run()? → new thread vs plain method call.
3. Race condition? → bank-ATM story + "count++ is 3 steps."
4. synchronized does what? → acquires the object's monitor; one thread at a time.
5. volatile? → visibility only, NOT atomicity → AtomicInteger/CAS for counters.
6. Deadlock + 2 preventions? → lock ordering, tryLock timeout.
7. Why ExecutorService over new Thread()? → reuse, control, backpressure; always shutdown().
8. Runnable vs Callable? → void vs returns value + checked exceptions.
9. Future vs CompletableFuture? → blocking get() vs chainable callbacks.

---

# 📋 THE NIGHT-BEFORE CHEAT SHEET

## Ten sentences that answer 80% of questions
1. Checked = external recoverable failure, compiler-enforced; unchecked = programming bug, extends RuntimeException.
2. `throw` raises an exception; `throws` declares it on the signature.
3. finally always runs (except System.exit); prefer try-with-resources for cleanup.
4. Generics = compile-time type safety, no casts; erased at runtime (type erasure).
5. `List<Dog>` is not `List<Animal>` — use wildcards; **PECS: Producer Extends, Consumer Super.**
6. start() spawns a real thread; run() is just a method call.
7. Race condition = shared data + unsynchronized read-modify-write; count++ is 3 steps.
8. synchronized = object's monitor lock, one thread at a time, reentrant.
9. volatile = visibility not atomicity; AtomicInteger uses CAS, lock-free.
10. Production code uses ExecutorService pools, not raw threads — and always shuts them down.

## Analogy index (your memory hooks)
| Concept | Hook |
|---|---|
| Exception propagation | complaint escalating employee → CEO |
| throws vs throw | wet-floor sign vs actually slipping |
| Raw list vs generic | unlabeled box → snake instead of rope |
| Type erasure | scaffolding removed after construction |
| Process vs thread | restaurant vs waiters sharing one kitchen |
| Race condition | two ATMs withdrawing from one account |
| synchronized | single-key washroom |
| Deadlock | two people on a narrow bridge |
| Livelock | corridor side-step dance |
| ThreadLocal | each waiter's own notepad |
| Thread pool | fixed staff of waiters + order queue |
| Future.get() | waiting at the counter for your order |

Sleep well. You know more than you think. 🚀
