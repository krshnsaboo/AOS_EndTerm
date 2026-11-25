# Topic 4: Synchronization Tools, Monitors & Classical Problems (Ch. 6–7)

> **Overview**: Dijkstra's motivation for semaphores, Progress condition, busy waiting vs. blocking, Solaris adaptive mutexes & turnstiles, monitors vs. semaphores (`c.wait`/`c.signal`), Single-Lane Bridge (semaphore & monitor), Readers-Writers, Jurassic Park, and lock-free partitioning.  
> **Reference Material**: [procSync_OS-2025.ppt](../Faculty_Notes/procSync_OS-2025.ppt) | [EWD1303.pdf](../Faculty_Notes/EWD1303.pdf)  
> **Questions**: 13 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [What Motivated Dijkstra to Introduce Semaphores?](#1-what-motivated-dijkstra-to-introduce-semaphores-frequency-1) — Frequency: 1
2. [Implication When a Critical Section Solution Violates the "Progress" Requirement](#2-implication-when-a-critical-section-solution-violates-the-progress-requirement-frequency-1) — Frequency: 1
3. [Busy Waiting (Spinlocks): Issue & System Implications](#3-busy-waiting-spinlocks-issue-system-implications-frequency-1) — Frequency: 1
4. [Solaris Synchronization: Adaptive Mutexes vs. Semaphores / RW Locks](#4-solaris-synchronization-adaptive-mutexes-vs-semaphores-rw-locks-frequency-1) — Frequency: 1
5. [Solaris Turnstiles: Memory Efficiency & Priority Inversion Resolution](#5-solaris-turnstiles-memory-efficiency-priority-inversion-resolution-frequency-1) — Frequency: 1
6. [T/F: "No Difference Between 'wait' in Semaphore and Conditional 'wait' in Monitor"](#6-tf-no-difference-between-wait-in-semaphore-and-conditional-wait-in-monitor-frequency-1) — Frequency: 1
7. [Why Monitors Need Condition Variables (c.wait / c.signal) & Differences vs. Semaphores](#7-why-monitors-need-condition-variables-cwait-csignal-differences-vs-semaphores-frequency-2) — **Frequency: 2**
8. [T/F: "With Monitors, It Is Possible to Reduce Busy Waiting Over Semaphores"](#8-tf-with-monitors-it-is-possible-to-reduce-busy-waiting-over-semaphores-frequency-3) — **Frequency: 3**
9. [Single-Lane Highway / Vermont Bridge Synchronization Using Semaphores](#9-single-lane-highway-vermont-bridge-synchronization-using-semaphores-frequency-3) — **Frequency: 3**
10. [Single-Lane Bridge Synchronization Using a Monitor (Mesa Semantics)](#10-single-lane-bridge-synchronization-using-a-monitor-mesa-semantics-frequency-1) — Frequency: 1
11. [IndixOS Starvation-Free Readers–Writers Problem Using POSIX Semaphores](#11-indixos-starvation-free-readerswriters-problem-using-posix-semaphores-frequency-1) — Frequency: 1
12. [Embarrassingly Parallel Disjoint File Record Processing: Do We Need Locks or Condition Variables?](#12-embarrassingly-parallel-disjoint-file-record-processing-do-we-need-locks-or-condition-variables-frequency-1) — Frequency: 1
13. [Jurassic Park Safari Rides & Passengers Synchronization Using Semaphores](#13-jurassic-park-safari-rides-passengers-synchronization-using-semaphores-frequency-1) — Frequency: 1

---

### 1. What Motivated Dijkstra to Introduce Semaphores? [Frequency: 1]

> **Question**: What motivated Dijkstra to introduce semaphores?

#### pre-requisite
- **Reference Paper**: Edsger W. Dijkstra, *"My recollections of operating system design"* (`EWD1303.pdf`, pages 22–24 & 29).

#### Answer (Exam Writing Format)
From Dijkstra's recollections (`EWD1303`), three core motivations led to introducing **semaphores (`P` and `V`)**:
1. **Avoiding Busy Waiting (Economic Reason)**: In multiprogramming systems, busy-wait loops waste expensive CPU cycles on blocked programs. Semaphores explicitly inform the scheduler when a process cannot proceed so the CPU can be granted to another runnable process.
2. **Symmetry Between CPU and Peripherals (Producer–Consumer Synchronization)**: Dijkstra noticed a symmetry between a fast CPU waiting for a slow typewriter to empty an output buffer and the typewriter waiting for the CPU to fill it. Binary and counting semaphores cleanly synchronized producer/consumer queues (e.g., command queues and unacknowledged completions between CPU and I/O channels).
3. **Solving the Mutual Exclusion / Critical Section Problem**: To allow cyclic concurrent processes sharing common memory to enter critical sections safely (at most one at a time, without timing assumptions or race conditions), clean explicit synchronization primitives were required.

#### IN DETAIL
In `EWD1303`, Dijkstra recounted two foundational constraints behind inventing **Semaphores (`P`/`wait` and `V`/`signal`)**:
- **CPU–Peripheral Speed Asymmetry & Symmetric Producer–Consumer Synchronization**: Interposing a bounded memory buffer between a high-speed processor and a slow asynchronous electric typewriter revealed exact producer–consumer symmetry—neither agent should overwrite a full slot nor read an empty slot. Dijkstra initially created binary semaphores (`0`/`1`), which Carel Scholten generalized to non-negative counting integers tracking buffer occupancy cleanly.
- **Eliminating Busy-Waiting Power/Cycle Waste**: Hardware test-and-set loops burn full CPU bandwidth waiting on asynchronous events. Defining `P()` to transition a waiting thread into kernel `Blocked` state until `V()` awakens it reconciled critical-section mutual exclusion with efficient multiprogramming CPU sharing.

---

### 2. Implication When a Critical Section Solution Violates the "Progress" Requirement [Frequency: 1]

> **Question**: What is the implication if a solution to the critical section problem does not satisfy "Progress" condition?

#### pre-requisite
- **Three Requirements for Critical Section**: (1) **Mutual Exclusion**, (2) **Progress** (if no process is in the critical section and some want to enter, only processes not in their remainder section participate in deciding who enters next, and this selection cannot be postponed indefinitely), and (3) **Bounded Waiting**.

#### Answer (Exam Writing Format)
If a critical-section solution violates **Progress**, the following severe issues occur:
1. **Unnecessary Blocking / Deadlock**: A process wishing to enter an **empty** critical section can get stuck waiting indefinitely—even when no other process is inside the critical section (e.g., strict alternation `while (turn != i);` blocks Process 0 forever if Process 1 stays idle in its remainder section!).
2. **Stalled System Throughput & Livelock**: Processes outside the critical section (doing unrelated remainder-section work or terminated) can block eligible processes from entering, causing system freeze/deadlock despite available resources.

#### IN DETAIL
Among the three canonical requirements for a valid Critical Section solution (**Mutual Exclusion**, **Progress**, and **Bounded Waiting**), violating **Progress** cripples liveness even when safety holds:
- **Strict Alternation Failure Mode**: Consider Peterson/Dekker-style naive alternation (`while (turn != i);`), which guarantees mutual exclusion ($\le 1$ process inside CS). If `turn == 0` while $P_0$ remains indefinitely outside its CS (e.g., idle in its remainder section or terminated), $P_1$ is permanently prevented from entering an *unoccupied* critical section—a direct **livelock / progress failure** driven by coupling entry rights to a non-contending process.

---

### 3. Busy Waiting (Spinlocks): Issue & System Implications [Frequency: 1]

> **Question**: Explain the issue of "Busy Waiting". What are the implications?

#### pre-requisite
- **Spinlock / Polling Synchronization**: Continuously executing a tight loop (`while (lock == 1);`) waiting for a lock or condition variable to change without yielding the CPU.

#### Answer (Exam Writing Format)
- **What Busy Waiting Is**: Busy waiting occurs when a process/thread repeatedly checks a condition or lock in a tight CPU loop instead of blocking (`sleep()`) and releasing the processor until the awaited event occurs.
- **Implications**:
  1. **Wasted CPU Cycles & Lower Throughput**: The spinning thread burns 100% of its CPU quantum doing unproductive checking rather than letting useful ready processes execute.
  2. **Uniprocessor Deadlock / Priority Inversion Hazard**: On a single-core CPU, if a high-priority thread busy-waits for a lock held by a low-priority thread, the low-priority thread may never get scheduled to release the lock—hanging the system!
  3. **Power Waste & Bus Contention**: Increases energy/heat consumption and generates heavy cache-coherence bus traffic on multicore systems.

#### IN DETAIL
**Busy waiting (spinlocking)** continuously consumes CPU instruction cycles polling a shared flag (`while (TestAndSet(&lock));`) rather than releasing the processor via the kernel scheduler:
- **Uniprocessor Hazard**: On a single CPU core, if a low-priority thread holds `lock` and gets preempted by a higher-priority spinner, the spinner hogs 100% of remaining CPU quanta polling `lock`, preventing the lock holder from ever resuming to unlock—causing **priority inversion deadlock**.
- **Multicore Trade-off**: Spinlocks remain beneficial only on **SMP/multicore systems** when the lock holder is concurrently executing on a separate core and critical-section hold duration $T_{\text{hold}}$ is strictly shorter than two context switches ($2 \cdot T_{\text{ctx}}$); beyond that threshold, cache-line bouncing over the memory bus plus wasted core cycles far exceed blocking costs.

---

### 4. Solaris Synchronization: Adaptive Mutexes vs. Semaphores / RW Locks [Frequency: 1]

> **Question**: For synchronization, Solaris provides the system calls related to (i) adaptive mutex and (ii) semaphore. Explain the difference.

#### pre-requisite
- **Reference Slides**: `procSync_OS-2025.ppt` ("Solaris 2 Synchronization").

#### Answer (Exam Writing Format)
| Feature | Solaris Adaptive Mutex | Solaris Semaphore |
| :--- | :--- | :--- |
| **Use Case (Segment Length)** | Protects access to **short critical data segments** (only a few hundred instructions). | Used for **longer critical code segments**, resource counting, and general thread signaling/serialization. |
| **Waiting Behavior** | **Adaptive (Hybrid)**: On a multiprocessor, starts as a **spinlock** if the lock-holding thread is currently *running* on another CPU; if the holder is *not running* (or on a uniprocessor), it immediately **blocks/sleeps** rather than spinning. | **Always Blocking**: If unavailable, the calling thread immediately transitions to a **blocked/sleep queue** and yields the CPU (no adaptive spinning). |
| **Ownership** | Owned by the single locking thread (binary mutual exclusion + priority inheritance via turnstiles). | Non-owned counting primitive (can be signaled by any thread). |

#### IN DETAIL
Solaris **adaptive mutexes** eliminate the manual programmer dilemma between spinning and blocking by inspecting lock-holder runtime state dynamically at acquisition time:
- **Dynamic Spin-vs.-Block Heuristic**: When acquiring a contended mutex guarding a short kernel critical section, Solaris queries whether the thread currently owning the mutex is **actively running on another physical CPU core**. If yes, ownership is likely to end within nanoseconds, so the caller spins briefly rather than paying a multi-microsecond sleep/wakeup context switch. If the owner is **currently sleeping / descheduled** (or on a uniprocessor), spinning is futile, so the adaptive mutex immediately suspends the caller.
- **Contrast with Semaphores**: Kernel semaphores typically guard long-lived resource waits (such as disk DMA or lengthy I/O transactions), where spinning would waste milliseconds of core compute; hence semaphores unconditionally enqueue waiters to sleep.

---

### 5. Solaris Turnstiles: Memory Efficiency & Priority Inversion Resolution [Frequency: 1]

> **Question**: For synchronization, Solaris provides the concept of turnstiles. What is the purpose of turnstiles?

#### pre-requisite
- **Reference Slides**: `procSync_OS-2025.ppt` ("Solaris 2 Synchronization").

#### Answer (Exam Writing Format)
A **turnstile** in Solaris is a queue data structure containing threads blocked waiting to acquire either an **adaptive mutex** or a **reader-writer lock**. Its two primary purposes are:
1. **Preventing Priority Inversion via Priority Inheritance**: Turnstiles are organized using the **priority inheritance protocol**—when a high-priority thread blocks on a lock held by a lower-priority thread, the lock holder temporarily inherits the waiter's high priority until it releases the lock.
2. **Saving Kernel Memory**: Instead of embedding a heavy wait-queue inside *every* synchronization object (most locks are uncontended), Solaris attaches one turnstile **per thread** and assigns a thread's turnstile to a lock dynamically *only* when contention occurs.

#### IN DETAIL
**Solaris Turnstiles** solve both kernel metadata bloat and lock-ordering priority inversion for adaptive mutexes and reader–writer locks:
- **Memory Amortization via Thread-Owned Queues**: Large kernels manage hundreds of thousands of fine-grained locks simultaneously, yet at any instant almost all locks are uncontended. Embedding a full doubly linked wait-queue head inside every lock descriptor wastes megabytes of pinned kernel memory. Since one blocked thread can wait on **at most one lock at a time**, each thread statically carries a compact `turnstile` structure; only the *first* blocked thread waiting on a contended lock attaches its own turnstile via a global lock hash table, while subsequent blocked waiters join that attached queue.
- **Priority Inheritance Support**: Each turnstile organizes its blocked threads by priority, letting the kernel immediately trace ownership chains and boost a low-priority lock holder's scheduling priority when a higher-priority waiter enters the turnstile.

---

### 6. T/F: "No Difference Between 'wait' in Semaphore and Conditional 'wait' in Monitor" [Frequency: 1]

> **Question**: *"There is no difference between 'wait' in semaphore and 'wait (conditional wait)' in monitor."*

#### pre-requisite
- **Semaphore vs. Monitor**: A semaphore holds an integer counter (`sem_wait` only blocks if counter $\le 0$), whereas a monitor condition variable is stateless and tightly coupled with the monitor's mutual-exclusion lock.

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: There are fundamental semantic and operational differences between `wait()` in a semaphore and conditional `wait()` in a monitor.
- **Justification**: Semaphore `wait()` decrements an internal integer counter and only blocks if the counter becomes negative (or zero), whereas monitor `wait()` on a stateless condition variable **always** blocks the calling thread unconditionally. Furthermore, monitor `wait()` **atomically releases** the monitor's mutual-exclusion lock upon sleeping (and re-acquires it on wakeup), whereas semaphore `wait()` does not manage any external lock. Finally, semaphore `signal()` increments the counter for future calls, whereas monitor `signal()` is lost if no thread is currently waiting.

#### IN DETAIL
Semaphores and Monitor Condition Variables share identically named `wait()` / `signal()` primitives whose internal semantics are fundamentally inverted:
- **Semaphore (`wait(S)` / `signal(S)`) Statefulness**: Holds an internal integer counter. `wait()` only blocks if `S <= 0`; `signal()` increments `S` persistently, memorizing past signals even when zero threads are currently suspended. However, calling `wait(S)` while retaining another mutex never drops that mutex—causing deadlock.
- **Monitor Condition Variable (`c.wait()` / `c.signal()`) Atomicity**: Carries zero persistent counter. Calling `c.wait()` inside a monitor **unconditionally suspends** the thread while **atomically releasing the monitor's mutual-exclusion lock** so peer threads can enter to establish the awaited condition; `c.signal()` awakens one waiting thread if present, or becomes a permanent **no-op** if the condition queue is empty.

---

### 7. Why Monitors Need Condition Variables (c.wait / c.signal) & Differences vs. Semaphores [Frequency: 2]

> **Question**: Given that the monitor construct protects critical section by allowing one process at a time, why you require `c.wait` and `c.signal` operations in the Monitor. How they are different from `wait` and `signal` operations of semaphore?

#### pre-requisite
- **Monitor Construct**: Ensures automatic mutual exclusion—only **one** thread/process may be active inside the monitor at a time.

#### Answer (Exam Writing Format)
1. **Why `c.wait` and `c.signal` Are Required Inside Monitors**:
   - A monitor guarantees **mutual exclusion** (at most one active process inside), **but** it does not solve **conditional synchronization**—when a process inside the monitor finds a required precondition false (e.g., buffer empty for a consumer, buffer full for a producer).
   - If the process stays waiting inside the monitor without releasing the monitor lock, no other process can enter to satisfy the condition (**deadlock**). `c.wait()` **atomically releases the monitor lock and suspends the thread**, while `c.signal()` wakes a waiting thread once the condition becomes true.
2. **Differences from Semaphore `wait()` and `signal()`**:
   - **Lock Integration**: Monitor `c.wait()` **atomically releases** the monitor's mutual-exclusion lock and sleeps (re-acquiring it on wakeup); semaphore `wait()` has no awareness of any external lock.
   - **Statefulness vs. Statelessness**: A semaphore stores an integer count—if `S > 0`, `wait()` decrements `S` and does not block. A monitor condition variable has **no counter**—`c.wait()` **always blocks unconditionally**.
   - **Signal Memory**: Semaphore `signal()` increments the counter and is remembered for future `wait()` calls; monitor `c.signal()` has **no effect (is lost)** if no thread is currently waiting on `c`.

#### IN DETAIL
A **Monitor** uses two distinct synchronization constructs—an **implicit mutual-exclusion entry lock** plus explicit **condition variables (`c.wait`/`c.signal`)**—to separate mutual safety from conditional coordination:
- **Deadlock Without Condition Variables**: Only one process may execute inside monitor procedures concurrently. If a Consumer acquires the monitor lock and sees `buffer_count == 0`, simply spinning or calling semaphore-style sleep *while holding the monitor lock* prevents any Producer from ever entering `insert()` to increment `buffer_count`.
- **Atomic Release–Block–Reacquire Contract**: Condition `not_empty.wait()` moves the Consumer onto `not_empty`'s wait queue and atomically unlocks the monitor boundary. When a Producer deposits an item and invokes `not_empty.signal()`, one blocked Consumer moves toward monitor re-entry, reacquiring mutual exclusion cleanly before resuming.

---

### 8. T/F: "With Monitors, It Is Possible to Reduce Busy Waiting Over Semaphores" [Frequency: 3]

> **Question**: *"With monitors, it is possible to reduce busy waiting over semaphores."*

#### pre-requisite
- **Blocking Semaphores vs. Monitors**: Kernel-implemented OS semaphores maintain sleep queues (just like monitor condition variables), whereas naive spinlock semaphores or user-level polling loops busy-wait.

#### Answer (Exam Writing Format)
- **FALSE** *(per the 2022 official solution key comparing standard kernel blocking semaphores against monitors; note that from a software-engineering perspective against spin-polling loops, it is sometimes marked TRUE as in 2025 AOS Endsem Q1.2)*.
- **Corrected Statement**: Both OS-level blocking semaphores and monitors use sleep/wait queues to avoid busy waiting, so monitors primarily simplify writing error-free critical sections rather than inherently reducing busy waiting over blocking semaphores.
- **Justification**: A standard blocking semaphore places waiting processes onto an OS sleep queue (`wait()`/`signal()`), consuming no CPU cycles while blocked—the same underlying sleep/wakeup mechanism used by monitor condition variables. Monitors add language-level mutual exclusion and safer structure, not a different hardware sleep mechanism.

#### IN DETAIL
Evaluating whether monitors reduce **busy waiting** relative to semaphores requires distinguishing **kernel primitive semantics** from **application usage errors**:
- **OS Kernel Level (`FALSE` per OSN 2022 key)**: Both kernel blocking semaphores (`P`/`V`) and monitors suspend blocked threads inside OS kernel wait queues (`Blocked` state) without spinning on CPU flags—neither relies on busy waiting inside the primitive itself.
- **Software Engineering Level (`TRUE` per AOS 2025 key)**: Because semaphores decouple locking and condition state manually, programmers frequently wrap multi-variable invariants in error-prone user-space spin loops or polling retries (`unlock; sleep; lock; recheck`), whereas monitors natively package atomic condition queues (`c.wait` / `c.signal`), eliminating ad-hoc user polling entirely.

---

### 9. Single-Lane Highway / Vermont Bridge Synchronization Using Semaphores [Frequency: 3]

> **Question**: An old bridge on a busy highway is too narrow to permit two-way traffic, so one-way traffic is to be implemented on the bridge by alternatively permitting vehicles travelling in opposite direction to use the bridge. The following rules are formulated for use of bridge:
> (a) Any time the bridge is used by vehicle(s) travelling in one direction only.
> (b) If the vehicles are waiting to cross the bridge at both ends, only one vehicle from one end is allowed to cross the bridge before a vehicle from the other end starts crossing the bridge.
> (c) If no vehicles are waiting at one end, then any number of vehicles from the other end are permitted to cross the bridge.
> Develop a program (Pseudocode) to synchronize vehicles with semaphores.
> *(AOS 2025 Midsem Q7 frames this identically as North Tunbridge vs. South Tunbridge produce farmers using semaphores.)*

#### pre-requisite
- **Single-Lane Bridge Rules**:
  - **Safety / Mutual Exclusion**: Never have Northbound and Southbound vehicles on the bridge simultaneously. Note carefully rule (b) & (c): multiple vehicles in the *same* direction can cross if no opposite vehicle is waiting, **but** if vehicles are waiting at *both* ends, *only one vehicle from one end crosses before switching to a vehicle from the other end* (strict alternation under contention to prevent starvation!).
- **Semaphore Pattern (Passing the Baton)**: Using a `mutex` semaphore for shared counters plus directional waiting semaphores (`sem_north`, `sem_south`) initialized to `0`.

#### Answer (Exam Writing Format)
**Assumptions**:
1. Binary semaphores are strong (FIFO wakeup queue) and initialized as `mutex = 1`, `ok_north = 0`, `ok_south = 0`.
2. Variables:
   - `on_bridge`: count of vehicles currently on bridge (`0` = empty).
   - `dir`: current/last direction (`0` = none/south, `1` = north, `2` = south; let `last_dir` track who went most recently).
   - `wait_north`, `wait_south`: number of vehicles waiting at each end.
3. Because rule (b) requires strict 1-for-1 alternation when *both* sides have waiting cars, when `wait_south > 0` and a Northbound car enters/is on the bridge, no additional Northbound car enters until a Southbound car takes its turn (so `on_bridge` for alternating mode stays 1, whereas when `wait_south == 0`, consecutive Northbound cars may cross freely per rule (c)).

```c
// Shared State & Semaphores
semaphore mutex = 1;       // Protects shared variables
semaphore ok_north = 0;    // Northbound waiting queue
semaphore ok_south = 0;    // Southbound waiting queue

int on_bridge = 0;         // Vehicles currently on the bridge
int cur_dir = 0;           // 0 = empty, 1 = North, 2 = South
int last_dir = 2;          // Direction of last vehicle that entered (1=North, 2=South)
int wait_north = 0;        // Waiting Northbound vehicles
int wait_south = 0;        // Waiting Southbound vehicles

void Northbound_Vehicle() {
    // --- Entry Protocol ---
    wait(mutex);
    if ((cur_dir == 0 && (wait_south == 0 || last_dir == 2)) ||
        (cur_dir == 1 && wait_south == 0)) {
        // Safe to enter: bridge empty & our turn (or nobody waiting south),
        // OR bridge already Northbound & nobody waiting South (rule c)
        cur_dir = 1;
        last_dir = 1;
        on_bridge++;
        signal(ok_north);
    } else {
        wait_north++;
    }
    signal(mutex);
    wait(ok_north);

    Cross_Bridge("North");

    // --- Exit Protocol ---
    wait(mutex);
    on_bridge--;
    if (on_bridge == 0) {
        if (wait_south > 0) {
            // Opposite side is waiting: alternate to exactly 1 Southbound vehicle (rule b)
            wait_south--;
            cur_dir = 2;
            last_dir = 2;
            on_bridge = 1;
            signal(ok_south);
        } else if (wait_north > 0) {
            // No Southbound waiting: let waiting Northbound cross (rule c)
            // Wake 1 or all waiting Northbound since wait_south == 0
            while (wait_north > 0) {
                wait_north--;
                cur_dir = 1;
                last_dir = 1;
                on_bridge++;
                signal(ok_north);
            }
        } else {
            cur_dir = 0; // Bridge is completely free
        }
    }
    signal(mutex);
}

void Southbound_Vehicle() {
    // --- Entry Protocol (symmetric) ---
    wait(mutex);
    if ((cur_dir == 0 && (wait_north == 0 || last_dir == 1)) ||
        (cur_dir == 2 && wait_north == 0)) {
        cur_dir = 2;
        last_dir = 2;
        on_bridge++;
        signal(ok_south);
    } else {
        wait_south++;
    }
    signal(mutex);
    wait(ok_south);

    Cross_Bridge("South");

    // --- Exit Protocol (symmetric) ---
    wait(mutex);
    on_bridge--;
    if (on_bridge == 0) {
        if (wait_north > 0) {
            wait_north--;
            cur_dir = 1;
            last_dir = 1;
            on_bridge = 1;
            signal(ok_north);
        } else if (wait_south > 0) {
            while (wait_south > 0) {
                wait_south--;
                cur_dir = 2;
                last_dir = 2;
                on_bridge++;
                signal(ok_south);
            }
        } else {
            cur_dir = 0;
        }
    }
    signal(mutex);
}
```

#### IN DETAIL
Solving bidirectional **Single-Lane Bridge synchronization via binary semaphores** enforces three concurrent invariants cleanly without busy waiting or spin-retries:
- **Single-Direction Mutual Exclusion (`cur_dir` + `on_bridge`)**: Simultaneous crossings occur only when `cur_dir` matches arriving traffic and zero opposite-direction vehicles are waiting (`wait_south == 0` / `wait_north == 0`). State transitions happen strictly inside short `mutex`-protected sections, never holding `mutex` across `Cross_Bridge()` or blocking calls.
- **Turn-Yielding Batched Alternation**: When opposite traffic arrives mid-stream (`wait_south > 0`), newly arriving same-direction vehicles block instead of joining the bridge, preventing infinite starvation of the opposite queue. When the final crossing vehicle exits (`on_bridge == 0`), prioritizing the opposite wait queue (`if wait_south > 0`) transfers control deterministically between queues (`handoff` wakeup).

---

### 10. Single-Lane Bridge Synchronization Using a Monitor (Mesa Semantics) [Frequency: 1]

> **Question**: *(Source scan page 3 shows Monitor Code Implementation for the Single-Lane Vermont Bridge problem—same problem statement as 2025 AOS Midsem Q7 / 2020 Quiz 5 Q1: prevent deadlock and starvation for Northbound and Southbound vehicles/farmers crossing a single-lane bridge using a **Monitor**).*

#### pre-requisite
- **Monitor Semantics**: Only one thread can be active inside the monitor at a time. Condition variable `free.wait()` atomically releases the monitor lock and sleeps until `free.broadcast()` / `free.signal()` wakes waiting threads (always re-checking the condition in a `while` loop!).
- **Key Fix over Scanned Key Bug**: Note that in the scanned key snippet (`OSN_mid_2022_M_key.pdf` p.3), the waiting counters were mistakenly declared as `semaphore` instead of `int`, `on_bridge` wasn't set to `1`, and `prev` checked the same value! Below is the **clean, bug-free** Monitor implementation.

#### Answer (Exam Writing Format)
**Assumptions**:
- Mesa-style Monitor (mutual exclusion across procedures; condition `free.wait()` releases lock and re-acquires upon wakeup, requiring a `while` loop).
- `on_bridge` tracks number of cars currently crossing in one direction (or boolean `0` empty / `1` occupied; setting `dir` + `on_bridge` allows multiple same-direction cars when opposite queue is empty, or 1-at-a-time strict alternation when both wait). Here is the clean starvation-free version allowing same-direction concurrency when opposite is empty and strict alternation when both wait:

```c
monitor Bridge {
    int on_bridge = 0;         // Number of vehicles currently on bridge (0 = empty)
    int cur_dir = 0;           // 0 = none, 1 = North, 2 = South
    int last_dir = 0;          // Last direction that entered (for alternation fairness)
    int num_waiting_north = 0; // Waiting Northbound vehicles
    int num_waiting_south = 0; // Waiting Southbound vehicles
    condition free;

    void enter_bridge_north() {
        num_waiting_north++;
        while ((on_bridge > 0 && cur_dir == 2) ||
               (on_bridge > 0 && cur_dir == 1 && num_waiting_south > 0) ||
               (on_bridge == 0 && last_dir == 1 && num_waiting_south > 0)) {
            free.wait();
        }
        num_waiting_north--;
        on_bridge++;
        cur_dir = 1;
        last_dir = 1;
    }

    void exit_bridge_north() {
        on_bridge--;
        if (on_bridge == 0) {
            cur_dir = 0;
        }
        free.broadcast();
    }

    void enter_bridge_south() {
        num_waiting_south++;
        while ((on_bridge > 0 && cur_dir == 1) ||
               (on_bridge > 0 && cur_dir == 2 && num_waiting_north > 0) ||
               (on_bridge == 0 && last_dir == 2 && num_waiting_north > 0)) {
            free.wait();
        }
        num_waiting_south--;
        on_bridge++;
        cur_dir = 2;
        last_dir = 2;
    }

    void exit_bridge_south() {
        on_bridge--;
        if (on_bridge == 0) {
            cur_dir = 0;
        }
        free.broadcast();
    }
}
```

#### IN DETAIL
Implementing the **Single-Lane Bridge under Mesa-style Monitor Semantics** relies heavily on wrapping `free.wait()` inside a **`while` predicate loop** rather than an `if`:
- **Why Mesa Semantics Mandate `while` Loops**: Unlike Hoare semantics (which instantaneously transfers the monitor lock directly from signaler to awakened thread), **Mesa semantics** place the awakened thread onto the ready/entry queue while the signaler keeps executing until monitor exit. Another thread can slip into the monitor first and invalidate the shared predicate (`on_bridge`, `cur_dir`, `last_dir`) before the woken thread runs.
- **Three Safety & Starvation Guards**: Re-evaluating `(on_bridge > 0 && cur_dir == OPPOSITE) || (wait_OPPOSITE > 0 && (on_bridge > 0 || last_dir == SAME))` on every wakeup safely enforces mutual collision exclusion, lets consecutive same-side vehicles pipeline when the opposite side is empty, and yields turn alternation when both sides wait.

---

### 11. IndixOS Starvation-Free Readers–Writers Problem Using POSIX Semaphores [Frequency: 1]

> **Question**: The *sem team* responsible for managing concurrency is testing out applications to test the concurrency aspects. They have implemented the notion of semaphore that is made available through the POSIX library. The team plans to test this by creating a simple application similar to Google Docs. However, they want you to test their semaphore implementation. The constraint they have for the application is that multiple threads should be able to read a document at the same time, but only one writer should be able to write at any given instant of time. Moreover, they also want to ensure that there is no starvation of thread that is waiting to write to the document. Can you devise a solution to address these constraints (Pseudocode)? (Clearly state the assumptions made and reason on the solution)

#### pre-requisite
- **First vs. Second/Third Readers-Writers Problem**: A naive reader-priority solution (`if read_count == 1 wait(rw_mutex)`) starves waiting writers whenever a continuous stream of readers overlaps! Adding a **`service_queue` / `order_mutex`** (or writer-preference counter) blocks new readers behind a waiting writer so writers never starve.

#### Answer (Exam Writing Format)
**Assumptions**:
1. POSIX semaphores (`sem_t`) have FIFO wake-up fairness (strong semaphores) and are initialized to `1`.
2. Shared variables: `read_count = 0`; semaphores `order_mutex = 1` (FIFO turnstile), `rw_mutex = 1` (document write lock), `r_mutex = 1` (protects `read_count`).

```c
sem_t order_mutex;   // Initialized to 1: enforces arrival order (prevents writer starvation)
sem_t rw_mutex;      // Initialized to 1: exclusive access for writer or active reader group
sem_t r_mutex;       // Initialized to 1: protects read_count
int read_count = 0;

void reader() {
    sem_wait(&order_mutex);    // 1. Wait in FIFO turnstile behind any waiting writer
    sem_wait(&r_mutex);
    read_count++;
    if (read_count == 1) {
        sem_wait(&rw_mutex);   // 2. First reader locks out writers
    }
    sem_post(&r_mutex);
    sem_post(&order_mutex);    // 3. Allow next reader/writer to enter turnstile

    /* --- CRITICAL SECTION: Concurrent Reading of Google Doc --- */
    read_document();

    sem_wait(&r_mutex);
    read_count--;
    if (read_count == 0) {
        sem_post(&rw_mutex);   // 4. Last reader releases writer lock
    }
    sem_post(&r_mutex);
}

void writer() {
    sem_wait(&order_mutex);    // 1. Wait in turnstile; blocks subsequent arriving readers!
    sem_wait(&rw_mutex);       // 2. Wait for any currently active readers/writer to finish
    sem_post(&order_mutex);    // 3. Release turnstile once write access is secured

    /* --- CRITICAL SECTION: Exclusive Writing to Google Doc --- */
    edit_document();

    sem_post(&rw_mutex);       // 4. Release document lock
}
```

**Reasoning on Constraints**:
1. **Concurrent Readers**: Once the first reader acquires `rw_mutex` and releases `order_mutex`, subsequent readers pass `order_mutex` and `r_mutex` immediately without blocking on `rw_mutex`, reading simultaneously.
2. **Exclusive Writer**: A writer must hold `rw_mutex`; no reader (`read_count == 1` waits on `rw_mutex`) or other writer can enter while a writer holds `rw_mutex`.
3. **No Writer Starvation**: When a writer arrives and calls `sem_wait(&order_mutex)`, it queues in arrival order. Any *new* readers arriving afterward block behind that writer on `order_mutex` rather than bypassing it. As soon as currently active readers drain (`read_count == 0`), the writer acquires `rw_mutex` immediately—preventing starvation for both writers and readers!
4. **Deadlock Free**: Lock acquisition order is strictly hierarchical (`order_mutex` $\to$ `r_mutex` $\to$ `rw_mutex` for readers; `order_mutex` $\to$ `rw_mutex` for writers), eliminating circular wait.

#### IN DETAIL
Standard First-Readers-Preference solutions suffer **Writer Starvation** because readers bypass `rw_mutex` whenever `read_count >= 1`, allowing continuous overlapping readers to keep `read_count > 0` forever:
- **FIFO Turnstile Barrier (`order_mutex`)**: Every arriving Reader and Writer acquires `order_mutex` at entry and holds it purely until admitted past the admission gate (Readers release it immediately upon updating `read_count`; Writers release it once they acquire `rw_mutex` or right at entry before waiting on `rw_mutex` under symmetric queueing).
- **Fairness Guarantee**: Once a Writer enters the queue on `order_mutex`, all subsequently arriving Readers block behind that Writer at `order_mutex`. Once active inside readers drain (`read_count` drops to `0`), `rw_mutex` transfers immediately to the queued Writer—bounding wait time for both classes symmetrically (assuming FIFO-strong POSIX semaphores).

---

### 12. Embarrassingly Parallel Disjoint File Record Processing: Do We Need Locks or Condition Variables? [Frequency: 1]

> **Question**: The *sem team* wants to test their implementation with an I/O intensive operation where they want to create a program that can process a file consisting of 100000 records. In order to make it faster and to test the efficiency of their implementation, the thread team are planning to write a program which will use 10 threads where each thread reads 10000 records and processes them independently, so in total, all the 100000 records will be processed. However, the team is not sure which of these is required to complete the implementation. Which of these do you recommend and why? (3 points)
> - a. Make use of locks. No need for condition variables
> - b. Only condition variables are needed. No need for locks
> - c. Both locks and condition variables are needed.
> - d. No need for either locks or condition variables.

#### Answer (Exam Writing Format)
- **Correct Option**: **d. No need for either locks or condition variables.** *(Assuming each thread opens its own independent file descriptor / uses positional `pread()` on its disjoint 10,000-record byte range and writes to disjoint outputs).*
- **Reasoning**:
  - Locks and condition variables are only required when threads share **mutable state** or must coordinate **ordering/waiting** on shared conditions.
  - Here, the 100,000 records are cleanly partitioned into **10 strictly disjoint slices** of 10,000 records each, processed completely independently (**embarrassingly parallel**). Since no thread writes to data accessed by another thread (and assuming separate file offsets/descriptors), there are **no race conditions**, so synchronization overhead is unnecessary.

#### IN DETAIL
Race conditions arise *only* when three structural elements coincide: **shared mutable memory**, **unsynchronized concurrent access**, and **at least one writer**:
- **Disjoint Partition Invariant**: Dividing an input array or file cleanly into non-overlapping record indices $[s_i, e_i)$ guarantees zero write–write or read–write conflicts in shared memory across worker threads—making mutual-exclusion locks unnecessary (`embarrassingly parallel`).
- **System-Call File Offset Caveat**: In POSIX I/O, multiple threads sharing one process-wide `int fd` implicitly share a **single kernel open-file offset pointer** modified atomically or raced by sequential `read()`/`lseek()` calls. Safe lock-free concurrent I/O therefore requires either **independent `open()` file descriptors per thread** or stateless positional **`pread(fd, buf, count, offset)`** syscalls.

---

### 13. Jurassic Park Safari Rides & Passengers Synchronization Using Semaphores [Frequency: 1]

> **Question**: Jurassic Park consists of a dinosaur museum and a park for safari riding. There are "$m$" passengers and "$n$" single-passenger cars. Passengers wander around the museum for a while, then line up to take ride in a safari car. When a car is available, it loads the one passenger it can hold and rides around the park for a random amount of time. If the "$n$" cars are all out riding passengers around, then the passenger who wants to ride waits; if a car is ready to load but there are no waiting passengers, then the car waits. Use semaphores to synchronize the "$m$" passenger processes and the "$n$" car processes. Is your solution deadlock and/or starvation free? Discuss.

#### pre-requisite
- **Two-Way Rendezvous with Semaphores**: Cars and passengers must pair up cleanly (`1 car ↔ 1 passenger`), ensuring a car doesn't drive off before its passenger boards AND a passenger doesn't exit/re-line-up before the ride ends!

#### Answer (Exam Writing Format)
**Semaphores**:
- `car_avail = 0` (signaled by an empty car ready to load)
- `pass_waiting = 0` (signaled by a passenger ready to board)
- `boarded = 0` (handshakes boarding completion before car departs)
- `ride_over = 0` (signals passenger that the safari tour finished)

```c
// Initialization
semaphore car_avail = 0;
semaphore pass_waiting = 0;
semaphore boarded = 0;
semaphore ride_over = 0;

void Passenger(int id) {
    while (true) {
        wander_museum();

        signal(pass_waiting);   // Announce arrival at ride line
        wait(car_avail);        // Wait for an empty safari car

        board_car();
        signal(boarded);        // Confirm seated in car

        wait(ride_over);        // Wait until safari tour finishes
        unboard_car();
    }
}

void SafariCar(int id) {
    while (true) {
        signal(car_avail);      // Announce empty car is ready
        wait(pass_waiting);     // Wait for a waiting passenger

        wait(boarded);          // Wait until passenger is seated
        take_safari_ride();     // Ride around park for random time
        signal(ride_over);      // End tour so passenger exits
    }
}
```
- **Deadlock Analysis**: **Deadlock-Free.** Notice `Passenger` and `SafariCar` both signal availability (`pass_waiting` / `car_avail`) *before* waiting on their complementary semaphore (`car_avail` / `pass_waiting`), eliminating any circular wait! Similarly, `boarded` and `ride_over` follow strict unidirectional cause-and-effect ordering (`boarded` $\to$ `ride_over`).
- **Starvation Analysis**: **Starvation-Free** provided the underlying OS semaphores use **FIFO (strong) wakeup queues**—both waiting passengers and waiting cars are matched in strict arrival order. *(With weak/unfair semaphores, a waiting passenger/car could theoretically be bypassed repeatedly.)*

#### IN DETAIL
Synchronizing two independent producer/consumer agent types (`m` Passengers, `n` single-seat Safari Cars) demonstrates the **asymmetric semaphore rendezvous handshake**:
- **Preventing Handshake Deadlock**: If both Passenger and Car waited first before signaling (`wait(car_avail)` before `signal(pass_waiting)` and `wait(pass_waiting)` before `signal(car_avail)`), both parties block forever in a symmetric hold-and-wait deadlock. Ordering one side to advertise availability (`signal(pass_waiting)` -> `wait(car_avail)`) breaks circular dependency cleanly.
- **Ensuring Synchronous Board & Unload Boundaries**: Introducing explicit barrier semaphores (`boarded`, `ride_done`, `unloaded`) enforces exact state ordering so cars never depart mid-boarding, passengers never exit mid-ride, and each car resets cleanly before accepting its next passenger.

---

