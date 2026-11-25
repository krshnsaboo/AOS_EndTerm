# Topic 5: Deadlocks — Prevention, Avoidance & Recovery (Ch. 8)

> **Overview**: Global lock ordering to prevent circular wait, circular-wait prevention vs. Banker's avoidance, starvation/utilization tradeoffs across prevention protocols, detect-and-recover vs. Banker's, and complete Banker's Algorithm safety-sequence numericals.  
> **Reference Material**: [deadlocks_OS.ppt](../Faculty_Notes/deadlocks_OS.ppt)  
> **Questions**: 8 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [AgentOS Resource Lock Ordering (a < b < c < d) to Prevent Circular Wait](#1-agentos-resource-lock-ordering-a-b-c-d-to-prevent-circular-wait-frequency-1) — Frequency: 1
2. [Circular-Wait Deadlock Prevention vs. Deadlock Avoidance (Banker's Algorithm): Overhead & Throughput](#2-circular-wait-deadlock-prevention-vs-deadlock-avoidance-bankers-algorithm-overhead-throughput-frequency-3) — **Frequency: 3**
3. [YES/NO: Can Deadlock Prevention Violating "Circular Wait" Suffer from Starvation?](#3-yesno-can-deadlock-prevention-violating-circular-wait-suffer-from-starvation-frequency-1) — Frequency: 1
4. [T/F: "Deadlock Prevention Protocols Improve CPU Utilization"](#4-tf-deadlock-prevention-protocols-improve-cpu-utilization-frequency-1) — Frequency: 1
5. [T/F: "Deadlock Prevention Violating No-Preemption Suffers from Starvation"](#5-tf-deadlock-prevention-violating-no-preemption-suffers-from-starvation-frequency-1) — Frequency: 1
6. [T/F: "Deadlock Avoidance Improves Resource Utilization Over Violating No-Preemption"](#6-tf-deadlock-avoidance-improves-resource-utilization-over-violating-no-preemption-frequency-1) — Frequency: 1
7. [Deadlock Avoidance (Banker's Algorithm) vs. Detection-and-Recovery (Kill Thread)](#7-deadlock-avoidance-bankers-algorithm-vs-detection-and-recovery-kill-thread-frequency-1) — Frequency: 1
8. [Banker's Algorithm Numerical: Four Resource Types <6,4,4,2> Safe State Check](#8-bankers-algorithm-numerical-four-resource-types-6442-safe-state-check-frequency-1) — Frequency: 1

---

### 1. AgentOS Resource Lock Ordering (a < b < c < d) to Prevent Circular Wait [Frequency: 1]

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

### 2. Circular-Wait Deadlock Prevention vs. Deadlock Avoidance (Banker's Algorithm): Overhead & Throughput [Frequency: 3]

> **Question**: Compare the deadlock prevention algorithms by preventing circular-wait with the deadlock avoidance algorithm (i.e., the banker's algorithm) with respect to the following: runtime overheads and system throughput.

#### pre-requisite
- **Preventing Circular Wait**: Assign a strict total ordering $F(R_1) < F(R_2) < \dots < F(R_m)$ to all resource types and enforce that processes only request resources in strictly increasing order.
- **Banker's Algorithm (Avoidance)**: Requires maximum resource claims up front and runs a safety-check algorithm ($O(m \cdot n^2)$) before granting any resource request.

#### Answer (Exam Writing Format)
| Criterion | Deadlock Prevention (Eliminating Circular Wait) | Deadlock Avoidance (Banker's Algorithm) |
| :--- | :--- | :--- |
| **Runtime Overhead** | **Very Low / Nearly Nil ($O(1)$)**: Resource acquisition order is enforced statically at design/compile time (or via a simple $O(1)$ index comparison). No expensive state-graph or matrix checking per request. | **High ($O(m \cdot n^2)$ per request)**: Where $m$ = resource types and $n$ = processes. The safety algorithm must execute on *every* resource allocation request. |
| **System Throughput** | **Lower**: Imposes rigid ordering constraints—processes must acquire resources in fixed order even if they don't need an earlier resource yet (holding locks longer) or reject/restart out-of-order requests, reducing concurrency and throughput. | **Higher (under safe contention)**: No processes are aborted or artificially forced to hold unused early locks just for numbering; resources are dynamically granted whenever a safe sequence exists, maintaining higher concurrency and throughput. |

#### IN DETAIL
Think of these two approaches like managing intersections:
1. **Preventing Circular Wait (Static Rules)**: Like a rule saying *"You MUST visit Shop 1 before Shop 2 before Shop 3."*
   - **Overhead**: Almost zero! The traffic cop doesn't need to calculate anything—just check that your next shop ID is bigger than your current shop ID.
   - **Throughput**: Poor! Even if Shop 3 is completely empty right now and you won't need Shop 1 for an hour, you're forced to grab Shop 1 early and hold it, blocking others and lowering overall throughput!
2. **Banker's Algorithm (Dynamic Safety Check)**: Like an air-traffic controller checking the entire radar before clearing every takeoff.
   - **Overhead**: Expensive! Every time *any* process asks for *any* resource, the OS simulates finishing all $n$ processes across $m$ resource types ($O(m \cdot n^2)$ work).
   - **Throughput**: Much higher concurrency—as long as there's at least one guaranteed way for everyone to finish ("safe state"), requests are granted flexibly without artificial ordering restrictions!

---

### 3. YES/NO: Can Deadlock Prevention Violating "Circular Wait" Suffer from Starvation? [Frequency: 1]

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

### 4. T/F: "Deadlock Prevention Protocols Improve CPU Utilization" [Frequency: 1]

> **Question**: Identify whether the following statement is TRUE or FALSE. If the statement is FALSE, correct it and justify the corrected sentence. If the statement is TRUE, justify it. Restrict the justification to a few sentences: "Deadlock prevention protocols improve CPU utilization"

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: Deadlock prevention protocols generally **reduce (degrade)** CPU utilization (and device/system throughput).
- **Justification**: Deadlock prevention enforces restrictive static rules to negate one of Coffman's four conditions—such as requiring processes to request all resources upfront (violating Hold-and-Wait), releasing/rolling back held resources (violating No Preemption), or acquiring resources in a strict global order (violating Circular Wait). These restrictions cause resources to remain idle while held prematurely, force repeated process aborts/restarts, and block eligible work unnecessarily, thereby lowering overall concurrency and **reducing CPU utilization**.

#### IN DETAIL
Deadlock prevention buys **100% deadlock safety** at a heavy performance price:
- **Negating Hold-and-Wait**: A process must ask for a printer, tape drive, and 10 locks at $t=0$ even if it only needs the printer in the last 10 seconds—keeping the printer locked and idle so other processes block instead of using the CPU.
- **Negating No Preemption**: If a process can't get lock B, the OS strips lock A away and forces rollback/restart, wasting CPU cycles already done!
- **Negating Circular Wait**: Rigid ordering restricts parallelism and delays work.

Because processes wait longer, hold resources unnecessarily, or restart work, fewer runnable processes stay active and **CPU utilization drops** compared to optimistic or dynamic approaches!

---

### 5. T/F: "Deadlock Prevention Violating No-Preemption Suffers from Starvation" [Frequency: 1]

> **Question**: Identify whether the following statement is TRUE or FALSE. If the statement is FALSE, correct it and justify the corrected sentence. If the statement is TRUE, justify it. *"Deadlock prevention algorithm designed by violating no preemption condition suffers from starvation problem."*

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: To negate the "No Preemption" condition, when a process holding resources requests another unavailable resource, its held resources are forcibly preempted (or released), forcing the process to roll back and retry later. Under continuous contention from other processes, an unlucky process can repeatedly acquire partial resources, get preempted, and restart indefinitely without ever completing—resulting in **starvation**.

#### IN DETAIL
Imagine trying to assemble a 4-piece puzzle under a strict "violate No-Preemption" rule: *"Whenever piece #4 isn't on the table, drop pieces #1, #2, and #3 back into the box and start completely over!"*
Every time you pick up pieces #1–#3, another faster player grabs #4—forcing you to dump your progress and retry from zero forever! That endless acquire-preempt-rollback loop is classic **starvation**!

---

### 6. T/F: "Deadlock Avoidance Improves Resource Utilization Over Violating No-Preemption" [Frequency: 1]

> **Question**: *"Deadlock avoidance approach improves resource utilization over deadlock prevention approach based on the violation of no pre-emption condition."*

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: Deadlock prevention by violating "No Preemption" forces the OS to forcibly take away held resources whenever a process's additional request cannot be met, causing aborted work, costly rollbacks, and wasted restarts. In contrast, deadlock avoidance (e.g., Banker's Algorithm) dynamically verifies safe states and lets processes retain their allocated resources and complete uninterruptedly whenever safe, sustaining significantly higher hardware and CPU utilization.

#### IN DETAIL
Violating "No Preemption" is destructive: if Process A holds Resource 1 and Resource 2 fails, the OS steals Resource 1 and throws away Process A's progress! Deadlock avoidance checks in advance whether granting Resource 2 stays safe—if yes, nobody loses work or rolls back, keeping resources productively utilized!

---

### 7. Deadlock Avoidance (Banker's Algorithm) vs. Detection-and-Recovery (Kill Thread) [Frequency: 1]

> **Question**: Consider the following ways of handling deadlock: (1) banker's algorithm (2) deadlock detect and kill thread, releasing all resources ... *(truncated in source PDF)*

#### pre-requisite
- **Deadlock Handling Strategies**: Avoidance (prevent unsafe states dynamically via Banker's Algorithm) vs. Detection & Recovery (allow allocations freely, periodically detect cycles/deadlock, and recover by aborting/preempting processes).

#### Answer (Exam Writing Format)
*(Comparing Banker's Algorithm vs. Deadlock Detection & Recovery on overhead, throughput, and safety)*:
1. **Banker's Algorithm (Deadlock Avoidance)**:
   - *Pros*: System never enters a deadlocked state; no process ever needs to be killed or lose completed work.
   - *Cons*: High runtime overhead ($O(m \cdot n^2)$) on **every** resource request; requires processes to declare maximum resource needs in advance (often impossible in general-purpose OSes); overly conservative, reducing concurrency.
2. **Deadlock Detection & Kill Thread (Recovery)**:
   - *Pros*: Zero runtime overhead during normal resource requests; higher throughput and concurrency when deadlocks are rare; does not require advance maximum-claim declarations.
   - *Cons*: Periodic detection overhead ($O(m \cdot n^2)$); when deadlock occurs, killing threads discards partially completed computation, risks leaving shared data inconsistent, and can cause starvation if the same victim is repeatedly killed.

#### IN DETAIL
Think of **Banker's Algorithm** as checking every single intersection across a city before letting *any* car pull out of a driveway: zero gridlocks, high continuous planning overhead, and lots of cars forced to wait even when they'd be fine!
**Detection & Recovery** is how real OSes and databases operate: let cars drive freely (fast!) and only call a tow truck if a deadlock cycle actually happens—accepting the occasional cost of aborting and rolling back one victim transaction!

---

### 8. Banker's Algorithm Numerical: Four Resource Types <6,4,4,2> Safe State Check [Frequency: 1]

> **Question**: Assume a system with four resource types $C = \langle 6, 4, 4, 2 \rangle$, and the maximum claim table and current allocation table shown below. The resource allocator is considering allocating resources according to the table shown below. Is this safe state? Why and why not?
>
> | Process | Max ($R_0, R_1, R_2, R_3$) | Current Alloc ($R_0, R_1, R_2, R_3$) |
> | :---: | :---: | :---: |
> | **$P_0$** | `3  2  1  1` | `2  0  1  1` |
> | **$P_1$** | `1  2  0  2` | `1  1  0  0` |
> | **$P_2$** | `1  1  2  0` | `1  1  0  0` |
> | **$P_3$** | `3  2  1  0` | `1  0  1  0` |
> | **$P_4$** | `2  1  0  1` | `0  1  0  1` |

#### pre-requisite
- **Banker's Safety Algorithm**:
  1. Compute $\text{Available} = C - \sum_i \text{Alloc}_i$.
  2. Compute $\text{Need}_i = \text{Max}_i - \text{Alloc}_i$.
  3. Find a sequence $\langle P_{i_1}, \dots, P_{i_5} \rangle$ where each $\text{Need}_{i_k} \le \text{Work}$, updating $\text{Work} \leftarrow \text{Work} + \text{Alloc}_{i_k}$ after each step. If all processes finish, the state is **Safe**.

#### Answer (Exam Writing Format)
1. **Compute `Total Allocated` and `Available`**:
   - $\sum \text{Alloc} = \langle 2+1+1+1+0,\; 0+1+1+0+1,\; 1+0+0+1+0,\; 1+0+0+0+1 \rangle = \langle 5, 3, 2, 2 \rangle$.
   - $\text{Available} = C - \sum \text{Alloc} = \langle 6, 4, 4, 2 \rangle - \langle 5, 3, 2, 2 \rangle = \mathbf{\langle 1, 1, 2, 0 \rangle}$.
2. **Compute `Need = Max - Alloc`**:
   - $P_0$: $\langle 3,2,1,1 \rangle - \langle 2,0,1,1 \rangle = \langle 1, 2, 0, 0 \rangle$
   - $P_1$: $\langle 1,2,0,2 \rangle - \langle 1,1,0,0 \rangle = \langle 0, 1, 0, 2 \rangle$
   - $P_2$: $\langle 1,1,2,0 \rangle - \langle 1,1,0,0 \rangle = \langle 0, 0, 2, 0 \rangle$
   - $P_3$: $\langle 3,2,1,0 \rangle - \langle 1,0,1,0 \rangle = \langle 2, 2, 0, 0 \rangle$
   - $P_4$: $\langle 2,1,0,1 \rangle - \langle 0,1,0,1 \rangle = \langle 2, 0, 0, 0 \rangle$
3. **Find Safe Sequence** (starting with $\text{Work} = \langle 1, 1, 2, 0 \rangle$):
   - **Step 1**: $\text{Need}(P_2) = \langle 0,0,2,0 \rangle \le \langle 1,1,2,0 \rangle$ $\implies$ Run **$P_2$**! $\text{Work} = \langle 1,1,2,0 \rangle + \langle 1,1,0,0 \rangle = \langle 2, 2, 2, 0 \rangle$.
   - **Step 2**: $\text{Need}(P_0) = \langle 1,2,0,0 \rangle \le \langle 2,2,2,0 \rangle$ $\implies$ Run **$P_0$**! $\text{Work} = \langle 2,2,2,0 \rangle + \langle 2,0,1,1 \rangle = \langle 4, 2, 3, 1 \rangle$.
   - **Step 3**: $\text{Need}(P_3) = \langle 2,2,0,0 \rangle \le \langle 4,2,3,1 \rangle$ $\implies$ Run **$P_3$**! $\text{Work} = \langle 4,2,3,1 \rangle + \langle 1,0,1,0 \rangle = \langle 5, 2, 4, 1 \rangle$.
   - **Step 4**: $\text{Need}(P_4) = \langle 2,0,0,0 \rangle \le \langle 5,2,4,1 \rangle$ $\implies$ Run **$P_4$**! $\text{Work} = \langle 5,2,4,1 \rangle + \langle 0,1,0,1 \rangle = \langle 5, 3, 4, 2 \rangle$.
   - **Step 5**: $\text{Need}(P_1) = \langle 0,1,0,2 \rangle \le \langle 5,3,4,2 \rangle$ $\implies$ Run **$P_1$**! $\text{Work} = \langle 6, 4, 4, 2 \rangle = C$.
- **Conclusion**: **YES, this is a SAFE state**, because there exists a valid safe execution sequence (e.g., **$\langle P_2, P_0, P_3, P_4, P_1 \rangle$**) allowing every process to satisfy its maximum claim and terminate without deadlock.

#### IN DETAIL
Notice how tight Step 1 is: when we start with $\text{Available} = \langle 1, 1, 2, 0 \rangle$, **no process except $P_2$** can finish! ($P_0$ needs $R_1=2$, $P_1$ needs $R_3=2$, $P_3$ & $P_4$ need $R_0=2$.)
Luckily, $P_2$ only needs $\langle 0, 0, 2, 0 \rangle$, finishes cleanly, and returns its $\langle 1, 1, 0, 0 \rangle$, raising `Work` to $\langle 2, 2, 2, 0 \rangle$! That immediately unlocks $P_0$, $P_3$, and $P_4$—and as soon as $P_0$ or $P_4$ returns $R_3$, $P_1$ finishes too! Since *all* 5 processes finish along $\langle P_2, P_0, P_3, P_4, P_1 \rangle$, deadlock can be completely avoided.

---
