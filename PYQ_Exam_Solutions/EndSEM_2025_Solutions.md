# End-Semester 2025 Examination — Complete Solutions

> **Includes**: **Part 1 — AOS Monsoon 2025 EndSem** (`CS3.304 Advanced Operating Systems`, 25 Nov 2025, 80 Marks — all Q1.1–Q13) and **Part 2 — OSN Monsoon 2025 EndSem** (`CS3.301`, AgentOS — all Q1–Q10).  
> **Back to Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Quick Question Overview

#### Part 1: AOS EndSem 2025 (`CS3.304`)
- [Question 1.1 [1 Mark]](#question-11-1-mark-frequency-1) — Frequency: 1
- [Question 1.2 & 1.4 [1 + 1 = 2 Marks]](#question-12-14-1-1-2-marks-frequency-3) — Frequency: 3
- [Question 1.3 [1 Mark]](#question-13-1-mark-frequency-1) — Frequency: 1
- [Question 1.5 [1 Mark]](#question-15-1-mark-frequency-1) — Frequency: 1
- [Question 1.6 [1 Mark]](#question-16-1-mark-frequency-2) — Frequency: 2
- [Question 1.7 [1 Mark]](#question-17-1-mark-frequency-3) — Frequency: 3
- [Question 1.8 [1 Mark]](#question-18-1-mark-frequency-1) — Frequency: 1
- [Question 1.9 [1 Mark]](#question-19-1-mark-frequency-2) — Frequency: 2
- [Question 1.10 [1 Mark]](#question-110-1-mark-frequency-3) — Frequency: 3
- [Question 2.1 [3 Marks]](#question-21-3-marks-frequency-1) — Frequency: 1
- [Question 2.2 [3 Marks]](#question-22-3-marks-frequency-1) — Frequency: 1
- [Question 2.3 [3 Marks]](#question-23-3-marks-frequency-1) — Frequency: 1
- [Question 2.4 [3 Marks]](#question-24-3-marks-frequency-1) — Frequency: 1
- [Question 2.5 [3 Marks]](#question-25-3-marks-frequency-1) — Frequency: 1
- [Question 3 [5 Marks]](#question-3-5-marks-frequency-3) — Frequency: 3
- [Question 4 [5 Marks]](#question-4-5-marks-frequency-1) — Frequency: 1
- [Question 5 [5 Marks]](#question-5-5-marks-frequency-3) — Frequency: 3
- [Question 6 [5 Marks]](#question-6-5-marks-frequency-1) — Frequency: 1
- [Question 7 [5 Marks]](#question-7-5-marks-frequency-1) — Frequency: 1
- [Question 8 [5 Marks]](#question-8-5-marks-frequency-2) — Frequency: 2
- [Question 9 [5 Marks]](#question-9-5-marks-frequency-3) — Frequency: 3
- [Question 10 [5 Marks]](#question-10-5-marks-frequency-1) — Frequency: 1
- [Question 11 [5 Marks]](#question-11-5-marks-frequency-3) — Frequency: 3
- [Question 12 [5 Marks]](#question-12-5-marks-frequency-3) — Frequency: 3
- [Question 13 [5 Marks]](#question-13-5-marks-frequency-1) — Frequency: 1

#### Part 2: OSN EndSem 2025 (`AgentOS`)
- [Question 1 [3 Points] — Round Robin Scheduling (q = 1 ms)](#agentos-question-1-3-points-round-robin-scheduling-q-1-ms-frequency-1) — Frequency: 1
- [Question 2 [3 Points] — TCP Retransmission Timeout Calculation](#agentos-question-2-3-points-tcp-retransmission-timeout-calculation-frequency-1) — Frequency: 1
- [Question 3 [3 Points] — Segmentation Address Translation (0x5100)](#agentos-question-3-3-points-segmentation-address-translation-0x5100-frequency-1) — Frequency: 1
- [Question 4(a) [5 Points] — TLB Miss & Async Page Fault Handling](#agentos-question-4a-5-points-tlb-miss-async-page-fault-handling-frequency-1) — Frequency: 1
- [Question 4(b) [5 Points] — Clock / Second-Chance Page Replacement (evictPages)](#agentos-question-4b-5-points-clock-second-chance-page-replacement-evictpages-frequency-1) — Frequency: 1
- [Question 5 [3 Points] — TCP Three-Way Handshake Sequence & Ack Numbers](#agentos-question-5-3-points-tcp-three-way-handshake-sequence-ack-numbers-frequency-1) — Frequency: 1
- [Question 6 [10 Points] — Cold-Start End-to-End Network Stack (DHCP, DNS, TCP, ARP)](#agentos-question-6-10-points-cold-start-end-to-end-network-stack-dhcp-dns-tcp-arp-frequency-1) — Frequency: 1
- [Question 7 [3 Points] — Resource Lock Ordering to Prevent Deadlock](#agentos-question-7-3-points-resource-lock-ordering-to-prevent-deadlock-frequency-1) — Frequency: 1
- [Question 8 [10 Points] — SL-FS Inode/Block Capacity, Max File Size & NAS RAID](#agentos-question-8-10-points-sl-fs-inodeblock-capacity-max-file-size-nas-raid-frequency-1) — Frequency: 1
- [Question 9 [15 Points] — Core Router Routing-Table RW Locks & Intra/Inter-AS Routing](#agentos-question-9-15-points-core-router-routing-table-rw-locks-intrainter-as-routing-frequency-1) — Frequency: 1
- [Question 10 (Bonus) [4 Points] — Parameterized RR Quantum & Asymmetric MLFQ Design](#agentos-question-10-bonus-4-points-parameterized-rr-quantum-asymmetric-mlfq-design-frequency-1) — Frequency: 1

---

## Part 1: AOS Monsoon 2025 End-Semester Examination (`CS3.304` — 80 Marks)

### Question 1.1 [1 Mark] [Frequency: 1]

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

### Question 1.2 & 1.4 [1 + 1 = 2 Marks] [Frequency: 3]

> **Question**: *"With monitors, it is possible to reduce busy waiting over semaphores."* *(Also asked identically as Q1.4: "It is possible to reduce 'busy waiting' with monitors over semaphores.")*

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

### Question 1.3 [1 Mark] [Frequency: 1]

> **Question**: *"Deadlock avoidance approach improves resource utilization over deadlock prevention approach based on the violation of no pre-emption condition."*

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: Deadlock prevention by violating "No Preemption" forces the OS to forcibly take away held resources whenever a process's additional request cannot be met, causing aborted work, costly rollbacks, and wasted restarts. In contrast, deadlock avoidance (e.g., Banker's Algorithm) dynamically verifies safe states and lets processes retain their allocated resources and complete uninterruptedly whenever safe, sustaining significantly higher hardware and CPU utilization.

#### IN DETAIL
Comparing **Deadlock Avoidance (Banker's Algorithm)** with **Preemption-Based Prevention** evaluates conservative admission control against destructive rollbacks:
- **Preservation of Completed Work**: Violating No-Preemption discards all CPU time, I/O operations, and intermediate state accumulated prior to preemption whenever a resource request fails—wasting throughput and requiring checkpoint/undo infrastructure (only trivial for stateless/idempotent resources like CPU registers or swapable memory pages).
- **Non-Destructive Scheduling in Avoidance**: Banker's Algorithm never confiscates resources from an active process; once a request passes the safety check, the process proceeds deterministically to completion, preserving 100% of execution progress while keeping utilization high (rejecting only requests leading into unsafe states).

---

### Question 1.5 [1 Mark] [Frequency: 1]

> **Question**: *"The notion of 'Thread pools' is employed to control MPL."*

#### pre-requisite
- **MPL (Degree of Multiprogramming)**: Number of *processes* concurrently loaded in main memory (regulated by the **Long-Term** and **Medium-Term Schedulers**).
- **Thread Pool**: Bounded pool of pre-created worker *threads* inside a multithreaded process/server.

#### Answer (Exam Writing Format)
- **FALSE**. *(In strict OS terminology; note thread pools limit intra-process thread concurrency rather than OS-level process MPL.)*
- **Corrected Statement**: The notion of **thread pools** is employed to **limit the number of concurrent threads within a process** and eliminate thread-creation overhead, whereas **MPL (Degree of Multiprogramming)** is controlled by the **Long-Term and Medium-Term Schedulers**.
- **Justification**: MPL refers to the total number of processes residing in main memory, regulated by admitting or swapping processes to avoid thrashing. Thread pools operate inside a server/process to pre-create a bounded number of reusable worker threads, preventing unbounded thread creation from exhausting CPU and memory resources.

#### IN DETAIL
OS terminology distinguishes strictly between **process-level** multiprogramming and **thread-level** concurrency control:
- **Degree of Multiprogramming (MPL)**: Governed by the **Long-Term** and **Medium-Term Schedulers**, reflecting the number of distinct *processes* (separate address spaces) residing simultaneously in main memory.
- **Thread Pool Concurrency Cap**: Operates *within a single process's address space*, fixing only the maximum number of active worker *threads* inside that application so it avoids per-request creation overhead and intra-process thread thrashing without altering OS-wide process MPL.

---

### Question 1.6 [1 Mark] [Frequency: 2]

> **Question**: *"The notion of 'load balancing' counteracts with the notion of 'processor affinity'."*

#### pre-requisite
- **Reference Slides**: `cpuScheduling_CS.ppt` ("Multiple-Processor Scheduling").
- **Processor Affinity**: Keeping a process running on the *same* CPU core so its warm L1/L2 cache stays valid.
- **Load Balancing**: Migrating queued processes from busy cores to idle cores so work stays evenly distributed.

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: **Processor affinity** tries to keep a process executing on the exact same CPU core to preserve its warm L1/L2 cache state and avoid costly cache misses. Conversely, **load balancing** (via push or pull migration) actively moves processes away from overloaded cores onto idle cores, which breaks processor affinity and invalidates cache warmth—creating a direct performance trade-off.

#### IN DETAIL
**Processor Affinity** and **Load Balancing** pursue fundamentally competing hardware vs. queueing objectives in Symmetric Multiprocessing (SMP) kernels:
- **Microarchitectural Basis of Affinity**: Keeping thread $T$ pinned to Core $k$ preserves warm **L1/L2 private caches** and active **TLB translation entries** across context switches, avoiding massive compulsory cache-miss storms and memory-bus invalidation traffic.
- **Load-Balancing Trade-off**: If Core $k$'s run queue accumulates multiple runnable threads while Core $j$ sits idle, leaving $T$ on Core $k$ wastes compute capacity across Core $j$. Pulling or pushing $T$ via migration evens out core queue depths (**work conservation**) at the immediate cost of invalidating $T$'s warm private cache state on Core $k$ and forcing cold cache re-population on Core $j$.

---

### Question 1.7 [1 Mark] [Frequency: 3]

> **Question**: *"Global frame allocation algorithm gives better throughput over local frame allocation algorithm."*

#### pre-requisite
- **Local vs. Global Replacement**: In **local** allocation, a page-faulting process can only replace one of its *own* fixed frames; in **global** allocation, it can steal an unused/free frame from *any* process in the global pool.

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: Global frame allocation allows a process whose working set is actively growing to claim free or infrequently used frames from other processes rather than thrashing inside a rigid fixed partition. By dynamically adjusting each process's frame count to match its current locality needs, global allocation reduces total page faults, improves memory utilization, and yields higher overall system throughput than local allocation.

#### IN DETAIL
**Global vs. Local Page Replacement Allocation** trades strict inter-process **performance isolation** against dynamic **aggregate memory efficiency**:
- **Why Global Allocation Achieves Higher Throughput**: Application working sets expand and contract dynamically across execution phases (compile parsing vs. linking, heap bursts vs. idle waits). Global replacement pools free and least-recently-used frames across the entire system, dynamically shifting unused frames from quiescent processes to temporarily memory-hungry active processes—dramatically cutting total page-fault rates and raising aggregate CPU throughput.
- **Inherent Stability Trade-off**: Because one aggressively faulting workload can steal frames belonging to well-behaved neighbors, local or minimum-guaranteed working-set floors (like Linux cgroup `memory.min` / per-process WS caps) are paired in multi-tenant environments to bound noisy-neighbor thrashing.

---

### Question 1.8 [1 Mark] [Frequency: 1]

> **Question**: *"In the given network, it does not make any difference if we replace the 'bridge' with the 'switch'."*

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slide 60: "Repeater, Hub, Bridge and Switch").

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: Replacing a **bridge** with a **switch** improves network performance, port capacity, and collision isolation (though both perform Layer-2 MAC forwarding).
- **Justification**: A traditional **bridge** connects a few multi-host LAN segments using software-based frame inspection, often leaving multiple hosts sharing a collision domain per port. A **switch** connects many individual hosts directly on dedicated **full-duplex hardware (ASIC) ports**, providing wire-speed forwarding, lower latency, higher aggregate bandwidth, and dedicated per-host collision-free links.

#### IN DETAIL
Although both **Layer-2 Bridges** and **Ethernet Switches** execute identical Data-Link **backward MAC address learning** and spanning-tree loop avoidance (`OverviewOfNetworking.pptx` Slides 60–63), their internal hardware and collision-domain topology differ radically:
- **Collision Domain Isolation**: A traditional 2-to-4-port bridge connects shared coax/hub LAN segments where stations behind each port still share half-duplex broadcast segments and contend via CSMA/CD. A modern **switch** dedicates one isolated physical wire directly to each host port (**micro-segmentation**)—reducing every collision domain to a single full-duplex point-to-point link with **zero collisions** (`CSMA/CD` disabled).
- **Hardware Switching Fabric**: Bridges historically forwarded frames sequentially in CPU software over shared buses, whereas switches use dedicated hardware **ASIC crossbars** enabling simultaneous wire-speed parallel forwarding across independent port pairs.

---

### Question 1.9 [1 Mark] [Frequency: 2]

> **Question**: *"Reading a file leads to writing of disk blocks."*

#### pre-requisite
- **Inode Metadata (`atime`)**: File systems store metadata for every file in an **inode**, including file size, permissions, block pointers, and timestamps (`mtime`, `ctime`, and **`atime` = last access time**).

#### Answer (Exam Writing Format)
- **T/F Verdict**: **TRUE** — Reading a file normally generates disk block writes unless disabled via `noatime`.

1. **Access Timestamp (`atime`) Update**: Whenever a file (and each directory along its path) is read, POSIX/UNIX file systems update the **`atime` (last-accessed time)** metadata field inside the file's (and traversed directories') inode.
2. **Resulting Disk Writes**: Modifying `atime` dirties the inode block in memory, which must eventually be flushed back as **block-level write operations to disk** (even though the user only requested a `read`!).

#### IN DETAIL
Even read-only file workloads trigger hidden metadata disk writes in default POSIX-compliant file systems due to mandatory **access-timestamp (`atime`) bookkeeping**:
- **Write Amplification from `atime`**: Opening or reading a file requires the VFS to update the inode's `st_atime` field in memory (marking the inode buffer dirty), flushing periodic dirty-inode block writes back to disk—hurting flash SSD wear and saturating read-heavy web/mail servers.
- **Mitigation Mount Options**: Modern Linux distributions default to **`relatime`** (updating `atime` only if it predates `mtime`/`ctime` or exceeds 24 hours) or **`noatime`** (disabling access timestamp updates entirely to keep pure read paths strictly read-only at block layer).

---

### Question 1.10 [1 Mark] [Frequency: 3]

> **Question**: *"As compared to CPU-bound process, OS gives higher priority for I/O bound process."*

#### pre-requisite
- **Scheduling Principle**: I/O-bound processes execute short CPU bursts before submitting slow I/O requests; prioritizing them overlaps I/O and CPU execution.

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: I/O-bound processes execute very short CPU bursts before blocking for lengthy I/O operations. By assigning higher priority to I/O-bound processes, the OS dispatches them quickly so they can initiate I/O immediately and keep I/O devices busy, while CPU-bound processes utilize the remaining CPU time while I/O is in progress. This avoids the convoy effect and maximizes overall CPU utilization, I/O utilization, and interactive response time.

#### IN DETAIL
Prioritizing **I/O-bound processes** over **CPU-bound processes** improves global hardware utilization across both processors and slow peripheral subsystems simultaneously:
- **Short Critical Dispatch Requirement**: An I/O-bound process requires only a brief CPU burst (typically microseconds to a few milliseconds) to parse a read buffer and enqueue its next multi-millisecond DMA request. Giving it preemptive priority lets the OS keep disk arms, SSD channels, and NICs continuously saturated.
- **Minimal Penalty to CPU-Bound Jobs**: Because I/O-bound jobs block swiftly, they consume negligible total CPU percentage before relinquishing the processor back to waiting CPU-bound threads—maximizing overlap between electronic computation and slow I/O transfers simultaneously.

---

### Question 2.1 [3 Marks] [Frequency: 1]

> **Question**: Deadlock prevention protocols which violate "circular wait" may suffer from starvation problem. YES/NO. Justify.

#### pre-requisite
- **Breaking Circular Wait vs. No Preemption**: Breaking circular wait assigns a fixed static number $F(R_i)$ to every resource and requires acquiring resources strictly in ascending order ($F(R_i) < F(R_j)$)—**no resources are ever preempted or forcibly rolled back**.

#### Answer (Exam Writing Format)
- **NO** *(does NOT inherently suffer from starvation due to the circular-wait protocol itself, assuming fair/FIFO lock queues)*.
- **Justification**:
  1. Breaking circular wait enforces a **static total ordering** on resource acquisition (e.g., always acquire lock $A$ before lock $B$) **without preempting or aborting** any process.
  2. Since there are no cycles in the Wait-For Graph, the process holding the highest-ordered resource $R_{\max}$ can always finish and release its locks, allowing waiting processes holding lower locks to advance monotonically in finite time.
  3. Unlike violating "No Preemption" (which causes livelock/starvation via repeated forced rollbacks), circular-wait prevention merely enforces acquisition order and does not itself introduce starvation (provided underlying mutexes/semaphores are FIFO-fair).

#### IN DETAIL
Distinguishing **Deadlock** from **Starvation** clarifies why breaking Circular Wait via strict resource ranking remains starvation-free under standard fair lock queues:
- **No Circular Blocking Cycles**: Strict lock ordering ensures every process holding maximal resource $R_k$ either needs no further locks or requests strictly higher locks $R_j > R_k$, terminating eventually and freeing locks downward without deadlock cycles.
- **Fair Admission Guarantees Bounded Wait**: As long as each individual resource lock uses a FIFO-fair (strong semaphore / queued) wake-up policy rather than arbitrary LIFO or random contention, every thread waiting on $R_i$ advances steadily to the front as finite holders finish—preventing indefinite postponement (**starvation**). *(Only starvation-prone unfair schedulers or unbounded retry spin-loops reintroduce starvation externally.)*

---

### Question 2.2 [3 Marks] [Frequency: 1]

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

### Question 2.3 [3 Marks] [Frequency: 1]

> **Question**: When developing an application, how do you decide whether you should employ "spinlock" or "semaphore"?

#### Answer (Exam Writing Format)
The choice depends on **expected lock-hold duration** and **processor architecture (uniprocessor vs. multiprocessor)**:
1. **Use a Spinlock when**:
   - **Very Short Critical Sections**: Expected wait time is **less than the cost of two context switches** (e.g., updating a few counters/pointers taking nanoseconds), so spinning is faster than sleeping and waking up.
   - **Multiprocessor / Multicore Systems**: Another thread running simultaneously on a different CPU core will release the lock momentarily. *(Never use spinlocks on a single-core CPU—the holder can't run while the waiter hogs the sole CPU!)*
   - **Interrupt Handlers / Non-Sleepable Kernel Contexts**: Where blocking/sleeping is prohibited.
2. **Use a Semaphore (Blocking Lock) when**:
   - **Long Critical Sections or I/O Waits**: Waiting may last microseconds to milliseconds, where spinning would burn vast CPU cycles.
   - **Uniprocessor Systems** or **High Contention**: Sleeping yields the CPU immediately to runnable threads.
   - **Condition Signaling / Counting Resources** (e.g., Producer–Consumer bounded buffers).

#### IN DETAIL
*(Heading aligns with actual question content: **Design Choice: Spinlock vs. Blocking Semaphore Selection**.)*
Spinlocks and blocking semaphores differ fundamentally in how waiting threads consume processor cycles relative to expected lock hold duration:
- **Spinlock Break-Even Economics ($T_{\text{hold}} < 2 \cdot T_{\text{ctx}}$)**: A spinlock executes a tight atomic test-and-set loop (`while (TAS(&lock))`), staying scheduled on its CPU core. If the lock holder is running concurrently on another physical core and holds the lock for nanoseconds, spinning avoids **two expensive OS context switches** (voluntary block + wakeup reschedule, typically microseconds). However, on a **single-core CPU**, spinning is catastrophic—the spinning waiter burns its entire quantum preventing the preempted lock holder from ever running to unlock (`priority inversion / live-hang`). Moreover, under heavy multicore contention, spinning generates sustained **cache-line invalidation traffic** over the shared memory bus.
- **Blocking Semaphore Semantics**: When locks guard long operations (disk I/O, bulk allocations, contended queues), semaphores suspend the caller onto a kernel wait queue (`Blocked` state), immediately freeing the CPU core for other runnable work while also supporting counting resources and cross-thread condition signaling.

---

### Question 2.4 [3 Marks] [Frequency: 1]

> **Question**: In HDFS, explain the concept of checkpoint node and backup node.

#### pre-requisite
- **Reference Paper/Slides**: `HDFS-AdoopFileSystem.pdf` & `HDFSPresentation-PKREDDY-2023.pptx`.
- **HDFS NameNode Persistence**: The NameNode keeps the namespace metadata in RAM and persists it as two disk structures: **`checkpoint` (`fsimage`)** (snapshot of metadata) + **`journal` (`edits`)** (append-only commit log of recent mutations).

#### Answer (Exam Writing Format)
In HDFS, an auxiliary NameNode host can run in one of two protective roles (preventing journal growth and protecting namespace metadata):
1. **CheckpointNode**:
   - Periodically downloads the current `checkpoint` (`fsimage`) and `journal` (`edits`) from the active NameNode.
   - Merges them locally in memory to generate a **new compact checkpoint** and resets an empty journal, uploading the fresh checkpoint back to the NameNode.
   - *Purpose*: Keeps the journal small so NameNode restarts don't take hours replaying massive logs, and maintains an up-to-date backup checkpoint.
2. **BackupNode**:
   - Performs everything a CheckpointNode does (creates periodic checkpoints), **AND** additionally maintains an **always up-to-date in-memory namespace image** synchronized in real time with the active NameNode.
   - Receives a live stream of journal transactions directly from the active NameNode, persists them locally, and applies them to its own RAM image—so if the active NameNode fails, the BackupNode already holds the latest namespace state ready for fast recovery.

#### IN DETAIL
Because HDFS stores active metadata changes purely as sequential appends to **`edits`**, leaving only the initial **`fsimage`** untouched causes journal bloat and slow NameNode restarts without periodic state compaction:
- **CheckpointNode (Periodic Asynchronous Log Merging)**: Periodically pulls `fsimage` and `edits` over HTTP from the active NameNode, replays transactions locally into an updated unified snapshot `fsimage.ckpt`, and uploads it back to truncate the old `edits` log—keeping active NameNode startup recovery bounded without taxing the active NameNode CPU/RAM during merges *(note: runs strictly out-of-band without live failover state)*.
- **BackupNode (Real-Time In-Memory Warm Standby)**: Maintains a complete live namespace state synchronized in its own RAM in real time alongside `fsimage`/`edits`, streaming every active NameNode mutation synchronously/in-step so it simultaneously performs local checkpointing *and* preserves an up-to-the-instant metadata mirror for rapid NameNode recovery.

---

### Question 2.5 [3 Marks] [Frequency: 1]

> **Question**: *"One of the common causes that lengthens the total time taken for a MapReduce operation is a 'straggler': a machine that takes an unusually long time to complete one of the last few map or reduce tasks in the computation."* Explain how MapReduce handles the issue of stragglers?

#### pre-requisite
- **MapReduce Barrier**: A MapReduce job cannot finish until the **slowest** final Map/Reduce task completes. Stragglers occur due to failing disks, background CPU contention, network slowdowns, or misconfigured machines.

#### Answer (Exam Writing Format)
MapReduce handles stragglers using **Speculative Execution (Backup Tasks)**:
1. **Detection**: When a MapReduce job is close to completion (almost all map or reduce tasks have finished), the Master detects any remaining in-progress tasks running unusually slowly compared to peer tasks.
2. **Launching Backup Executions**: The Master schedules **duplicate ("backup") copies** of those remaining in-progress tasks on idle worker machines across the cluster.
3. **First-to-Finish Wins**: Whichever execution (original or backup) finishes first reports completion to the Master, which commits its output and kills/ignores the slower duplicate—cutting total job latency dramatically (e.g., Google observed up to ~44% speedup in MapReduce benchmarks) using only a tiny fraction of extra cluster resources.

#### IN DETAIL
In synchronous barrier-coupled MapReduce stages, job latency is governed strictly by the slowest **tail task ($\max_i T_i$)** rather than mean worker speed (**Straggler / Tail-Latency Problem**):
- **Root Causes of Stragglers**: Hardware heterogeneity, failing ECC DRAM or degrading disk sectors inducing silent DMA retries, OS background maintenance, network switch hotspot congestion, or **Key Skew** (hot Partition keys routing disproportionate records to one Reducer).
- **Speculative Execution Trade-off**: Once progress monitors observe almost all tasks finished (`~99%`) with outlier trailing tasks lagging significantly behind peer mean progress rate, launching duplicate **backup tasks** on idle cluster nodes lets the first completed instance commit cleanly while killing the slower sibling—cutting overall job completion time substantially at tiny marginal cluster compute cost (unless skew itself drives slowness, where all duplicates experience identical key volume).

---

### Question 3 [5 Marks] [Frequency: 3]

> **Question**: Justify the requirement of medium-term scheduler by explaining the corresponding issues if you do not employ medium term scheduler.

#### pre-requisite
- **Medium-Term Scheduler (Swapper)**: Removes partially executed processes from RAM to disk (**swap out**) and later brings them back into RAM (**swap in**).

#### Answer (Exam Writing Format)
The **medium-term scheduler** is required to dynamically regulate the **degree of multiprogramming (MPL)** and balance the process mix via **swapping**:
1. **Balancing the I/O- vs. CPU-Bound Mix**:
   - If all resident processes are **I/O-bound**, the ready queue empties and the CPU sits idle.
   - If all resident processes are **CPU-bound**, I/O queues empty and devices go underutilized.
   - The medium-term scheduler swaps out excess processes of one type and swaps in processes of the other type to keep both CPU and I/O devices busy.
2. **Relieving Memory Contention / Preventing Thrashing**: When main memory is overcommitted (page fault rates spike), swapping out a subset of processes frees frames so remaining resident processes run smoothly.

**Issues If No Medium-Term Scheduler Is Employed**:
Without a **medium-term scheduler** (swapping), an operating system suffers from four major runtime issues:
1. **Thrashing & Memory Exhaustion**: As resident processes grow their dynamic working sets (or too many interactive/background apps remain in RAM), total frame demand exceeds physical RAM. Without swapping some processes out, all processes thrash (spending most time paging from disk instead of executing).
2. **Degraded I/O–CPU Process Mix**: Even if initial admission balanced CPU-bound and I/O-bound processes, workloads shift over time. Without a medium-term scheduler to swap out excess CPU-bound (or I/O-bound) jobs, the system can end up with all CPU-bound processes (devices sit idle) or all I/O-bound processes (CPU sits idle).
3. **Inability to Admit New High-Priority / Interactive Processes**: If RAM is full of idle/long-running processes, new urgent tasks cannot be loaded until old ones exit permanently.
4. **Poor System Throughput & Response Time**: Overcommitted memory and skewed queues lower overall hardware utilization and stall interactive jobs.

#### IN DETAIL
Without a **Medium-Term Scheduler (Swapper)**, admission control becomes irreversible once a process enters physical memory:
- **Dynamic Working-Set Oversubscription**: Even if the Long-Term Scheduler picks a sensible initial multiprogramming level $N$, processes dynamically allocate heap memory (`brk`/`mmap`), expanding their aggregate working-set footprint beyond available DRAM frames.
- **System Failure Modes**: Without the ability to suspend active processes cleanly to backing swap storage (`Ready/Blocked -> Ready-Suspended/Blocked-Suspended`), page replacement degrades into global **thrashing**—CPU utilization collapses while disk paging saturates—and blocked long-idle processes permanently lock physical frames away from active workloads.

---

### Question 4 [5 Marks] [Frequency: 1]

> **Question**: Explain how the notions of "modularity", "abstraction" and "robustness principle" help in developing modern computer systems. What will happen if you do not follow them?

#### pre-requisite
- **Reference Slides**: `Lec1-AOS-Course Overview-2026.pdf` (Slides 1.27–1.34, Saltzer & Kaashoek *"Coping with System Complexity"*).

#### Answer (Exam Writing Format)
1. **Modularity (Divide & Conquer)**:
   - *How it helps*: Decomposes a huge system into isolated, cohesive subsystems (modules). As shown in `Lec1`, if a program has $N$ bugs taking $O(N^2)$ debugging effort, dividing it into $K$ independent modules reduces debugging effort by a factor of $K$ ($\sim K \cdot (N/K)^2 = N^2/K$) and allows upgrading individual modules independently.
   - *Without it*: A monolithic tangled codebase where every tiny change triggers cascading side-effects and debugging scales quadratically.
2. **Abstraction (Information Hiding via Clean Interfaces)**:
   - *How it helps*: Allows modules to interact solely through documented external specifications while hiding internal implementation details—limiting fault propagation (a bug affects only one module) and allowing internal replacement (e.g., swapping HDD for SSD under `read()`).
   - *Without it*: Tight coupling across components—changing a low-level data structure forces rewriting all upper callers, and faults propagate across the whole system ("leaky" collapse).
3. **Robustness Principle ("Be tolerant of inputs and strict on outputs")**:
   - *How it helps*: Components generate strictly spec-compliant outputs while gracefully handling malformed, out-of-range, or slightly non-standard inputs (using safety margins and boundary checks)—essential for interoperability across heterogeneous networks, human UIs, and fault-tolerant systems.
   - *Without it*: Minor input anomalies, version differences, or noisy sensor/network inputs cause brittle crashes, protocol deadlocks, and security/reliability failures across interacting systems.

#### IN DETAIL
Coping with software complexity relies on three complementary design disciplines (`Lec1` & Saltzer/Kaashoek):
- **Modularity**: Decomposes an intricate system into bounded, cohesive units with minimal cross-module coupling. Faults, reasoning scope, and lock contention stay localized inside individual subsystems instead of infecting global state.
- **Abstraction**: Exports clean, implementation-independent contracts (such as POSIX `read()`/`write()` or virtual address spaces) that hide hardware idiosyncrasies (cylinder geometry, DMA timings, interrupts), enabling portability across evolving hardware.
- **Robustness Principle (*Postel's Law*)**: Requires components to strictly validate and adhere to protocol/interface invariants when emitting requests or state, while gracefully bounding, sanitizing, and tolerating malformed, out-of-order, or degraded inputs from callers and hardware peers without catastrophic crashes.

---

### Question 5 [5 Marks] [Frequency: 3]

> **Question**: In TCP protocol, explain the issues with the connection establishment and connection release? Briefly explain the solutions.
> *(2020 Quiz 9 Q3 & 2022 Endsem Q10 focused on Connection Establishment and Three-Way Handshake justification.)*

#### pre-requisite
- **Reference Slides**: `OverviewOfTransportLayer.pptx` (Slides 18–23, Tomlison's Three-Way Handshake).
- **Unreliable Network Reality**: Packets can be **delayed, duplicated, and reordered** arbitrarily inside the network; old packets from previous closed connections may suddenly reappear ("delayed duplicates").
- **Connection Release Reality**: Stopping both directions cleanly faces the distributed **Two-Army Problem** impossibility.

#### Answer (Exam Writing Format)
- **Issues in Connection Establishment**:
  1. **Delayed Duplicate Packets**: In a simple two-way handshake (`CR` $\to$ `ACK`), an old delayed duplicate Connection Request (`CR`) (and old duplicate `DATA` packet) arriving at a server long after an earlier session closed would look like a brand-new valid connection, causing the server to accept phantom connections or corrupt data!
  2. **Clock/Sequence Number Wrap-around & Synchronization**: Both hosts must agree on initial sequence numbers without reusing recently active sequence numbers within $2 \times \text{MSL}$ (Maximum Segment Lifetime), and must survive host crashes without remembering every past connection.
- **Justification for Three-Way Handshake**:
  1. **Host 1 $\to$ Host 2 (`SYN`, $seq = x$)**: Host 1 proposes its random/clock-based initial sequence number $x$.
  2. **Host 2 $\to$ Host 1 (`SYN-ACK`, $seq = y, ack = x + 1$)**: Host 2 picks its *own* fresh sequence number $y$ and asks Host 1 to confirm both $x$ and $y$.
  3. **Host 1 $\to$ Host 2 (`ACK`, $ack = y + 1$)**: Host 1 confirms $y$ (connection opens!).
  - **Why this stops delayed duplicates**:
    - If an **old duplicate `CR` ($seq = x_{old}$)** arrives out of nowhere, Host 2 sends back a `SYN-ACK` asking Host 1 to verify $y$. Since Host 1 never initiated a new connection, Host 1 **rejects (`RST`)** it instead of sending the 3rd `ACK`, so no bogus connection opens!
    - If both an **old duplicate `CR` AND old duplicate `DATA`** arrive, Host 1's rejection (and mismatch with Host 2's fresh $y$) ensures neither the bogus connection nor the stale data is accepted.

**Part B — Connection Release Issues & Solution (2025 AOS Q5)**:
- **Issues**: (1) **Two-Army Problem** (neither host can be 100% certain its final `ACK` arrived without risking infinite `ACK-of-ACK` chains); (2) **Asymmetric Data Loss** if closing aborts opposite-direction in-flight segments.
- **Solution**: **Independent Half-Close (`FIN` + `ACK` in each direction) + Retransmission Timers + `TIME_WAIT` ($2\text{MSL}$)** so delayed segments drain and a lost final `ACK` can be re-sent safely.

#### IN DETAIL
TCP connection lifecycle design resolves two fundamental distributed-systems pitfalls across unreliable networks: **delayed duplicate replays (`Establishment`)** and the **Two-Army asymmetric halting impossibility (`Release`)**:
- **Three-Way Handshake & Delayed Duplicates**: Under naive two-way handshakes (`SYN -> ACK`), an obsolete delayed duplicate `SYN` arriving late opens a half-open ghost connection allocating server resources. With a **3-Way Handshake**, the server's `SYN(N_s) + ACK(N_c + 1)` forces client confirmation of `N_s + 1`; if the `SYN` was a ghost duplicate, the genuine client rejects the unrecognized `N_s + 1` with **`RST`** immediately, closing the ghost state cleanly.
- **Connection Release (`4-Way FIN` + `2 MSL TIME_WAIT`)**: Proving bidirectional agreement via finite unreliable messages hits the unsolvable **Two-Army Problem** (any final `ACK` could drop leaving the sender unsure whether its peer heard it, requiring infinite ACKs of ACKs). TCP settles pragmatically using four independent half-closes (`FIN`/`ACK` per direction) paired with a **`2 × MSL` (`TIME_WAIT`)** hold state on the active closer—retransmitting lost final ACKs if duplicate `FIN`s re-arrive while allowing stale network segments to drain completely before port reuse.

---

### Question 6 [5 Marks] [Frequency: 1]

> **Question**: Briefly explain the following operations of Virtual machine approach: (i) Multiplexing, (ii) Suspension, (iii) Provision, and (iv) Live migration. On cloud platforms, discuss how these operations will significantly enhance the utilization of server resources.

#### pre-requisite
- **Hypervisor / Virtual Machine Monitor (VMM)**: Decouples the OS + applications (packaged inside an isolated Virtual Machine image) from underlying physical server hardware.

#### Answer (Exam Writing Format)
1. **Four VM Operations**:
   - **(i) Multiplexing**: Running multiple independent VMs concurrently on a single physical host, dynamically sharing physical CPU, RAM, disk, and NIC hardware via the hypervisor.
   - **(ii) Suspension**: Pausing an executing VM and saving its entire execution state (CPU registers, memory pages, virtual device state) to disk so it can later be resumed from the exact same instruction.
   - **(iii) Provisioning**: Rapidly instantiating/cloning new VMs from pre-configured base VM templates/snapshots on demand without manual OS installation.
   - **(iv) Live Migration**: Moving a running VM—including its RAM and execution state—from one physical host to another over a network with near-zero downtime and without dropping active client connections.
2. **How Cloud Platforms Enhance Server Resource Utilization**:
   - **Server Consolidation (via Multiplexing)**: Replaces many underutilized physical servers (often running at 10–15% load) with high-density VMs packed onto fewer physical servers running at 70–85%+ utilization.
   - **Elastic Scaling & Zero-Idle Waste (via Provisioning & Suspension)**: Cloud providers rapidly spin up VMs during traffic peaks (**provisioning**) and **suspend** idle VMs to disk during off-peak hours, immediately reclaiming RAM and CPU for active paying tenants.
   - **Dynamic Load Balancing & Power Consolidation (via Live Migration)**: Underutilized servers can have their live VMs migrated onto fewer physical machines during low traffic (letting empty servers power down), and overloaded hotspots can be rebalanced seamlessly without service outages.

#### IN DETAIL
Cloud infrastructure management builds directly upon four fundamental properties unlocked by **encapsulating an entire machine's CPU, memory, and virtual disk state inside software abstractions**:
- **Multiplexing**: Time-slicing and spatial partitioning allow a hypervisor to consolidate multiple underutilized OS workloads onto one physical server with strict fault and performance isolation.
- **Suspension & Snapshotting**: Because every piece of virtual machine execution state (vCPU registers, device buffers, virtual RAM, disk blocks) lives in hypervisor-accessible software structures, a VM can be serialized cleanly to disk and resumed identically hours later or cloned (`Provisioning`).
- **Live Migration**: Encapsulation lets a VMM copy guest memory and virtual device state across a data-center network while the VM remains executing on the source host, resuming seamlessly on the target host with millisecond downtime.

---

### Question 7 [5 Marks] [Frequency: 1]

> **Question**: A small computer has 8 page frames, each containing a page. The page frames contain virtual pages `A, C, G, H, B, L, N` and `F` in that order. Their respective load times were `18, 23, 5, 7, 32, 19, 3` and `8`. Their reference bits are `1, 0, 1, 1, 0, 1, 1` and `0`, and their modified bits are `1, 1, 1, 0, 0, 0, 1` and `1`, respectively. What is the order that Enhanced Second Chance considers pages and which one is selected?

#### pre-requisite
- **Enhanced Second Chance Algorithm** (Silberschatz, `virtualMemory_OS.ppt`): Evaluates the ordered pair `(Reference Bit R, Modified Bit M)` across **4 priority classes**:
  1. **`(0, 0)`**: Neither recently used nor modified — **best page to replace** (no disk write needed).
  2. **`(0, 1)`**: Not recently used, but modified — must write to disk before eviction.
  3. **`(1, 0)`**: Recently used, clean — likely used again soon.
  4. **`(1, 1)`**: Recently used and modified — **worst page to replace**.
- The clock scans circularly starting from the **oldest loaded page** (`N`, load time = 3) — or from frame 0 (`A`) — searching first for class `(0,0)`, then `(0,1)` (clearing `R` bits on pass), etc.

#### Answer (Exam Writing Format)
**Page Table State Summary**:

| Frame # | Page | Load Time | `(R, M)` Class | Meaning |
| :---: | :---: | :---: | :---: | :--- |
| 0 | `A` | 18 | `(1, 1)` | Recently used & dirty |
| 1 | `C` | 23 | `(0, 1)` | Clean? No (dirty), unreferenced |
| 2 | `G` | 5 | `(1, 1)` | Recently used & dirty |
| 3 | `H` | 7 | `(1, 0)` | Recently used & clean |
| **4** | **`B`** | **32** | **`(0, 0)`** | **Unreferenced & clean (Best victim!)** |
| 5 | `L` | 19 | `(1, 0)` | Recently used & clean |
| 6 | `N` | 3 (Oldest) | `(1, 1)` | Recently used & dirty |
| 7 | `F` | 8 | `(0, 1)` | Unreferenced & dirty |

1. **Priority Class Order Considered by Enhanced Second Chance**:
   - Class 1: **`(0, 0)`** $\to$ Class 2: **`(0, 1)`** $\to$ Class 3: **`(1, 0)`** $\to$ Class 4: **`(1, 1)`**.
2. **Page Scan Order**:
   - **Pass 1 (Searching for class `(0, 0)` without clearing `R` bits)**:
     - Starting from the **oldest loaded page (FIFO clock pointer at `N`, load time 3)** in circular load-time order:
       `N (3, 1,1)` $\to$ `G (5, 1,1)` $\to$ `H (7, 1,0)` $\to$ `F (8, 0,1)` $\to$ `A (18, 1,1)` $\to$ `L (19, 1,0)` $\to$ `C (23, 0,1)` $\to$ **`B (32, 0,0)` — MATCH!**
     - *(Note: Even if scanning strictly by frame order `0→7` starting at `A`, the scan checks `A(1,1) → C(0,1) → G(1,1) → H(1,0) → B(0,0)` — still hitting `B` on Pass 1!)*
3. **Selected Victim Page**: **Page `B`** (the sole page in class `(0, 0)`).

#### IN DETAIL
**Enhanced Second Chance (NRU / Two-Bit Clock)** improves replacement decisions by pairing recency (`R`) with writeback cost (`M` / Dirty Bit) into four hierarchical priority classes:
- **Four-Class Lexicographic Ordering `(R, M)`**:
  - Class `(0, 0)`: Recently unused and **clean** $\implies$ zero disk write needed before reuse (optimal victim).
  - Class `(0, 1)`: Recently unused, **dirty** $\implies$ requires asynchronous swap writeback prior to reuse.
  - Class `(1, 0)`: Actively referenced, **clean**.
  - Class `(1, 1)`: Actively referenced, **dirty** $\implies$ highest retention priority.
- **Tie-Breaking by Sweep / FIFO Arrival**: The clock hand scans cyclically starting from the oldest candidate pointer (`load time = 3`, page `N`), sweeping up to four passes looking for the lowest non-empty `(R, M)` class while decaying `R = 1 -> 0` only during search passes—minimizing total swap disk writes alongside miss rate.

---

### Question 8 [5 Marks] [Frequency: 2]

> **Question**: Explain the concept of "RAID Level 4". What is the issue with "RAID Level 4"? How does RAID Level 5 overcome the issues with RAID Level 4?

#### pre-requisite
- **Reference Slides**: `SecondaryStorageStructure_AOS.ppt`.
- **RAID 3 vs. RAID 4 vs. RAID 5**: All three use a single parity disk worth of capacity ($N-1$ data + 1 parity equivalent) to tolerate 1 disk failure, differing in *striping granularity* and *parity placement*.

#### Answer (Exam Writing Format)
- **How RAID 4 is better than RAID 3**:
  1. **Striping Granularity**: RAID 3 uses **bit-level (or byte-level) striping**, forcing *all* disks to participate in every single I/O (requiring synchronized spindles). RAID 4 uses **block-level striping**, where entire blocks live on individual disks.
  2. **Parallel Independent Reads**: Because blocks reside on separate disks, RAID 4 can service **multiple small random read requests in parallel** across different data disks, whereas RAID 3 can handle only one I/O at a time.
- **Comparison: RAID 4 vs. RAID 5**:
| Feature | RAID Level 4 | RAID Level 5 |
| :--- | :--- | :--- |
| **Data Striping** | Block-level striping. | Block-level striping. |
| **Parity Placement** | **Dedicated single parity disk** (all parity blocks live on Disk $N$). | **Distributed parity** rotated cyclically across *all* disks. |
| **Write Bottleneck** | Severe bottleneck on the single parity disk—every write (even to unrelated data disks) must update the *same* parity disk, serializing parallel writes. | **Eliminates parity-disk bottleneck**—writes touching different stripes update parity blocks on *different* disks concurrently. |
| **Random Reads** | Reads from $N-1$ data disks only (parity disk stays mostly idle during normal reads). | Reads spread across **all $N$ disks** (higher read parallelism). |

#### IN DETAIL
**RAID 4** outperforms **RAID 3** on multi-user workloads by shifting from fine-grained bit/byte interleaving to **independent block-level striping**, yet inherits a fundamental write-path scalability flaw resolved by **RAID 5**:
- **Why RAID 4 Bottlenecks on Writes**: Even though independent data blocks reside across $N-1$ distinct drives, *all* XOR parity blocks across every stripe sit exclusively on **one dedicated parity drive (`Disk N`)**. Consequently, two unrelated writes to `Block A (Disk 0)` and `Block B (Disk 2)` must *both* read and overwrite `Disk N`—serializing 100% of random writes onto a single overloaded actuator while leaving `Disk N` completely idle on reads.
- **RAID 5 Rotating Distributed Parity**: Eliminates the dedicated parity drive by cycling parity placement diagonally across all $N$ disks (`Stripe i` stores parity on disk $i \bmod N$). Disjoint stripe writes update distinct parity drives simultaneously in parallel, and normal reads distribute across all $N$ spindles cleanly (though each individual small write still incurs 4 I/Os via Read-Modify-Write).

---

### Question 9 [5 Marks] [Frequency: 3]

> **Question**: A computer has a cache, main memory, and a disk used for virtual memory. If a referenced word is in cache, $20\text{ nsec}$ are required to access it. If it is in main memory but not in cache, $60\text{ nsec}$ are needed to load into the cache, and then the reference is started again. If the word is not in main memory, $12\text{ msec}$ are required to fetch the word from the disk, followed by $60\text{ nsec}$ to copy it to the cache, and then the reference is started again. The cache hit ratio is $0.9$ and the main memory hit ratio is $0.6$. What is the average time, in nanoseconds, required to access a referenced word on this system?

#### pre-requisite
- **"Reference is started again" Rule**: When a miss occurs at Cache ($20\text{ ns}$ to detect miss), after loading from RAM ($+60\text{ ns}$) or Disk ($+12\text{ ms} + 60\text{ ns}$), restarting the cache access adds another $20\text{ ns}$! *(Always convert $1\text{ ms} = 10^6\text{ ns}$.)*

#### Answer (Exam Writing Format)
- **Given Parameters**:
  - Cache access time ($t_c$) = $20\text{ ns}$; Cache hit ratio ($H_c$) = $0.9 \implies$ Cache miss = $0.1$.
  - RAM-to-cache load time ($t_m$) = $60\text{ ns}$; Main memory hit ratio (given cache miss, $H_m$) = $0.6 \implies$ Page fault = $0.4$.
  - Disk fetch time ($t_d$) = $12\text{ ms} = 12,000,000\text{ ns}$.
- **Three Disjoint Access Cases**:
  1. **Cache Hit** ($P_1 = 0.9$):
     $$T_1 = 20\text{ ns}$$
  2. **Cache Miss, Main Memory Hit** ($P_2 = (1 - 0.9) \times 0.6 = 0.1 \times 0.6 = 0.06$):
     $$T_2 = \underbrace{20\text{ ns}}_{\text{cache check}} + \underbrace{60\text{ ns}}_{\text{load to cache}} + \underbrace{20\text{ ns}}_{\text{restart reference}} = 100\text{ ns}$$
  3. **Cache Miss, Main Memory Miss (Page Fault)** ($P_3 = 0.1 \times (1 - 0.6) = 0.1 \times 0.4 = 0.04$):
     $$T_3 = \underbrace{20\text{ ns}}_{\text{cache check}} + \underbrace{12,000,000\text{ ns}}_{\text{fetch from disk}} + \underbrace{60\text{ ns}}_{\text{copy to cache}} + \underbrace{20\text{ ns}}_{\text{restart reference}} = 12,000,100\text{ ns}$$
- **Average Access Time**:
  $$\text{Avg Time} = (0.9 \times 20) + (0.06 \times 100) + (0.04 \times 12,000,100)$$
  $$= 18 + 6 + 480,004 = \mathbf{480,028\text{ nanoseconds}}$$

#### IN DETAIL
Multi-tier memory latency evaluation (**Cache $\to$ Main Memory $\to$ Swap Disk**) illustrates how tiny **page-fault probabilities ($p$)** dominate **Effective Access Time (EAT)** due to a $10^5\times$–$10^6\times$ mechanical speed gap:
- **Restart-Semantics Model**: When a memory reference misses both SRAM cache ($20\text{ ns}$, rate $1-H_c = 0.1$) and RAM (page fault, rate $p = 1 - H_m = 0.04$), the hardware first burns the initial lookup + OS swap latency ($12\text{ ms}$) to page the missing block into DRAM, and then **restarts the memory cycle cleanly**, paying a fresh cache miss ($20\text{ ns}$) plus RAM-to-cache refill ($120\text{ ns}$) before serving the datum.
- **Scale Sensitivity**: Even at $96\%$ RAM residency, $0.04 \times 12,000,000\text{ ns} = 480,000\text{ ns}$ crushes the $28\text{ ns}$ cache/RAM terms completely—explaining why real demand-paged systems must maintain page-fault frequencies below $p < 10^{-5}$ to avoid crippling slowdowns.

---

### Question 10 [5 Marks] [Frequency: 1]

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

### Question 11 [5 Marks] [Frequency: 3]

> **Question**: Developments in operating systems have generally occurred in an evolutionary rather than revolutionary fashion. For each of the following transitions, describe the primary motivations of operating systems designers that led them to produce the new type of system from the old:
> - **a.** Single user dedicated systems to multiprogramming.
> - **b.** Fixed partition multiprogramming systems with absolute translation and loading to fixed partition multiprogramming systems with re-locatable translation and loading.
> - **c.** Fixed partition multiprogramming to variable partition multiprogramming.
> - **d.** Contiguous storage allocation systems to non-contiguous storage allocation systems.
> - **e.** Single user dedicated systems with manual job-to-job transition to single user dedicated systems with single stream batch processing systems.

#### pre-requisite
- **Evolutionary Fix Pattern**: Each OS generation was engineered to solve the dominant CPU-idleness or memory-fragmentation bottleneck left by the previous generation.
- **Contiguous vs. Non-Contiguous**: Contiguous places a process in one unbroken block (`base`..`base+limit`); non-contiguous splits it across scattered physical frames/segments.

#### Answer (Exam Writing Format)
- **a. Single-user dedicated $\to$ Multiprogramming**:
  - *Motivation*: In single-user systems, the expensive CPU sat idle >90% of the time whenever a running job waited for slow I/O. Loading multiple jobs into RAM simultaneously lets the OS switch the CPU to another ready job during I/O waits, dramatically increasing **CPU utilization and throughput**.
- **b. Fixed partition with absolute loading $\to$ Fixed partition with relocatable loading**:
  - *Motivation*: With absolute loading, a job compiled for Partition 2 could **only** run in Partition 2—sitting queued on disk even while a larger Partition 3 sat completely empty! Relocatable loading allows any job to run in **any** sufficiently large free partition, improving memory utilization and scheduling flexibility.
- **c. Fixed partition $\to$ Variable partition multiprogramming**:
  - *Motivation*: Rigid fixed partition sizes suffered severe **internal fragmentation** (small jobs wasted unused space inside huge partitions) and rejected jobs slightly larger than the largest fixed partition. Variable partitions dynamically size each allocation to match the exact job size, eliminating internal fragmentation.
- **d. Contiguous storage allocation $\to$ Non-contiguous storage allocation**:
  - *Motivation*: Variable-size contiguous partitions created **external fragmentation** (enough total free RAM, yet scattered in unusable small holes requiring expensive compaction). Non-contiguous allocation (paging/segmentation) splits processes across scattered physical frames, eliminating external fragmentation and compaction overhead.
- **e. Single-user manual transition $\to$ Single-stream batch processing**:
  - *Motivation*: Manual human setup between jobs (mounting tapes, resetting switches, loading cards) left multi-million-dollar CPUs idle for long setup periods. Grouping jobs into batches and automating job-to-job transitions via a **Resident Monitor** eliminated human setup idle delays and improved machine throughput.

**(Additional Depth for Transition d — Contiguous $\to$ Non-Contiguous)**:
  - Eliminates **external fragmentation** and avoids expensive **memory compaction**.
  - Enables running programs larger than any single contiguous hole (plus **virtual memory / demand paging**) and clean per-page protection/code sharing.

#### IN DETAIL
Each OS evolutionary stage emerged to eliminate the dominant hardware bottleneck created by the huge speed gap between electronic CPUs ($\mu$s–ns) and electromechanical peripherals (ms–s):
- **Hardware Enablers**: Asynchronous DMA channels, programmable interval timers, hardware mode bits, and Memory Management Units (MMUs) decoupled CPU compute from slow I/O, prevented runaway loops from monopolizing the processor, and enabled non-contiguous virtual-to-physical memory translation.
- **Optimization Metric Shift**: Early systems maximized raw hardware throughput (CPU utilization in batch/multiprogramming), whereas later systems balanced throughput with bounded interactive latency (time-sharing) and physical RAM overcommitment (paging/virtual memory).

---

### Question 12 [5 Marks] [Frequency: 3]

> **Question**: Consider the following network. Distance vector routing is used and the following vectors have just come in to router $C$: From $B$: $(5, 0, 8, 1, 6, 2)$; From $D$: $(16, 12, 6, 0, 9, 10)$; From $E$: $(7, 6, 3, 9, 0, 4)$. The cost of the links from router $C$ to neighbors $B, D,$ and $E$ are $6, 3,$ and $5$ respectively. What is router $C$'s new routing table?

#### pre-requisite
- **Bellman-Ford Distance Vector Update**: For each destination $y \in \{A, B, C, D, E, F\}$:
  $$D_C(y) = \min_{v \in \{B, D, E\}} \big\{ c(C, v) + D_v(y) \big\}, \quad D_C(C) = 0$$

#### Answer (Exam Writing Format)
Given link costs $c(C, B) = 6$, $c(C, D) = 3$, $c(C, E) = 5$:

| Destination | Via $B$ ($6 + D_B$) | Via $D$ ($3 + D_D$) | Via $E$ ($5 + D_E$) | Min Cost | Next Hop |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **$A$** | $6 + 5 = \mathbf{11}$ | $3 + 16 = 19$ | $5 + 7 = 12$ | **11** | **$B$** |
| **$B$** | $6 + 0 = \mathbf{6}$ | $3 + 12 = 15$ | $5 + 6 = 11$ | **6** | **$B$** |
| **$C$** | — | — | — | **0** | **— (Self)** |
| **$D$** | $6 + 1 = 7$ | $3 + 0 = \mathbf{3}$ | $5 + 9 = 14$ | **3** | **$D$** |
| **$E$** | $6 + 6 = 12$ | $3 + 9 = 12$ | $5 + 0 = \mathbf{5}$ | **5** | **$E$** |
| **$F$** | $6 + 2 = \mathbf{8}$ | $3 + 10 = 13$ | $5 + 4 = 9$ | **8** | **$B$** |

- **Router $C$'s New Routing Table**:
  - **Distance Vector**: `(11, 6, 0, 3, 5, 8)`
  - **Outgoing Lines / Next Hops**: `(B, B, -, D, E, B)`

#### IN DETAIL
**Distance Vector Routing** implements **distributed asynchronous Bellman–Ford relaxation** purely from local link costs and neighbor-advertised cost vectors:
- **Bellman–Ford Transition Equation**: Each router $x$ stores solely direct edge costs $c(x, v)$ to immediate neighbors $v \in N(x)$ plus the latest distance vectors $\mathbf{D}_v = [D_v(y)]_{y \in V}$ advertised by each neighbor. Recomputing $x$'s new distance table is completely mechanical: $D_x(y) = \min_{v \in N(x)} \{ c(x, v) + D_v(y) \}$, setting the corresponding minimum-cost neighbor $v^*$ as forwarding `Next Hop` (and keeping self-distance $D_x(x) = 0$).
- **Convergence Mechanics**: Without global graph visibility, good cost reductions propagate at 1 hop per periodic exchange round, whereas arbitrary topology partitioning relies on loop-avoidance safeguards because old indirect neighbor paths retain stale finite hop estimates.

---

### Question 13 [5 Marks] [Frequency: 1]

> **Question**: Suppose we have a computer system with a 44-bit virtual address, page size of $64\text{ K}$ and $4\text{ bytes}$ per page table entry.
> - **(i)** How many pages are there in the virtual address space? [1]
> - **(ii)** Suppose we use two-level paging and arrange for all page tables to fit into a single page frame. How will the bits of address be divided up? [2]
> - **(iii)** Suppose we have a $4\text{ GB}$ program such that the entire program and all necessary page tables (using two-level paging) are in memory. How many frames of memory is used by the program, including its page tables? [2] *(Note: frame size = page size)*

#### pre-requisite
- **Powers of 2**: $1\text{ KB} = 2^{10}$, $64\text{ KB} = 2^6 \times 2^{10} = 2^{16}$, $4\text{ GB} = 2^2 \times 2^{30} = 2^{32}$.
- **PTEs per Page Table Frame**: $\frac{\text{Page Size}}{\text{PTE Size}}$.

#### Answer (Exam Writing Format)
Given: Virtual Address = $44\text{ bits}$, Page/Frame Size = $64\text{ KB} = 2^{16}\text{ bytes}$, PTE Size = $4\text{ bytes} = 2^2\text{ bytes}$.

- **(i) Number of pages in Virtual Address Space [1 Mark]**:
  $$\text{Number of virtual pages} = \frac{2^{44}}{2^{16}} = \mathbf{2^{28}\text{ pages}}\; (= 268,435,456\text{ pages})$$

- **(ii) Bit Division for Two-Level Paging [2 Marks]**:
  - **Offset bits ($d$)**: $\log_2(64\text{ KB}) = \log_2(2^{16}) = \mathbf{16\text{ bits}}$.
  - **Remaining VPN bits**: $44 - 16 = \mathbf{28\text{ bits}}$.
  - **Entries fitting in one 64 KB frame**: $\frac{2^{16}\text{ B}}{4\text{ B}} = \frac{2^{16}}{2^2} = 2^{14}\text{ PTEs} \implies$ each page table level indexes **$\log_2(2^{14}) = 14\text{ bits}$**.
  - **Address Bit Split**:
    $$\mathbf{14\text{ bits (Outer Page Table } P_1\text{)}} \;\mid\; \mathbf{14\text{ bits (Inner Page Table } P_2\text{)}} \;\mid\; \mathbf{16\text{ bits (Page Offset } d\text{)}}$$
    *(Verify: $14 + 14 + 16 = 44\text{ bits}$, and Outer Table fits in $2^{14} \times 4\text{ B} = 64\text{ KB} = 1\text{ frame}$!)*

- **(iii) Total Memory Frames Used by 4 GB Program [2 Marks]**:
  1. **Data/Program Frames**:
     $$\text{Program size} = 4\text{ GB} = 2^{32}\text{ bytes} \implies \frac{2^{32}}{2^{16}} = \mathbf{2^{16} = 65,536\text{ frames}}$$
  2. **Inner Page Table Frames ($P_2$)**:
     Each inner page table frame holds $2^{14} = 16,384$ PTEs (mapping $16,384$ data pages).
     $$\text{Inner Page Tables needed} = \frac{2^{16}}{2^{14}} = 2^2 = \mathbf{4\text{ frames}}$$
  3. **Outer Page Table Frame ($P_1$)**:
     $$\text{Outer Page Table} = \mathbf{1\text{ frame}}$$
  4. **Total Frames Used**:
     $$65,536 + 4 + 1 = \mathbf{65,541\text{ frames}}$$

#### IN DETAIL
Virtual-to-physical tree geometry balances two independent dimensional constraints: **intra-page branching capacity** and **global physical frame count**:
- **Index Bit Partitioning**: With $64\text{ KB} = 2^{16}\text{ B}$ pages and $4\text{ B} = 2^2\text{ B}$ PTEs, one page table node holds $2^{14}$ entries ($14$ bits). Subtracting $16$ offset bits from a $44$-bit virtual address leaves exactly $28$ VPN bits, splitting symmetrically into two $14$-bit levels ($L_1 = 14$, $L_2 = 14$, Offset $= 16$) so both outer directory and inner leaf tables occupy exact single $64\text{ KB}$ frames.
- **Large-Page Trade-off (Internal Fragmentation vs. Metadata)**: Coarse $64\text{ KB}$ pages dramatically collapse page-table depth and enlarge TLB coverage ($\text{TLB entries} \times 64\text{ KB}$), reducing capacity misses; however, each $4\text{ GB}$ footprint occupies $2^{16} = 65,536$ frames while small allocations suffer increased **internal fragmentation** inside the trailing frame.

---

## Part 2: OSN Monsoon 2025 End-Semester Examination (`AgentOS`)

### AgentOS — Question 1 [3 Points] — Round Robin Scheduling (q = 1 ms) [Frequency: 1]

> **Question**: The AgentOS team is evaluating whether Round Robin (RR) is a suitable scheduler for their workloads. They consider three processes:
> - **P1**: arrives at $0\text{ ms}$ and requires $5\text{ ms}$ of CPU time;
> - **P2**: arrives at $1\text{ ms}$ and exhibits mixed behaviour with a $2\text{ ms}$ CPU burst, followed by a $4\text{ ms}$ I/O operation, and then another $2\text{ ms}$ CPU burst;
> - **P3**: arrives at $3\text{ ms}$ and requires $4\text{ ms}$ of CPU time.
>
> The team simulates these processes under Round Robin scheduling with a $1\text{ ms}$ time quantum and no context-switch overhead, and they want to determine the resulting average turnaround time. What is the correct value and why?
> - a. `10.00 ms`
> - b. `8.66 ms`
> - c. `11.33 ms`
> - d. `9.00 ms`

#### pre-requisite
- **Turnaround Time**: $\text{TAT} = \text{Completion Time} - \text{Arrival Time}$ (includes time spent executing CPU bursts, waiting in the ready queue, *and* waiting during I/O!).
- **Queue Tie-Breaking Note**: In Round Robin, when a process's time quantum expires at the exact same timestamp $t$ that a new process arrives, standard textbook convention enqueues the preempted process *or* the newly arrived process in a consistent FIFO order (note that `11.33 ms` is mathematically impossible under *any* valid 1 ms RR tie-breaking!).

#### Answer (Exam Writing Format)
- **Correct Option**: **a. `10.00 ms`**.
- **Step-by-Step Gantt Simulation ($q = 1\text{ ms}$, preempted process re-queued before simultaneous arrival at $t$)** *(or standard queue order yielding `(9, 12, 9)`)*:
  - $0\text{–}1$: `P1` runs (4 left). At $t=1$, `P2` arrives $\implies$ Queue: `[P1, P2]`
  - $1\text{–}2$: `P1` runs (3 left). Queue: `[P2, P1]`
  - $2\text{–}3$: `P2` runs (1 left in 1st CPU burst). At $t=3$, `P3` arrives $\implies$ Queue: `[P1, P2, P3]`
  - $3\text{–}4$: `P1` runs (2 left). Queue: `[P2, P3, P1]`
  - $4\text{–}5$: `P2` finishes 1st CPU burst and blocks for $4\text{ ms}$ I/O (`t = 5 to 9`). Queue: `[P3, P1]`
  - $5\text{–}6$: `P3` runs (3 left). Queue: `[P1, P3]`
  - $6\text{–}7$: `P1` runs (1 left). Queue: `[P3, P1]`
  - $7\text{–}8$: `P3` runs (2 left). Queue: `[P1, P3]`
  - $8\text{–}9$: `P1` finishes! (**$C_{P1} = 9\text{ ms}$**). At $t=9$, `P2` finishes I/O $\implies$ Queue: `[P3, P2]`
  - $9\text{–}10$: `P3` runs (1 left). Queue: `[P2, P3]`
  - $10\text{–}11$: `P2` runs (1 left in 2nd CPU burst). Queue: `[P3, P2]`
  - $11\text{–}12$: `P3` finishes! (**$C_{P3} = 12\text{ ms}$**). Queue: `[P2]`
  - $12\text{–}13$: `P2` finishes! (**$C_{P2} = 13\text{ ms}$**).
- **Turnaround Times**:
  - $\text{TAT}(P_1) = 9 - 0 = 9\text{ ms}$
  - $\text{TAT}(P_2) = 13 - 1 = 12\text{ ms}$
  - $\text{TAT}(P_3) = 12 - 3 = 9\text{ ms}$
  - **Average Turnaround Time** $= \frac{9 + 12 + 9}{3} = \frac{30}{3} = \mathbf{10.00\text{ ms}}$.

#### IN DETAIL
When analyzing **Round Robin ($q = 1\text{ ms}$)** schedules that combine CPU bursts and asynchronous I/O intervals, turnaround accounting must separate active CPU service from blocked I/O intervals:
- **Wall-Clock Definition of Latency / Turnaround**: $\text{TAT}_i = t_{\text{final\_exit}}(i) - t_{\text{arrival}}(i)$ always spans from submission to completion—naturally absorbing any self-induced `Blocked` I/O gaps (such as $P_2$'s $4\text{ ms}$ I/O during $[5, 9]$) alongside ready-queue waiting time and CPU execution.
- **Event-Ordering Sensitivity**: In deterministic discrete-event simulation at $q=1\text{ ms}$, simultaneous queue insertions at integer tick $t$ (newly arriving job vs. I/O wakeup vs. time-slice-expired thread) determine exact tie-breaking order—though conservation of total CPU work ($\sum \text{burst}_i = 13\text{ ms}$ with zero CPU idle gaps) strictly bounds aggregate completion time.

---

### AgentOS — Question 2 [3 Points] — TCP Retransmission Timeout Calculation [Frequency: 1]

> **Question**: The AgentOS team was testing the networking stack when they observed significant network jitter and decided to inspect the TCP timeout behavior. At time $t$, the system's internal state is `EstimatedRTT = 50 ms` and `DevRTT = 4 ms`. For the most recent packet, the RTT was `60 ms`. Assuming the standard TCP parameters of $\alpha = 0.125$ and $\beta = 0.25$, what should the TCP Timeout Interval be updated to for the *next* transmission and why?
> - a. `67.25 ms`
> - b. `72.00 ms`
> - c. `51.25 ms`
> - d. `70.75 ms`

#### pre-requisite
- **RFC 6298 / Jacobson–Karels TCP RTO Equations**:
  1. $\text{EstimatedRTT} = (1 - \alpha) \cdot \text{EstimatedRTT} + \alpha \cdot \text{SampleRTT}$
  2. $\text{DevRTT} = (1 - \beta) \cdot \text{DevRTT} + \beta \cdot |\text{SampleRTT} - \text{EstimatedRTT}|$
  3. $\text{TimeoutInterval} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT}$

#### Answer (Exam Writing Format)
- **Correct Option**: **b. `72.00 ms`**.
- **Calculation**:
  1. **Updated `EstimatedRTT`**:
     $$\text{EstimatedRTT} = (1 - 0.125)(50) + 0.125(60) = 43.75 + 7.50 = \mathbf{51.25\text{ ms}}$$
  2. **Updated `DevRTT`**:
     $$\text{DevRTT} = (1 - 0.25)(4) + 0.25 \cdot |60 - 51.25| = 3.00 + 0.25(8.75) = 3.00 + 2.1875 = \mathbf{5.1875\text{ ms}}$$
  3. **Updated `TimeoutInterval`**:
     $$\text{TimeoutInterval} = 51.25 + 4 \times 5.1875 = 51.25 + 20.75 = \mathbf{72.00\text{ ms}}$$

#### IN DETAIL
Adaptive TCP **Retransmission Timeout ($\text{RTO}$)** uses Exponentially Weighted Moving Averages (**EWMA**) of *both* mean latency and statistical variance to minimize false retransmissions without waiting sluggishly on real losses:
- **Tracking Jitter Beyond Mean Latency**: Setting $\text{RTO} = \text{EstimatedRTT}$ alone causes **~50% spurious retransmissions** whenever normal round-trip times fluctuate symmetrically above average. Jacobson/Karels introduced **`DevRTT`** (smoothed $|\text{SampleRTT} - \text{EstimatedRTT}|$ with $\beta = 0.25$) and set **$\text{RTO} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT}$**—giving a wide statistical safety margin during volatile queuing spikes while tightening swiftly on stable fiber paths (`72.00 ms` here).
- **Karn's Algorithm Exclusion**: Retransmitted segments introduce ambiguity (did the returning ACK reply to the original or retried copy?); TCP excludes retransmitted RTT samples from EWMA updates and **doubles $\text{RTO}$ exponentially** upon consecutive timeouts to avoid congestion collapse.

---

### AgentOS — Question 3 [3 Points] — Segmentation Address Translation (0x5100) [Frequency: 1]

> **Question**: The AgentOS team was considering using segmentation as the approach for memory virtualisation. The underlying system uses 16-bit Virtual Addresses. The memory management hardware uses the top 2 bits of the address to select the segment. The current Segment Table is configured as follows:
>
> | Segment | ID (Binary) | Base Address (Physical) | Limit (Size) |
> | :--- | :--- | :--- | :--- |
> | **Code** | `00` | `0x2000` | `0x1000` |
> | **Heap** | `01` | `0x9000` | `0x2000` |
> | **Stack** | `10` | `0xE000` | `0x1000` |
>
> A process generates the Virtual Address `0x5100`. What is the corresponding Physical Address and why?
> - a. `0x3100`
> - b. `0xA100`
> - c. `0xF100`
> - d. Segmentation Fault

#### pre-requisite
- **Segmented Address Translation**:
  1. Extract Segment ID from top bits and Offset from remaining lower bits.
  2. **Bounds Check**: Verify $\text{Offset} < \text{Limit}$ (if violated $\implies$ Segmentation Fault).
  3. Compute $\text{Physical Address} = \text{Base} + \text{Offset}$.

#### Answer (Exam Writing Format)
- **Correct Option**: **b. `0xA100`**.
- **Step-by-Step Translation**:
  1. **Convert `0x5100` to 16-bit binary**:
     $$\texttt{0x5100} = \underbrace{\texttt{01}}_{\text{Segment ID}}\; \underbrace{\texttt{01 0001 0000 0000}}_{\text{14-bit Offset}} = \texttt{0101 0001 0000 0000}_2$$
  2. **Identify Segment**: Top 2 bits = **`01`** $\implies$ **Heap** segment (`Base = 0x9000`, `Limit = 0x2000`).
  3. **Extract Offset**: Remaining 14 bits = `01 0001 0000 0000`$_2$ = **`0x1100`**.
  4. **Bounds Check**: `0x1100 < 0x2000` (Valid!).
  5. **Physical Address**: $\texttt{0x9000} + \texttt{0x1100} = \mathbf{\texttt{0xA100}}$.

#### IN DETAIL
Segmentation hardware splits a $k$-bit virtual address explicitly into a discrete **Segment Selector (`s`)** and **Intra-Segment Offset (`d`)**, performing parallel bounds protection before relocation:
- **Binary Bit-Masking vs. Decimal Hex**: Hardware inspects the top $n$ most significant bits (here upper 2 bits `01` of `0x5100 = 0101_0001_0000_0000_2` $\implies s = 1$), rather than the whole leading hex nibble, extracting the remaining 14 bits (`0x1100`) as byte offset $d$.
- **Two-Stage MMU Check**: First verifying $d < \text{Limit}[s]$ traps invalid sparse cross-segment pointers immediately (`SIGSEGV` / Segment Fault); only valid offsets add $\text{Base}[s] + d$ to produce the linear physical address, cleanly supporting independent per-segment protection flags (`R/W/X`) and variable growth directions (upward heap vs. downward stack).

---

### AgentOS — Question 4(a) [5 Points] — TLB Miss & Async Page Fault Handling [Frequency: 1]

> **Question**: After the above translation, the AgentOS team wants you to lead the design and development of a subsystem of a new Kernel. The team wants the system to utilize **Demand Paging with a software-managed TLB** for memory management. There may be scenarios where the page translation is not present in the TLB and a TLB miss occurs. Further, post TLB miss, the page may not be present in the physical memory as well. Given this constraint:
> **a.** Design an algorithm/pseudocode for the OS Trap Handler to resolve conditions where the page is not in memory. Your algorithm must sequentially detail the lifecycle of this event from the moment the trap occurs to the moment the process resumes. You must explicitly account for CPU efficiency: *The CPU cannot sit idle while the disk spins.* (5 points)

#### pre-requisite
- **Software-Managed TLB + Demand Paging**: When a virtual address misses the TLB, hardware traps to the OS (`TLB_Miss_Handler`). If the page table shows the page is **invalid / not present in RAM**, a **Page Fault** occurs, requiring slow disk I/O (~milliseconds), during which the CPU must switch to another ready process!

#### Answer (Exam Writing Format)
**Lifecycle & Trap Handler Pseudocode (Non-Blocking CPU during Disk I/O)**:
```c
void OS_Trap_Handler(VirtualAddress va, Process *cur_proc) {
    // 1. Hardware saves PC/registers, switches to Kernel Mode (Mode bit = 0)
    VPN = Extract_VPN(va);
    PTE *pte = Lookup_PageTable(cur_proc->page_table, VPN);

    // 2. Check protection / valid allocation
    if (pte == NULL || !pte->allocated || !Has_Permission(pte, access_type)) {
        Terminate_Process(cur_proc, SIGSEGV); // Segmentation Fault
        Schedule_Next_Ready_Process();
        return;
    }

    // 3. Page is already in RAM (only a TLB Miss occurred)
    if (pte->present == 1) {
        TLB_Insert(VPN, pte->pfn, pte->prot_bits);
        Return_From_Trap_And_Retry(cur_proc); // Re-execute faulting instruction
        return;
    }

    // 4. Page Fault (pte->present == 0): Page is on swap/disk
    // 4a. Find a free frame (or run evictPages() if memory is full; flush if dirty)
    PFN free_frame = Get_Free_Frame_Or_Evict();

    // 4b. Transition cur_proc to BLOCKED and issue asynchronous DMA disk read
    //     CPU does NOT sit idle while the disk spins!
    cur_proc->state = BLOCKED;
    Save_Context(cur_proc->pcb);
    Issue_Async_Disk_DMA_Read(pte->disk_addr, free_frame, callback_info=(cur_proc, VPN, free_frame));

    // 4c. Immediately context-switch CPU to another READY process
    Process *next_proc = Scheduler_Pick_Ready();
    Context_Switch(next_proc); // CPU stays productively busy!
}

void Disk_Interrupt_Handler(Process *proc, VPN vpn, PFN frame) {
    // 5. Runs milliseconds later when DMA finishes loading page from disk
    PTE *pte = Lookup_PageTable(proc->page_table, vpn);
    pte->pfn = frame;
    pte->present = 1;
    pte->reference_bit = 1;
    pte->dirty_bit = 0;

    // 6. Move waiting process from BLOCKED -> READY queue
    proc->state = READY;
    Enqueue_Ready_Queue(proc);
}
// 7. When scheduler later dispatches 'proc', it retries the faulting instruction:
//    Now TLB_Miss_Handler finds pte->present == 1, loads TLB, and instruction succeeds!
```

#### IN DETAIL
Handling a combined **TLB-Miss and Page-Fault** on architectures with software-managed TLBs exposes the boundary between **lightweight synchronous refill** and **asynchronous disk-blocking context switches**:
- **Fast-Path Software TLB Miss**: On a TLB miss whose target PTE has `Present = 1` in the in-memory page table, the trap handler simply inserts `(VPN -> PFN)` into a hardware TLB slot and immediately executes `ReturnFromTrap` (`iret`) without context switching or scheduling (~tens of nanoseconds).
- **Slow-Path Page Fault (`Present = 0`)**: Because disk reads take milliseconds ($10^6\times$ longer than RAM accesses), the fault handler initiates asynchronous DMA read (`disk_read`), clears `Present` until completion, transitions the faulting thread to **`Blocked`** state, and invokes the scheduler to run other ready processes—resuming only when the disk ISR marks `Present = 1`, inserts the PTE/TLB mapping, moves the process to `Ready`, and re-executes the identical faulting instruction.

---

### AgentOS — Question 4(b) [5 Points] — Clock / Second-Chance Page Replacement (evictPages) [Frequency: 1]

> **Question**: The members of the AgentOS team taking care of the Persistence aspect, have completed the development of the virtual memory subsystem, but they are facing performance issues when memory becomes full. They need you to design the Page Replacement Policy by considering two constraints:
> i) The hardware does not support a timestamp register for every page access;
> ii) The hardware provides a single Reference Bit per page table entry.
> Outline the algorithm for the `evictPages()` routine that decides which pages to evict. Does your algorithm degenerate to FIFO under specific circumstances? (5 points)

#### pre-requisite
- **Approximating LRU without Timestamps**: True LRU requires updating a timestamp/stack on *every* memory reference (too expensive in hardware). With a single **Reference Bit (`R`)** set to `1` by hardware whenever a page is read/written, the OS can implement the **Clock (Second-Chance) Algorithm**.

#### Answer (Exam Writing Format)
- **Algorithm Choice**: **Second-Chance (Clock) Page Replacement Algorithm**.
- **Mechanism**: Resident physical frames are arranged in a circular FIFO list with a `clock_hand` pointer scanning circularly.
- **Pseudocode (`evictPages()`)**:
```c
// Frame list of size NUM_FRAMES arranged circularly; clock_hand persists across calls
PFN evictPages() {
    while (true) {
        PTE *pte = Get_Resident_PTE(clock_hand);

        if (pte->reference_bit == 0) {
            // Victim found: not referenced since the clock hand last swept past
            PFN victim = clock_hand;
            clock_hand = (clock_hand + 1) % NUM_FRAMES;

            if (pte->dirty_bit == 1) {
                Write_Page_To_Disk(victim, pte->disk_addr);
            }
            pte->present = 0;
            return victim;
        } else {
            // Give a "second chance": clear Reference bit and advance hand
            pte->reference_bit = 0;
            clock_hand = (clock_hand + 1) % NUM_FRAMES;
        }
    }
}
```
- **Does It Degenerate to FIFO?**:
  - **Yes!** Under conditions of **extremely heavy locality / high working-set activity where every resident page has `reference_bit == 1`** when `evictPages()` is called:
    - The clock hand sweeps a full circle clearing all `reference_bit`s from `1` to `0`, and returns to the **exact starting frame (the oldest loaded/scanned frame)**—evicting pages in **strict FIFO order**! *(Similarly, if all pages have `R == 0`, it also picks the oldest hand position directly like FIFO.)*

#### IN DETAIL
The **Second-Chance (Clock) Algorithm** approximates LRU cleanly in kernel software with $O(1)$ per-hit hardware cost using a single MMU **Reference Bit (`R`)**:
- **Circular Scan Invariant**: Instead of paying expensive heap/doubly-linked-list pointer updates on every CPU memory access (impossible at nanosecond cache speeds), MMU hardware simply flips `R = 1` on read/write. Upon page fault, the Clock hand inspects frames around a circular buffer: encountering `R = 1` clears `R = 0` (granting one full sweep chance) and advances; encountering `R = 0` selects the victim immediately.
- **Degeneration to Pure FIFO**: When the active working set is accessed densely enough within one clock sweep that *every* frame holds `R = 1`, the hand clears all $M$ bits through one full revolution and evicts whichever frame stood initially at the oldest pointer position—reducing identically to FIFO.

---

### AgentOS — Question 5 [3 Points] — TCP Three-Way Handshake Sequence & Ack Numbers [Frequency: 1]

> **Question**: As part of diagnosing intermittent network failures in AgentOS, the networking team asks you to verify whether the system is correctly following the TCP three-way handshake. A TCP client module $P$ initiates a connection to a remote TCP service $Q$. Let $N_p$ denote the sequence number in the `SYN` sent from $P$ to $Q$, and let $N_q$ denote the acknowledgement number in the `SYN-ACK` sent from $Q$ back to $P$. Based on standard TCP behaviour, which of the following statements is/are CORRECT and why?
> - a. The sequence number $N_p$ is randomly chosen
> - b. The sequence number is set to 0
> - c. The acknowledgement number is $N_q = N_p$
> - d. The acknowledgement number is $N_q = N_p + 1$ *(right edge of scan crops `1` after `+`)*

#### Answer (Exam Writing Format)
- **Correct Options**: **a** and **d**.
  - **a. ($N_p$ is randomly/pseudo-randomly chosen)**: TCP selects a random/clock-based Initial Sequence Number (ISN) rather than `0` to prevent confusion with delayed duplicate packets from old connections and to defend against TCP sequence-prediction/spoofing attacks.
  - **d. ($N_q = N_p + 1$)**: Because the `SYN` flag itself consumes **1 byte of sequence space**, $Q$ acknowledges receipt of $P$'s `SYN` by setting the acknowledgment number to the *next expected byte*: $N_q = N_p + 1$.

#### IN DETAIL
Diagnosing **TCP Three-Way Handshake** trace parameters verifies protection against **stale delayed segments** and **blind spoofing attacks**:
- **Randomized Initial Sequence Numbers (`ISN`)**: Choosing fixed `ISN = 0` invites both **cross-incarnation wrap/replay collisions** (old delayed segments from a prior closed port reuse matching sequence space of a fresh session) and trivial **blind sequence-prediction / session-hijacking attacks** where off-path attackers forge `SYN/ACK/RST` segments. Standard RFC 6528 hashes clock ticks with secret per-connection cryptographic keys (`N_p` pseudo-random).
- **Exact Byte-Consumption Accounting (`N_q = N_p + 1`)**: Although a pure **`SYN`** (and `FIN`) segment carries zero application payload bytes, TCP intentionally assigns **1 sequence-number coordinate** to the `SYN` flag bit itself so the peer can unambiguously acknowledge receipt of `SYN` via `ack = N_p + 1` (preventing ambiguous zero-progress `SYN` retransmissions).

---

### AgentOS — Question 6 [10 Points] — Cold-Start End-to-End Network Stack (DHCP, DNS, TCP, ARP) [Frequency: 1]

> **Question**: A node running AgentOS connects to a network without having any IP address assigned. Immediately after joining, a user process attempts to send an HTTP request to `http://anotherdomain.com` (server IP: `179.20.10.6`). The AgentOS team requires your help in understanding the end-to-end sequence of steps from the moment the node connects until the HTTP request is ready to leave the link layer and move toward the next hop.
> **a.** Describe what the Operating System must provide to enable network communication. (2 points)
> **b.** Explain the mechanisms the node uses to acquire its own IP address and to resolve the destination server's IP address. (4 points)
> **c.** Describe the activities at the Application, Transport, and Link layers that prepare the HTTP request for transmission toward the next hop (excluding network-layer routing details). (4 points)

#### pre-requisite
- **Bootstrap Sequence**: A freshly joined node needs (1) OS network stack + NIC drivers, (2) **DHCP** to obtain its local IP/gateway/DNS, (3) **DNS** to map hostname $\to$ IP, and (4) **ARP** to resolve the next-hop gateway MAC address before sending an HTTP/TCP/Ethernet frame.

#### Answer (Exam Writing Format)
##### **(a) What the OS Must Provide (2 Points)**
1. **Network Interface Card (NIC) Hardware Drivers**: To interact with physical Ethernet/Wi-Fi hardware and transmit/receive raw frames via interrupts/DMA.
2. **Kernel Protocol Stack & Socket API**: Implementation of Link (ARP/Ethernet), Network (IP/ICMP), and Transport (TCP/UDP) protocols in the kernel, exposed to user processes via BSD **socket syscalls** (`socket`, `connect`, `send`, `recv`) and kernel socket buffers.

##### **(b) Mechanisms to Acquire Own IP & Resolve Destination IP (4 Points)**
1. **Acquiring Own IP Address — DHCP (DORA)**:
   - Node broadcasts `DHCPDISCOVER` $\to$ DHCP server replies with `DHCPOFFER` $\to$ Node broadcasts `DHCPREQUEST` $\to$ Server sends `DHCPACK`.
   - Along with its own **IP address + Subnet Mask**, the node simultaneously receives the **Default Gateway Router's IP** and **Local DNS Server's IP**.
2. **Resolving Destination Server's IP (`anotherdomain.com`) — DNS**:
   - The node sends a **DNS Query** (over UDP port 53) to the configured local DNS resolver (checking local hosts/DNS cache first).
   - If uncached, the resolver queries Root $\to$ TLD (`.com`) $\to$ Authoritative DNS servers recursively/iteratively and returns `179.20.10.6` to the node.

##### **(c) Activities at Application, Transport, and Link Layers (4 Points)**
1. **Application Layer**:
   - Browser/client formats the plaintext **HTTP GET request** (`GET / HTTP/1.1\r\nHost: anotherdomain.com\r\n...`), opens a TCP socket (`socket()`, `connect(179.20.10.6, 80)`), and passes the HTTP message to the OS via `send()`.
2. **Transport Layer (TCP)**:
   - Assigns an ephemeral **Source Port** and Destination Port **`80`** (HTTP), performs the **TCP Three-Way Handshake** (`SYN`, `SYN-ACK`, `ACK`) with `179.20.10.6`, segments the HTTP payload, computes the 16-bit **TCP Checksum**, sets sequence/window fields, and passes segments to IP.
3. **Link Layer (Ethernet + ARP)**:
   - Because `179.20.10.6` is outside the local subnet, IP determines the next hop is the **Default Gateway Router**.
   - Link layer checks its **ARP cache** for the Default Gateway's IP (broadcasting an **ARP Request** if missing to learn the Gateway's MAC address).
   - Encapsulates the IP packet inside an **Ethernet Frame** with `Src MAC = Node's NIC MAC`, `Dst MAC = Default Gateway Router's MAC`, appends a **32-bit CRC/FCS trailer**, and pushes bits onto the wire via the NIC driver.

#### IN DETAIL
Tracing a **cold-start web request (`http://anotherdomain.com`)** from an unconfigured NIC integrates the entire OS kernel networking subsystem across Layers 1–7:
- **OS Device & Socket Infrastructure**: Requires NIC DMA ring drivers, a modular TCP/IP state machine, routing/ARP tables, and POSIX socket syscalls (`socket/connect/send/recv`) decoupling application streams from kernel frame assembly.
- **Control-Plane Prerequisite Chain**: First **DHCP (DORA over UDP broadcast)** provisions host identity (`IP`, mask, gateway, DNS server); next **DNS (`UDP:53`)** queries via the default gateway (resolved via **ARP**) map hostname `"anotherdomain.com"` $\to$ IP `"179.20.10.6"`.
- **Encapsulation Data Path**: Application emits HTTP payload $\to$ TCP establishes a **3-Way Handshake** toward `179.20.10.6:80` tagging sequence numbers/checksum $\to$ IP attaches global L3 headers and selects the default-gateway route $\to$ Link layer sets **`Dst MAC = Default Gateway Router MAC`** (via ARP) + CRC-32 FCS before NIC DMA transmission.

---

### AgentOS — Question 7 [3 Points] — Resource Lock Ordering to Prevent Deadlock [Frequency: 1]

> **Question**: The AgentOS team recently encountered an issue related to concurrency in one of its subsystems. Three concurrent processes X, Y, and Z execute different critical code segments that update shared variables. Process X performs `Wait(a)`, `Wait(b)`, and `Wait(c)` before entering its critical section; process Y performs `Wait(b)`, `Wait(c)`, and `Wait(d)`; and process Z performs `Wait(c)`, `Wait(d)`, and `Wait(a)` before entering its critical section. After completing their respective critical sections, each process executes `Signal()` on the same three semaphores in the same order. All semaphores are binary semaphores initialized to 1. Which one of the following represents a deadlock-free order of invoking the `Wait()` operations?
> - a. `X: Wait(a) Wait(b) Wait(c)    Y: Wait(b) Wait(c) Wait(d)    Z: Wait(c) Wait(d) Wait(a)`
> - b. `X: Wait(a) Wait(b) Wait(c)    Y: Wait(b) Wait(c) Wait(d)    Z: Wait(a) Wait(c) Wait(d)`
> - c. `X: Wait(b) Wait(a) Wait(c)    Y: Wait(c) Wait(b) Wait(d)    Z: Wait(a) Wait(c) Wait(d)`
> - d. `X: Wait(a) Wait(b) Wait(c)    Y: Wait(c) Wait(b) Wait(d)    Z: Wait(c) Wait(d) Wait(a)`

#### pre-requisite
- **Resource Ordering (Breaking Circular Wait)**: If all processes acquire resources/locks in a **strict global total order** (e.g., $a < b < c < d$), a cycle in the Wait-For Graph is mathematically impossible $\implies$ **deadlock-free**!

#### Answer (Exam Writing Format)
- **Correct Option**: **b**.
  - `X`: `Wait(a) Wait(b) Wait(c)` ($a < b < c$)
  - `Y`: `Wait(b) Wait(c) Wait(d)` ($b < c < d$)
  - `Z`: `Wait(a) Wait(c) Wait(d)` ($a < c < d$)
- **Justification**: Option **b** enforces a consistent **total global ordering $a < b < c < d$** across all three processes, making **Circular Wait impossible** (guaranteed deadlock-free). All other options contain cycles:
  - In **a**: $X$ holds $b \to$ wants $c$; $Y$ holds $c \to$ wants $d$; $Z$ holds $d \to$ wants $a$; $X$ holds $a$ (cycle $a \to b \to c \to d \to a$).
  - In **c**: $X$ acquires $b \to a$ while $Z$ acquires $a \to c$ and $Y$ acquires $c \to b$ (cycle $b \to a \to c \to b$).
  - In **d**: $Y$ acquires $c \to b$, $Z$ acquires $c \to d \to a$, $X$ acquires $a \to b \to c$ (cycle $a \to c \to d \to a$).

#### IN DETAIL
Enforcing a **strict total resource acquisition ordering** ($R_1 < R_2 < \dots < R_n$) eliminates deadlock structurally by making **Circular Wait** mathematically impossible:
- **Graph-Theoretic Proof of Acyclicity**: Define a Resource-Allocation Wait-For Graph where a directed edge $P_i \to P_j$ exists only if $P_i$ currently holds some lock $r$ and blocks requesting lock $s$ held by $P_j$. Under strict lock ordering, every process requesting $s$ while holding $r$ must obey $r < s$. Along any wait cycle $P_0 \to P_1 \to \dots \to P_k \to P_0$, the minimum lock held along the chain would satisfy $r_0 < r_1 < \dots < r_k < r_0$—a mathematical contradiction!
- **Design Constraint**: Because processes acquire resources strictly in monotonically ascending rank ($a \to b \to c$, etc.), no runtime Banker's check or wait-for graph probe is needed; deadlock is prevented statically at compile/design time.

---

### AgentOS — Question 8 [10 Points] — SL-FS Inode/Block Capacity, Max File Size & NAS RAID [Frequency: 1]

> **Question**: The AgentOS media engineering team is designing a custom file system, **SL-FS**, optimized for a high-throughput video streaming server. The underlying storage cluster consists of **$16\text{ TB}$** of raw disk space. The SL-FS design specifications are as follows:
> i) Block size: $8\text{ KB}$ ($8192$);
> ii) Pointer size: $64\text{-bit}$ ($8\text{ bytes}$) addresses are used for all block pointers;
> iii) Inode size: $256\text{ bytes}$;
> iv) Inode structure: To optimize for large media files, the inode contains **12 direct pointers, 1 single-indirect pointer, and 1 double-indirect pointer**.
> Further, for metadata allocation, the system reserves **exactly $2\%$ of the total disk capacity** specifically for the Inode Table (where inodes are stored). The remaining **$98\%$** is for data blocks.
>
> **a.** Given the allocation policy above, what is the maximum number of files that can be stored in this file system? *Scenario*: If the users strictly store small text logs of size $1\text{ KB}$ each, will the system run out of Data Blocks or Inodes first? Justify your answer mathematically. (3 points)
> **b.** Calculate the maximum file size (in GB or TB) that a single file can reach in SL-FS. *(Show your derivation for Direct, Single-Indirect, and Double-Indirect capacities separately. Assume $1\text{ KB} = 2^{10}\text{ bytes}$, $1\text{ MB} = 2^{20}\text{ bytes}$, etc.)* (3 points)
> **c.** The team decides to move the file system onto a Network Attached Storage (NAS) array consisting of **8 disks of $4\text{ TB}$ each**. They are debating between using **RAID 1** and **RAID 5**. What is your suggestion, considering the need for high storage efficiency and the fact that the system is expected to have a larger number of random small writes? Reason by calculating the usable capacity under both RAID setups and demonstrating performance under write/read workloads. (4 points)

#### pre-requisite
- **Inode & File Size Math**:
  - Total Inodes = $\lfloor \frac{\text{Inode Table Space}}{\text{Inode Size}} \rfloor$.
  - Pointers per Indirect Block = $\frac{\text{Block Size}}{\text{Pointer Size}}$.
  - Max File Size = $(\text{Direct} + \text{Single} \times P + \text{Double} \times P^2) \times \text{Block Size}$.

#### Answer (Exam Writing Format)
##### **(a) Maximum Files & Exhaustion on 1 KB Logs (3 Points)**
- **Total Disk Capacity**: $16\text{ TB} = 16 \times 2^{40}\text{ bytes} = 2^{44}\text{ bytes}$.
- **Inode Table Space ($2\%$)**:
  $$0.02 \times 16 \times 2^{40}\text{ bytes} = 0.32 \times 2^{40}\text{ bytes} = 327.68\text{ GiB} = 351,843,720,888.32\text{ bytes}$$
  $$\text{Maximum Inodes (Max Files)} = \left\lfloor \frac{0.02 \times 2^{44}}{256} \right\rfloor = \lfloor 0.02 \times 2^{36} \rfloor = \mathbf{1,374,389,534\text{ files}}\; (\approx 1.374 \times 10^9)$$
- **Data Block Space ($98\%$)**:
  - Block size = $8\text{ KB} = 8192 = 2^{13}\text{ bytes}$.
  $$\text{Total Data Blocks} = \left\lfloor \frac{0.98 \times 2^{44}}{2^{13}} \right\rfloor = \lfloor 0.98 \times 2^{31} \rfloor = \mathbf{2,104,533,975\text{ blocks}}\; (\approx 2.105 \times 10^9)$$
- **1 KB Small Text Logs Scenario**:
  - Because block allocation granularity is $8\text{ KB}$, each $1\text{ KB}$ file occupies **1 inode** and **1 full $8\text{ KB}$ data block** ($7\text{ KB}$ internal fragmentation).
  - Storing 1 file consumes 1 inode and 1 data block (`1:1` ratio).
  - Comparing limits: $\mathbf{1.374 \times 10^9\text{ inodes} < 2.105 \times 10^9\text{ data blocks}}$.
  - **Conclusion**: The system will run out of **Inodes first**! (At $1.374 \times 10^9$ files, all inodes are exhausted while $\sim 7.30 \times 10^8$ data blocks remain unused.)

##### **(b) Maximum Single File Size (3 Points)**
- **Pointers per $8\text{ KB}$ indirect block**:
  $$P = \frac{8192\text{ bytes}}{8\text{ bytes/pointer}} = 1024 = 2^{10}\text{ pointers}$$
- **1. Direct Pointers Capacity**:
  $$12 \times 8\text{ KB} = \mathbf{96\text{ KB}}\; (= 98,304\text{ B})$$
- **2. Single-Indirect Pointer Capacity**:
  $$1 \times 1024 \times 8\text{ KB} = 8192\text{ KB} = \mathbf{8\text{ MB}}\; (= 8,388,608\text{ B})$$
- **3. Double-Indirect Pointer Capacity**:
  $$1 \times 1024 \times 1024 \times 8\text{ KB} = 2^{20} \times 8\text{ KB} = \mathbf{8\text{ GB}}\; (= 8,589,934,592\text{ B})$$
- **Total Maximum File Size**:
  $$\text{Max File Size} = 8\text{ GB} + 8\text{ MB} + 96\text{ KB} = 8,598,421,504\text{ bytes} \approx \mathbf{8.0079\text{ GB}}$$

##### **(c) NAS RAID Comparison: RAID 1 vs. RAID 5 on $8 \times 4\text{ TB}$ Disks (4 Points)**
- **Usable Capacity Calculations ($N = 8$, $C = 4\text{ TB}$, Raw = $32\text{ TB}$)**:
  - **RAID 1 (Mirroring)**: $\frac{N}{2} \times 4\text{ TB} = 4 \times 4\text{ TB} = \mathbf{16\text{ TB}}$ (**50% storage efficiency**).
  - **RAID 5 (Distributed Parity)**: $(N - 1) \times 4\text{ TB} = 7 \times 4\text{ TB} = \mathbf{28\text{ TB}}$ (**87.5% storage efficiency**).
- **Performance Under Workloads**:
  - **Reads**: Both provide high parallel read throughput across all 8 disks.
  - **Random Small Writes**:
    - **RAID 5** suffers the **Small-Write Penalty**: every random small write requires **4 disk I/Os** (Read old data, Read old parity, Write new data, Write new parity) to maintain XOR parity—causing severe throughput degradation under heavy random small writes.
    - **RAID 1** writes directly to the 2 mirrored copies (**2 parallel writes, zero read-modify-write**), delivering superior random small-write IOPS.
- **Suggestion / Trade-Off Recommendation**:
  - Because the team faces **two conflicting goals**:
    - Recommend **RAID 1** (or RAID 10) if **random small-write IOPS performance** is the primary bottleneck, accepting 50% capacity (16 TB).
    - Recommend **RAID 5** *only* if **87.5% storage efficiency (28 TB)** is strictly mandatory and write throughput can be absorbed via NVRAM/SSD write-back caching (which coalesces small random writes into full-stripe writes).

#### IN DETAIL
Evaluating `SL-FS` highlights three critical file-system sizing and redundancy tradeoffs across disk capacity, workload block granularity, and parity overhead:
- **Metadata Starvation on Small Files (Part a)**: Because every file consumes at least one discrete **Inode (`128 B`)**, reserving a fixed metadata budget ($2\%$ of $16\text{ TB} \approx 2^{31}$ potential inodes) still easily hits artificial **inode-table exhaustion** if workloads generate billions of tiny $1\text{ KB}$ logs (which fill fewer data blocks proportionally while consuming 1 inode each)—blocking new `open(O_CREAT)` calls despite abundant free data gigabytes!
- **Multi-Level Indexing Math (Part b)** & **RAID Tradeoff (Part c)**: Expanding address pointers inside $4\text{ KB}$ blocks ($1024$ $4\text{ B}$ pointers/block) grows max file size exponentially via $D + P + P^2 + P^3$, while `8 × 4 TB` NAS designs force a fundamental choice between **RAID 1** ($16\text{ TB}$ usable, optimal random small-write IOPS via 2 mirrored writes) and **RAID 5** ($28\text{ TB}$ usable, degraded small random writes due to the 4-I/O Read-Modify-Write parity penalty).

---

### AgentOS — Question 9 [15 Points] — Core Router Routing-Table RW Locks & Intra/Inter-AS Routing [Frequency: 1]

> **Question**: The AgentOS team is in the process of developing a firmware for a high-end Core Router placed at the edge of an Autonomous System (AS). This router maintains a massive `Global_Routing_Table` in shared memory. The router has two critical tasks running as concurrent threads:
> i) Hundreds of threads representing incoming packets. They need to read the `Global_Routing_Table` to find the "Next Hop" interface. Speed is critical; multiple readers must access the table simultaneously;
> ii) A background process that receives `BGP_UPDATE` messages from other ISPs. When a route changes (e.g., a cable cut in some location), this daemon must acquire exclusive access to modify the table to prevent readers from routing packets into a black hole.
>
> **a.** Write pseudocode `Lookup_Route(Dest_IP)` and `Update_BGP_Route(New_Path)` using Semaphores and Mutexes. (8 points)
> **b.** The router described above is in an Autonomous System (AS 300). Considering a node with a public IP address `179.81.26.18` in this autonomous system wants to send data to another node in another autonomous system (AS 700) with an IP address `17.65.21.89`. Given this scenario:
>    - **i.** Explain how such a communication can work using different protocols. (4 points)
>    - **ii.** Do we need different protocols like OSPF and BGP, or will one protocol suffice for the entire internet routing? Explain. (3 points)

#### pre-requisite
- **Fair / Writer-Safe Readers-Writers Synchronization**: Prevents a continuous stream of high-speed packet-forwarding reader threads from starving the critical `BGP_UPDATE` writer thread (which would cause stale/black-hole routing!).
- **Intra-AS vs. Inter-AS Routing**:
  - **IGP (e.g., OSPF / RIP)**: Interior Gateway Protocols optimize shortest paths (hop count / link cost) *inside* a single Autonomous System.
  - **EGP (BGP)**: Exterior Gateway Protocol routes across *different* Autonomous Systems using reachability + **business policies / AS-paths**.

#### Answer (Exam Writing Format)
##### **(a) Pseudocode: `Lookup_Route` and `Update_BGP_Route` (8 Points)**
**Assumptions**: `order_mutex = 1` (semaphore/mutex enforcing arrival order so BGP updates never starve behind endless packet bursts), `rw_mutex = 1` (exclusive writer semaphore), `r_mutex = 1` (protects `reader_count`).

```c
semaphore order_mutex = 1; // Prevents BGP writer starvation by queuing new readers behind waiting writer
semaphore rw_mutex = 1;    // Mutual exclusion between writers and reader groups
mutex r_mutex = 1;         // Protects read_count
int read_count = 0;

NextHop Lookup_Route(IP Dest_IP) {
    wait(order_mutex);         // Wait behind any pending BGP update
    lock(r_mutex);
    read_count++;
    if (read_count == 1) {
        wait(rw_mutex);        // First reader blocks BGP writer
    }
    unlock(r_mutex);
    signal(order_mutex);       // Allow other readers immediately while no writer waits

    /* --- Concurrent Read Critical Section --- */
    NextHop hop = Search_Table(Global_Routing_Table, Dest_IP);

    lock(r_mutex);
    read_count--;
    if (read_count == 0) {
        signal(rw_mutex);      // Last active reader lets BGP writer proceed
    }
    unlock(r_mutex);
    return hop;
}

void Update_BGP_Route(Route New_Path) {
    wait(order_mutex);         // Stop new incoming packet lookups from bypassing the update
    wait(rw_mutex);            // Wait for currently active lookups to finish
    signal(order_mutex);

    /* --- Exclusive Write Critical Section --- */
    Modify_Table(Global_Routing_Table, New_Path);

    signal(rw_mutex);          // Resume packet forwarding with updated table
}
```

##### **(b)(i) How `179.81.26.18` (AS 300) Communicates with `17.65.21.89` (AS 700) (4 Points)**
1. **Inter-AS Routing (BGP) — Discovering External Path & Border Router**:
   - Border routers of AS 700 advertise reachability to prefix `17.65.0.0/16` (or `17.65.21.0/24`) via **eBGP** across peering links to neighbor ASes until reaching AS 300's border router.
   - Inside AS 300, border routers propagate this external BGP route via **iBGP** to all internal AS 300 routers, identifying *which AS 300 border (egress) router* leads toward AS 700.
2. **Intra-AS Routing (OSPF) — Reaching the Border Router Inside AS 300**:
   - The sender's source subnet sends packets to its local router inside AS 300. Internal AS 300 routers run an IGP like **OSPF** (link-state Dijkstra shortest path) to forward the packet hop-by-hop across AS 300 to the selected AS 300 egress border router.
3. **Across ISP Backbone to Destination (AS 300 $\to$ AS 700)**:
   - The AS 300 border router forwards the packet across inter-AS peer/transit links (using BGP paths) until it reaches AS 700's ingress border router.
   - Inside AS 700, AS 700's **IGP (e.g., OSPF)** routes the packet internally to the host `17.65.21.89`.

##### **(b)(ii) Why We Need Different Protocols (OSPF vs. BGP) (3 Points)**
One single routing protocol **cannot** suffice for the entire Internet due to three fundamental requirements:
1. **Scale**: Flooding OSPF link-state advertisements or flat distance-vector tables for every router/link on Earth across billions of hosts would explode router memory and control-plane bandwidth. BGP aggregates prefixes and routes at the **AS level** (hierarchical routing).
2. **Policy vs. Performance**: Inside one organization (OSPF), routing optimizes pure **performance** (shortest/lowest-cost path). Across independent competing ISPs (**BGP**), routing is governed by **economic & political policies** (e.g., customer vs. peer vs. provider agreements, avoiding transit through unapproved networks/countries)—not just shortest path!
3. **Autonomy & Privacy**: Competing ISPs refuse to expose their internal router topologies, link capacities, and link failures to outsiders; BGP hides internal AS structure behind border summaries.

#### IN DETAIL
This architecture unites two distinct concurrency and routing domains in modern edge routers:
- **Writer-Fair Readers–Writers Synchronization (Part a)**: Packet-forwarding threads perform lockless or concurrent shared reads (`read_count > 1`) on `Global_Routing_Table` via `read_mutex` while acquiring `rw_mutex` only when the first reader enters and releasing it when the last exits. Introducing a FIFO arrival gate (`order_mutex`) ensures an arriving `BGP_UPDATE` writer blocks new incoming readers once queued, preventing writer starvation and stale black-hole forwarding loops under continuous traffic bursts.
- **Hierarchical Internet Routing (Part b)**: **OSPF (Intra-AS IGP)** distributes complete link-state topologies inside AS 300 so internal routers compute shortest-delay/cost paths (`Dijkstra`). **BGP (Inter-AS EGP)** interconnects autonomous systems (`AS 300` to `AS 700`) via policy-driven Path-Vector routing—scaling globally by aggregating CIDR prefixes and honoring commercial peering/transit policies rather than exposing raw internal topologies or metrics globally.

---

### AgentOS — Question 10 (Bonus) [4 Points] — Parameterized RR Quantum & Asymmetric MLFQ Design [Frequency: 1]

> **Question**: Many CPU-scheduling algorithms are parameterized. For example, the RR algorithm requires a parameter to indicate the time slice. Multilevel feedback queues require parameters to define the number of queues, the scheduling algorithms for each queue, the criteria used to move processes between queues, and so on. These algorithms are thus really sets of algorithms. What (if any) relation holds between the following pairs of algorithm sets?
> - a. Priority and SJF
> - b. Multilevel feedback queues and FCFS
> - c. Priority and FCFS
> - d. RR and SJF

#### pre-requisite
- **Algorithm Set Inclusion**: Set $A$ **includes** (is a generalization of) Set $B$ ($B \subset A$) if choosing specific parameter values for $A$ produces every behavior of $B$.

#### Answer (Exam Writing Format)
- **a. Priority and SJF**: **$\text{SJF} \subset \text{Priority}$** — SJF is a **special case (strict subset)** of Priority scheduling where a process's priority is set inversely proportional to its next predicted CPU burst length ($\text{priority} \propto 1 / \tau_i$).
- **b. Multilevel Feedback Queues (MLFQ) and FCFS**: **$\text{FCFS} \subset \text{MLFQ}$** — FCFS is a **special case (strict subset)** of MLFQ configured with **1 single queue** whose scheduling policy is FCFS (or infinite time quantum with zero demotion).
- **c. Priority and FCFS**: **$\text{FCFS} \subset \text{Priority}$** — FCFS is a **special case (strict subset)** of Priority scheduling where priority is assigned inversely to process arrival time (earliest arrival time = highest priority, non-preemptive).
- **d. RR and SJF**: **Disjoint / Neither includes the other** (independent sets in general). RR uses a time quantum ignoring job lengths, while SJF orders by burst length without time-slicing. *(Note: On trivial degenerate cases—such as a single process or all equal bursts with $q \ge \text{burst}$—both can yield the same schedule, but in general neither algorithm family can simulate the other.)*

#### IN DETAIL
Classifying scheduling algorithms by **reduction / subset equivalence** reveals which frameworks subsume others purely through parameter configuration:
- **Universal Frameworks**: **Priority Scheduling** strictly subsumes **FCFS** (assign priority = $-\text{arrival\_time}$) and **SJF** (assign priority = $1 / \text{next\_CPU\_burst}$). Similarly, **MLFQ** with $k=1$ single non-preemptive FIFO queue reduces identically to **FCFS**.
- **Incompatible Primitives (RR vs. SJF)**: Round Robin is strictly preemptive and purely **burst-agnostic** (time-slicing every $q$ ms regardless of remaining CPU need), whereas non-preemptive SJF uses **a priori burst estimates** without any timer-quantum slicing—making neither a valid parameterization of the other across arbitrary job streams.

---
