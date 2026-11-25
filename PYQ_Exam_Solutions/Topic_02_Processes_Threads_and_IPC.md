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
9. [IndixOS Process IPC Design: Shared Memory vs. Message Passing](#9-indixos-process-ipc-design-shared-memory-vs-message-passing-frequency-1) — Frequency: 1
10. [Merits & Demerits of Synchronous (Blocking) vs. Asynchronous (Non-Blocking) IPC](#10-merits-demerits-of-synchronous-blocking-vs-asynchronous-non-blocking-ipc-frequency-1) — Frequency: 1
11. [How Multithreading Exploits Multiprocessor Systems](#11-how-multithreading-exploits-multiprocessor-systems-frequency-1) — Frequency: 1
12. [Does Multithreading Improve Both Concurrency and Parallelism on Multicore Systems?](#12-does-multithreading-improve-both-concurrency-and-parallelism-on-multicore-systems-frequency-1) — Frequency: 1
13. [Many-to-Many Threading Model on Multiprocessors: Performance for K < P, K = P, and P < K < U](#13-many-to-many-threading-model-on-multiprocessors-performance-for-k-p-k-p-and-p-k-u-frequency-2) — **Frequency: 2**
14. [Concept and Purpose of Thread Pools](#14-concept-and-purpose-of-thread-pools-frequency-2) — **Frequency: 2**
15. [T/F: "The Notion of 'Thread Pools' Is Employed to Control MPL"](#15-tf-the-notion-of-thread-pools-is-employed-to-control-mpl-frequency-1) — Frequency: 1
16. [Why a Real-Time User-Level Thread Must Be Bound to a Kernel LWP](#16-why-a-real-time-user-level-thread-must-be-bound-to-a-kernel-lwp-frequency-1) — Frequency: 1
17. [Design Choice: When to Use Multiple Processes vs. Multiple Threads](#17-design-choice-when-to-use-multiple-processes-vs-multiple-threads-frequency-1) — Frequency: 1

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
A `.exe` or compiled binary on your SSD is just a **recipe book** sitting passively on a shelf (**a program**). The moment you actually start cooking it, **a process** is born: you now have a live cook tracking *which line of the recipe they're on* (`PC`), ingredients on the counter (`Heap/Data`), scratchpad notes (`Stack`), and a clipboard for the kitchen manager (`PCB`)!
Without the **process** abstraction, an OS running 50 concurrent tasks on 4 CPU cores would have no clean unit of **isolation**, **resource ownership**, or **pausing/resuming**!

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
Why doesn't the shell just call `execute("fwc")` directly? Because `execute()` (`exec()`) **completely overwrites** the calling process's code, data, and stack! If the shell ran `execute()` itself, the shell would vanish forever the moment `fwc` finished!

Instead, UNIX uses a three-step dance:
1. **`repl()` (`fork()`)** clones the shell into a disposable child copy.
2. **Child calls `execute("fwc", ...)`**: Only the *child* gets replaced by the word-counting program, runs, prints `20`, and dies.
3. **Parent shell calls `wait()`**: The real shell naps politely until the child finishes (keeping terminal I/O tidy) and then wakes up ready for your next command!

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
Why not the other three?
- **b. FCFS**: Terrible convoy effect—one CPU hog freezes all I/O-bound jobs behind it!
- **c. Round Robin**: Gives equal slices to everyone—great response time, **bad turnaround time** because short jobs keep getting stuck behind long CPU jobs at the same queue level!
- **d. STCF**: Optimal *if* you have a crystal ball telling you exact completion times ahead of time—impossible in a real general-purpose OS!
- **a. MLFQ**: The real-world winner! It guesses *"short burst = interactive/I/O-bound"* by observing behavior dynamically, keeping interactive tasks snappy at the top while letting batch work run smoothly at the bottom (with periodic priority boosts so nothing starves!).

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
Trace a process through a quick restaurant analogy:
- **Ready Queue** = Customers holding menus who are **100% ready to order** as soon as the cashier (`CPU`) is free.
- **Waiting Queue** = Customers who already placed an order and are sitting at tables **waiting for their food (`I/O event`) to finish cooking**. Even if the cashier has zero customers at the counter, a customer waiting on food cannot order again until the kitchen dings the bell (**interrupt**), sending them back to the Ready Queue!

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
Picture a popular restaurant:
- **Long-Term Scheduler (The Host at the Front Door)**: Controls how many people are allowed inside the restaurant (**degree of multiprogramming**). If too few are let in, tables sit empty (underutilized CPU). If too many crowd in, waiters can't move (thrashing / memory overload). And a smart host tries to seat both fast-eating diners (**I/O-bound**) and slow-course diners (**CPU-bound**) so the kitchen and waitstaff stay evenly busy.
- Compare this to:
  - **Short-Term Scheduler (CPU Scheduler)**: Runs every few milliseconds to pick *which ready process in RAM* runs on the CPU next.
  - **Medium-Term Scheduler (Swapper)**: Temporarily swaps already-admitted processes out to disk and back in to adjust the mix or free RAM.

*(Note: Many modern time-sharing OSes like desktop Linux/Windows do not use a strict long-term scheduler—they admit all started processes directly and rely on memory limits / medium-term swapping.)*

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
Compare the **three schedulers**:
- **Long-Term Scheduler** decides *who gets admitted from disk into RAM* (runs every few minutes).
- **Medium-Term Scheduler** decides *who gets swapped out/in* to balance RAM & process mix (runs every few seconds).
- **Short-Term Scheduler** is the sprinter running every **1–10 milliseconds**: it picks the very next ready process in RAM to put onto the CPU! If the Short-Term Scheduler took even 2 ms to decide, you'd waste 20% of the CPU just picking who runs! That's why it has to be blazingly fast.

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
Without a medium-term scheduler, once a process enters RAM via the long-term scheduler, **the ONLY way it ever leaves RAM is by terminating!**
Think what goes wrong on a real system:
- Ten idle browser tabs or suspended jobs hog 95% of RAM forever while active jobs starve and **thrash**!
- Or 5 jobs all enter an intense CPU-crunching phase at once, leaving the disk completely idle, while an I/O job sits locked outside RAM!
The medium-term scheduler solves both by temporarily **suspending and swapping out** selected processes to disk, then swapping them back in cleanly when RAM frees up or the mix needs rebalancing!

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
Imagine two coworkers collaborating in an office:
- **Shared Memory** is like putting a **shared whiteboard** between their desks. Once the manager (OS) sets up the whiteboard, both workers can write and read directly at lightning speed without asking the manager each time. **Catch**: If both grab the same marker or write over each other's numbers at the same time, chaos happens! So they must strictly coordinate turns (mutexes/semaphores), and they must be in the *same room* (same physical machine).
- **Message Passing** is like **sending sealed memos** through an office courier (the OS kernel). It's slower because every memo requires handing it to the courier (`send()` system call) and copying it to the recipient's inbox (`receive()`). **Advantage**: Nobody overwrites anyone else's desk, there are no shared-state corruption bugs, and the courier works just as well if the coworker is in another building (across a network!).

---

### 9. IndixOS Process IPC Design: Shared Memory vs. Message Passing [Frequency: 1]

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
Think of **cooperative multitasking** like relying on speakers at a conference to voluntarily pass the microphone—if one speaker gets stuck in an infinite `while(1)` loop, nobody else ever speaks!
**Preemption** installs an automated stage alarm (**hardware timer**) wired directly to the soundboard (**CPU hardware + IVT**):
1. Every 10 ms, the alarm rings (**timer interrupt**). Process B has *zero say* in the matter—the CPU immediately switches to Kernel Mode and runs the OS handler.
2. **Context Switch**: Think of pausing a video game—save every register, stack pointer, and memory map pointer of Process B into its **PCB save file**, load Process C's **PCB save file**, switch back to User Mode, and Process C resumes without ever realizing it was paused!

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
Think of **phone call (Synchronous)** vs. **text message / email (Asynchronous)**:
- On a **phone call (`blocking send/receive`)**, both people synchronize directly (*rendezvous*), and you know instantly they heard you—no inbox queue needed! **Catch**: if the other person doesn't pick up, you just sit stuck holding the phone doing nothing!
- With a **text message (`non-blocking send`)**, you fire off the message into the carrier's queue and immediately move on with your day (**higher concurrency!**)—at the cost that the carrier must maintain message storage buffers and you don't know immediately if/when they read it!

---

### 11. How Multithreading Exploits Multiprocessor Systems [Frequency: 1]

> **Question**: Explain how the multi-threading framework can exploit the power of multi-processing architectures as compared to the process framework?

#### Answer (Exam Writing Format)
- **Single-Threaded Process Limitation**: A traditional single-threaded process has only **one stream of execution** (one PC/register state), so at any instant it can execute on **at most one CPU core**, leaving additional cores unused by that application.
- **How Multithreading Exploits Multiprocessing**:
  1. A multithreaded process contains **multiple independent execution streams (threads)** sharing the same code, data, and open files.
  2. On a multiprocessor/multicore system, the OS schedules different threads of the *same* process onto **separate CPU cores simultaneously** (true parallelism), achieving significant speedup with much lower creation and context-switch overhead than spawning heavyweight processes.

#### IN DETAIL
Suppose you have a massive video file to encode on an 8-core laptop:
- A classic **single-threaded process** can only ever run on **1 core** at a time (using just 12.5% of your CPU!). The other 7 cores can't touch that single thread's instruction stream.
- A **multithreaded** video encoder splits the work into 8 threads inside the same process (sharing the video buffer in RAM with zero IPC copying!). The OS drops Thread 1 onto Core 0, Thread 2 onto Core 1, ..., Thread 8 onto Core 7—finishing the task up to ~8× faster!

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
Consider a web server or modern text editor (like MS Word):
- On a **single-core CPU**, multithreading still gives **concurrency**: while Thread 1 waits for a disk save, Thread 2 keeps accepting your keystrokes, and Thread 3 checks spelling by taking rapid turns on that one core.
- On a **multicore CPU** (say 4 cores), a traditional single-threaded process is stuck using only 1 core (25% of the machine's CPU power!). With **multithreading**, the OS puts Thread 1 on Core 0, Thread 2 on Core 1, Thread 3 on Core 2, and Thread 4 on Core 3 **at the exact same nanosecond**. So you get *both* responsive overlapping structure (concurrency) and true multi-core simultaneous computation (parallelism).

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
Think of **processors ($P=4$)** as 4 factory machines, **kernel threads ($K$)** as licensed machine operators, and **user threads ($U=20$)** as tasks to perform:
- **(i) $K = 2 < 4$**: Only 2 operators exist, so 2 machines gather dust permanently!
- **(ii) $K = 4 = 4$**: All 4 machines stay busy *until* one operator steps away to wait on a delivery truck (**blocking syscall**)—leaving Machine #4 sitting empty while 16 tasks wait in line!
- **(iii) $K = 8$ ($4 < 8 < 20$)**: You have 8 operators for 4 machines! Whenever one operator waits on a delivery truck, one of the spare operators immediately steps onto Machine #4—keeping all 4 machines humming near **100% utilization**, without paying the memory cost of hiring 20 separate kernel-level operators!

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
Imagine a busy restaurant hiring and firing a brand-new waiter every single time a customer walks in, then firing them 30 seconds later when the bill is paid! That's what spawning a new thread per request does—it wastes huge OS overhead allocating stacks and kernel structures, and during a rush hour (10,000 simultaneous requests), the restaurant runs out of space and collapses.

A **thread pool** hires a fixed team of, say, 20 waiters at opening time:
- When requests arrive, an idle thread picks up the job immediately (zero startup delay).
- When done, the thread returns to the pool rather than dying.
- If 500 requests arrive at once, 20 run smoothly and 480 wait politely in the request queue without crashing RAM!

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
Look closely at the exact terms from `MultithreadedProgramming-ch4-2025.ppt` vs `Lec7-processConcept_ch3-2025.ppt`:
- **Thread pools** limit *how many threads* a multithreaded server spawns (so 10,000 requests don't crash RAM with 10,000 threads).
- **MPL (Multiprogramming Level)** is the number of *processes* in main memory controlled by the OS **Long-Term / Medium-Term Scheduler**! Keep that distinction sharp on exams!

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
Think of **LWPs** as airport VIP shuttle vans, and **unbound user threads** as passengers sharing a common pool of 4 shuttle vans:
- Normally, sharing 4 vans among 50 passengers is super efficient.
- Now imagine an **ambulance patient (Real-Time Thread)**! If you put the ambulance patient in the shared waiting lounge without assigning a dedicated van:
  1. Air traffic / traffic cops (**the OS Kernel**) only see 4 generic vans—they have **no idea** an urgent emergency patient is sitting inside the lounge!
  2. If all 4 vans are stuck in traffic (**blocked on disk syscalls**) or carrying normal tourists, the real-time patient misses its deadline and dies!
- **Binding the real-time thread to its own dedicated LWP** gives the ambulance its own dedicated vehicle with flashing sirens visible directly to the kernel scheduler!

---

### 17. Design Choice: When to Use Multiple Processes vs. Multiple Threads [Frequency: 1]

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
