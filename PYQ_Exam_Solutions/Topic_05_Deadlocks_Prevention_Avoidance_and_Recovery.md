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
Enforcing a **strict total resource acquisition ordering** ($R_1 < R_2 < \dots < R_n$) eliminates deadlock structurally by making **Circular Wait** mathematically impossible:
- **Graph-Theoretic Proof of Acyclicity**: Define a Resource-Allocation Wait-For Graph where a directed edge $P_i \to P_j$ exists only if $P_i$ currently holds some lock $r$ and blocks requesting lock $s$ held by $P_j$. Under strict lock ordering, every process requesting $s$ while holding $r$ must obey $r < s$. Along any wait cycle $P_0 \to P_1 \to \dots \to P_k \to P_0$, the minimum lock held along the chain would satisfy $r_0 < r_1 < \dots < r_k < r_0$—a mathematical contradiction!
- **Design Constraint**: Because processes acquire resources strictly in monotonically ascending rank ($a \to b \to c$, etc.), no runtime Banker's check or wait-for graph probe is needed; deadlock is prevented statically at compile/design time.

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
Comparing **Deadlock Prevention (eliminating Circular Wait)** against **Deadlock Avoidance (Banker's Algorithm)** contrasts **static structural invariants** against **dynamic future-state reachability verification**:
- **Prevention (Total Resource Ordering)**: Zero runtime CPU computation per request (`Wait(r)` checks no safety table); works even without a priori maximum resource declarations. Its cost is architectural inflexibility (developers must fix lock acquisition order across all modules) and potential idle waiting when a process must lock a low-numbered resource early solely to satisfy rank order.
- **Avoidance (Banker's Algorithm)**: Permits arbitrary dynamic acquisition order while guaranteeing zero deadlock, *provided* every process declares exact `Max[i][j]` bounds upfront and $O(m \cdot n^2)$ safety graph checks verify that a complete finishing sequence remains reachable after every grant—incurring heavy per-request locking/CPU overhead and false rejections whenever unsafe states merely *risk* deadlock without making it certain.

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
Distinguishing **Deadlock** from **Starvation** clarifies why breaking Circular Wait via strict resource ranking remains starvation-free under standard fair lock queues:
- **No Circular Blocking Cycles**: Strict lock ordering ensures every process holding maximal resource $R_k$ either needs no further locks or requests strictly higher locks $R_j > R_k$, terminating eventually and freeing locks downward without deadlock cycles.
- **Fair Admission Guarantees Bounded Wait**: As long as each individual resource lock uses a FIFO-fair (strong semaphore / queued) wake-up policy rather than arbitrary LIFO or random contention, every thread waiting on $R_i$ advances steadily to the front as finite holders finish—preventing indefinite postponement (**starvation**). *(Only starvation-prone unfair schedulers or unbounded retry spin-loops reintroduce starvation externally.)*

---

### 4. T/F: "Deadlock Prevention Protocols Improve CPU Utilization" [Frequency: 1]

> **Question**: Identify whether the following statement is TRUE or FALSE. If the statement is FALSE, correct it and justify the corrected sentence. If the statement is TRUE, justify it. Restrict the justification to a few sentences: "Deadlock prevention protocols improve CPU utilization"

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: Deadlock prevention protocols generally **reduce (degrade)** CPU utilization (and device/system throughput).
- **Justification**: Deadlock prevention enforces restrictive static rules to negate one of Coffman's four conditions—such as requiring processes to request all resources upfront (violating Hold-and-Wait), releasing/rolling back held resources (violating No Preemption), or acquiring resources in a strict global order (violating Circular Wait). These restrictions cause resources to remain idle while held prematurely, force repeated process aborts/restarts, and block eligible work unnecessarily, thereby lowering overall concurrency and **reducing CPU utilization**.

#### IN DETAIL
Every **Deadlock Prevention** protocol trades lower hardware/resource utilization for an absolute static guarantee against circular deadlock:
- **Violating Hold-and-Wait**: Releasing all held resources before requesting additional ones—or pre-allocating a process's entire lifetime peak resource demand at startup—locks idle devices/buffers away from concurrent processes for long execution intervals, throttling multiprogramming concurrency and CPU utilization.
- **Violating Circular Wait / No-Preemption**: Strict lock ordering forces processes to acquire low-rank resources prematurely before they are actually needed (extending lock hold time unnecessarily), while forced resource preemption aborts partially completed computations, wasting CPU cycles on repeated rollbacks and retries.

---

### 5. T/F: "Deadlock Prevention Violating No-Preemption Suffers from Starvation" [Frequency: 1]

> **Question**: Identify whether the following statement is TRUE or FALSE. If the statement is FALSE, correct it and justify the corrected sentence. If the statement is TRUE, justify it. *"Deadlock prevention algorithm designed by violating no preemption condition suffers from starvation problem."*

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: To negate the "No Preemption" condition, when a process holding resources requests another unavailable resource, its held resources are forcibly preempted (or released), forcing the process to roll back and retry later. Under continuous contention from other processes, an unlucky process can repeatedly acquire partial resources, get preempted, and restart indefinitely without ever completing—resulting in **starvation**.

#### IN DETAIL
Deadlock prevention via **violating No-Preemption** eliminates permanent deadlock by forcibly rolling back blocked holders, directly creating a **livelock / starvation hazard**:
- **Rollback Amplification Under Contention**: Suppose Process $P$ holds $\{R_1, R_2, R_3\}$ and blocks requesting $R_4$. Violating No-Preemption forces $P$ to surrender $\{R_1, R_2, R_3\}$ so peers can proceed. If competing threads repeatedly hold $R_4$ or re-acquire $\{R_1, R_2, R_3\}$ whenever $P$ releases them, $P$ can cycle infinitely through partial progress $\to$ forced rollback $\to$ restart without ever completing (**starvation / livelock**).
- **Mitigation**: Practical systems avoiding starvation under rollback-based concurrency control (such as transactional memory or database 2PL wait-die) add monotonic timestamps (`Wait-Die` / `Wound-Wait`) or randomized exponential backoff to guarantee eventual completion.

---

### 6. T/F: "Deadlock Avoidance Improves Resource Utilization Over Violating No-Preemption" [Frequency: 1]

> **Question**: *"Deadlock avoidance approach improves resource utilization over deadlock prevention approach based on the violation of no pre-emption condition."*

#### Answer (Exam Writing Format)
- **TRUE**.
- **Justification**: Deadlock prevention by violating "No Preemption" forces the OS to forcibly take away held resources whenever a process's additional request cannot be met, causing aborted work, costly rollbacks, and wasted restarts. In contrast, deadlock avoidance (e.g., Banker's Algorithm) dynamically verifies safe states and lets processes retain their allocated resources and complete uninterruptedly whenever safe, sustaining significantly higher hardware and CPU utilization.

#### IN DETAIL
Comparing **Deadlock Avoidance (Banker's Algorithm)** with **Preemption-Based Prevention** evaluates conservative admission control against destructive rollbacks:
- **Preservation of Completed Work**: Violating No-Preemption discards all CPU time, I/O operations, and intermediate state accumulated prior to preemption whenever a resource request fails—wasting throughput and requiring checkpoint/undo infrastructure (only trivial for stateless/idempotent resources like CPU registers or swapable memory pages).
- **Non-Destructive Scheduling in Avoidance**: Banker's Algorithm never confiscates resources from an active process; once a request passes the safety check, the process proceeds deterministically to completion, preserving 100% of execution progress while keeping utilization high (rejecting only requests leading into unsafe states).

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
Deciding between **proactive Avoidance (Banker's Algorithm)** and **reactive Detection & Recovery** hinges on **deadlock probability**, **check overhead**, and **recovery blast radius**:
- **Banker's Avoidance Overhead**: Runs an $O(m \cdot n^2)$ matrix simulation on *every single dynamic allocation*, assumes fixed process counts and static `Max` claims, and rejects safe-in-practice allocations whenever worst-case concurrent demands conflict—impractical for general-purpose OSes where dynamic memory/file requests occur millions of times per second.
- **Optimistic Detection & Recovery (Real-World Standard)**: Assumes deadlocks are rare, permitting immediate zero-overhead allocation. Background cycle detection (Wait-For Graph traversal $O(V+E)$) runs periodically or upon long timeouts. Recovery via victim termination/rollback incurs clean localized loss only during genuine deadlocks—yielding far higher everyday system throughput (used by databases like PostgreSQL/MySQL via lock wait-for detection).

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
The **Banker's Safety Algorithm** verifies whether the current state $(\text{Available}, \text{Allocation}, \text{Need})$ admits at least one global completion sequence $\langle P_{k_1}, P_{k_2}, \dots, P_{k_n} \rangle$:
- **Monotonic Work-Vector Invariant**: Initialize $\text{Work} = \text{Available}$. Any process $P_i$ satisfying $\text{Need}_i \le \text{Work}$ element-wise can be granted its entire worst-case remaining demand, execute to completion, and release all its held resources: $\text{Work} \leftarrow \text{Work} + \text{Allocation}_i$.
- **Why Safe Sequence Order Matters**: Because $\text{Work}$ grows **strictly monotonically** at each step, once a process qualifies ($\text{Need}_i \le \text{Work}$), it *never* becomes disqualified later. Thus checking reachability greedily—starting with tight vectors like $\langle 1, 1, 2, 0 \rangle$ where only $P_2$ qualifies initially—is guaranteed both sound and complete ($O(m \cdot n^2)$) without needing exponential backtracking across permutations.

---

