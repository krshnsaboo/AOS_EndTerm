# Topic 8: Virtual Memory, Demand Paging & Page Replacement (Ch. 10)

> **Overview**: Pending-I/O swap hazards, demand-paging hardware/OS support, TLB-miss + async page-fault handling, thrashing detection/control, global vs. local frame allocation, 3-tier EAT calculations, FIFO vs. LRU & Belady's Anomaly, and Clock / Enhanced Second Chance algorithms.  
> **Reference Material**: [virtualMemory_OS.ppt](../Faculty_Notes/virtualMemory_OS.ppt)  
> **Questions**: 9 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [Problems When Swapping Out a Process with Pending Asynchronous I/Os](#1-problems-when-swapping-out-a-process-with-pending-asynchronous-ios-frequency-2) — **Frequency: 2**
2. [Hardware and OS Support Components Required to Implement Demand Paging](#2-hardware-and-os-support-components-required-to-implement-demand-paging-frequency-1) — Frequency: 1
3. [AgentOS Virtual Memory: Designing the TLB-Miss & Asynchronous Page-Fault Handler](#3-agentos-virtual-memory-designing-the-tlb-miss-asynchronous-page-fault-handler-frequency-1) — Frequency: 1
4. [Thrashing: Root Cause, OS Detection Mechanisms & Mitigation](#4-thrashing-root-cause-os-detection-mechanisms-mitigation-frequency-1) — Frequency: 1
5. [T/F: "Global Frame Allocation Algorithm Gives Better Throughput Over Local Frame Allocation"](#5-tf-global-frame-allocation-algorithm-gives-better-throughput-over-local-frame-allocation-frequency-3) — **Frequency: 3**
6. [Three-Tier Memory Hierarchy Effective Access Time Numerical (Cache + RAM + Disk = 480,028 ns)](#6-three-tier-memory-hierarchy-effective-access-time-numerical-cache-ram-disk-480028-ns-frequency-3) — **Frequency: 3**
7. [FIFO vs. LRU Page Replacement Traces & Belady's Anomaly (3 vs. 4 Frames)](#7-fifo-vs-lru-page-replacement-traces-beladys-anomaly-3-vs-4-frames-frequency-2) — **Frequency: 2**
8. [AgentOS evictPages(): Second-Chance (Clock) Algorithm & Degeneration to FIFO](#8-agentos-evictpages-second-chance-clock-algorithm-degeneration-to-fifo-frequency-1) — Frequency: 1
9. [Enhanced Second Chance (NRU with (R, M) Classes) Page Replacement Numerical](#9-enhanced-second-chance-nru-with-r-m-classes-page-replacement-numerical-frequency-1) — Frequency: 1

---

### 1. Problems When Swapping Out a Process with Pending Asynchronous I/Os [Frequency: 2]

> **Question**: Explain the problems if you swap a process with pending I/Os.

#### pre-requisite
- **Swapping & DMA**: When a process initiates disk/network I/O (e.g., `read()`), the DMA controller writes directly into the target physical memory buffer asynchronously while the OS switches the CPU to other processes.

#### Answer (Exam Writing Format)
If the OS swaps out Process $P_1$ to disk while it still has an active/pending DMA I/O transfer into physical memory buffer $M$:
1. **Silent Memory Corruption of Another Process**: After $P_1$ is swapped out, the OS may allocate physical frame $M$ to a newly swapped-in Process $P_2$. When $P_1$'s asynchronous I/O completes, the DMA controller writes $P_1$'s incoming data into physical address $M$, **corrupting $P_2$'s memory and causing security/correctness failures**.
2. **Lost I/O Data for $P_1$**: Even if $M$ stays free, when $P_1$ is later swapped back in, it may be placed at a different physical address, missing the completed I/O data.
- **Standard OS Solutions**:
  - **Pinning / Locking pages**: Never swap out pages involved in pending I/O until the transfer completes.
  - **Double Buffering (Kernel Buffers)**: Perform all DMA into fixed kernel-space buffers, then copy to user memory only when the process is resident.

#### IN DETAIL
Suppose Process A asks the disk controller: *"Fetch 4 KB from the disk and drop it into physical RAM room #5."* The disk takes 10 ms, so the OS puts Process A to sleep and swaps Process A out to disk to make room for Process B. The OS hands physical room #5 to Process B!

Ten milliseconds later, the clueless disk controller finishes fetching Process A's data and faithfully writes it straight into **physical room #5**—smashing right over Process B's variables or code! Process B crashes unpredictably or reads Process A's private data!

That's why OS kernels either **pin** I/O pages in RAM so they cannot be swapped out during active DMA, or **double-buffer** I/O through pinned kernel memory buffers first.

---

### 2. Hardware and OS Support Components Required to Implement Demand Paging [Frequency: 1]

> **Question**: In addition to hardware support such as page table and secondary memory what kind of software support is needed to implement demand paging? Explain clearly.

#### pre-requisite
- **Reference Slides**: `virtualMemory_OS.ppt` (Instruction Restart & Hardware/Software Demand Paging Constraints).
- **Demand Paging Trap**: A page fault can strike **mid-instruction** (during opcode fetch, operand read, or operand write)—requiring the OS/architecture to cleanly restart the interrupted instruction once the missing page loads from disk!

#### Answer (Exam Writing Format)
Implementing demand paging requires OS/software support beyond basic page tables and swap disk, especially for **page-fault handling, frame replacement, and cleanly restarting faulted instructions**:
1. **Page-Fault Trap Handler & Frame/Swap Management Software**:
   - Software routine to verify valid vs. illegal memory accesses (`SIGSEGV`), allocate/evict physical frames (Clock/LRU replacement), schedule non-blocking disk I/O, update PTE valid/frame bits, and reschedule the waiting process.
2. **Clean Instruction Restart Support**:
   - After loading the missing page, the OS must restart the trapped instruction from the beginning (re-fetching and re-executing it seamlessly). For simple instructions (`ADD A, B -> C`), re-running from the saved `PC` works if no destination was partially overwritten before the fault.
3. **Handling Multi-Location Block Instructions (e.g., IBM 360/370 `MVC` Move Character)**:
   - Instructions moving e.g. 256 bytes between overlapping source/destination blocks can page-fault midway after overwriting part of the source!
   - *Solution*: Either touch/probe both ends of both blocks via microcode before starting the move (triggering any page fault *before* modifying memory), or save overwritten values in temporary registers to restore state before restarting.
4. **Handling Auto-Increment / Auto-Decrement Addressing Modes (e.g., `MOV (R2)+, -(R3)`)**:
   - If an instruction modifies pointer registers (`R2` incremented or `R3` decremented) *before* faulting while writing to memory, blindly restarting would increment/decrement the registers **a second time**!
   - *Solution*: Software uses hardware status/tracking records recording which registers were modified and by how much, **undoing partial register side effects** before restarting the instruction.

#### IN DETAIL
Think of a page fault like a sudden power outage hitting right in the middle of executing a single assembly instruction:
- **Easy case (`ADD A, B -> C`)**: Suppose `C` isn't in RAM when storing the answer! Easy—just load `C`'s page from disk and **restart `ADD A, B -> C` from the top** (since `A` and `B` weren't overwritten).
- **Hard case #1 (`MVC` block copy of 256 bytes where source and destination overlap)**: Suppose bytes `0..100` overwrite part of the source, and byte `101` page-faults! You can't simply restart from byte 0 because bytes `0..100` of the source are already destroyed! So the system either checks *both ends* of source & destination first so any fault happens *before* copying begins, or saves backup values!
- **Hard case #2 (`MOV (R2)+, -(R3)`)**: If `R2` already added `+4` before the write to `(R3)` faulted, restarting without undoing `R2 -= 4` would silently skip 4 bytes! The OS/hardware status word records partial register changes so the page-fault handler **rolls back the registers** before restarting!

---

### 3. AgentOS Virtual Memory: Designing the TLB-Miss & Asynchronous Page-Fault Handler [Frequency: 1]

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

### 4. Thrashing: Root Cause, OS Detection Mechanisms & Mitigation [Frequency: 1]

> **Question**: What is the cause of thrashing? How does the system detect thrashing? Once it detects thrashing, how does it resolve/prevent it?

#### pre-requisite
- **Reference Slides**: `virtualMemory_OS.ppt` ("Thrashing").
- **Working Set**: The set of active memory pages a process is currently referencing frequently during its current execution phase.

#### Answer (Exam Writing Format)
1. **Cause of Thrashing**:
   - Occurs when the **sum of working-set sizes** of all concurrently running processes exceeds total available physical RAM frames ($\sum \text{WS}_i > \text{Total Frames}$).
   - Processes continuously steal frames from each other, triggering **constant page faults** and spending almost all their time swapping pages to/from disk rather than executing instructions.
2. **How the System Detects Thrashing**:
   - **CPU Utilization vs. Degree of Multiprogramming (MPL) Monitoring**: As MPL increases initially, CPU utilization rises; once thrashing begins, **CPU utilization plunges sharply while disk paging activity spikes** (threads sit blocked waiting on the paging device).
   - **Page-Fault Frequency (PFF) Monitoring**: Detecting that per-process page-fault rates exceed an upper threshold.
3. **How the OS Resolves / Prevents Thrashing**:
   - **Reduce Degree of Multiprogramming (Medium-Term Swapping)**: Suspend and swap out one or more processes to disk, reallocating their frames to remaining processes so their full working sets fit in RAM.
   - **Local / Working-Set / PFF Frame Allocation**: Use local replacement (preventing one thrashing process from stealing another's frames) and enforce the **Working-Set Model** or **Page-Fault Frequency (PFF)** algorithm so a process runs *only* if enough frames exist for its working set.

#### IN DETAIL
Think of 10 chefs trying to cook simultaneously in a kitchen with **only 4 cutting boards** (RAM frames)!
Instead of cooking, every 2 seconds Chef A steals Chef B's cutting board, washes it (disk page-out), puts their vegetables on it (disk page-in), and before making 1 chop, Chef C steals it back! **Zero meals get cooked** (CPU utilization crashes to ~5%!) while the sink (disk) runs at 100%!
- **How the OS catches it**: It notices *"Wait—I just allowed MORE chefs into the kitchen (higher MPL), yet meals-per-hour suddenly DROPPED off a cliff while the disk is thrashing nonstop!"*
- **The Fix**: Kick 5 chefs out to the hallway temporarily (**medium-term scheduler swaps them out**). Immediately the remaining 5 chefs keep their cutting boards steady and CPU productivity jumps right back up!

---

### 5. T/F: "Global Frame Allocation Algorithm Gives Better Throughput Over Local Frame Allocation" [Frequency: 3]

> **Question**: Identify whether the following statement is TRUE or FALSE. If the statement is FALSE, correct it and justify the corrected sentence. If the statement is TRUE, justify it. Restrict the justification to few (less than five) sentences: **"Global frame allocation algorithm gives better throughput over local frame allocation algorithm."**

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

### 6. Three-Tier Memory Hierarchy Effective Access Time Numerical (Cache + RAM + Disk = 480,028 ns) [Frequency: 3]

> **Question**: A computer has a cache, main memory, and a disk used for virtual memory. If a referenced word is in cache, $20\text{ nsec}$ (nano second) are required to access it. If it is in main memory but not in cache, $60\text{ nsec}$ are needed to load into the cache, and then the reference is started again. If the word is not in main memory, $12\text{ msec}$ (milli seconds) are required to fetch the word from the disk, followed by $60\text{ nsec}$ to copy it to the cache, and then the reference is started again. The cache hit ratio is $0.9$ and the main memory hit ratio is $0.6$. What is the average time, in nanoseconds, required to access a referenced word on this system?
> *(2021 Endsem Q11 had the prompt truncated after "referenced word is in"; 2022 Endsem Q6 had an official-key calculation error `4.8 ms` fixed here to `480,028 ns`.)*

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

### 7. FIFO vs. LRU Page Replacement Traces & Belady's Anomaly (3 vs. 4 Frames) [Frequency: 2]

> **Question**: You observed that, like many teams, another team has also used paging for memory management, where they have devised a technique using swap space to manage even processes with large virtual address spaces. However, the exact policy for page replacement has yet to be decided. The team has two types of traces of page access in mind, and they are looking at the main memory as a cache with a size of **3 pages**:
> - **Trace 1**: `0, 1, 2, 0, 1, 3, 0, 3, 1, 2, 1`
> - **Trace 2**: `1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`
>
> Demonstrate using the necessary metrics and the trace table for the different policies — **FIFO** and **LRU**, and further, which among them would you recommend and why? What if the main memory can hold **4 pages**?

#### pre-requisite
- **Page Faults vs. Hits**: A page fault occurs when a referenced page is not currently in RAM frames.
- **Belady's Anomaly**: A counter-intuitive phenomenon (possible in FIFO, impossible in stack algorithms like LRU) where **increasing** the number of physical frames causes **more** page faults!

#### Answer (Exam Writing Format)
##### **1. Cache Size = 3 Pages**
- **Trace 1 (`0, 1, 2, 0, 1, 3, 0, 3, 1, 2, 1`, length 11)**:
  - **FIFO (3 frames)**:
    `0(M) 1(M) 2(M) 0(H) 1(H) 3(M, evict 0) 0(M, evict 1) 3(H) 1(M, evict 2) 2(M, evict 3) 1(H)` $\implies$ **7 Faults, 4 Hits** (Fault Rate = $7/11 = 63.6\%$).
  - **LRU (3 frames)**:
    `0(M) 1(M) 2(M) 0(H) 1(H) 3(M, evict 2) 0(H) 3(H) 1(H) 2(M, evict 0) 1(H)` $\implies$ **5 Faults, 6 Hits** (Fault Rate = $5/11 = 45.5\%$).
- **Trace 2 (`1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`, length 12)**:
  - **FIFO (3 frames)**:
    `1(M) 2(M) 3(M) 4(M, evict 1) 1(M, evict 2) 2(M, evict 3) 5(M, evict 4) 1(H) 2(H) 3(M, evict 1) 4(M, evict 2) 5(H)` $\implies$ **9 Faults, 3 Hits**.
  - **LRU (3 frames)**:
    `1(M) 2(M) 3(M) 4(M, evict 1) 1(M, evict 2) 2(M, evict 3) 5(M, evict 4) 1(H) 2(H) 3(M, evict 5) 4(M, evict 1) 5(M, evict 2)` $\implies$ **10 Faults, 2 Hits**.

##### **2. Cache Size = 4 Pages**
- **Trace 1 (`0, 1, 2, 0, 1, 3, 0, 3, 1, 2, 1`)**:
  - **FIFO (4 frames)**: Only 4 distinct pages (`0,1,2,3`) exist in the entire trace! After `0(M), 1(M), 2(M), 3(M)`, every remaining access is a Hit $\implies$ **4 Faults, 7 Hits**.
  - **LRU (4 frames)**: Same—4 distinct pages fit in 4 frames $\implies$ **4 Faults, 7 Hits**.
- **Trace 2 (`1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`)**:
  - **FIFO (4 frames)**:
    `1(M) 2(M) 3(M) 4(M) 1(H) 2(H) 5(M, evict 1) 1(M, evict 2) 2(M, evict 3) 3(M, evict 4) 4(M, evict 5) 5(M, evict 1)` $\implies$ **10 Faults, 2 Hits**!
  - **LRU (4 frames)**:
    `1(M) 2(M) 3(M) 4(M) 1(H) 2(H) 5(M, evict 3) 1(H) 2(H) 3(M, evict 4) 4(M, evict 5) 5(M, evict 1)` $\implies$ **8 Faults, 4 Hits**.

##### **Summary Table & Recommendation**
| Trace | Frames | FIFO Faults | LRU Faults |
| :--- | :---: | :---: | :---: |
| **Trace 1** | 3 | 7 | **5** |
| **Trace 1** | 4 | 4 | 4 |
| **Trace 2** | 3 | **9** | 10 |
| **Trace 2** | 4 | **10 (Belady's Anomaly!)** | **8** |

- **Recommendation**: Recommend **LRU** (or an efficient Clock approximation in real hardware). Unlike FIFO—which exhibits **Belady's Anomaly** on Trace 2 (faults **rose from 9 to 10** when RAM increased from 3 to 4 pages!)—LRU is a stack algorithm guaranteed **never** to suffer Belady's Anomaly and exploits temporal locality effectively (beating FIFO on Trace 1 at 3 frames and Trace 2 at 4 frames).

#### IN DETAIL
Look at **Trace 2** (`1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5`)—that's the famous textbook sequence specifically constructed to expose **Belady's Anomaly** in FIFO!
- With **3 frames**, FIFO suffered **9 faults** (getting lucky hits on `1, 2` and `5` at the end!).
- You buy more RAM (**4 frames**), expecting fewer faults—and FIFO actually gets **WORSE (10 faults!)** because with 4 frames, `1` and `2` survived just long enough for `5` to evict `1` right before `1` was accessed, triggering a chain reaction of misses!
- **LRU** is mathematically guaranteed to be **anomaly-free**: adding more RAM frames can *never* increase page faults under LRU!

**Why FIFO Also Lost on Trace 1 Alone (2023 Q2b Check)**:
Look closely at why FIFO stumbled right at step 6 (`3` arrives) and step 7 (`0` arrives):
- Pages `0` and `1` were just actively used (`0, 1, 2, 0, 1`), yet FIFO kicked `0` out simply because `0` arrived first—forcing an immediate extra page fault on the very next instruction when `0` was needed again, which then kicked `1` out right before `1` was needed!
- **LRU** recognized that `2` hadn't been touched since step 3, evicted `2` instead, and kept `0` and `1` warm in memory—cutting misses from **7 down to 5**!

---

### 8. AgentOS evictPages(): Second-Chance (Clock) Algorithm & Degeneration to FIFO [Frequency: 1]

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

### 9. Enhanced Second Chance (NRU with (R, M) Classes) Page Replacement Numerical [Frequency: 1]

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
