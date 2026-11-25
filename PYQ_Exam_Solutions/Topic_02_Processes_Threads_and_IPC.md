# Topic 2: Processes, Threading & Concurrency, and IPC (Ch. 3–4)

> **Overview**: Process concept (`fork`/`exec`), ready vs. waiting queues, long/short/medium-term schedulers, shared-memory vs. message-passing IPC, sync vs. async communication, many-to-many threading, thread pools vs. MPL, and real-time LWP binding.  
> **Reference Material**: [Lec7-Process Concept](../Faculty_Notes/Lec7-processConcept_ch3-2025.ppt) | [Multithreaded Programming](../Faculty_Notes/MultithreadedProgramming-ch4-2025.ppt)  
> **Questions**: 17 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [Notion of a Process & Why the Process Abstraction Was Introduced](#1-notion-of-a-process-why-the-process-abstraction-was-introduced-frequency-1) — Frequency: 1
2. [Process Virtualization APIs: fork() and exec() Separation & Semantics](#2-process-virtualization-apis-fork-and-exec-separation-semantics-frequency-1) — Frequency: 1
3. [Seamless Context Switching Between Concurrent Processes](#3-seamless-context-switching-between-concurrent-processes-frequency-1) — Frequency: 1
4. [Ready Queue vs. Waiting Queue & Queue Transitions](#4-ready-queue-vs-waiting-queue-queue-transitions-frequency-1) — Frequency: 1
5. [Role of the Long-Term Scheduler (Job Scheduler)](#5-role-of-the-long-term-scheduler-job-scheduler-frequency-1) — Frequency: 1
6. [Role of the Short-Term Scheduler (CPU Scheduler / Dispatcher)](#6-role-of-the-short-term-scheduler-cpu-scheduler-dispatcher-frequency-1) — Frequency: 1
7. [Medium-Term Scheduler (Swapper): Purpose & Issues If Omitted](#7-medium-term-scheduler-swapper-purpose-issues-if-omitted-frequency-3) — **Frequency: 3**
8. [Strengths & Weaknesses of Shared Memory vs. Message Passing IPC Models](#8-strengths-weaknesses-of-shared-memory-vs-message-passing-ipc-models-frequency-1) — Frequency: 1
9. [IndixOS Preemptive Multitasking: Hardware Timer Interrupt & Context-Switch Protocol](#9-indixos-preemptive-multitasking-hardware-timer-interrupt-context-switch-protocol-frequency-1) — Frequency: 1
10. [Merits & Demerits of Synchronous (Blocking) vs. Asynchronous (Non-Blocking) IPC](#10-merits-demerits-of-synchronous-blocking-vs-asynchronous-non-blocking-ipc-frequency-1) — Frequency: 1
11. [How Multithreading Exploits Multiprocessor Systems](#11-how-multithreading-exploits-multiprocessor-systems-frequency-1) — Frequency: 1
12. [Does Multithreading Improve Both Concurrency and Parallelism on Multicore Systems?](#12-does-multithreading-improve-both-concurrency-and-parallelism-on-multicore-systems-frequency-1) — Frequency: 1
13. [Many-to-Many Threading Model on Multiprocessors: Performance for K < P, K = P, and P < K < U](#13-many-to-many-threading-model-on-multiprocessors-performance-for-k-p-k-p-and-p-k-u-frequency-2) — **Frequency: 2**
14. [Concept and Purpose of Thread Pools](#14-concept-and-purpose-of-thread-pools-frequency-2) — **Frequency: 2**
15. [T/F: "The Notion of 'Thread Pools' Is Employed to Control MPL"](#15-tf-the-notion-of-thread-pools-is-employed-to-control-mpl-frequency-1) — Frequency: 1
16. [Why a Real-Time User-Level Thread Must Be Bound to a Kernel LWP](#16-why-a-real-time-user-level-thread-must-be-bound-to-a-kernel-lwp-frequency-1) — Frequency: 1
17. [Design Choice: Spinlock vs. Blocking Semaphore Selection](#17-design-choice-spinlock-vs-blocking-semaphore-selection-frequency-1) — Frequency: 1

---

### 1. Notion of a Process & Why the Process Abstraction Was Introduced [Frequency: 1]

> **Question**: Explain the notion of process? Explain the reason for introduction of process concept.

#### Answer (Exam Writing Format)
- **Notion of a Process**:
  - A **process** is a **program in execution** (an active entity, unlike a passive executable file stored on disk).
  - It encapsulates the executing code (**Text**), current execution state (**Program Counter & CPU registers**), **Stack** (temporary function parameters/local variables), **Data/Heap** (global/dynamic memory), and OS-tracked resources in a **Process Control Block (PCB)** (PID, state, open files, memory map).
- **Reason for Introducing the Process Concept**:
  - Early computers ran one sequential program at a time. With the arrival of **multiprogramming and time-sharing**, multiple programs had to coexist in RAM, take turns on the CPU, block for I/O independently, and stay isolated from crashing or corrupting one another.
  - Formalizing the **process** abstraction allowed the OS to cleanly track each running program's independent state (PCB), pause/resume execution via context switching, enforce memory/resource protection, and allocate CPU/I/O resources fairly.

#### IN DETAIL
An OS distinguishes strictly between a **static program** and a **dynamic process**:
- **Passive Program vs. Active Execution Context**: A compiled binary on secondary storage consists only of immutable ELF headers, `.text` machine opcodes, and initialized `.data`. A **process** binds that image to runtime hardware resources: a virtual address space (`code, data, heap, stack`), processor execution state (`PC, SP, general registers, flags` saved inside its **Process Control Block — PCB**), open file descriptor tables, and signal masks.
- **Architectural Motivation**: Early batch systems executed one raw job directly on physical hardware with zero isolation. Introducing the process abstraction enabled hardware-isolated time-sharing, independent failure containment, and standardized resource accounting by giving every executing task its own virtual CPU and virtual memory domain.

---

### 2. Process Virtualization APIs: fork() and exec() Separation & Semantics [Frequency: 1]

> **Question**: The *fork team* have created two APIs for supporting process virtualization: i) `repl()`, which is similar to `fork()`; ii) `execute()`, which is the same as the `exec()`; and iii) `wait()` as offered by the standard Unix OS. The team has developed a kernel and a shell on top of it. However, they want to implement support for a new shell command, `fwc` which takes a file name as input and gives out the number of words in the file as output (eg: `fwc file.txt` gives "20" as output indicating 20 words). Since shell itself is an executing process, they want your suggestion on implementing this command. Please explain how the existing system calls can be used to achieve this.

#### pre-requisite
- **UNIX Process Creation Separation (`fork` + `exec` + `wait`)**: Separating child creation (`repl`/`fork`) from program replacement (`execute`/`exec`) allows the shell to stay alive while running external commands in isolated child processes.

#### Answer (Exam Writing Format)
To run `fwc file.txt` without terminating or overwriting the shell process itself:
1. **Parse Input**: The shell reads `"fwc file.txt"` and tokenizes it into command `"fwc"` and argument `"file.txt"`.
2. **Create Child Process (`repl()`)**: The shell calls `pid = repl()` to create an independent child process that is a clone of the shell.
3. **Replace Child Image (`execute()`)**: Inside the child (`pid == 0`), the child calls `execute("fwc", ["fwc", "file.txt", NULL])` to overwrite its memory image with the `fwc` binary, which counts words, prints `"20"`, and exits.
4. **Wait for Completion (`wait()`)**: Inside the parent shell (`pid > 0`), the shell calls `wait()` to pause until the child finishes executing, then prints the next shell prompt.

#### IN DETAIL
UNIX splits process creation (`fork()` / `repl()`) from program replacement (`exec()` / `execute()`) so parent processes can manipulate the child's execution environment before launching new code:
- **Structural Separation**: `fork()` clones the parent's PCB, virtual page mappings (efficiently via **Copy-on-Write**), and open file descriptor table, returning `0` in the child and child PID in the parent. `exec()` preserves the existing PID and open file descriptors (unless marked `FD_CLOEXEC`) while completely overwriting the calling process's virtual memory segments with the target binary.
- **Why Shells Need Both**: Had `exec()` been called directly in the shell without `fork()`, the shell binary itself would be erased upon completion of one command. Calling `exec()` only inside the freshly forked child lets the shell set up I/O redirection (`dup2()`), pipes (`pipe()`), signal handlers, and `uid`/`cwd` changes prior to executing the target binary while waiting via `waitpid()`.

---

### 3. Seamless Context Switching Between Concurrent Processes [Frequency: 1]

> **Question**: The *Fork team* wants to check how IndixOS can seamlessly switch between different running processes at any given point. The team expects IndixOS to have a mix of workloads with a lot of I/O-intensive workloads and some CPU-intensive workloads. Assuming that, as in any real case, there will not be any information on the completion time of each process and any process can arrive at any time, the team wants to check with you which could be a better policy to use for scheduling. Which one of the following scheduling policies do you recommend and why? (3 points)
> - a. Multi-level feedback queues
> - b. First come, first serve
> - c. Round Robin
> - d. Shortest Completion time first

#### Answer (Exam Writing Format)
- **Correct Option**: **a. Multi-level feedback queues (MLFQ)**.
- **Reasoning**:
  1. **No Prior Burst Knowledge Needed**: Unlike Shortest Completion Time First (STCF/SJF), MLFQ does **not** require knowing process completion times in advance—it learns behavior dynamically from past CPU usage.
  2. **Optimal for Mixed Workloads**: Processes keeping CPU bursts short (I/O-intensive / interactive jobs) stay in **high-priority queues** with fast response times, while CPU-intensive jobs automatically sink to **lower-priority queues** with larger quanta (better throughput, less switch overhead).
  3. **No Convoy Effect or Starvation**: Avoids FCFS's convoy effect on I/O-bound jobs, improves turnaround over plain Round Robin, and uses **periodic priority boost (aging)** to prevent starvation of CPU-bound jobs.

#### IN DETAIL
Choosing a CPU scheduler when switching seamlessly among heterogeneous unknown processes highlights why **Multi-Level Feedback Queues (MLFQ)** dominate general-purpose OS design:
- **Limitations of Alternatives**: **FCFS** is non-preemptive and suffers severe convoy effects when long CPU bursts precede short interactive tasks. **SCTF / SJF** requires knowing exact future CPU-burst lengths in advance, which is unobtainable for arbitrary live workloads. Fixed **Round Robin** prevents starvation but treats short interactive latency-sensitive tasks identically to background CPU hogs.
- **MLFQ Adaptive Mechanism**: MLFQ maintains $k$ priority queues with progressively larger quanta at lower tiers. New and I/O-blocking tasks remain in top high-priority queues (achieving low interactive response time), whereas CPU-intensive processes consuming full quanta are dynamically demoted to lower queues (preventing frequent preemption overhead), with periodic priority boost (aging) eliminating lower-tier starvation.

---

### 4. Ready Queue vs. Waiting Queue & Queue Transitions [Frequency: 1]

> **Question**: Explain the terms "waiting queue" and "Ready queue" in the Operating System. When the process enters and exits from waiting queue.

#### Answer (Exam Writing Format)
- **Ready Queue**: Holds all processes loaded in main memory that are **ready and runnable**, waiting only for the CPU scheduler to assign a processor core.
- **Waiting (Blocked) Queue**: Holds processes that **cannot execute even if the CPU is idle**, because they are waiting for a specific event (e.g., disk/network I/O completion, semaphore/lock, child exit).
- **Waiting Queue Transitions**:
  - **Enters Waiting Queue**: When a currently **running** process requests an asynchronous event or resource that is not immediately ready (e.g., initiates disk I/O or waits on a semaphore/timer).
  - **Exits Waiting Queue**: When the awaited event completes (e.g., disk interrupt arrives or semaphore is signaled), the OS transitions the process from the waiting queue back to the **ready queue**.

#### IN DETAIL
Process scheduling relies on distinct OS kernel queues—implemented via intrusive linked pointers inside Process Control Blocks (PCBs)—to separate **runnable CPU contention** from **asynchronous resource waiting**:
- **Ready Queue (Single or Per-Core)**: Holds PCBs in the `READY` state whose memory mappings are active and whose only prerequisite to execute is CPU availability. Dispatched purely by the Short-Term Scheduler.
- **Waiting / Blocked Queues (Per-Event / Per-Device)**: Held separately per I/O controller, mutex, semaphore, or condition variable. Moving blocked processes out of the Ready Queue ensures the Short-Term Scheduler wastes zero cycles scanning unrunnable processes; when an ISR signals event completion, the OS moves the specific PCB back to the Ready Queue (`Blocked -> Ready`).

---

### 5. Role of the Long-Term Scheduler (Job Scheduler) [Frequency: 1]

> **Question**: What do you understand by the term "long term scheduler"?

#### pre-requisite
- **Three Scheduler Types**: Operating systems can employ Long-Term (Job), Medium-Term (Swapper), and Short-Term (CPU) schedulers to manage process admission, memory presence, and CPU dispatch.

#### Answer (Exam Writing Format)
- **Definition**: The **Long-Term Scheduler** (also called the **Job Scheduler**) selects processes from the job pool (on disk) and loads them into main memory (the ready queue) for execution.
- **Key Functions**:
  1. **Controls Degree of Multiprogramming (MPL)**: Decides how many processes are simultaneously resident in memory.
  2. **Balances Job Mix**: Attempts to admit a balanced mix of **I/O-bound** and **CPU-bound** processes so neither the CPU nor I/O devices sit idle.
  3. **Execution Frequency**: Executes infrequently (seconds or minutes apart, only when a job arrives or terminates), so it can afford more complex selection logic.

#### IN DETAIL
The **Long-Term Scheduler (Job Scheduler)** acts as the coarse-grained admission-control gate between secondary spool storage (`NEW` state) and resident multiprogramming (`READY` state):
- **Controlling Degree of Multiprogramming ($N$)**: By capping how many active processes enter DRAM concurrently, it guarantees sufficient resident physical frames per process so page-fault rates remain low and CPU utilization stays high.
- **Workload Mix Balancing**: Admitting exclusively CPU-bound jobs leaves disk/network DMA engines idle, while admitting exclusively I/O-bound jobs leaves the CPU starved for compute work; the Long-Term Scheduler maintains an interleaved mix of both so hardware pipelines stay balanced over long time horizons.

---

### 6. Role of the Short-Term Scheduler (CPU Scheduler / Dispatcher) [Frequency: 1]

> **Question**: Explain role of short term scheduler.

#### Answer (Exam Writing Format)
- **Definition**: The **Short-Term Scheduler** (also called the **CPU Scheduler**) selects one process from the in-memory **Ready Queue** and allocates a CPU core to it via the dispatcher.
- **Key Characteristics & Role**:
  1. **Execution Frequency**: Executes very frequently (every few milliseconds—whenever a process blocks on I/O, terminates, or a timer quantum expires).
  2. **Speed Requirement**: Because it runs so often, its decision logic must be **extremely fast** ($O(1)$ or $O(\log N)$) to avoid wasting significant CPU overhead on scheduling itself.
  3. **Policy Goal**: Directly governs CPU utilization, response time, turnaround time, and fairness across ready processes.

#### IN DETAIL
The three classical OS schedulers form a clear **time-scale hierarchy**:
- **Long-Term Scheduler** (seconds to minutes): Controls overall **process admission** from disk spool into RAM, shaping the static degree of multiprogramming and coarse CPU/I/O workload mix.
- **Medium-Term Scheduler** (hundreds of milliseconds to seconds): Performs **dynamic swapping** (`suspend/resume`) to temporarily evict memory-heavy processes during acute RAM pressure and break thrashing spirals.
- **Short-Term Scheduler / Dispatcher** (milliseconds to microseconds): Selects the next runnable PCB from the in-memory **Ready Queue** on every timer tick, blocking syscall, or wakeup interrupt, switching hardware address spaces (`CR3`) and CPU register state via sub-millisecond context switches.

---

### 7. Medium-Term Scheduler (Swapper): Purpose & Issues If Omitted [Frequency: 3]

> **Question**: Justify the requirement of medium-term scheduler by explaining the corresponding issues if you do not employ medium term scheduler.
> *(2022 Quiz 1 Q7 & 2025 AOS Midsem Q1(d) variants: "Justify the requirement of / Explain why medium-term scheduler is introduced.")*

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

### 8. Strengths & Weaknesses of Shared Memory vs. Message Passing IPC Models [Frequency: 1]

> **Question**: What are the strengths and weaknesses of the two models of inter-process communication?

#### pre-requisite
- **Two IPC Models**: Processes have isolated virtual address spaces by default. To cooperate, they use either **Shared Memory** (a common RAM region mapped into both address spaces) or **Message Passing** (`send()`/`receive()` via OS kernel).

#### Answer (Exam Writing Format)
| Feature | Shared Memory Model | Message Passing Model |
| :--- | :--- | :--- |
| **Mechanism** | OS maps a shared RAM region into participating processes' address spaces; reads/writes occur directly at memory speed. | Processes exchange messages via OS kernel primitives (`send()` and `receive()`). |
| **Strengths** | 1. **High Speed / Low Overhead**: After setup, zero kernel intervention or data copying per access.<br>2. **Ideal for Large Data**: Efficiently shares bulk arrays/buffers. | 1. **Easier & Safer**: No explicit locks needed from programmers; avoids race conditions on shared state.<br>2. **Distributed Ready**: Works seamlessly across networked machines as well as single machines. |
| **Weaknesses** | 1. **Synchronization Burden**: Programmers must explicitly synchronize access (locks/semaphores) to prevent race conditions.<br>2. **Single-Machine Only**: Does not natively work over networks. | 1. **Higher Overhead**: Every message incurs system calls (mode switches) and copying between user and kernel space.<br>2. **Lower Bandwidth** for very large frequent data transfers. |

#### IN DETAIL
**Shared Memory** and **Message Passing** present an architectural tradeoff between raw throughput and kernel-enforced isolation:
- **Shared Memory Architecture**: Two or more processes ask the OS kernel (`shm_open` + `mmap`) to map identical physical RAM frames into each process's virtual page tables. After setup, communication executes at **full DRAM memory speed** without kernel traps or buffer copies; however, correctness falls entirely on user-space synchronization primitives (mutexes, atomics, memory barriers) to prevent data races.
- **Message Passing Architecture**: Every communication invokes explicit kernel traps (`send()`/`recv()`, pipes, sockets) where the kernel copies payload data into protected kernel buffers and across address-space boundaries. Although syscalls and copies add per-message overhead, the kernel enforces implicit synchronization, boundary safety, and seamless extension across distributed nodes where physical memory is not shared.

---

### 9. IndixOS Preemptive Multitasking: Hardware Timer Interrupt & Context-Switch Protocol [Frequency: 1]

> **Question**: At any given point, there will be many processes executing in the IndixOS. At present, the team relies on the cooperative approach to pass control back to the OS. However, the team is unclear about how a pre-emptive approach can be implemented to get control back to the OS during process execution... The team requires your suggestion based on your understanding of Unix about the mechanism and protocol that the OS needs to use to ensure that it can seamlessly switch between multiple processes.

#### pre-requisite
- **Cooperative vs. Preemptive**: Cooperative multitasking relies on processes voluntarily yielding via system calls (`yield`, `read`, `exit`); a buggy `while(1)` loop hangs the whole CPU. Preemptive multitasking uses hardware to force periodic OS intervention.

#### Answer (Exam Writing Format)
To switch from cooperative to **preemptive multitasking**, IndixOS needs two coordinated mechanisms:
1. **Hardware Timer Interrupt Mechanism**:
   - Configure a programmable **hardware timer** (e.g., PIT/APIC) during OS boot to raise periodic interrupts every time quantum (e.g., every 1–10 ms).
   - Register the OS **Timer Interrupt Service Routine (ISR)** in the hardware **Interrupt Vector Table (IVT)** at boot time.
   - Ensure timer configuration/disabling instructions are **privileged** (Kernel Mode only) so user processes cannot disable preemption.
2. **Context-Switching Protocol**:
   - When the timer expires while Process B runs, CPU hardware automatically switches from **User Mode $\to$ Kernel Mode**, saves B's Program Counter (`PC`) and flags onto B's kernel stack, and jumps to the OS Timer ISR.
   - The ISR saves Process B's remaining CPU registers into **B's Process Control Block (PCB)** and moves B to the `READY` queue.
   - The **CPU Scheduler** selects Process C from the `READY` queue, loads C's saved registers and **Page Table base register** (e.g., `CR3`) from C's PCB, and executes a **return-from-interrupt** (`iret`/`sysret`) switching back to User Mode to resume C seamlessly.

#### IN DETAIL
Implementing **preemptive multitasking** removes reliance on cooperative user-space `yield()` behavior, guaranteeing the OS kernel regains hardware control reliably even if user tasks enter infinite loops:
- **Protected Hardware Timer & IVT Hook**: During boot, the kernel configures a programmable hardware timer (e.g., LAPIC timer / PIT) to raise periodic interrupts every quantum $q$, registering the OS Timer ISR inside the hardware Interrupt Vector Table (IVT). Marking timer-control instructions/MMIO registers privileged ensures user processes can never mask or reprogram the preemption clock.
- **Atomic Trap & Context Switch Protocol**: On timer expiration, CPU hardware atomically saves minimal hardware state (`PC, SP, PSW/EFLAGS`) onto the kernel stack, transitions from User Mode (`Ring 3`) to Kernel Mode (`Ring 0`), and branches to the Timer ISR. The ISR saves remaining general-purpose registers into the running process's PCB, calls the Short-Term Scheduler to pick the next Ready PCB, reloads `CR3` (if switching address spaces) and the new process's saved registers, and executes `iret`/`sysret` back into User Mode.

---

### 10. Merits & Demerits of Synchronous (Blocking) vs. Asynchronous (Non-Blocking) IPC [Frequency: 1]

> **Question**: Explain merits and demerits of synchronous and asynchronous communication.

#### pre-requisite
- **Message-Passing Synchronization Modes**: In IPC (`send()` / `receive()`), **synchronous** means **blocking** (sender/receiver wait until message transfer completes), whereas **asynchronous** means **non-blocking** (sender resumes immediately after handing the message to the OS buffer).

#### Answer (Exam Writing Format)
1. **Synchronous (Blocking) Communication**:
   - *Merits*: Simple design with zero unbounded intermediate buffering overhead; creates a clean **rendezvous** when both `send()` and `receive()` are blocking, making synchronization and error confirmation straightforward.
   - *Demerits*: Reduces concurrency—sender stalls idle waiting for a slow or busy receiver, risking deadlocks if cyclic waits occur.
2. **Asynchronous (Non-Blocking) Communication**:
   - *Merits*: Maximizes concurrency and throughput—sender immediately continues computation after sending; decouples execution speeds/clocks of sender and receiver.
   - *Demerits*: Requires OS kernel memory buffers (queues can overflow under fast producers), complicates error handling/delivery confirmation, and requires extra polling/callbacks to check completion.

#### IN DETAIL
Synchronous vs. asynchronous IPC reflects fundamentally different **control-flow coupling** and **kernel buffering** assumptions:
- **Synchronous (Blocking Send/Receive)**: Enforces a strict **rendezvous**—whichever endpoint arrives early suspends in a kernel wait queue until its peer executes the matching operation. This bounds intermediate kernel buffer usage to zero (direct copy from sender space to receiver space once matched) and simplifies ordering proofs, at the cost of reduced pipeline concurrency and vulnerability to deadlock if circular dependencies arise.
- **Asynchronous (Non-Blocking Send / Buffered Queues)**: Decouples execution rates by introducing bounded kernel FIFO message queues. Senders immediately resume execution after enqueuing, improving producer–consumer throughput; however, kernel memory must be allocated and bounded carefully—requiring explicit flow-control backpressure (blocking or `EAGAIN` errors) if consumers lag indefinitely.

---

### 11. How Multithreading Exploits Multiprocessor Systems [Frequency: 1]

> **Question**: Explain how the multi-threading framework can exploit the power of multi-processing architectures as compared to the process framework?

#### Answer (Exam Writing Format)
- **Single-Threaded Process Limitation**: A traditional single-threaded process has only **one stream of execution** (one PC/register state), so at any instant it can execute on **at most one CPU core**, leaving additional cores unused by that application.
- **How Multithreading Exploits Multiprocessing**:
  1. A multithreaded process contains **multiple independent execution streams (threads)** sharing the same code, data, and open files.
  2. On a multiprocessor/multicore system, the OS schedules different threads of the *same* process onto **separate CPU cores simultaneously** (true parallelism), achieving significant speedup with much lower creation and context-switch overhead than spawning heavyweight processes.

#### IN DETAIL
Multithreading unlocks parallel scalability on multicore/multiprocessor hardware without duplicating heavy per-process OS structures:
- **Single-Threaded Limit**: Because a traditional process owns only **one hardware execution context** (`PC, SP, register file`), the OS scheduler can dispatch it onto at most **one CPU core** at any single instant; all extra cores remain unused by that application unless multiple heavyweight processes are spawned and linked via slower IPC.
- **Multi-Core Thread Dispatching**: Because threads within one process share the same virtual address space (page tables, heap, `.data`, open file descriptors) while maintaining independent **Thread Control Blocks (TCBs)**, stacks, and register contexts, the OS kernel scheduler assigns distinct runnable kernel threads concurrently to separate physical cores—performing true parallel computation over shared heap data with minimal inter-thread communication cost and zero TLB flush overhead.

---

### 12. Does Multithreading Improve Both Concurrency and Parallelism on Multicore Systems? [Frequency: 1]

> **Question**: Do you agree that "multithreading framework improves both Concurrency and Parallelism on multicore system?" Justify your answer.

#### pre-requisite
- **Concurrency**: Structuring a program so multiple tasks make progress over overlapping time intervals (even via time-slicing on 1 core).
- **Parallelism**: Actually executing multiple tasks *simultaneously* at the exact same physical instant on multiple cores.

#### Answer (Exam Writing Format)
- **Yes, I agree** that a multithreading framework improves **both Concurrency and Parallelism** on a multicore system:
  1. **Improved Concurrency**: Breaking an application into multiple independent execution threads (e.g., UI thread, network listener thread, worker threads) allows tasks to interleave cleanly. When one thread blocks on I/O or a lock, other threads within the same process immediately continue executing without stalling the entire application.
  2. **Improved Parallelism**: Unlike a single-threaded process (which can execute on at most **one** CPU core at any instant), a multithreaded process exposes multiple independent instruction streams that the OS kernel can schedule onto **different CPU cores simultaneously**, achieving true parallel speedup.

#### IN DETAIL
**Concurrency** and **parallelism** address orthogonal execution dimensions, and multithreading strengthens both on multicore hardware:
- **Improved Concurrency (Logical Structural Decoupling)**: Separating logically independent activities (UI event loop, network socket reader, disk flusher, worker compute) into distinct threads lets an application keep making progress whenever individual threads block on I/O—even on a single core via OS time-slicing.
- **Improved Parallelism (Simultaneous Hardware Execution)**: On multicore processors, those concurrent threads map directly onto independent CPU cores simultaneously (`T1` on Core 0, `T2` on Core 1), reducing wall-clock execution latency roughly proportionally to core count for parallelizable workloads (modulo synchronization and Amdahl's serial fraction).

---

### 13. Many-to-Many Threading Model on Multiprocessors: Performance for K < P, K = P, and P < K < U [Frequency: 2]

> **Question**: Consider a multiprocessor system and a multi threaded program using many-to-many threading model. Let the number of user-level threads in the program be more than the number of processors in the system. Discuss the performance implications of the following scenarios:
> (i) The number of kernel threads allocated to the program is less than the number of processors.
> (ii) The number of kernel threads allocated to the program is equal to the number of processors.
> (iii) The number of kernel threads allocated to the program is greater than the number of processors but less than the number of user-level threads.

#### pre-requisite
- **Many-to-Many Threading Model**: $U$ user-level threads are multiplexed onto $K$ kernel-level threads, and only kernel threads ($K$) are scheduled onto $P$ physical CPUs. Given $U > P$.

#### Answer (Exam Writing Format)
- **(i) $K < P$ (Fewer kernel threads than processors)**:
  - **System Underutilization**: Only at most $K$ processors can run threads of this program simultaneously, leaving $(P - K)$ processors idle for this program despite having $U > P$ runnable user threads. Any blocking syscall reduces active cores even further.
- **(ii) $K = P$ (Kernel threads equal to processors)**:
  - **Full Peak Utilization, Vulnerable to Blocking**: All $P$ processors can execute simultaneously when threads are CPU-bound. However, if any user thread executes a **blocking system call** through its kernel thread, that kernel thread blocks and leaves one physical processor idle (even though other user threads are ready to run!).
- **(iii) $P < K < U$ (More kernel threads than processors, fewer than user threads)**:
  - **High CPU Utilization & Resilience to Blocking (Optimal)**: If one kernel thread blocks on I/O, another runnable kernel thread is immediately scheduled onto that freed CPU core, keeping all $P$ processors saturated while multiple user threads efficiently multiplex over the $K$ kernel threads without excessive kernel-thread overhead.

#### IN DETAIL
Under the **Many-to-Many (`M:N`)** threading architecture, execution concurrency spans two decoupled scheduling layers: user-space threads ($U$) multiplexed by a runtime library onto Kernel-Schedulable Entities / LWPs ($K$), which the OS schedules onto physical processors ($P$):
- **$K < P$ (Kernel-Thread Bottleneck)**: The OS kernel perceives only $K$ dispatchable execution contexts, leaving at least $P - K$ CPU cores idle regardless of how many $U$ user threads are runnable.
- **$K = P$ (Saturated Optimal Mapping for Pure Compute)**: Every core can execute one thread concurrently without excess kernel context switches, provided none of the $K$ LWPs block in synchronous syscalls.
- **$P < K < U$ (Resilience to Blocking Syscalls)**: Having surplus kernel threads ($K > P$) guarantees that if one or more LWPs block inside kernel I/O, the OS seamlessly schedules alternate ready LWPs onto freed cores, maintaining full $P$-way core utilization at modest extra kernel TCB overhead.

---

### 14. Concept and Purpose of Thread Pools [Frequency: 2]

> **Question**: What is the purpose of thread pools?
> *(2022 Midsem Q1(c) variant: "Discuss the concept of thread pools.")*

#### Answer (Exam Writing Format)
A **thread pool** pre-creates a bounded number of worker threads at startup that sit idle waiting for tasks in a shared queue. Its key purposes are:
1. **Reduced Thread Creation/Destruction Overhead**: Servicing an incoming request with an already-existing thread from the pool is much faster than dynamically creating and destroying a thread per request.
2. **Bounds Concurrency / Resource Exhaustion**: Caps the maximum number of concurrent threads in the system, preventing memory/CPU exhaustion on busy servers (controls degree of multiprogramming / MPL).
3. **Predictable Performance & Graceful Degradation**: When all pool threads are busy, new requests wait cleanly in a task queue rather than crashing the system by spawning thousands of threads.

#### IN DETAIL
Thread pools amortize thread lifecycle costs and enforce bounded concurrency over dynamic bursty workloads:
- **Kernel Lifecycle Amortization**: Spawning and tearing down native OS threads repeatedly per short request incurs repeated `clone()` syscalls, stack `mmap()` allocation, page-table initialization, kernel TCB setup, and scheduler enqueueing—often exceeding the compute duration of short tasks themselves. Pre-allocated worker threads reuse persistent stacks and TCBs continuously across tasks from a shared synchronization queue.
- **Resource Exhaustion Protection**: Unbounded thread-per-request architectures under heavy traffic surges exhaust virtual/physical stack memory (`OOM`) and trigger severe scheduler context-switch thrashing; a bounded pool caps active execution contexts cleanly at an optimal core-tuned concurrency limit.

---

### 15. T/F: "The Notion of 'Thread Pools' Is Employed to Control MPL" [Frequency: 1]

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

### 16. Why a Real-Time User-Level Thread Must Be Bound to a Kernel LWP [Frequency: 1]

> **Question**: Assume an OS maps user-level threads to the kernel using many-to-many model where a mapping is done through LWPs (Light Weight Processes). Furthermore the system allows the developers to create real-time threads. Is it necessary to bound a real-time thread to an LWP? What will happen if you do not map a real-time thread to LWP? Explain.

#### pre-requisite
- **LWP (Light-Weight Process)**: An intermediate virtual processor data structure connecting user-level threads to kernel-level threads in a Many-to-Many (or Two-Level) threading model (e.g., Solaris). The OS kernel schedules **LWPs**, *not* unbound user-level threads directly.

#### Answer (Exam Writing Format)
- **Yes, it is strictly necessary to bind a real-time thread to a dedicated LWP** (creating a 1-to-1 **bound thread** inside the two-level model).
- **What Happens If a Real-Time Thread Is NOT Bound to an LWP**:
  1. **Invisible to the Kernel Real-Time Scheduler**: In an unbound many-to-many pool, the kernel only sees and assigns priorities to the underlying LWPs—not the user-level threads multiplexed on top. Thus, the OS cannot honor the real-time thread's strict priority or preempt lower-priority threads immediately for it.
  2. **Unbounded Contention & Starvation Behind Unbound Pool**: If all shared LWPs in the pool are currently busy running normal threads or **blocked in kernel system calls / page faults**, the ready real-time user thread gets stuck waiting in user space for a free LWP—causing unpredictable dispatch latency and **missed real-time deadlines**.
- **With LWP Binding**: Binding dedicates an exclusive kernel thread/LWP with real-time priority (`SCHED_FIFO`/`SCHED_RR`) directly to that thread, ensuring immediate kernel preemption and deterministic scheduling.

#### IN DETAIL
In a **Two-Level (`M:N` + Bound)** threading model, an OS kernel schedules only **LWPs / Kernel Threads**, remaining completely blind to multiplex rules running inside user-space thread libraries:
- **Hazards of Unbound Multiplexing for Real-Time Work**: An unbound real-time user thread faces two failure modes: (1) it can be preempted or starved inside user space when all pooled LWPs block on synchronous syscalls or run other lower-priority user tasks; and (2) the OS kernel cannot apply hard real-time scheduling classes (`SCHED_FIFO`/`SCHED_RR`) or global real-time priority inheritance directly to user threads it cannot distinguish.
- **Role of Dedicated Binding**: Binding a critical thread **1-to-1 to a dedicated LWP** guarantees continuous kernel visibility, isolated kernel scheduling priority, and direct preemption safety on physical CPU cores.

---

### 17. Design Choice: Spinlock vs. Blocking Semaphore Selection [Frequency: 1]

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

