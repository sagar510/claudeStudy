# 🧵 THREADING — HOUR 2: Creating Threads — Thread, Runnable, Callable & Futures
### Course chapters 17–22 · LinkedIn carousel page 2, Q1–Q2

> *The instructor's flow here mirrors a real interview: "how do you create threads?" → "which would you choose and why?" → "do you know Callable?" → "difference?" — he literally calls this the trap-chain. We walk it in that exact order.*

---

## 2.1 Way #1 — extend Thread (the beginner door)

```java
class MyThread extends Thread {          // ① inherit from Thread
    @Override
    public void run() {                  // ② the work goes in run()
        System.out.println("baking on " + getName());
    }
}

MyThread t1 = new MyThread();
MyThread t2 = new MyThread();
t1.start();                              // ③ START, not run — spawns a real thread
t2.start();
```

What actually happens — follow the timeline:

1. Your program begins on the **main thread** (Hour 1: mom — the kitchen opens with her).
2. Main thread creates objects t1, t2 — *just objects so far, no new workers yet.*
3. `t1.start()` → a **new thread is spawned**, and that new thread goes looking for the `run()` method and executes it.
4. Meanwhile — and this is the key word, *meanwhile* — the **main thread didn't wait**. It moved on immediately to `t2.start()`. Now three workers are alive at once: main, t1, t2.

**And the output order?** Run it twice, get two different interleavings: `t1, t2, t1, t1, t2…` one time, `t2, t1, t2…` the next. Neither is wrong. The instructor stresses this hard: **thread scheduling is non-deterministic** — a thread might get delayed, stuck, or scheduled late. *Kitchen: you and your sister both baking — who finishes step 3 first varies by the day. Nobody promised an order.*

🗣️ *"start() returns immediately and the new thread runs asynchronously; execution order across threads is unpredictable unless we explicitly coordinate them."* ← this sentence pre-answers three future questions (join, wait, synchronization).

His side-note on `Thread.sleep(1000)` inside run(), with his exact analogy — keep it: *the thread is a person who, mid-task, **puts up his bed and sleeps** right there.* Wakes after the timeout, resumes exactly where he lay down. (Full sleep() treatment is H3 — hold the bed image.)

---

## 2.2 Way #2 — implement Runnable (the RIGHT door)

Same work, different packaging:

```java
class BakeTask implements Runnable {     // ① implement the INTERFACE
    @Override
    public void run() { System.out.println("baking!"); }
}

Runnable task = new BakeTask();
Thread t1 = new Thread(task);            // ② hand the TASK to a Thread
t1.start();

// modern short form (Runnable has one method → lambda works):
new Thread(() -> System.out.println("baking!")).start();
```

Only two changes: `extends Thread` → `implements Runnable`, and you **pass the task into** a Thread's constructor instead of *being* one.

**So why is this THE answer to "which would you choose?"** The instructor's core argument, and it's the exact one interviewers expect:

**Java allows extending only ONE class.** If your class extends Thread, its inheritance slot is *spent*. But real business classes often need to extend something meaningful (`ReportGenerator extends BaseGenerator`). Extend Thread and you're — his word — *screwed*. Implementing an interface costs nothing: you can implement many, and still extend whatever you need.

There's a second, deeper reason worth adding (it makes you sound senior): extending Thread **confuses two identities** — WHAT to do (the task) with WHO does it (the worker). Runnable separates them: the task is one object, the worker another. *Kitchen: a recipe card (Runnable) handed to any cook (Thread) — versus surgically attaching the recipe to one specific cook's arm.* This separation is exactly why Runnable plugs into thread pools later (H4): pools want recipe cards they can hand to any of their workers.

His honest production note — quote-worthy: *"Is `new Thread(...)` used in production? **Never.** Real code uses ExecutorService to manage threads. But Runnable itself? Used everywhere — it's the task format everything else consumes."* (Carousel page 8 Q5 — "why is new Thread() rarely used in production" — just got pre-answered.)

His analogy for "why not still use Thread sometimes?": *you own a pocket knife and a pencil knife of the same size — why keep the pencil knife at all?* Runnable does everything Thread-extension does, minus the drawback. Just always use Runnable.

> ### 💬 INTERVIEW CHECKPOINT — carousel p.2 Q1, and the #1 threading opener anywhere
>
> **Q: "What are the ways to create a thread in Java, and which do you prefer?"**
> 🗣️ *"Three ways: extend Thread, implement Runnable, or implement Callable when I need a result. I prefer Runnable — Java has single inheritance, so extending Thread burns the only extends slot; Runnable also separates the task from the execution mechanism, which is exactly the shape thread pools and executors consume. Extending Thread is fine only for quick demos."*
>
> **Follow-up trap: "Can you make a thread WITHOUT extending Thread or implementing Runnable?"**
> 🗣️ *"Callable with an ExecutorService — or lambda syntax — but underneath, everything reduces to a Runnable/Callable handed to some Thread. There's no fourth mechanism, just packaging."*

---

## 2.3 Way #3 — Callable: "run() returning void felt bad"

The instructor's setup for Callable is the best way to feel the need for it:

> *"Imagine every function you ever wrote could only return **void**. How bad would you feel?"*

That's Runnable's life: `void run()`. The thread does work... and can't hand anything back. What if the task is "compute the report total" or "fetch the user from the API"? You NEED the result. Enter **Callable**:

```java
Callable<Integer> countCookies = () -> {
    Thread.sleep(1000);                  // pretend: slow counting
    return 42;                           // ← RETURNS a value! (run() never could)
};
```

Two upgrades over Runnable, both from its signature `V call() throws Exception`:

1. **Returns a value** — `Callable<Integer>` returns an Integer, `Callable<User>` a User.
2. **Can throw checked exceptions** — `run()` can't (its signature has no `throws`); `call()` can throw `IOException`, `SQLException`, etc. and let the caller handle them properly.

(He does a checked/unchecked exceptions detour here — you already own this cold from our exceptions deep-dive: checked = declared & handled at compile time, IOException/SQLException; unchecked = runtime bugs, NullPointer/ArithmeticException. One connection to make: **InterruptedException** — the one `sleep()` throws — is *checked*, which is why sleep always forces a try-catch. Now you know why.)

---

## 2.4 Futures — the receipt for a result that's still cooking

Here's the puzzle Callable creates. The whole POINT of a thread is that the caller doesn't wait. But Callable returns a value... **when?** The caller is 10 lines ahead already; the value doesn't exist yet!

Java's answer: you don't get the value — **you get a `Future<V>`: a receipt.** His framing, worth keeping verbatim: *"a Future is something that will come... in the future."*

```java
ExecutorService pool = Executors.newFixedThreadPool(2);   // (H4's topic — for now: "a thread manager")

Future<Integer> receipt = pool.submit(countCookies);      // task starts on another thread;
                                                          // I get the receipt IMMEDIATELY
System.out.println("meanwhile, main thread keeps working...");   // not blocked!

Integer cookies = receipt.get();       // ⛔ NOW I block — stand at the counter
System.out.println(cookies);           //    until the result is actually ready
pool.shutdown();
```

*Kitchen analogy, extended: you order a custom cake at the bakery counter. They don't make you stand there for 40 minutes — they hand you a **token slip (Future)** and you go do your shopping (main thread continues). When you return and present the slip — `get()` — either the cake is ready (instant hand-over) or you **wait at the counter** until it is. The slip is not the cake; it's the promise of the cake.*

The critical behavior to state precisely: **`get()` BLOCKS.** Call it too early and your "asynchronous" code quietly becomes synchronous — the main thread stands at the counter. That's Future's known weakness, and (say this to sound ahead of the curve) it's exactly what **CompletableFuture** fixes in H7 with non-blocking callbacks. Useful extras on the receipt: `isDone()` (peek without waiting), `get(2, TimeUnit.SECONDS)` (wait at most 2s, then TimeoutException), `cancel()`.

Note the pairing he emphasizes: **Callable doesn't work with bare `new Thread()`** (Thread's constructor only accepts Runnable). Callable is *designed* for `executor.submit()` → Future. Runnable rides bare Threads; Callable rides executors.

---

## 2.5 The three-way table — carousel p.2 Q2, made permanent

| | extends Thread | Runnable | Callable |
|---|---|---|---|
| Type | class | interface | interface |
| Work method | `void run()` | `void run()` | `V call() throws Exception` |
| Returns a value? | ❌ | ❌ | ✅ |
| Throws checked exceptions? | ❌ | ❌ | ✅ |
| Blocks the extends slot? | ✅ the drawback | ❌ | ❌ |
| Runs via | itself | `new Thread(r)` / pools | `executor.submit()` → Future |
| Use it when | demos only | fire-and-forget tasks | need the result / exception back |

🗣️ **Carousel Q2, ready to recite:** *"Runnable's run() returns nothing and can't throw checked exceptions — fire-and-forget. Callable's call() returns a value and can throw checked exceptions, and it's used with an ExecutorService, which returns a Future to retrieve the result later."*

---

## 2.6 Best practices (Ch 22, compressed to what's actually said in interviews)

1. **Always Runnable/Callable over extending Thread** (inheritance + separation).
2. **Never raw `new Thread()` in production** — executors manage lifecycle, reuse, limits (H4).
3. **Name your threads** (`new Thread(task, "report-worker")`) — when production hangs, a thread-dump full of "Thread-47" is misery; names are free diagnostics.
4. **Threads are cheap, not free** — his "mark my words" line from H1; don't create-and-destroy in loops.

---

## 2.7 The 30-second story

> *"Three creation routes: extend Thread — works but burns the single-inheritance slot and welds task to worker, so it's demo-only; implement Runnable — the standard: a task object handed to a Thread or, in real code, to an executor; and Callable when I need a result — call() returns a value and can throw checked exceptions, submitted to an executor which hands back a Future, a receipt whose get() blocks until the result exists. Production code never spawns raw threads — it submits Runnables and Callables to pools."*

---

## 🎤 HOUR 2 DRILL — out loud

1. Three ways to create a thread + which and why? *(Runnable — inheritance slot + task/worker separation)*
2. What happens on the main thread when t1.start() is called? *(nothing waits — it moves on; new thread runs run() asynchronously)*
3. Why is thread output order different every run? *(scheduling is non-deterministic — no promised order)*
4. Deeper reason Runnable beats extending Thread, beyond inheritance? *(separates WHAT from WHO — recipe card vs cook; pool-compatible)*
5. Runnable vs Callable — the two signature differences? *(returns V; throws checked exceptions)*
6. Why does sleep() force a try-catch? *(InterruptedException is a CHECKED exception)*
7. What is a Future in one line? *(a receipt for a result still being computed)*
8. What's the danger of future.get()? *(it blocks — call it early and async becomes sync)*
9. Can Callable run on a bare new Thread()? *(no — Thread takes only Runnable; Callable pairs with executor.submit())*
10. Why is new Thread() absent from production code? *(no reuse/limits/lifecycle — executors do that; H4 incoming)*

---

**Next → Hour 3:** The classic comparisons + lifecycle — start() vs run(), sleep() vs wait(), notify() vs notifyAll(), and the thread state diagram (course chapters 23–25, 27 · carousel p.1 Q4, p.2 Q3–Q7).
