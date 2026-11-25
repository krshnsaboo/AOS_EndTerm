# Topic 3: CPU Scheduling Algorithms & Numerical Calculations (Ch. 5)

> **Overview**: Multiprogramming CPU/IO overlap bounds, CPU-bound vs. I/O-bound classification, FCFS convoy effect, SJF vs. RR vs. Progressive RR, UNIX dynamic priority & preemption points, priority inheritance, MLFQ, Linux CFS `vruntime`, and processor affinity vs. load balancing.  
> **Reference Material**: [cpuScheduling_CS.ppt](../Faculty_Notes/cpuScheduling_CS.ppt) | [thompson-bstj78-unix.pdf](../Faculty_Notes/thompson-bstj78-unix.pdf)  
> **Questions**: 20 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [Multiprogramming Total Execution Time: How T Can Exceed Σ t_i](#1-multiprogramming-total-execution-time-how-t-can-exceed-σ-t_i-frequency-1) — Frequency: 1
2. [Multiprogramming Total Execution Time: How T Can Approach max(t_1, ..., t_N)](#2-multiprogramming-total-execution-time-how-t-can-approach-maxt_1-t_n-frequency-1) — Frequency: 1
3. [CPU-Bound vs. I/O-Bound Processes & Scheduling Based on Recent CPU Usage](#3-cpu-bound-vs-io-bound-processes-scheduling-based-on-recent-cpu-usage-frequency-2) — **Frequency: 2**
4. [T/F: OS Gives Higher Priority to I/O-Bound Over CPU-Bound Processes](#4-tf-os-gives-higher-priority-to-io-bound-over-cpu-bound-processes-frequency-3) — **Frequency: 3**
5. [Consequence of Misclassifying CPU-Bound and I/O-Bound Processes on Throughput & Response Time](#5-consequence-of-misclassifying-cpu-bound-and-io-bound-processes-on-throughput-response-time-frequency-2) — **Frequency: 2**
6. [How FCFS CPU Scheduler Penalizes I/O-Bound Processes (Convoy Effect)](#6-how-fcfs-cpu-scheduler-penalizes-io-bound-processes-convoy-effect-frequency-2) — **Frequency: 2**
7. [Problem in Non-Preemptive CPU Scheduling (Convoy Effect & Unresponsive Starvation)](#7-problem-in-non-preemptive-cpu-scheduling-convoy-effect-unresponsive-starvation-frequency-2) — **Frequency: 2**
8. [Conflicting Pairs of CPU Scheduling Criteria](#8-conflicting-pairs-of-cpu-scheduling-criteria-frequency-1) — Frequency: 1
9. [Multilevel Queue Scheduling: Starvation Risks & Mitigation Strategies](#9-multilevel-queue-scheduling-starvation-risks-mitigation-strategies-frequency-1) — Frequency: 1
10. [Progressive Round Robin vs. Ordinary Round Robin (Batch vs. Interactive Users)](#10-progressive-round-robin-vs-ordinary-round-robin-batch-vs-interactive-users-frequency-3) — **Frequency: 3**
11. [UNIX CPU Scheduling Algorithm (Dynamic Negative-Feedback Priority)](#11-unix-cpu-scheduling-algorithm-dynamic-negative-feedback-priority-frequency-1) — Frequency: 1
12. [Why Classic Non-Preemptive UNIX Kernels Are Unsuitable for Real-Time Applications](#12-why-classic-non-preemptive-unix-kernels-are-unsuitable-for-real-time-applications-frequency-1) — Frequency: 1
13. [Why Preemption Points Are Included in Kernel System Calls](#13-why-preemption-points-are-included-in-kernel-system-calls-frequency-1) — Frequency: 1
14. [Application of the Priority Inheritance Protocol (Fixing Priority Inversion)](#14-application-of-the-priority-inheritance-protocol-fixing-priority-inversion-frequency-1) — Frequency: 1
15. [T/F: "The Notion of 'Load Balancing' Counteracts with 'Processor Affinity'"](#15-tf-the-notion-of-load-balancing-counteracts-with-processor-affinity-frequency-2) — **Frequency: 2**
16. [Process Scheduling Overhead: Why Context Switch Duration Impacts CPU Efficiency](#16-process-scheduling-overhead-why-context-switch-duration-impacts-cpu-efficiency-frequency-1) — Frequency: 1
17. [LangOS Numerical Scheduling: SJF vs. Round Robin & Equal-Burst Equivalence](#17-langos-numerical-scheduling-sjf-vs-round-robin-equal-burst-equivalence-frequency-1) — Frequency: 1
18. [AgentOS Numerical Round Robin (q = 1 ms): Average Latency = 10.00 ms & Fairness](#18-agentos-numerical-round-robin-q-1-ms-average-latency-1000-ms-fairness-frequency-1) — Frequency: 1
19. [Linux CFS Scheduler Mechanics: Does vruntime Ever Overflow?](#19-linux-cfs-scheduler-mechanics-does-vruntime-ever-overflow-frequency-1) — Frequency: 1
20. [Parameterized Round Robin Quantum Selection & Asymmetric Interactive/CPU Workload Design](#20-parameterized-round-robin-quantum-selection-asymmetric-interactivecpu-workload-design-frequency-1) — Frequency: 1

---

### 1. Multiprogramming Total Execution Time: How T Can Exceed Σ t_i [Frequency: 1]

> **Question**: Suppose a multi-programmed system has a load of $N$ processes with individual total execution times of $t_1, t_2, \dots, t_N$. How would it be possible that $T > t_1 + t_2 + \dots + t_N$? That is, what would cause the total execution time to exceed the sum of individual process execution times?

#### pre-requisite
- **Ideal vs. Real Multiprogramming Total Time ($T$)**: Ideally, overlapping CPU and I/O makes $T \le \sum t_i$. However, OS and hardware sharing introduce extra runtime overheads not present when each process runs alone.

#### Answer (Exam Writing Format)
Total execution time $T$ can exceed $\sum_{i=1}^N t_i$ due to **multiprogramming overheads**:
1. **Context-Switch Overhead**: Frequently saving/restoring CPU registers, PCBs, and kernel stacks, plus running the OS scheduler across $N$ processes adds pure non-productive time.
2. **Cache & TLB Pollution (Cold Caches)**: Interleaving $N$ processes evicts each other's hot L1/L2 cache lines and TLB entries (or flushes TLBs on context switch), forcing extra cache misses and page-table walks that slow down every process beyond its solo time $t_i$.
3. **Memory Contention & Thrashing (Page Faults / Swapping)**: Sharing finite RAM among $N$ processes shrinks per-process frame allocations, triggering excessive page faults and slow disk/swap I/O delays not incurred when running solo.
4. **Resource & Lock Contention**: Processes competing for shared locks, disk heads (extra seek latency), memory bus bandwidth, or non-overlapped I/O waits waste additional execution time.

#### IN DETAIL
While multiprogramming aims to make total completion time $T < \sum t_i$ by overlapping one process's I/O waits with another process's CPU execution, several systemic overheads can push $T > \sum t_i$:
- **Context-Switch & Cache/TLB Pollution**: Each preemption burns pure OS cycles saving/restoring PCB registers and triggers cold L1/L2 cache misses plus TLB misses once a new address space takes over, slowing individual instruction throughput compared to isolated execution.
- **Hardware Contention & Thrashing**: Concurrent processes contend for shared DRAM channels, memory buses, and disk seek queues; worst of all, exceeding resident RAM capacity induces heavy swapping/page-fault thrashing ($T \gg \sum t_i$).

---

### 2. Multiprogramming Total Execution Time: How T Can Approach max(t_1, ..., t_N) [Frequency: 1]

> **Question**: Suppose that a multi-programmed system has a load of $N$ processes with individual total execution times of $t_1, t_2, \dots, t_N$. How would it be possible that total execution time could be as small as $\text{maximum}(t_1, t_2, \dots, t_N)$?

#### pre-requisite
- **Overlapping Execution in Multiprogramming**: Each process's total time $t_i$ consists of both **CPU bursts** and **I/O waits** ($t_i = t_{i,\text{CPU}} + t_{i,\text{IO}}$).

#### Answer (Exam Writing Format)
Total execution time $T$ can approach $\max(t_1, t_2, \dots, t_N)$ in two ways:
1. **CPU–I/O Overlap on a Single CPU**: Suppose the process $P_k$ with the longest total time $t_k = \max(t_i)$ spends significant portions of $t_k$ blocked waiting on I/O devices. Whenever $P_k$ blocks on I/O, the multiprogramming OS schedules the remaining $N-1$ processes to run on the CPU (and independent I/O devices) **inside $P_k$'s waiting gaps**, so all shorter processes finish *before or alongside* $P_k$'s completion time $t_k$ (assuming $\sum t_{i,\text{CPU}} \le t_k$ and negligible context-switch overhead).
2. **Parallel Execution on Multiprocessor/Multicore Hardware**: With $\ge N$ CPU cores, all $N$ processes execute simultaneously on separate cores, finishing at $\max(t_1, \dots, t_N)$.

#### IN DETAIL
Under ideal concurrency without contention, overlapping computation across time or hardware resources lowers aggregate wall-clock completion time $T$ toward $\max(t_1, t_2, \dots, t_N)$:
- **Single-CPU Compute/I/O Overlap**: If one process executes mostly CPU bursts (duration $T_{\max}$) while $N-1$ other processes spend most of their lifetimes waiting asynchronously in independent DMA disk/network I/O queues with minimal CPU slices, all I/O latencies run entirely inside the dominant job's compute shadow, bounding total wall-clock makespan near $T_{\max}$.
- **Multi-Core / Multi-Processor Parallelism**: On an $M$-core SMP machine where $M \ge N$ and no data dependencies or lock serialization constrain execution, every process runs independently on its own core simultaneously, achieving true wall-clock makespan equal to the longest individual process ($\max_i t_i$).

---

### 3. CPU-Bound vs. I/O-Bound Processes & Scheduling Based on Recent CPU Usage [Frequency: 2]

> **Question**: Explain the terms "CPU-bound process" and "I/O bound process" through examples. Suppose a CPU scheduling algorithm favors those programs that have used little processor time in the recent past. Explain why this algorithm favours I/O-bound processes and yet does not permanently deny processor time to CPU-bound processes?

#### pre-requisite
- **CPU-Bound vs I/O-Bound**: I/O-bound jobs have short CPU bursts followed by long I/O waits (e.g., interactive editor); CPU-bound jobs execute long uninterrupted CPU bursts (e.g., matrix multiplication).
- **Aging / Priority Decay**: Dynamic priorities adjust over time based on recent CPU usage (`recent_cpu / real_time`).

#### Answer (Exam Writing Format)
1. **Definitions & Examples**:
   - **CPU-Bound Process**: Spends the vast majority of its execution time performing CPU computations with long CPU bursts and minimal I/O (e.g., computing Fast Fourier Transforms, training an ML model, matrix multiplication).
   - **I/O-Bound Process**: Spends most of its time waiting on peripheral/user I/O with very short CPU bursts in between (e.g., reading/copying files, interactive text editors, web servers).
2. **Why the Algorithm Favors I/O-Bound Processes**:
   - I/O-bound processes execute brief CPU bursts and spend most time blocked in I/O wait queues, so their **recent CPU usage remains very low**.
   - Thus, the scheduler assigns them high priority, letting them run immediately, quickly start their next I/O request, and maximize **both I/O utilization and interactive responsiveness**.
3. **Why CPU-Bound Processes Are Not Permanently Denied CPU Time**:
   - Whenever I/O-bound jobs block waiting for I/O completion, the ready queue has only CPU-bound jobs left, so **remaining CPU capacity runs CPU-bound jobs**.
   - Furthermore, **priority decay / aging** reduces recorded recent CPU usage as a process waits, periodically boosting waiting CPU-bound processes so they never starve.

#### IN DETAIL
Dynamic priority decay based on **recent CPU consumption** (exemplified by classic UNIX `decay` scheduling) adapts automatically without requiring prior workload labels:
- **Negative-Feedback Mechanism**: Every timer tick, the OS increments a running process's recent-CPU counter (`p_cpu`) and periodically decays all counters exponentially (e.g., $\text{p\_cpu} \leftarrow \text{p\_cpu} / 2$), setting scheduling priority inversely proportional to `p_cpu`.
- **Automatic Convergence**: I/O-bound processes run briefly ($< 1\text{ ms}$) before blocking for disk/network completion, keeping their `p_cpu` tiny and yielding top scheduling priority the instant their I/O returns. CPU-bound processes exhaust full timer quanta, accumulating large `p_cpu` values and automatically drifting to lower priority queues where they soak up idle background cycles without starving I/O devices.

---

### 4. T/F: OS Gives Higher Priority to I/O-Bound Over CPU-Bound Processes [Frequency: 3]

> **Question**: Identify whether the following statement is TRUE or FALSE. If the statement is FALSE, correct it and justify the corrected sentence. If the statement is TRUE, justify it. Restrict the justification to a few sentences. "As compared to CPU-bound process, OS gives higher priority for I/O bound process."

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

### 5. Consequence of Misclassifying CPU-Bound and I/O-Bound Processes on Throughput & Response Time [Frequency: 2]

> **Question**: The kernel of a multiprogramming system classifies a program as CPU-bound or I/O-bound and assigns appropriate priority to it. **What would be the consequence of a wrong classification of programs for throughput and response times in a multi-programming system?**
> *(2021 Endsem Q1 had the opening stem truncated before asking the priority/classification rationale.)*

#### pre-requisite
- **Correct Classification Behavior**: Normally, **I/O-bound** processes receive **higher priority** (short CPU bursts $\to$ start I/O fast $\to$ overlap CPU & I/O + great interactive response), while **CPU-bound** processes receive **lower priority** (utilize remaining CPU while I/O runs).

#### Answer (Exam Writing Format)
If the OS **misclassifies** processes—assigning **high priority to CPU-bound** programs and **low priority to I/O-bound** programs—the system suffers severe degradation:
1. **Consequences for Response Time**:
   - **Degraded (Very High) Interactive Response Time**: Short I/O-bound / interactive processes sit queued behind long uninterrupted CPU bursts (the **Convoy Effect**). Under a continuous stream of CPU-bound jobs, I/O-bound processes face severe delays or **starvation**.
2. **Consequences for Throughput & Hardware Utilization**:
   - **Lower System Throughput & Poor I/O Utilization**: While high-priority CPU-bound jobs monopolize the CPU, I/O-bound jobs wait in the ready queue instead of submitting I/O requests—leaving **disks, NICs, and peripherals sitting idle**.
   - Later, when all I/O-bound jobs finally run and hit I/O simultaneously, the CPU risks idling if CPU jobs finish—eliminating CPU–I/O overlap and reducing total jobs completed per unit time (**throughput drops**, even though CPU-bound jobs individually finish slightly earlier).

**Summary of Correct Classification (Comparison)**:
- Correctly giving higher priority to I/O-bound processes maximizes CPU–I/O overlap and keeps CPU-bound jobs running whenever I/O-bound processes block (no starvation via aging/recent-usage decay).

#### IN DETAIL
Inverting scheduling preference to prioritize **CPU-bound processes** above **I/O-bound processes** reverses hardware pipelining and triggers systemic underutilization:
- **Device Starvation & Convoy Effect**: Short I/O-bound processes stall indefinitely in the Ready Queue waiting behind multi-second CPU computations. Meanwhile, disk and network subsystems fall completely idle because their initiating processes cannot obtain the tiny CPU slices needed to dispatch subsequent I/O commands.
- **Interactive Latency Collapse**: Interactive user threads (keyboard, GUI, RPC handlers) are fundamentally I/O-bound; delaying them destroys perceived responsiveness and lowers overall job completion throughput across the mixed workload.

---

### 6. How FCFS CPU Scheduler Penalizes I/O-Bound Processes (Convoy Effect) [Frequency: 2]

> **Question**: Explain how FCFS CPU scheduler penalizes I/O bound processes.

#### pre-requisite
- **Convoy Effect**: In non-preemptive First-Come, First-Served (FCFS), once a process gets the CPU, it holds it until it finishes its CPU burst or blocks for I/O.

#### Answer (Exam Writing Format)
- **Workload Difference**: I/O-bound processes have **very short CPU bursts** followed by long I/O operations; CPU-bound processes have **very long CPU bursts**.
- **Mechanism of Penalty (Convoy Effect)**:
  1. If a long CPU-bound process enters the ready queue ahead of I/O-bound processes, FCFS runs the CPU-bound job uninterruptedly until completion/blocking.
  2. All I/O-bound processes sit stuck waiting in the ready queue behind it, leaving I/O devices idle.
  3. When the CPU-bound process finally finishes, the I/O-bound processes race through their short CPU bursts and all rush to I/O devices together—leaving the CPU idle until I/O finishes.
- **Result**: Severe increase in waiting/turnaround time for I/O-bound jobs and sharp drops in **both CPU and I/O device utilization**.

#### IN DETAIL
First-Come, First-Served (**FCFS**) exhibits the classic **Convoy Effect** because it is strictly non-preemptive and ignores remaining burst durations:
- **Mathematical Delay Amplification**: If one CPU-bound process arrives just ahead of $k$ short I/O-bound processes (say CPU burst $B = 100\text{ ms}$, short bursts $b = 1\text{ ms}$), all $k$ short processes wait the full $100\text{ ms}$ in the Ready Queue while all I/O peripherals sit idle. Once the CPU-bound process finally yields for I/O, the $k$ I/O jobs flash through $1\text{ ms}$ CPU bursts and immediately bunch up together back in I/O queues—repeatedly creating periodic CPU idle valleys followed by long I/O-bound starvation delays.

---

### 7. Problem in Non-Preemptive CPU Scheduling (Convoy Effect & Unresponsive Starvation) [Frequency: 2]

> **Question**: Some simple OSs use non-preemptive scheduling. Give an example of a problem that can occur when the OSs uses non-preemptive scheduling.

#### Answer (Exam Writing Format)
- **Problem (Unbounded Delay / Failure to Meet Time Constraints)**: In non-preemptive scheduling, once a process (or kernel routine) holds the CPU, it cannot be forcibly interrupted until it blocks or finishes.
- **Example**: If a low-priority batch process (or infinite loop) holds the CPU and an urgent **high-priority / real-time** process (or interactive user event) becomes ready via an interrupt, the urgent process cannot run until the running process voluntarily yields—violating real-time deadlines and hurting interactive response time.

#### IN DETAIL
Non-preemptive scheduling creates **unbounded worst-case scheduling latency**, making it structurally incompatible with multi-user time-sharing and real-time guarantees:
- **Arbitrary CPU Hold Duration**: Once any process enters `Running` state, neither timer interrupts nor higher-priority wakeups can reclaim the core until that thread explicitly invokes a blocking syscall, exits, or executes `yield()`.
- **Cascading Failure Modes**: An infinite compute loop freezes an entire uniprocessor system permanently; even well-behaved long computations violate strict real-time deadlines (e.g., audio buffer underflows, sensor drop-outs, or safety-critical control loops) whenever high-priority tasks become runnable mid-burst.

---

### 8. Conflicting Pairs of CPU Scheduling Criteria [Frequency: 1]

> **Question**: Discuss how the following pairs of scheduling criteria conflict in certain settings: **CPU utilization** and **response time**.

#### Answer (Exam Writing Format)
- **Conflict**: Optimizing for maximum **CPU utilization** can degrade (increase) **response time**, and vice versa.
- **Explanation & Setting**:
  - To maximize **CPU utilization**, the scheduler avoids frequent preemptions/context switches and lets long CPU-bound jobs run uninterruptedly (e.g., FCFS or huge time quanta), eliminating switch overhead—**but** newly arrived interactive processes must wait long periods, severely worsening **response time**.
  - Conversely, achieving fast **response time** requires tiny time slices (frequent preemptions/Round Robin), which increases context-switch overhead and lowers effective **CPU utilization**.

#### IN DETAIL
Scheduling criteria form inherent trade-offs where optimizing one metric mathematically penalizes another across fixed CPU capacity:
- **Throughput / CPU Utilization vs. Response Time**: Every preemptive context switch burns non-productive kernel instructions ($T_{\text{ctx}}$) plus warm L1/L2 cache and TLB state (indirect cold-cache cycles afterward). Maximizing pure CPU efficiency prefers huge time quanta or zero preemption (maximizing useful-instruction percentage per second), whereas guaranteeing rapid **response time** for interactive tasks requires short quanta ($q$) and frequent preemptions—trading higher context-switch overhead for bounded latency.

---

### 9. Multilevel Queue Scheduling: Starvation Risks & Mitigation Strategies [Frequency: 1]

> **Question**: Consider a system implementing multilevel queue scheduling. What strategy can a computer user employ to maximize the amount of CPU time allocated to the user's process?

#### pre-requisite
- **Multilevel Feedback Queue (MLFQ) Behavior**: Processes start in the highest-priority queue (shortest quantum $q_1$); if they consume their full time quantum, they are demoted to lower-priority queues (which only run when higher queues are empty). If they yield early for I/O, they stay at high priority.

#### Answer (Exam Writing Format)
- **How MLFQ Works**: Higher-priority queues get smaller time quanta and strictly preempt/run before lower-priority queues. A process that uses its entire quantum is demoted downward.
- **User Strategy to Maximize CPU Priority/Time ("Gaming the Scheduler")**:
  - Program the process so that just **before** its time quantum in the top queue expires, it issues a **dummy/quick I/O operation** (or yields voluntarily) to relinquish the CPU slightly early.
  - Because it never consumes a full quantum continuously, the scheduler misclassifies it as an interactive/I/O-bound job and **keeps it permanently in the highest-priority queue**, maximizing its CPU allocation.

#### IN DETAIL
Multilevel Feedback Queue (**MLFQ**) promotion/demotion rules classify tasks empirically by monitoring whether a running process exceeds its allotted time quantum before blocking:
- **Gaming Heuristic Exploit**: Because MLFQ assumes any thread yielding before quantum expiration $q$ is interactive/I/O-bound, a malicious or greedy CPU-bound application can inject synthetic voluntary pauses (`usleep(1)`, dummy I/O read, or `sched_yield`) immediately prior to consuming $q$—staying permanently pinned inside the highest-priority queue and starving genuine workloads.
- **System Countermeasures**: Modern OS schedulers defend against gaming via **cumulative CPU accounting** across slices (like `vruntime` in Linux CFS or total-time tracking per queue level in MLFQ) rather than per-burst reset timers, alongside automatic **aging / periodic priority resets** to lift starved lower-tier tasks.

---

### 10. Progressive Round Robin vs. Ordinary Round Robin (Batch vs. Interactive Users) [Frequency: 3]

> **Question**: A system is receiving both batch jobs and interactive jobs. Normally, round robin makes the users of interactive jobs happy. Consider a variation of round robin we will call **progressive round robin**. In progressive round-robin, each process has its own time quantum. This starts out at 50ms, and increases by 50 ms each time it goes through the round-robin queue. Give the advantages and disadvantages of this variant over ordinary-round robin with respect to the users of batch jobs and interactive jobs.

#### pre-requisite
- **Ordinary Round Robin (RR)**: Every process receives the exact same fixed time quantum $q$ every time it reaches the front of the ready queue.
- **Interactive vs. Batch Jobs**: Interactive jobs have tiny CPU bursts (user typing/clicking) and care about **low response time**; batch jobs have huge CPU bursts and care about **high throughput / low overhead**.

#### Answer (Exam Writing Format)
- **Context**: In **ordinary RR**, every job gets a fixed small slice (say 50 ms). In **progressive RR**, process $i$'s quantum follows $50\text{ ms}, 100\text{ ms}, 150\text{ ms}, \dots$ across successive rounds.
- **1. Impact on Batch Job Users**:
  - **Advantages**:
    - Fewer context switches as their quantum grows larger ($100, 150, 200\text{ ms}\dots$), reducing scheduling/cache-flush overhead and improving CPU throughput.
    - Batch jobs no longer suffer unfairly against streams of short interactive arrivals (unlike fixed small-quantum RR where batch jobs only get tiny slices).
  - **Disadvantages**:
    - If a *newly submitted* short batch job enters behind an *old* batch job whose quantum has grown huge (e.g., 1000 ms), the newer job experiences long delays.
- **2. Impact on Interactive Job Users**:
  - **Advantages**:
    - Short interactive jobs starting with $50\text{ ms}$ generally finish their burst within 1–2 quanta and enjoy good response time—*provided no long batch job currently holds an inflated quantum*.
  - **Disadvantages**:
    - **Return of the Convoy Effect**: Once long-running batch jobs accumulate large quanta (e.g., $500\text{ ms}$ or $2000\text{ ms}$), progressive RR behaves increasingly like FCFS. Newly arriving interactive jobs can get stuck waiting seconds for a batch job's huge quantum to expire, severely degrading interactive response time!
    - Also, a process whose quantum resets/doesn't reset properly after blocking can make waiting times unpredictable.

#### IN DETAIL
**Progressive Round Robin** reconciles interactive latency and batch throughput by coupling each process's time quantum to its accumulated execution duration within a single scheduler framework:
- **Dynamic Per-Process Slicing ($50, 100, 150\dots\text{ ms}$)**: Newly arrived or freshly unblocked interactive tasks start at small initial quanta ($50\text{ ms}$), delivering fast initial response latency without starving peers.
- **Amortized Batch Overhead**: Long CPU-bound batch jobs remain runnable through consecutive turns; doubling or incrementing their quantum geometrically reduces total context switches from $O(T_{\text{job}} / q)$ to $O(\log(T_{\text{job}}))$ or $O(\sqrt{T_{\text{job}}})$, substantially improving cache locality and raw CPU efficiency relative to fixed small-quantum Round Robin.

---

### 11. UNIX CPU Scheduling Algorithm (Dynamic Negative-Feedback Priority) [Frequency: 1]

> **Question**: Describe the scheduling algorithm followed in UNIX.

#### pre-requisite
- **Reference Paper**: Ken Thompson, *"UNIX Implementation"* (`thompson-bstj78-unix.pdf`, Section 2.3).

#### Answer (Exam Writing Format)
Traditional UNIX uses a **dynamic priority-based preemptive scheduling algorithm** with negative feedback:
1. **System vs. User Priority Tiers**: System/kernel events (disk, terminal, clock) are assigned higher priorities than *all* user processes; disk events rank above teletype and time-of-day events.
2. **User Priority Formula (Recent CPU Usage)**: User-process priority is computed periodically (every second) based on the recent ratio of **CPU compute time to real time** consumed by the process (plus optional `nice` bias).
   - High recent CPU usage $\implies$ **Lower priority** (penalizes CPU hogs).
   - Low recent CPU usage $\implies$ **Higher priority** (automatically favors interactive & I/O-bound processes).
3. **Dispatch & Preemption**: The scheduler always picks the runnable process with the highest priority. Processes of equal priority (e.g., looping CPU-bound jobs) rotate Round-Robin with a **1-second quantum**. A high-priority process waking up preempts a running lower-priority user process.
4. **Negative Feedback (No Starvation)**: As a process consumes CPU, its priority drops; as a neglected process waits, its recent usage decays and its priority rises.

#### IN DETAIL
Classic **UNIX SVR3/BSD time-sharing scheduling** maintains dynamic priorities through an integer table updated at every clock tick and recomputed every second:
- **Priority Separation**: Kernel/system wait events receive fixed low numerical priorities (higher urgency, e.g., $0\text{--}49$, ordered so disk I/O wakes above TTY above user processes), while user-mode computation sits in higher numerical tiers ($\ge 50$).
- **Decay Formula**: User priority updates roughly as $P_j(i) = \text{Base} + \frac{\text{CPU}_j(i)}{2} + \text{nice}_j$, where $\text{CPU}_j$ increments during active execution and decays exponentially (`CPU_j /= 2`) every second. This mathematically enforces negative feedback: recently idle/blocked jobs surge to top priority, while sustained CPU hogs progressively sink downward.

---

### 12. Why Classic Non-Preemptive UNIX Kernels Are Unsuitable for Real-Time Applications [Frequency: 1]

> **Question**: UNIX is unsuitable for real-time applications because a process executing in kernel mode may not be interrupted. Explain the reasons.

#### pre-requisite
- **Non-Preemptive Kernel (Traditional UNIX)**: Once a user process enters kernel mode via a system call (e.g., `read()`, `fork()`, `fsync()`), the classic UNIX kernel does not preempt it in favor of another process until the syscall finishes or explicitly sleeps/blocks. *(Note: hardware interrupts can still pause it briefly to run an ISR, after which execution returns to the same kernel-mode process—the scheduler cannot switch processes mid-syscall!)*

#### Answer (Exam Writing Format)
1. **Unbounded Dispatch Latency**: In traditional non-preemptive UNIX kernels, if a low-priority process invokes a lengthy system call (e.g., flushing buffers or searching large kernel tables), the kernel executes it to completion/blocking before calling the scheduler.
2. **Missed Hard Real-Time Deadlines**: Even if a high-priority real-time process becomes ready via an interrupt, it **cannot preempt** the low-priority process executing in kernel mode, resulting in unpredictable, unbounded worst-case waiting delays.
3. **Priority Inversion & Lack of Bounded Worst-Case Execution**: Real-time applications (control systems, avionics, robotics, live audio/video) require strict **deterministic latency guarantees**, which non-preemptive kernels cannot provide. *(Modern RTOS kernels and `PREEMPT_RT` Linux fix this by making the kernel itself preemptible.)*

#### IN DETAIL
Traditional non-preemptive UNIX kernels forbid kernel-mode preemption to protect internal kernel invariants cheaply without fine-grained locking:
- **Design Assumption**: If only one thread executes inside Kernel Mode at a time on a uniprocessor, kernel structures (inode lists, buffer-cache chains, process tables) cannot be interleaved mid-mutation by another system call—removing the need for mutex locks on every kernel subsystem.
- **Real-Time Dispatch Latency Penalty**: When an application enters a complex system call (e.g., `fsync()`, recursive path lookup, bulk `fork()` copy), any real-time thread awakened by a hardware interrupt must wait until that syscall voluntarily completes and returns across the User/Kernel boundary—introducing arbitrary, multi-millisecond **dispatch latency** that breaks real-time deadline determinism.

---

### 13. Why Preemption Points Are Included in Kernel System Calls [Frequency: 1]

> **Question**: Why preemption points are included in system calls?

#### pre-requisite
- **Non-Preemptive vs. Preemptive Kernels**: In traditional non-preemptive UNIX kernels, a process executing a system call in kernel mode cannot be preempted until the syscall finishes—causing high dispatch latency for urgent/real-time tasks.

#### Answer (Exam Writing Format)
**Preemption points** are strategically placed safe checkpoints inside long-running kernel system calls where the kernel voluntarily checks if a higher-priority process is waiting and allows a context switch. They are included to:
1. **Reduce Dispatch Latency (Real-Time / Interactive Responsiveness)**: Prevents a high-priority or real-time process from getting stuck waiting unpredictably while a low-priority process runs a lengthy system call (e.g., scanning large directories, zeroing big memory regions, or flushing buffers).
2. **Maintain Kernel Data Consistency Safely**: Unlike arbitrary preemption anywhere in kernel mode (which risks corrupting shared kernel tables without fine-grained locking everywhere), placing explicit preemption points **only where kernel data structures are in a consistent, lock-free state** achieves low latency safely with minimal locking complexity (e.g., used in Solaris 2 / preemptible UNIX kernels before full kernel preemption).

#### IN DETAIL
**Preemption points** represent a deliberate engineering compromise between fully non-preemptive monolithic kernels and fully lock-instrumented preemptive kernels:
- **Mechanism**: Developers insert explicit voluntary yield checks (`if (need_resched) schedule();`) at well-defined **quiescent locations** inside lengthy kernel loops (buffer scans, large memory clears, filesystem traversals) where all shared kernel invariants and linked structures are temporarily consistent and zero spinlocks are held.
- **Bounded Latency Benefit**: Urgent real-time or higher-priority threads no longer wait for an entire multi-millisecond system call to unwind—reducing worst-case dispatch latency to the interval between safe checkpoints without incurring fine-grained locking overhead across the entire legacy kernel codebase.

---

### 14. Application of the Priority Inheritance Protocol (Fixing Priority Inversion) [Frequency: 1]

> **Question**: Explain the application of priority inheritance protocol?

#### pre-requisite
- **Priority Inversion**: A synchronization bug where a high-priority process $H$ waits on a resource/lock currently held by a low-priority process $L$, while a medium-priority process $M$ continuously preempts $L$—effectively blocking $H$ indefinitely behind $M$!

#### Answer (Exam Writing Format)
- **Problem Addressed (Priority Inversion)**:
  - Suppose priorities satisfy $H > M > L$.
  - $L$ acquires lock $R$. Then $H$ arrives, preempts $L$, requests $R$, and blocks waiting for $L$.
  - Now $M$ becomes ready; since $M > L$, $M$ preempts $L$ and runs indefinitely, preventing $L$ from finishing and releasing $R$—so $H$ is blocked by lower-priority $M$ (unbounded priority inversion).
- **How Priority Inheritance Protocol Solves It**:
  1. When high-priority process $H$ blocks waiting for a resource held by low-priority process $L$, $L$ **temporarily inherits the higher priority of $H$**.
  2. Because $L$ now runs at priority $H$ ($> M$), medium-priority process $M$ **cannot preempt $L$**.
  3. $L$ quickly finishes its critical section and releases lock $R$.
  4. Upon release, $L$ immediately reverts to its original low priority, and $H$ acquires $R$ and executes.
- **Application**: Widely used in real-time operating systems (e.g., VxWorks/Mars Pathfinder fix, POSIX `PTHREAD_PRIO_INHERIT`) and Solaris turnstiles for adaptive mutexes/reader-writer locks.

#### IN DETAIL
**Priority Inversion** arises whenever synchronization blocking interacts with preemptive priority scheduling across three or more priority levels ($H > M > L$):
- **Inversion Mechanism**: Low-priority thread $L$ holds shared lock $R$. High-priority thread $H$ arrives, preempts $L$, requests $R$, and blocks. Subsequently, unrelated medium-priority thread $M$ (which never uses $R$) becomes runnable and preempts $L$ continuously because $M > L$—indirectly blocking highest-priority thread $H$ for $M$'s entire unbounded execution duration (a real failure observed in NASA's 1997 Mars Pathfinder RTOS).
- **Priority Inheritance Protocol (PIP)**: Temporarily elevates $L$'s effective scheduling priority to $\max(P_L, \max_{t \in \text{waiters}(R)} P_t) = P_H$ while $L$ retains $R$. $M$ can no longer preempt $L$, ensuring $L$ promptly finishes its critical section, releases $R$, drops back to $P_L$, and immediately wakes $H$.

---

### 15. T/F: "The Notion of 'Load Balancing' Counteracts with 'Processor Affinity'" [Frequency: 2]

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

### 16. Process Scheduling Overhead: Why Context Switch Duration Impacts CPU Efficiency [Frequency: 1]

> **Question**: As a first step to process virtualisation, the team has identified that a process at any point can be in one of the three states: **ready**, **running**, or **blocked**. Since you know Unix OS principles, they wanted to check with you when a process should alternate between these states.
> Considering the scenario of two different processes:
> i) **Interactive collaborative coding sessions** (user-driven, lots of I/O); and
> ii) **Periodic retraining of language models** (assume these are purely CPU-intensive jobs with no disk or network I/O),
> please explain briefly (with a diagram) the states and their transitions. Briefly describe the conditions under which transitions occur, considering the above two examples.

#### pre-requisite
- **Three-State Process Model**:
  - **Running**: Currently executing instructions on a CPU core.
  - **Ready**: Runnable and waiting in the ready queue for CPU assignment.
  - **Blocked (Waiting)**: Waiting for an external event/I/O; cannot run even if a CPU is free.

#### Answer (Exam Writing Format)
**State Transition Diagram**:
```text
                (1) Scheduled / Dispatched
         +--------------------------------------+
         |                                      v
    [  READY  ] <-------------------------- [ RUNNING ]
         ^       (2) Timer Preempt / Yield      |
         |                                      |
         | (4) I/O or Event Completes           | (3) Issues I/O / Waits
         |                                      v
         +--------------------------------- [ BLOCKED ]
```

**Transitions in the Two LangOS Scenarios**:
1. **Scenario (i): Interactive Collaborative Coding Session (I/O-bound)**:
   - Frequently cycles through **all three states**: **`Running` $\to$ `Blocked` $\to$ `Ready` $\to$ `Running`**.
   - **Running $\to$ Blocked**: Occurs whenever the editor waits for user keystrokes, reads/saves code files, or waits for collaborator updates over the network.
   - **Blocked $\to$ Ready**: Occurs when keyboard/disk/network I/O completes via hardware interrupt.
   - **Ready $\to$ Running**: Scheduler dispatches the process when CPU is free (often prioritized for fast interactive response).
   - **Running $\to$ Ready**: Happens rarely (only if a heavy syntax check/compile consumes an entire timer quantum).
2. **Scenario (ii): Periodic Retraining of Language Models (Purely CPU-intensive, Zero I/O)**:
   - Alternates **only between `Ready` and `Running`** (never enters `Blocked`, as it performs no disk/network I/O).
   - **Ready $\to$ Running**: Dispatched by the CPU scheduler.
   - **Running $\to$ Ready**: Preempted when its **hardware timer quantum expires** (so other processes get fair CPU turns).

#### IN DETAIL
Process state transitions and scheduling frequency dictate how context-switch latency ($T_{\text{ctx}}$) impacts net CPU efficiency:
- **Workload Divergence**: I/O-bound collaborative tasks traverse `Running -> Blocked -> Ready -> Running` after tiny sub-millisecond CPU bursts, incurring a high rate of context switches per unit work; compute-heavy batch workloads (`LangOS retraining`) traverse strictly `Running <-> Ready` purely at timer-quantum boundaries without `Blocked` transitions.
- **Efficiency Formula**: Over one scheduling interval of active execution $Q$ followed by switch latency $S$, true CPU utilization efficiency is bounded by $\eta = \frac{Q}{Q + S}$. Increasing $S$ linearly degrades efficiency unless $Q$ scales proportionally—which in turn increases interactive latency.

---

### 17. LangOS Numerical Scheduling: SJF vs. Round Robin & Equal-Burst Equivalence [Frequency: 1]

> **Question**: The LangOS process virtualisation team is considering the different workload types in the system to decide on a scheduling policy. The team is planning to build a scenario-based scheduling policy selector that dynamically selects a policy based on a given scenario. The team has identified two broad scenarios:
> - **Scenario A**: Three identical system processes, each requiring $20\text{ ms}$ execution time, arrive simultaneously (at $t=0$).
> - **Scenario B**: Three processes with different execution requirements: Process 1 ($10\text{ ms}$), Process 2 ($20\text{ ms}$), Process 3 ($30\text{ ms}$) arrive simultaneously (at $t=0$).
>
> Based on the above scenarios, help the team decide on the scheduling policy:
> **a.** In your opinion, what will be the benefit of using SJF and FIFO schedulers for Scenario A, considering the different metrics?
> **b.** For Scenario B, express your opinion by calculating the metrics for **SJF**, **FIFO**, and **Round Robin (time slice = 1 ms)** schedulers. Show your work clearly.
> **c.** According to you, under what characteristics would SJF deliver the same turnaround times as FIFO in LangOS?
> **d.** For what types of workloads and quantum lengths would SJF deliver the same response times as Round Robin?

#### pre-requisite
- **Scheduling Metrics** (with arrival time $A_i$, first-run time $S_i$, completion time $C_i$, burst $B_i$):
  - **Turnaround Time**: $\text{TAT}_i = C_i - A_i$
  - **Waiting Time**: $\text{WT}_i = \text{TAT}_i - B_i$
  - **Response Time**: $\text{RT}_i = S_i - A_i$
- Assume zero context-switch overhead and tie-breaking by process order ($P_1, P_2, P_3$).

#### Answer (Exam Writing Format)
##### **(a) Scenario A: Three identical 20 ms processes arriving at $t=0$**
- **Behavior**: Since all burst times ($20\text{ ms}$) and arrival times ($0\text{ ms}$) are identical, **SJF and FIFO produce the exact same execution order** ($P_1 \to P_2 \to P_3$).
- **Metrics for both SJF & FIFO**:
  - Completion times: $P_1=20, P_2=40, P_3=60\text{ ms}$.
  - $\text{Avg Turnaround Time} = \frac{20+40+60}{3} = \mathbf{40\text{ ms}}$.
  - $\text{Avg Waiting Time} = \frac{0+20+40}{3} = \mathbf{20\text{ ms}}$.
  - $\text{Avg Response Time} = \frac{0+20+40}{3} = \mathbf{20\text{ ms}}$.
- **Benefit**: Both avoid context-switch overhead (maximizing CPU throughput and turnaround/waiting optimality for batch jobs), **and FIFO is preferable in practice** over SJF because it achieves identical metrics without needing burst-length estimation.

##### **(b) Scenario B: $P_1=10\text{ ms}, P_2=20\text{ ms}, P_3=30\text{ ms}$ arriving at $t=0$**
1. **FIFO & SJF** *(here arrival order $10, 20, 30$ is already shortest-first, so FIFO and SJF give identical schedules)*:
   - Gantt: `[0–10: P1] [10–30: P2] [30–60: P3]`
   - **Completion / TAT**: $P_1=10, P_2=30, P_3=60 \implies \text{Avg TAT} = \frac{10+30+60}{3} = \mathbf{33.33\text{ ms}}$
   - **Waiting Time**: $P_1=0, P_2=10, P_3=30 \implies \text{Avg WT} = \frac{0+10+30}{3} = \mathbf{13.33\text{ ms}}$
   - **Response Time**: $P_1=0, P_2=10, P_3=30 \implies \text{Avg RT} = \frac{0+10+30}{3} = \mathbf{13.33\text{ ms}}$

2. **Round Robin ($q = 1\text{ ms}$)**:
   - Round 1 ($t=0..30$): Cycles `P1, P2, P3` for 10 rounds (each runs $10\text{ ms}$):
     - First start times ($S_i$): $P_1=0, P_2=1, P_3=2\text{ ms}$.
     - **$P_1$ completes at $t = 28\text{ ms}$** (10th turn of P1 is $[27, 28]$). Remaining: $P_2=10, P_3=20$.
   - Round 2 ($t=30..50$, wait: after P1 ends at $28$, $P_2$ runs $[28,29]$, $P_3$ runs $[29,30]$, then 9 more 2-process rounds of `P2, P3` over $18\text{ ms}$? Let's trace carefully!):
     - Wait: at $t=0..27$ (9 full 3-way cycles = $27\text{ ms}$): each ran $9\text{ ms}$ ($P_1:1, P_2:11, P_3:21$ left).
     - $t=27..28$: $P_1$ finishes! ($C_1 = 28$).
     - $t=28..29$: $P_2$ (10 left), $t=29..30$: $P_3$ (20 left).
     - Next 9 full 2-way cycles ($t=30..48$, $18\text{ ms}$): each runs $9\text{ ms}$ ($P_2:1, P_3:11$ left).
     - $t=48..49$: **$P_2$ finishes!** ($C_2 = 49$).
     - $t=49..60$: $P_3$ runs remaining $11\text{ ms}$ alone $\implies$ **$P_3$ finishes at $C_3 = 60\text{ ms}$**.
   - **Avg Turnaround Time**: $\frac{28 + 49 + 60}{3} = \frac{137}{3} = \mathbf{45.67\text{ ms}}$
   - **Avg Waiting Time**: $\frac{(28-10) + (49-20) + (60-30)}{3} = \frac{18 + 29 + 30}{3} = \frac{77}{3} = \mathbf{25.67\text{ ms}}$
   - **Avg Response Time**: $\frac{0 + 1 + 2}{3} = \mathbf{1.00\text{ ms}}$

- **Recommendation**: Choose **SJF** when minimizing **average turnaround/waiting time** (batch efficiency); choose **Round Robin** when minimizing **response time** (interactive fairness).

##### **(c) When SJF Delivers the Same Turnaround Time as FIFO**
1. **Equal Burst Lengths**: All jobs have identical CPU burst times (as in Scenario A).
2. **Sorted Arrival Order**: Jobs arrive in non-decreasing order of burst times (or simultaneous arrivals are already ordered shortest-to-longest, as in Scenario B).
3. **Zero Queueing Overlap**: Jobs arrive spaced apart such that each job finishes before the next arrives (at most 1 job in the ready queue).

##### **(d) When SJF Delivers the Same Response Time as Round Robin**
1. **Single Process**: Only 1 process exists in the ready queue.
2. **Large Quantum + Shortest-First Arrival Order**: Time quantum $q \ge \max_i(\text{burst}_i)$ (so RR becomes non-preemptive FIFO) **AND** the processes are queued in ascending order of burst length (or have equal burst lengths), making RR execute the exact same non-preemptive order and first-run times as SJF.

#### IN DETAIL
Scenario comparison between **SJF/FCFS** and **Round Robin ($q=1\text{ ms}$)** exposes the fundamental tension between **Average Turnaround Time** and **Average Response Time**:
- **SJF Optimality Proof**: Running non-preemptive shortest jobs sequentially minimizes $\sum \text{TAT}_i$ because every $1\text{ ms}$ given to a shorter job shrinks waiting time for fewer future jobs—here eliminating prolonged concurrent execution ($P_1$ exits at $10\text{ ms}$, $P_2$ at $30\text{ ms}$, avg $\text{TAT}=33.33\text{ ms}$), yet starves $P_3$'s first execution until $t=30\text{ ms}$ (avg $\text{RT}=13.33\text{ ms}$).
- **Round Robin Interactivity**: Slicing at $q=1\text{ ms}$ bounds initial response time proportionally to $q \cdot (N-1)$ (avg $\text{RT}=1.00\text{ ms}$), but interleaving all three bursts stretches earlier jobs' completion times dramatically until near the end of the entire schedule ($P_1$ exits at $28\text{ ms}$, avg $\text{TAT}=45.67\text{ ms}$). When bursts and arrivals are identical, SJF degenerates to FCFS, whereas RR sacrifices TAT even for equal bursts unless bursts align cleanly.

---

### 18. AgentOS Numerical Round Robin (q = 1 ms): Average Latency = 10.00 ms & Fairness [Frequency: 1]

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

### 19. Linux CFS Scheduler Mechanics: Does vruntime Ever Overflow? [Frequency: 1]

> **Question**: The team is considering using a CFS scheduler, but they have a curiosity. What was the main reason the CFS scheduler replaced the Linux $O(1)$ scheduler? Justify your answer:
> - a. Scheduling a task under high loads took an unpredictable amount of time
> - b. Low-priority task could wait indefinitely and starve
> - c. Interactive tasks could wait for unpredictable amounts of time to be scheduled

#### pre-requisite
- **Linux $O(1)$ Scheduler vs. CFS**: The $O(1)$ scheduler ran in constant $O(1)$ time using active/expired priority arrays, relying on complex sleep-time heuristics to guess which tasks were "interactive" and boost them. **CFS (Completely Fair Scheduler)** replaced it in Linux 2.6.23 using a red-black tree sorted by `vruntime` (virtual runtime).

#### Answer (Exam Writing Format)
- **Correct Option**: **c. Interactive tasks could wait for unpredictable amounts of time to be scheduled.**
- **Justification**:
  - While the Linux $O(1)$ scheduler always selected the next task in $O(1)$ constant time (ruling out **a**), its **complex, brittle heuristics** for identifying "interactive" tasks (based on sleep averages) often misclassified interactive workloads (e.g., bursty audio/GUI apps) or caused tasks to sit in the *expired* array until all *active* tasks finished—leading to **unpredictable scheduling delays and poor responsiveness** for interactive tasks.
  - CFS fixed this by cleanly tracking exact virtual CPU time (`vruntime`) so every runnable process—especially newly woken interactive tasks with low `vruntime`—gets fair, predictable latency without heuristic guesswork.

#### IN DETAIL
Replacing the Linux **$O(1)$ Scheduler** with the **Completely Fair Scheduler (CFS)** eliminated fragile interactivity heuristics in favor of mathematically explicit fairness:
- **Why $O(1)$ Failed on Mixed Workloads**: Although $O(1)$ selected tasks in constant time using `active` and `expired` bitmapped priority arrays, keeping audio/UI responsive forced complex bonus/penalty heuristics estimating "sleep vs. run" time. Adversarial or atypical burst patterns misclassified interactive tasks, dropping them into the `expired` array behind heavy CPU hogs for unpredictable stalls.
- **CFS Virtual Runtime Architecture**: CFS models an ideal multitasking CPU giving $N$ equal-weight tasks exact $1/N$ processor bandwidth simultaneously. Each task accumulates weighted virtual execution (`vruntime` += $\Delta t \cdot \frac{\text{NICE\_0\_LOAD}}{\text{weight}}$) inside a **Red-Black Tree** ($O(\log N)$ operations), always dispatching the leftmost node (lowest `vruntime`). Comparing unsigned wraparound via signed differences (`(s64)(a->vruntime - b->vruntime) < 0`) handles 64-bit counter wrap cleanly across centuries of continuous uptime.

---

### 20. Parameterized Round Robin Quantum Selection & Asymmetric Interactive/CPU Workload Design [Frequency: 1]

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

