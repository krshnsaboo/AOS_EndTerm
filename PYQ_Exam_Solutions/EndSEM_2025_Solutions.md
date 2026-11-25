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
- [Question 9 [15 Points] — Multi-Stage Bootloader Firmware Architecture](#agentos-question-9-15-points-multi-stage-bootloader-firmware-architecture-frequency-1) — Frequency: 1
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
Calling a semaphore `wait()` while holding a mutex lock freezes the entire system because semaphore `wait()` has no idea you're holding a lock! Monitor `wait(cv)` was explicitly designed to **atomically release the monitor lock AND put the thread to sleep** in one step, and remember: a `signal(cv)` inside a monitor vanishes if nobody is waiting right then, whereas a semaphore `signal()` sticks around in its integer counter!

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
Be aware of this classic course nuance!
- In **OSN Endsem 2022**, the key evaluated **blocking OS semaphores** vs. **monitors**: since kernel semaphores already sleep without spinning, monitors have the same blocking behavior (**FALSE**).
- In **AOS Endsem 2025 Q1.2/Q1.4**, the comparison emphasized how monitors stop developers from writing ad-hoc user-level polling/spin-loops around low-level semaphores (**TRUE**). Mentioning that *kernel-level semaphores already block via wait queues, whereas monitors reduce application-level polling bugs* covers both angles clearly!

---

### Question 1.3 [1 Mark] [Frequency: 1]

> **Question**: *"Deadlock avoidance approach improves resource utilization over deadlock prevention approach based on the violation of no pre-emption condition."*

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: Deadlock prevention by violating "No Preemption" forces the OS to forcibly take away held resources whenever a process's additional request cannot be met, causing aborted work, costly rollbacks, and wasted restarts. In contrast, deadlock avoidance (e.g., Banker's Algorithm) dynamically verifies safe states and lets processes retain their allocated resources and complete uninterruptedly whenever safe, sustaining significantly higher hardware and CPU utilization.

#### IN DETAIL
Violating "No Preemption" is destructive: if Process A holds Resource 1 and Resource 2 fails, the OS steals Resource 1 and throws away Process A's progress! Deadlock avoidance checks in advance whether granting Resource 2 stays safe—if yes, nobody loses work or rolls back, keeping resources productively utilized!

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
Look closely at the exact terms from `MultithreadedProgramming-ch4-2025.ppt` vs `Lec7-processConcept_ch3-2025.ppt`:
- **Thread pools** limit *how many threads* a multithreaded server spawns (so 10,000 requests don't crash RAM with 10,000 threads).
- **MPL (Multiprogramming Level)** is the number of *processes* in main memory controlled by the OS **Long-Term / Medium-Term Scheduler**! Keep that distinction sharp on exams!

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
Think of **Processor Affinity** as keeping a chef at their own prepped kitchen station (all knives and ingredients already warm in their local cache!). **Load Balancing** taps that chef on the shoulder and yells *"Switch to Station #4 because it's empty!"*—evening out the workload across cores, at the exact cost of leaving behind all their warm cache contents!

---

### Question 1.7 [1 Mark] [Frequency: 3]

> **Question**: *"Global frame allocation algorithm gives better throughput over local frame allocation algorithm."*

#### pre-requisite
- **Local vs. Global Replacement**: In **local** allocation, a page-faulting process can only replace one of its *own* fixed frames; in **global** allocation, it can steal an unused/free frame from *any* process in the global pool.

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: Global frame allocation allows a process whose working set is actively growing to claim free or infrequently used frames from other processes rather than thrashing inside a rigid fixed partition. By dynamically adjusting each process's frame count to match its current locality needs, global allocation reduces total page faults, improves memory utilization, and yields higher overall system throughput than local allocation.

#### IN DETAIL
Imagine two cooks sharing a 10-burner stove:
- **Local allocation** draws a permanent tape line down the middle: *"Cook A gets 5 burners forever, Cook B gets 5 burners forever."* Even if Cook B is just simmering 1 small pot and has 4 burners idle while Cook A desperately needs 7 burners, Cook A isn't allowed to touch Cook B's 4 empty burners!
- **Global allocation** lets Cook A borrow idle burners dynamically whenever Cook A's workload spikes—keeping total kitchen productivity (**system throughput**) higher! *(Catch: one greedy runaway process can steal too many frames from a neighbor unless minimum frame limits / working-set controls are maintained.)*

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
While both bridges and switches operate at **Layer 2** using backward-learning MAC tables (`OverviewOfNetworking.pptx` Slide 60–63), replacing a traditional software bridge with a switch makes a big practical difference: switches have dozens of dedicated full-duplex ASIC ports (one computer per port $\implies$ zero collisions and simultaneous multi-port parallel forwarding), whereas bridges historically joined shared-bus LAN segments in software!

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
Wait—when you run `cat /dir1/file.txt`, you think *"I'm only reading!"*, right?
Under the hood, the file system keeps a diary entry (**inode**) for every file and directory recording *"When was this file last accessed (`atime`)?"*
- To open `/dir1/file.txt`, the OS traverses `/` and `/dir1`, and reads `file.txt`'s data blocks.
- Each read updates the **`atime`** timestamp in the corresponding inodes—forcing the OS to write those modified inode blocks back to the disk!
*(Fun real-world note: on busy Linux servers with millions of reads, updating `atime` on every read caused so many extra disk writes that modern Linux introduced `relatime` and `noatime` mount options to skip unnecessary `atime` writes!)*

---

### Question 1.10 [1 Mark] [Frequency: 3]

> **Question**: *"As compared to CPU-bound process, OS gives higher priority for I/O bound process."*

#### pre-requisite
- **Scheduling Principle**: I/O-bound processes execute short CPU bursts before submitting slow I/O requests; prioritizing them overlaps I/O and CPU execution.

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: I/O-bound processes execute very short CPU bursts before blocking for lengthy I/O operations. By assigning higher priority to I/O-bound processes, the OS dispatches them quickly so they can initiate I/O immediately and keep I/O devices busy, while CPU-bound processes utilize the remaining CPU time while I/O is in progress. This avoids the convoy effect and maximizes overall CPU utilization, I/O utilization, and interactive response time.

#### IN DETAIL
In general-purpose OSes (and explicitly in Prof. Krishna Reddy's scheduling lectures and UNIX), prioritizing I/O-bound jobs is a win-win:
- Suppose Job A needs **1 ms CPU then 99 ms disk**, and Job B needs **100 ms CPU**.
- If **A runs first**: at $t=1\text{ ms}$, A starts its disk operation, and from $t=1\text{ ms}$ to $t=100\text{ ms}$ **both the CPU (running B) and the disk (serving A) work in parallel at ~100% utilization!**
- If **B ran first** (FCFS): the disk sits 100% idle for the first 100 ms, wasting hardware throughput and delaying A's response by 100×!
*(And aging/priority decay ensures B never starves.)*

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
Compare **AOS Quiz 2 Q4** (*violating No Preemption $\implies$ YES, starvation!*) vs **AOS Endsem Q2.1** (*violating Circular Wait $\implies$ NO!*). Don't mix them up!
Why? Because violating **No Preemption** keeps stealing your tools away and restarting you from zero—so you can starve forever! Violating **Circular Wait** just makes everyone wait in an acyclic line (`Lock 1 -> Lock 2 -> Lock 3`). Whoever already holds `Lock 3` has nothing higher left to wait for, finishes guaranteed, and releases `Lock 3`, letting the next person step forward cleanly without ever rolling back!

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
Think of **Strict Alternation** (`turn = 0` gives turn to P0, `turn = 1` gives turn to P1):
- Mutual exclusion holds 100%.
- **Progress FAILS!** Suppose P0 finishes, sets `turn = 1`, and goes home for vacation (stays in its remainder section). Now P1 enters, finishes, sets `turn = 0`, and immediately needs to enter the critical section again. Because `turn == 0` and P0 isn't even running, **P1 sits locked outside an empty room forever!** That's what violating **Progress** looks like: *empty critical section, ready processes outside, yet nobody can enter!*

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
Think of waiting for a coworker to finish using the office printer:
- If they're just grabbing **1 sheet already printed** (takes 2 seconds $\implies$ **short critical section**), walking all the way back to your desk, sitting down, opening your laptop, and walking back later (**2 context switches**) wastes way more time than standing there for 2 seconds (**spinlock**)!
- If they're printing a **500-page textbook** (takes 20 minutes $\implies$ **long critical section**), standing there staring at the printer (**spinlock**) wastes 20 minutes of your work—go back to your desk and let them ping you when done (**blocking semaphore**)!

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
Think of the NameNode like a busy bank cashier keeping a giant **Account Balance Ledger (`checkpoint / fsimage`)** plus a **Daily Receipt Roll (`journal / edits`)**:
- If nobody ever folds the daily receipts back into the main ledger, after a month the receipt roll grows 10 miles long—so if the bank reboots after a power outage, it takes *hours* to re-read every receipt!
- **CheckpointNode** periodically walks over, copies the ledger + receipt roll onto its own computer, tallies them into a clean brand-new ledger, and hands it back!
- **BackupNode** sits right next to the cashier listening to *every transaction live as it happens*, keeping an **identical live copy in its own RAM**—acting like both a checkpointer AND an instant hot-standby mirror!

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
Imagine 100 runners in a team relay where the team's finish time equals the **last** runner crossing the line! 99 runners finish in 10 minutes, while Runner #47 trips on a bad shoelace (a flaky disk or noisy neighbor VM) and is limping at 1 mph—making the entire team wait an extra 30 minutes!

Instead of waiting helplessly, the MapReduce Master uses **Backup Tasks (Speculative Execution)**: as soon as the job nears the finish line, it assigns a healthy idle runner to **run a duplicate copy of Task #47 in parallel**! Whichever finishes first wins and cancels the other—turning a 40-minute stall into an 11-minute finish!

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
Without a medium-term scheduler, once a process enters RAM via the long-term scheduler, **the ONLY way it ever leaves RAM is by terminating!**
Think what goes wrong on a real system:
- Ten idle browser tabs or suspended jobs hog 95% of RAM forever while active jobs starve and **thrash**!
- Or 5 jobs all enter an intense CPU-crunching phase at once, leaving the disk completely idle, while an I/O job sits locked outside RAM!
The medium-term scheduler solves both by temporarily **suspending and swapping out** selected processes to disk, then swapping them back in cleanly when RAM frees up or the mix needs rebalancing!

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
These three pillars come directly from **Lecture 1 (`Lec1-AOS-Course Overview-2026.pdf`)**:
- **Modularity** splits a 10-million-line OS into clean separate pieces (Scheduler, File System, Network Stack) so debugging shrinks by $K\times$!
- **Abstraction** ensures the File System calls `block_read()` without caring *how* the disk driver spins motors or flashes SSD cells—so failures stay quarantined inside one box!
- **Robustness Principle** (*Postel's Law: "Be conservative in what you send, liberal in what you accept"*): your browser follows strict HTTP standards when sending requests, yet still renders a webpage even if a messy server omits a minor whitespace header instead of crashing!

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
Imagine a malicious or slow postal system where letters can get stuck in a mailbag for **2 minutes** and show up late!
- **Why 2 steps (`Connect` $\to$ `Accepted`) fails catastrophically**: Suppose Alice ordered a pizza (`Connect` + `Charge $50`) an hour ago, got her pizza, and hung up. A delayed duplicate of that envelope shows up at the pizza shop now! In a 2-way handshake, the shop thinks it's a brand-new order, bakes a phantom pizza, and charges Alice $50 again!
- **How the 3-Way Handshake fixes it**: Now, when an envelope `"Connect (ID = x)"` arrives, the shop does NOT bake the pizza yet! Instead, the shop replies: *"Did you really just ask for ID=x? Prove you're alive right now by echoing back my secret fresh number y!"*
  - If it was a **ghost delayed packet**, Alice says *"Huh? I didn't order anything!"* (sends `RST` or ignores it)—so the shop safely drops the ghost request!
  - Only a **live, real Alice** replies *"Yes, confirmed y!"* (Step 3), proving both sides are alive *right now* and synchronized on both directions' sequence numbers!

**And Closing Safely (`Two-Army Problem`)**:
(Two-Army Problem)**: Imagine two Blue armies trying to agree via messengers through enemy territory when to both hang up! Army 1 sends *"Let's hang up!"* Army 2 replies *"Agreed!"*—wait, how does Army 2 know Army 1 heard *"Agreed"*?! If Army 1 sends *"I heard your agreement!"*, how does Army 1 know *that* arrived? **Mathematically unsolvable with 100% perfection!**
  So TCP compromises brilliantly: **close each direction separately (`FIN` / `ACK`)**, retry with a **timeout** if a `FIN` is lost (closing forcefully after $N$ retries), and wait **`2 * MSL` (`TIME_WAIT`)** so any stray packets die off cleanly!

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
Before Virtual Machines, every enterprise app got its own physical server sitting at 10% CPU utilization—massive electricity, space, and money wasted!
Cloud platforms (AWS/Azure/GCP) exploit these four VM superpowers together:
- **Multiplexing**: Packs 20 virtual tenants onto 1 physical server safely.
- **Provisioning**: Clones a fresh server image in seconds when demand spikes.
- **Suspension**: Freezes idle VMs onto disk like hibernating a laptop, freeing RAM for busy neighbors.
- **Live Migration**: Slides a live, running VM seamlessly from an overheating or lightly loaded host onto another host with sub-second pause—letting data centers consolidate workloads and power off empty racks automatically!

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
Look at why the question listed **load times**: normally a clock hand points at the **oldest page in memory (`N`, load time = 3)**.
- In **Enhanced Second Chance**, Pass 1 asks: *"Is there ANY page with `(R=0, M=0)` (not recently used AND clean so we don't even have to pay for a disk write)?"*
- Watch the clock scan in load-time order (`N -> G -> H -> F -> A -> L -> C -> B`): it skips every `(1,1)`, `(1,0)`, and `(0,1)` until it reaches **`B` (`(0,0)`)**!
- Because `B` is in the highest-priority replacement class `(0,0)`, **Page `B` is evicted immediately on Pass 1** (even though `B` was loaded most recently at time 32!), and **no reference bits need to be cleared**!

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
Imagine 4 cashiers and 1 ledger book:
- **RAID 3** forces all 4 cashiers to hold one pen together to write 1 bit each of every single number! Fast if you're printing one massive continuous report, terrible if 4 different customers want quick receipts at once!
- **RAID 4** gives each cashier their own block of receipts, so **4 customers can read different data in parallel**! **Catch**: Whenever *anyone* writes/changes a receipt, they have to walk over to the **single dedicated Parity Desk (Disk 5)** to update the checksum. So if 4 threads try to do random small writes simultaneously, they all crash into a traffic jam at Disk 5!
- **RAID 5** simply removes the dedicated Parity Desk and spreads the parity entries across *all 5 desks* in rotation. Now writes to different stripes hit different parity disks in parallel, AND normal reads utilize all 5 disks instead of leaving Disk 5 idle! *(Note: both RAID 4 and RAID 5 still have the "read-modify-write" small-write penalty per single write: 2 reads + 2 writes.)*

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
Follow the exact phrasing of the problem: *"and then the reference is started again"*!
1. You look in the cache first (**20 ns**).
2. If it's in RAM (6% chance), you spend **60 ns** loading the word into cache, and **restart the reference (+20 ns)** = **100 ns** total.
3. If it's on Disk (4% chance), you spend **12,000,000 ns** fetching from disk, **60 ns** copying into cache, and **restart the reference (+20 ns)** = **12,000,100 ns** total.
Multiplying out: $18 + 6 + 480,004 = \mathbf{480,028\text{ ns}}$! Notice how 99.995% of the average access time comes purely from that 4% disk fault rate!

**Note on 2022 Key Correction**:
- The scanned 2022 key mistakenly reported `4.8 ms` (`4,800,000 ns`, off by a factor of $10\times$!). Computing carefully gives $0.9(20) + 0.06(100) + 0.04(12,000,100) = 18 + 6 + 480,004 = \mathbf{480,028\text{ ns}}$.

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
Look carefully at the subtle deadlock trap in two-way handshakes:
- If `Passenger` called `wait(car_avail)` *before* `signal(pass_waiting)`, and `Car` called `wait(pass_waiting)` *before* `signal(car_avail)`, **both would sleep forever on the very first step!**
- By having **both sides signal their presence first** (`signal(pass_waiting)` before `wait(car_avail)`, and `signal(car_avail)` before `wait(pass_waiting)`), the rendezvous opens cleanly with zero circular wait!
- Adding `boarded` ensures the car doesn't start driving before the passenger sits down, and `ride_over` ensures the passenger stays in the car until the tour finishes! *(Wait: what if $n$ cars run concurrently and ride durations differ? Notice `ride_over` unblocks *a* riding passenger when *any* car finishes—for exact per-car pairing, each car $j$ uses its own private `car_ride_over[j]` passed via a bounded queue, though the 4-semaphore counting model above is the standard clean exam solution!)*

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
Notice how these 5 transitions literally tell the entire **evolutionary saga of Operating Systems**! Every single step fixed the exact crisis left by the step before it:
1. **(e) Manual $\to$ Batch**: Humans were too slow swapping jobs $\implies$ Resident Monitor automated transitions!
2. **(a) Single-user $\to$ Multiprogramming**: Slow disks left CPU idle $\implies$ keep multiple jobs in RAM so CPU never sits idle on I/O!
3. **(b) Absolute $\to$ Relocatable**: Jobs stuck waiting for *one specific memory slot* while other slots were empty $\implies$ base/relocation registers let a job run anywhere!
4. **(c) Fixed $\to$ Variable Partitions**: Putting a 10 KB job in a 100 KB fixed slot wasted 90 KB (**internal fragmentation**) $\implies$ cut partitions dynamically to fit each job!
5. **(d) Contiguous $\to$ Non-contiguous**: Cutting dynamic sizes left Swiss-cheese gaps (**external fragmentation**) $\implies$ chop memory into fixed pages/frames!

**Extra Memory Analogy (Contiguous $\to$ Non-Contiguous)**:
Imagine parking 20-foot limousines along a curb with no splitting allowed (**contiguous allocation**). After several cars leave, you might have a 10-foot gap near the corner and another 12-foot gap down the block. You have **22 feet** of empty curb total, yet a 20-foot limousine still can't park unless you tow every other parked car down the street (**memory compaction**, which freezes the system!).

Now switch to **paging (non-contiguous allocation)**: chop every car (process) into uniform 4 KB bike-sized slots (**pages**) and chop the curb into 4 KB slots (**frames**). Now you can park the pieces anywhere free frames exist! Zero external fragmentation, zero compaction needed, and as a bonus, you don't even have to keep parts in RAM that aren't currently being used!

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
Watch how simple and mechanical Distance Vector Routing is!
Router $C$ doesn't know the whole map—it only knows the cost to step onto its three neighbor bridges ($B=6$, $D=3$, $E=5$). When $B$, $D$, and $E$ mail in their distance vectors (each a 6-tuple for `A, B, C, D, E, F`), $C$ simply adds $6$ to every number from $B$, $3$ to every number from $D$, and $5$ to every number from $E$, then picks the smallest total column-by-column (always keeping $C \to C = 0$)!

**Scan Trivia (2022 Endsem Q11 Check)**:
- In `OSN_end_2022_M_key.pdf`, the printed solution key had a minor typo `6+1=18` on D's entry via B (correct is `6+1=7`), which fortunately did not alter the final routing table `(11, 6, 0, 3, 5, 8)` via `(B, B, -, D, E, B)` because direct `C→D = 3 < 7`!

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
Look how neatly the numbers click together:
- **Offset**: $64\text{ KB} = 2^{16} \implies 16$ bits. That leaves $44 - 16 = \mathbf{28}$ bits for the Virtual Page Number (**$2^{28}$ pages**).
- **One frame** holds $64\text{ KB} / 4\text{ B} = 16,384 = \mathbf{2^{14}}$ entries! And $28 / 2 = \mathbf{14}$! So the 28-bit VPN divides symmetrically into **$14\text{ bits} \mid 14\text{ bits}$**, meaning *both* the Outer Page Directory and every Inner Page Table fit neatly into exactly 1 page frame ($64\text{ KB}$)!
- For a **$4\text{ GB}$ ($2^{32}\text{ B}$) program**:
  - Data needs $2^{32} / 2^{16} = 65,536$ frames.
  - Since 1 inner page table maps $16,384$ pages, mapping $65,536$ pages needs $65,536 / 16,384 = \mathbf{4}$ inner page-table frames.
  - Plus **1** outer page-directory frame pointing to those 4 inner tables!
  - Total = $65,536 + 4 + 1 = \mathbf{65,541\text{ frames}}$!

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
Watch out for a common exam pitfall: on the scanned paper, a student ticked `c. 11.33 ms`, which is **mathematically impossible**! Let's see why:
- Total CPU needed across all 3 processes = $5 + (2 + 2) + 4 = 13\text{ ms}$, and the CPU is never idle from $t=0$ to $t=13$.
- Even if you test *every possible* tie-breaking order when simultaneous events occur (new arrival vs. I/O return vs. preempted process), the only achievable averages are `9.67 ms`, **`10.00 ms`**, `10.33 ms`, and `10.67 ms` (`32/3`). So **`a. 10.00 ms`** is unequivocally the correct MCQ option!
- Also notice why $P_2$ has $\text{TAT} = 12\text{ ms}$: Turnaround Time always measures **wall-clock duration from arrival (`1 ms`) to final exit (`13 ms`)**, which naturally includes its 4 ms I/O wait inside `[5, 9]`.

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
Why does TCP add **`4 * DevRTT`** onto `EstimatedRTT` instead of just using `EstimatedRTT` (`51.25 ms`, which is distractor `c`)?
Because network latency has **jitter**! Even if average RTT is `51.25 ms`, half of healthy packets naturally take a bit longer than the average. Adding a **4-standard-deviation safety margin (`4 * DevRTT`)** prevents TCP from triggering false timeouts when a normal packet merely experiences slight jitter! *(Also note: always compute `DevRTT` using the **newly updated** `EstimatedRTT = 51.25`, giving exactly `72.00 ms` = option **b**.)*

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
Watch out for the classic hex trap when splitting top 2 bits!
People often look at the first hex digit `5` (`0101`) and forget that the top 2 bits (`01`) belong to the Segment ID while the **other 2 bits (`01`) of that `5` remain inside the offset!**
So the offset is **`0x1100`** (NOT `0x0100`, and NOT `0x5100`!).
- Since `0x1100` (4,352 bytes) is strictly smaller than Heap's limit `0x2000` (8,192 bytes), it does **not** cause a Segmentation Fault (note the stray pen marks on the scanned PDF circling `d`—ignore student marks!).
- Adding Base `0x9000` + Offset `0x1100` gives **`0xA100`** (**b**)!

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
Follow the story of a single memory access when **both** the TLB and RAM miss:
1. **Trap!**: CPU checks the software-managed TLB, doesn't find `VPN`, and traps into Kernel Mode.
2. **Page Table Walk**: The OS inspects the process's page table. Suppose `present == 0`—the page is sitting out on slow disk/swap!
3. **Never Wait Idle!**: Loading a page from disk takes ~10 ms (tens of millions of CPU cycles!). So the OS allocates a physical frame (evicting a victim if RAM is full), **starts an asynchronous DMA disk read**, marks the faulting process **`BLOCKED`**, and **immediately runs some other `READY` process on the CPU**!
4. **Disk Interrupt & Resume**: 10 ms later, the disk controller fires an interrupt saying *"Done!"*. The OS marks `pte->present = 1` and moves our process back to `READY`. When the scheduler picks it again, the CPU **re-executes the exact instruction that trapped**—this time finding the page in RAM, updating the TLB, and running smoothly!

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
Think of the **Clock Algorithm** like a security guard walking around a circular hallway of rooms checking whether anyone is awake (`R = 1` means *"used recently"*, `R = 0` means *"sleeping / untouched"*):
- If the guard checks Room #1 and sees `R = 1`: *"Okay, you get a second chance!"*—the guard flips the light switch off (`R = 0`) and walks to Room #2.
- If the page isn't touched before the guard makes a full lap back to Room #1, `R` will still be `0`—so the guard evicts Room #1!
- **When does it act just like plain FIFO?** Suppose *every single page* in RAM is actively used (`R = 1, 1, 1, 1`) before the next page fault! The guard walks a full circle flipping every switch (`0, 0, 0, 0`) and lands right back on **the oldest page it started at**—literally executing FIFO!

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
Notice that the exam instructions explicitly state *"you can select one or more options (if necessary)"*!
1. Why **not `b` (set to 0)**? If every TCP connection started at `seq = 0`, an attacker could easily forge packets—and worse, a delayed packet #5 from an old crashed session could crash straight into a brand-new session! So TCP picks a pseudo-random 32-bit Initial Sequence Number ($N_p$). *(Note: Wireshark only shows `Seq = 0` as a "relative sequence number" helper for readability—on the real wire, it's random!)*
2. Why **`d` ($N_q = N_p + 1$)**? Even though a `SYN` packet carries 0 application payload bytes, TCP treats the `SYN` control bit as occupying **1 logical sequence number** so that `SYN` itself can be reliably acknowledged (`N_p + 1`).

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
Walk through this like flipping a light switch when you plug a brand-new laptop into campus Wi-Fi and open `http://anotherdomain.com`:
1. **OS Foundations (Part a)**: You need NIC hardware drivers + a TCP/IP stack + sockets (`connect`/`send`), otherwise apps can't even talk to the wire.
2. **Who am I? Where are they? (Part b)**:
   - **DHCP** gives your laptop an identity card (`IP, Mask, Router IP, DNS Server IP`).
   - **DNS** acts like a phonebook translating human name `"anotherdomain.com"` into `"179.20.10.6"`.
3. **Packing the Russian Nesting Dolls (Part c)**:
   - **Application Layer**: Writes the letter (`HTTP GET /`).
   - **Transport Layer (TCP)**: Calls `179.20.10.6:80` to establish a reliable 3-way handshake, stamps ports/sequence numbers/checksum onto the TCP segment.
   - **Link Layer**: Notices `"179.20.10.6"` isn't in the same room, uses **ARP** to find the local **Router's MAC address** (note: *NOT* the server's MAC!), wraps everything in an Ethernet frame with `Dst MAC = Router MAC` + CRC, and shoots it out the cable!

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
Think of locks `a, b, c, d` as numbered steps `1, 2, 3, 4`:
- As long as every single process **only walks UP the stairs** (never holding a higher-numbered lock while asking for a lower-numbered lock), can two processes ever trap each other in a circle? **Never!** Whoever already holds the highest lock can always finish without needing anything above it!
- Only Option **b** keeps every process walking strictly upward: `X` goes `1→2→3`, `Y` goes `2→3→4`, and `Z` goes `1→3→4`!

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
Look closely at Part **(a)**—don't make a rushed arithmetic mistake!
- $2\%$ of $16\text{ TB}$ is $0.32\text{ TB} \approx 327.68\text{ GiB}$. Dividing by $256\text{ B}$ gives **$1.374\text{ billion}$ inodes**!
- $98\%$ of $16\text{ TB}$ is $15.68\text{ TB}$. Dividing by $8\text{ KB}$ ($2^{13}\text{ B}$) gives **$2.105\text{ billion}$ data blocks**!
- Because $1.374\text{ billion} < 2.105\text{ billion}$, even when every tiny $1\text{ KB}$ log wastes $7\text{ KB}$ inside an $8\text{ KB}$ block, **inodes run out first**! (Think: an average file size $< \frac{2.105}{1.374} \times 8\text{ KB} \approx 12.25\text{ KB}$ always runs out of inodes before blocks!)
- And in Part **(c)**, always explicitly state **both** sides of the engineering trade-off clearly: RAID 5 wins on capacity (**28 TB vs 16 TB**), whereas RAID 1 wins on random small-write performance (**2 I/Os vs 4 I/Os per write**)!

---

### AgentOS — Question 9 [15 Points] — Multi-Stage Bootloader Firmware Architecture [Frequency: 1]

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
Look at why Part **(a)** is a *classic* real-world networking concurrency problem:
- If you have 500 packet-lookup threads constantly reading `Global_Routing_Table`, a naive reader-first lock would **starve the BGP writer forever** whenever traffic is busy! That means when an undersea fiber cable gets cut, your router keeps forwarding millions of packets into a dead link (**black hole**)!
- By placing **`order_mutex`** at the front door, new lookups keep flowing simultaneously in parallel whenever no BGP update is pending—yet the instant a `BGP_UPDATE` arrives, it steps into `order_mutex`, waits a few microseconds for in-flight lookups to finish, updates the route atomically, and resumes forwarding!
- And in Part **(b)**, remember the **Airport vs. Airline analogy**: **OSPF** is the internal airport terminal signage (*"Gate B12 is a 3-minute walk"*—pure efficiency inside one building), whereas **BGP** is the global airline alliance network (*"Don't book a flight connecting through a rival airline we don't have a contract with"*—scale + business policy + privacy across independent organizations!).

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
This classic Silberschatz textbook question tests whether you recognize **Priority** and **MLFQ** as universal "super-frameworks":
- **Priority** lets you plug in *any* key to pick who runs next: plug in `1 / burst_time` and you get **SJF**; plug in `-arrival_time` and you get **FCFS**!
- **MLFQ** lets you configure $k$ queues with any scheduling rule inside each: set $k = 1$ queue running FIFO and you instantly recover **FCFS** (and set $k = 1$ with quantum $q$ and you recover **RR**)!
- **RR vs. SJF** have completely different axes: RR slices by **clock time** without knowing job durations, whereas SJF sorts by **job duration** without slicing by clock time—so neither can mimic the other across arbitrary workloads!

---
