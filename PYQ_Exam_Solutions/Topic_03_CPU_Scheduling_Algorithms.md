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
Notice why students sometimes find this surprising: *"Wait, isn't multiprogramming supposed to overlap I/O so $T$ is LESS than $t_1 + t_2 + \dots + t_N$?"*
Usually yes—*unless* the hidden taxes of sharing a single machine overwhelm the I/O overlap!
Think of cooking 3 recipes one-at-a-time vs. juggling all 3 simultaneously in a tiny kitchen:
- Every time you switch recipes (**context switch**), you waste time putting away pans and reading where you left off (**TLB/cache pollution**).
- If all 3 recipes don't fit on the counter (**RAM full $\to$ thrashing**), you spend half your time running back and forth to the basement fridge (**disk page faults**)!
- Result: Juggling all $N$ together can take *longer* ($T > \sum t_i$) than running them sequentially one by one!

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
Contrast this with **2025 AOS Midsem Q3** (which asked how $T$ can be *greater* than $\sum t_i$)!
Here, suppose Process A takes $t_A = 100\text{ ms}$ (**10 ms CPU + 90 ms Disk**), and Process B takes $t_B = 40\text{ ms}$ (**20 ms CPU + 20 ms Network**).
- Run sequentially: $100 + 40 = 140\text{ ms}$.
- Run concurrently in a multiprogrammed OS: Process A uses the CPU for 10 ms then waits 90 ms on the disk—during that 90 ms disk wait, Process B easily finishes its entire 40 ms CPU+Network job! So BOTH finish in **100 ms = $\max(t_A, t_B)$**!

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
This is actually the exact mechanism used in **classic UNIX scheduling** (and MLFQ with aging)!
Think of priority like "how tired/full you are from eating CPU time recently":
- An **I/O-bound process** runs for just 1 ms, then leaves for 50 ms to wait for disk or keyboard input. When it comes back, it has consumed almost *zero* CPU recently, so the scheduler lets it jump right to the front, do its tiny 1 ms burst, and get back to I/O—keeping disks and users happy!
- Meanwhile, a **CPU-bound process** eats 100% of the CPU when it gets a turn, so its "recent CPU usage" shoots up and its priority drops. Does it starve forever? **No!** First, whenever the I/O jobs are waiting on the disk, the CPU-bound job gets all the remaining CPU cycles. Second, every second that passes, the kernel decays everyone's "recent CPU" counter (aging), so a neglected CPU-bound job naturally becomes "fresh" again and gets scheduled!

---

### 4. T/F: OS Gives Higher Priority to I/O-Bound Over CPU-Bound Processes [Frequency: 3]

> **Question**: Identify whether the following statement is TRUE or FALSE. If the statement is FALSE, correct it and justify the corrected sentence. If the statement is TRUE, justify it. Restrict the justification to a few sentences. "As compared to CPU-bound process, OS gives higher priority for I/O bound process."

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
Look at what happens when you flip priorities upside down (putting 10-hour heavy math jobs ahead of 1-ms keyboard/disk jobs):
- **I/O devices sit starving**: Disks and network cards sit completely empty doing **zero work** while the CPU crunches a long math loop—because the tiny 1 ms I/O-bound jobs that feed work to those disks aren't allowed onto the CPU!
- **Users freeze**: Interactive terminals lag for seconds waiting behind batch math jobs!
- Thus, wrong classification destroys *both* **interactive responsiveness** AND **overall multi-device system throughput**!

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
Imagine a grocery checkout lane with 10 people buying a single bottle of water (**I/O-bound processes**, needing 1 second at the register) standing behind 1 person with three overflowing shopping carts (**CPU-bound process**, needing 15 minutes).
Under **FCFS** (strict line, no express lane, no preemption):
- Everyone with 1 bottle waits 15 minutes just to spend 1 second at the register! Their average waiting time skyrockets.
- Worse for the whole system: while the CPU is busy scanning the 3 carts, the disk/I/O lanes sit empty doing zero work! Then all 10 water-bottle buyers leave to wait on I/O at once, leaving the CPU sitting completely idle! That traffic jam is called the **Convoy Effect**.

---

### 7. Problem in Non-Preemptive CPU Scheduling (Convoy Effect & Unresponsive Starvation) [Frequency: 2]

> **Question**: Some simple OSs use non-preemptive scheduling. Give an example of a problem that can occur when the OSs uses non-preemptive scheduling.

#### Answer (Exam Writing Format)
- **Problem (Unbounded Delay / Failure to Meet Time Constraints)**: In non-preemptive scheduling, once a process (or kernel routine) holds the CPU, it cannot be forcibly interrupted until it blocks or finishes.
- **Example**: If a low-priority batch process (or infinite loop) holds the CPU and an urgent **high-priority / real-time** process (or interactive user event) becomes ready via an interrupt, the urgent process cannot run until the running process voluntarily yields—violating real-time deadlines and hurting interactive response time.

#### IN DETAIL
Suppose a simple non-preemptive OS starts running a heavy scientific simulation that won't touch I/O for 60 seconds. Half a second later, a pacemaker monitor or audio-playback thread wakes up needing 1 ms of CPU to avoid skipping or missing a hard deadline! Because the scheduler cannot preempt the running job, the urgent task sits helplessly in the ready queue for 59.5 seconds—making non-preemptive scheduling unusable for interactive or hard real-time workloads.

---

### 8. Conflicting Pairs of CPU Scheduling Criteria [Frequency: 1]

> **Question**: Discuss how the following pairs of scheduling criteria conflict in certain settings: **CPU utilization** and **response time**.

#### Answer (Exam Writing Format)
- **Conflict**: Optimizing for maximum **CPU utilization** can degrade (increase) **response time**, and vice versa.
- **Explanation & Setting**:
  - To maximize **CPU utilization**, the scheduler avoids frequent preemptions/context switches and lets long CPU-bound jobs run uninterruptedly (e.g., FCFS or huge time quanta), eliminating switch overhead—**but** newly arrived interactive processes must wait long periods, severely worsening **response time**.
  - Conversely, achieving fast **response time** requires tiny time slices (frequent preemptions/Round Robin), which increases context-switch overhead and lowers effective **CPU utilization**.

#### IN DETAIL
Imagine a surgeon trying to maximize **"percentage of hour spent cutting"** (CPU utilization) vs. **"how fast new emergency walk-ins get greeted"** (response time):
- If the scheduler never interrupts long jobs, 100% of CPU cycles go to useful job work (zero context-switch waste!), **but** interactive users wait seconds for a single keypress (**bad response time**).
- If the scheduler slices time every 0.1 ms to give instant response, interactive feel is blazing fast, **but** half the CPU time is wasted saving/restoring registers and flushing caches (**lower effective CPU utilization**)!

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
Think of MLFQ as a bouncer giving VIP wristbands to anyone who takes quick 9-millisecond turns on a game machine (assuming anyone playing for 10+ ms is a greedy hog and sending them to the slow basement!).
A sneaky user can **game** this rule easily: run heavy computation for **9.9 ms**, then call `write(1 byte to /dev/null)` or `sleep(1 µs)`! The OS thinks *"Oh, short I/O-bound interactive program!"* and puts you right back at the front of the VIP queue—letting you hog the CPU! *(Modern MLFQs defeat this by tracking **total CPU time consumed** at each priority level rather than resetting the budget on every I/O.)*

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
Look at why ordinary Round Robin frustrates batch jobs: a batch job needing 10 seconds with a 50 ms quantum gets interrupted **200 times**, wasting time on 200 context switches and cold caches.
**Progressive Round Robin** tries to be clever: *"Start small (50 ms) so short interactive jobs finish fast; if you're still running on lap 2, lap 3, lap 4, we know you're a long batch job, so we'll give you 100 ms, 150 ms, 200 ms so you stop wasting context-switch overhead!"*

Sounds great—until you spot the fatal flaw:
- Suppose Batch Job A has been running for 20 rounds. Its personal quantum is now **$1000\text{ ms}$ (1 full second)**!
- Right now, a user hits a key in an interactive editor. Even though the interactive job only needs 2 ms, it has to sit and wait until Batch Job A finishes its huge 1-second time slice (plus any other long batch jobs ahead of it!).
- In other words, **giving older jobs huge quanta destroys the worst-case response-time guarantee of ordinary Round Robin ($q \times (N-1)$)** and brings back the **Convoy Effect**! (That's why real systems like MLFQ keep interactive jobs in *high-priority* short-quantum queues and drop batch jobs to *lower-priority* long-quantum queues, where incoming interactive jobs immediately *preempt* the batch job!)

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
Classic UNIX didn't try to guess how long a program would run in advance. Instead, it measured **what the process actually did in the last second**:
- **Interactive processes** (typing in an editor, running shell commands) spend 99% of their time waiting on humans or disks, so their `CPU / real_time` ratio is tiny. Result: UNIX automatically gives them top user priority, keeping the terminal snappy!
- **CPU-heavy loops** burn 100% of their slice, so their priority drops every second—preventing them from starving interactive users. Meanwhile, if multiple CPU hogs exist at the bottom tier, they take fair 1-second turns, and aging ensures nobody gets permanently stuck!

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
Suppose a low-priority background backup script calls a heavy `sync()` or `unlink()` system call in classic UNIX, stepping inside the kernel for **50 milliseconds**.
At millisecond 1, an emergency missile-guidance or airbag-sensor process (**hard real-time process**, deadline = 5 ms) wakes up!
- The interrupt controller runs the quick wakeup handler and marks the real-time thread `READY` with top priority.
- **Problem**: Because classic UNIX protected kernel data structures simply by forbidding *process preemption inside kernel mode*, the scheduler isn't allowed to kick the backup script off the CPU until its 50 ms syscall finishes!
- Result: The real-time process misses its 5 ms deadline by 45 ms—catastrophic for hard real-time systems!

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
Suppose a low-priority background job calls a system call that loops 100,000 times inside the kernel.
- **No Preemption in Kernel**: An urgent high-priority thread wakes up and waits helplessly for the entire 100,000-step syscall to finish!
- **Preemption Everywhere**: Every single line of kernel code now needs complex spinlocks/mutexes—hard and slow in classic kernels!
- **Preemption Points (Smart Compromise!)**: Inside that 100,000-step kernel loop, at clean moments where no internal kernel table is half-updated, the kernel puts a quick check (`if (need_resched) yield_cpu()`). Now the high-priority thread runs almost immediately while kernel invariants stay 100% intact!

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
A famous real-world disaster illustrates this exact bug: **NASA's Mars Pathfinder (1997)** kept crashing and rebooting on Mars because of **Priority Inversion**!
Here's how it happens:
- **VIP ($H$)** needs a conference room key currently held by **Intern ($L$)** for 2 minutes.
- VIP arrives and waits at the door for Intern to finish.
- Suddenly **Middle Manager ($M$)** walks in. Because Middle Manager outranks Intern, the scheduler kicks Intern off the CPU to run Middle Manager's 3-hour presentation!
- Now Intern can't finish the 2-minute work, so Intern never drops the key, and **VIP stays stuck waiting 3 hours behind Middle Manager!**

**Priority Inheritance Protocol** fixes this elegantly: the moment VIP starts waiting on Intern's lock, the OS says *"While holding VIP's key, Intern temporarily wears VIP's badge!"*. Now Middle Manager cannot interrupt Intern; Intern finishes in 2 minutes, hands over the key, returns the VIP badge, and VIP runs right away!

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
Think of **Processor Affinity** as keeping a chef at their own prepped kitchen station (all knives and ingredients already warm in their local cache!). **Load Balancing** taps that chef on the shoulder and yells *"Switch to Station #4 because it's empty!"*—evening out the workload across cores, at the exact cost of leaving behind all their warm cache contents!

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
Think of these two workloads in action:
- **Collaborative Coding** runs for just 0.5 ms to draw a character on screen, then immediately hits `read(keyboard)` or `recv(network)` and **blocks**—spending 99% of its life jumping between `Blocked` and `Ready`, getting tiny quick sips of `Running`!
- **Language Model Retraining** (with zero I/O assumed) never waits for a disk or user—it wants to crunch matrix math nonstop! Without a timer interrupt, it would stay `Running` forever until finished. Because UNIX/LangOS is preemptive, every time its time slice ticks out, the timer kicks it **Running $\to$ Ready**, giving interactive sessions a turn, and then schedules it right back **Ready $\to$ Running**—skipping `Blocked` completely!

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
Look at the sharp contrast in **Scenario B** between **SJF/FIFO** and **Round Robin ($q=1\text{ ms}$)**:
- **SJF** finishes quick job $P_1$ at $10\text{ ms}$ and $P_2$ at $30\text{ ms}$ (**Avg TAT = 33.33 ms**), **but** poor $P_3$ sits completely untouched for **30 ms** before running a single instruction (**Avg RT = 13.33 ms**)!
- **Round Robin** lets $P_1, P_2, P_3$ *all* start within 2 ms (**Avg RT = 1.00 ms**—amazing interactivity!), **but** by stretching everyone out simultaneously, $P_1$ doesn't finish until **28 ms** and $P_2$ until **49 ms** (**Avg TAT = 45.67 ms**)!
That is the fundamental **Response Time vs. Turnaround Time trade-off** in CPU scheduling!

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
Watch out for a common exam pitfall: on the scanned paper, a student ticked `c. 11.33 ms`, which is **mathematically impossible**! Let's see why:
- Total CPU needed across all 3 processes = $5 + (2 + 2) + 4 = 13\text{ ms}$, and the CPU is never idle from $t=0$ to $t=13$.
- Even if you test *every possible* tie-breaking order when simultaneous events occur (new arrival vs. I/O return vs. preempted process), the only achievable averages are `9.67 ms`, **`10.00 ms`**, `10.33 ms`, and `10.67 ms` (`32/3`). So **`a. 10.00 ms`** is unequivocally the correct MCQ option!
- Also notice why $P_2$ has $\text{TAT} = 12\text{ ms}$: Turnaround Time always measures **wall-clock duration from arrival (`1 ms`) to final exit (`13 ms`)**, which naturally includes its 4 ms I/O wait inside `[5, 9]`.

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
Why did Linux throw away the super-fast "$O(1)$ scheduler"?
- Look carefully at the options:
  - **a** is the *opposite* of $O(1)$—the $O(1)$ scheduler got its name specifically because picking a task took constant $O(1)$ time regardless of load! (CFS is actually $O(\log N)$!)
  - **b** isn't the primary bug—$O(1)$ swapped the `active` and `expired` arrays cleanly once `active` emptied, so tasks eventually ran.
  - **c** was the real pain point! To keep mice/keyboards/audio snappy, the $O(1)$ scheduler contained hundreds of lines of messy guesswork trying to reward "sleepy" interactive jobs. Certain interactive patterns fooled the heuristic, dumping them into the `expired` queue behind CPU hogs for unpredictable stretches! **CFS** threw out all those heuristics in favor of one elegant rule: *track `vruntime` in a red-black tree and always run whoever received the least CPU time so far!*

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
This classic Silberschatz textbook question tests whether you recognize **Priority** and **MLFQ** as universal "super-frameworks":
- **Priority** lets you plug in *any* key to pick who runs next: plug in `1 / burst_time` and you get **SJF**; plug in `-arrival_time` and you get **FCFS**!
- **MLFQ** lets you configure $k$ queues with any scheduling rule inside each: set $k = 1$ queue running FIFO and you instantly recover **FCFS** (and set $k = 1$ with quantum $q$ and you recover **RR**)!
- **RR vs. SJF** have completely different axes: RR slices by **clock time** without knowing job durations, whereas SJF sorts by **job duration** without slicing by clock time—so neither can mimic the other across arbitrary workloads!

---
