# Topic 7: Memory Management, Segmentation, Paging & Page Tables (Ch. 9)

> **Overview**: Static vs. dynamic memory binding, dynamic loading vs. linking, First/Best/Worst-fit allocation, segmentation translation, paging fundamentals (`TLB`, dirty bits, inverted page tables), and 32/44/64-bit multi-level page table sizing.  
> **Reference Material**: [memManage_CS.ppt](../Faculty_Notes/memManage_CS.ppt)  
> **Questions**: 12 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [Static (Absolute) Memory Allocation Ps Failure vs. Dynamic Allocation Pd Success](#1-static-absolute-memory-allocation-ps-failure-vs-dynamic-allocation-pd-success-frequency-1) — Frequency: 1
2. [Difference Between Dynamic Loading and Dynamic Linking](#2-difference-between-dynamic-loading-and-dynamic-linking-frequency-1) — Frequency: 1
3. [Contiguous Memory Partition Allocation: First-Fit vs. Best-Fit vs. Worst-Fit Trace](#3-contiguous-memory-partition-allocation-first-fit-vs-best-fit-vs-worst-fit-trace-frequency-1) — Frequency: 1
4. [AgentOS Segmentation Address Translation Numerical (0x5100 → 0xA100)](#4-agentos-segmentation-address-translation-numerical-0x5100-0xa100-frequency-1) — Frequency: 1
5. [Why Paging Replaces Contiguous Allocation & Coordination Needed with fork()](#5-why-paging-replaces-contiguous-allocation-coordination-needed-with-fork-frequency-1) — Frequency: 1
6. [Design Justification: Why Virtual Page Size Must Equal Physical Frame Size](#6-design-justification-why-virtual-page-size-must-equal-physical-frame-size-frequency-1) — Frequency: 1
7. [Purpose of the Translation Lookaside Buffer (TLB)](#7-purpose-of-the-translation-lookaside-buffer-tlb-frequency-1) — Frequency: 1
8. [Purpose of the Dirty (Modified) Bit in a Page Table Entry](#8-purpose-of-the-dirty-modified-bit-in-a-page-table-entry-frequency-1) — Frequency: 1
9. [Inverted Page Table: Positive and Negative Aspects](#9-inverted-page-table-positive-and-negative-aspects-frequency-1) — Frequency: 1
10. [32-Bit 2 KB Pages (100 Processes = 800 MB Metadata): Fixing Overhead & Double Access with Multi-Level Paging + TLB](#10-32-bit-2-kb-pages-100-processes-800-mb-metadata-fixing-overhead-double-access-with-multi-level-paging-tlb-frequency-1) — Frequency: 1
11. [Numerical: Number of Page Table Levels Required for a 64-Bit Address Space (4 KB Page, 4 B PTE)](#11-numerical-number-of-page-table-levels-required-for-a-64-bit-address-space-4-kb-page-4-b-pte-frequency-1) — Frequency: 1
12. [Numerical: 44-Bit Virtual Address, 64 KB Page, Two-Level Paging & 4 GB Program Frame Count](#12-numerical-44-bit-virtual-address-64-kb-page-two-level-paging-4-gb-program-frame-count-frequency-1) — Frequency: 1

---

### 1. Static (Absolute) Memory Allocation Ps Failure vs. Dynamic Allocation Pd Success [Frequency: 1]

> **Question**: Two versions of a program P, called Ps and Pd, are developed using static and dynamic memory allocation, respectively. When an effort is made to execute Ps, an OS gives the message "Insufficient memory, can not run the program." However, the same OS executes program Pd without any difficulties. Explain why this happens.

#### pre-requisite
- **Address Binding Times**: Compile/Load-time **absolute** binding fixes physical memory addresses before execution, whereas execution-time **dynamic relocation** translates logical addresses via hardware base/limit registers (or MMU paging) at runtime.

#### Answer (Exam Writing Format)
- **Why $P_s$ (Static Allocation / Absolute Code) Fails**:
  - $P_s$ is compiled/linked with **fixed physical addresses**. It can only execute if that exact predetermined physical memory region is completely free.
  - Even if plenty of total free RAM exists elsewhere, if that specific fixed address range is occupied (or fragmented), the OS reports *"Insufficient memory, cannot run."*
- **Why $P_d$ (Dynamic / Relocatable Allocation) Succeeds**:
  - $P_d$ generates **relocatable / logical addresses** resolved at execution time using hardware relocation registers (base + limit) or paging.
  - The OS can load $P_d$ into **any** available memory space (or even dynamically allocate/grow heap/stack or non-contiguous frames on demand), allowing $P_d$ to execute without difficulty.

#### IN DETAIL
Imagine printing a theater ticket with a permanent rule *"You MUST sit in Row 3, Seat 5"* (**static absolute binding** for $P_s$). Even if the theater has 99 empty seats in Rows 1, 2, and 4, if one person is sitting in Row 3 Seat 5, you get turned away at the door!
Meanwhile, $P_d$ gets a **general-admission relocatable ticket** (*"Seat = Base + Offset"*). The usher (OS + hardware MMU/base register) seats $P_d$ in *any* empty block—and with dynamic allocation/virtual memory, it doesn't even have to reserve max worst-case memory upfront!

---

### 2. Difference Between Dynamic Loading and Dynamic Linking [Frequency: 1]

> **Question**: Explain the difference between dynamic loading and dynamic linking.

#### Answer (Exam Writing Format)
| Feature | Dynamic Loading | Dynamic Linking |
| :--- | :--- | :--- |
| **Core Mechanism** | Program routines (stored on disk in relocatable format) are **not loaded into RAM until actually called** during execution. | Linking of external **system/shared libraries** (`.so` / `.dll`) is postponed until **execution time**; a small **stub** locates/loads the routine on first call. |
| **Primary Benefit** | Saves memory within a single program by never loading unused routines (e.g., rare error handlers). | Allows **multiple processes to share one physical copy** of common library code in RAM and lets libraries be updated independently without recompiling apps. |
| **OS Support Needed** | **No special OS support required**—can be implemented entirely by the user/program design. | **Requires OS/dynamic-loader support** (e.g., stub resolution, shared memory/code page mapping across processes). |

#### IN DETAIL
- **Dynamic Loading** is like keeping your rarely used winter coat in storage until it actually snows—your single suitcase (**process RAM**) stays small because error-handling routines that never execute never get loaded from disk. You can code this yourself without special OS magic.
- **Dynamic Linking** is like **100 apartments sharing one public library** instead of buying 100 personal copies of `printf()`! When your program calls `printf()`, a tiny **stub** checks if `libc.so` is already in RAM; if yes, the OS maps that *existing* shared copy straight into your address space—saving huge amounts of disk space and RAM across all running apps!

---

### 3. Contiguous Memory Partition Allocation: First-Fit vs. Best-Fit vs. Worst-Fit Trace [Frequency: 1]

> **Question**: Given memory partitions (holes) of $100\text{ K}, 500\text{ K}, 200\text{ K}, 300\text{ K}$ and $600\text{ K}$ (in order), how would each of the First-fit, Best-fit, and Worst-fit algorithms place processes of $212\text{ K}, 417\text{ K}, 112\text{ K}$ and $426\text{ K}$ (in order)? Which algorithm makes efficient use of memory?

#### pre-requisite
- **First-Fit**: Allocates the *first* free hole large enough.
- **Best-Fit**: Allocates the *smallest* free hole large enough (minimizes immediate leftover fragment).
- **Worst-Fit**: Allocates the *largest* free hole available (leaves the biggest leftover hole).

#### Answer (Exam Writing Format)
Initial Holes: `[100K, 500K, 200K, 300K, 600K]`. Requests: `P1=212K`, `P2=417K`, `P3=112K`, `P4=426K`.

1. **First-Fit**:
   - `212K` $\to$ placed in **`500K`** (2nd hole) $\to$ leaves `288K`. Holes: `[100K, 288K, 200K, 300K, 600K]`
   - `417K` $\to$ placed in **`600K`** (5th hole) $\to$ leaves `183K`. Holes: `[100K, 288K, 200K, 300K, 183K]`
   - `112K` $\to$ placed in **`288K`** (leftover of 2nd hole) $\to$ leaves `176K`. Holes: `[100K, 176K, 200K, 300K, 183K]`
   - `426K` $\to$ **Cannot be allocated / Must wait** (largest remaining hole is `300K < 426K`).

2. **Best-Fit**:
   - `212K` $\to$ placed in **`300K`** (4th hole) $\to$ leaves `88K`. Holes: `[100K, 500K, 200K, 88K, 600K]`
   - `417K` $\to$ placed in **`500K`** (2nd hole) $\to$ leaves `83K`. Holes: `[100K, 83K, 200K, 88K, 600K]`
   - `112K` $\to$ placed in **`200K`** (3rd hole) $\to$ leaves `88K`. Holes: `[100K, 83K, 88K, 88K, 600K]`
   - `426K` $\to$ placed in **`600K`** (5th hole) $\to$ leaves `174K`. Holes: `[100K, 83K, 88K, 88K, 174K]`

3. **Worst-Fit**:
   - `212K` $\to$ placed in **`600K`** (5th hole) $\to$ leaves `388K`. Holes: `[100K, 500K, 200K, 300K, 388K]`
   - `417K` $\to$ placed in **`500K`** (2nd hole) $\to$ leaves `83K`. Holes: `[100K, 83K, 200K, 300K, 388K]`
   - `112K` $\to$ placed in **`388K`** (leftover of 5th hole) $\to$ leaves `276K`. Holes: `[100K, 83K, 200K, 300K, 276K]`
   - `426K` $\to$ **Cannot be allocated / Must wait** (largest remaining hole is `300K < 426K`).

- **Conclusion**: **Best-Fit** makes the most efficient use of memory for this workload because it is the **only** algorithm that successfully accommodates all four processes (`1167K` allocated out of `1700K`), whereas both First-Fit and Worst-Fit fail to place the `426K` process.

#### IN DETAIL
Let's check why **Best-Fit** won here:
- Under **First-Fit**, `212K` grabbed the first big hole (`500K`), leaving only `600K` for `417K`—which consumed the only hole large enough for `426K` later!
- Under **Worst-Fit**, `212K` immediately chewed up the biggest hole (`600K`), again leaving no contiguous `426K` slot at the end.
- **Best-Fit** matched each job tightly (`212K -> 300K`, `417K -> 500K`, `112K -> 200K`), preserving the `600K` hole intact until `426K` arrived! *(Note: in general workloads over long periods, First-Fit is faster and often comparable in fragmentation because Best-Fit tends to leave tiny unusable slivers, though for this classic textbook trace Best-Fit clearly wins.)*

---

### 4. AgentOS Segmentation Address Translation Numerical (0x5100 → 0xA100) [Frequency: 1]

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

### 5. Why Paging Replaces Contiguous Allocation & Coordination Needed with fork() [Frequency: 1]

> **Question**: The *fork team* learned from the *Mem Team* that they might be using Paging for Memory management. Assuming that it will be paging, Team fork wants your input on how the memory management will work when performing calls like `repl()` and `execute()`.

#### pre-requisite
- **Paging & Copy-on-Write (COW)**: Each process has its own Page Table mapping Virtual Pages to Physical Frames. Copying every physical page eagerly during `fork()` is wasteful if `exec()` immediately wipes the memory space.

#### Answer (Exam Writing Format)
1. **During `repl()` (`fork()`) — Copy-on-Write (COW)**:
   - Instead of immediately duplicating all physical memory frames of the parent, the OS creates a **new Page Table** for the child pointing to the **exact same physical frames** as the parent.
   - All shared writable pages (data, heap, stack) are marked **Read-Only** in both parent and child page tables.
   - Only if either process attempts a **write** does a hardware protection fault occur; the kernel then lazily allocates a fresh physical frame, copies that single page, updates the writer's PTE to **Read-Write**, and resumes execution.
2. **During `execute()` (`exec()`) — Address Space Replacement**:
   - The kernel **deallocates/unmaps** the child's inherited page table entries and decrements reference counts on shared COW physical frames (freeing any frame whose count reaches 0).
   - It constructs a **fresh Page Table** for the new executable's segments (Code/Text mapped Read-Only, Data, BSS, Heap, and a new Stack), typically using **demand paging** so code/data pages are loaded from disk into physical frames only when first referenced.

#### IN DETAIL
Think about what happened in Question 1(a): right after `repl()` cloned a 100 MB shell, the child immediately called `execute("fwc")` 1 microsecond later! If `repl()` had physically copied all 100 MB (25,600 pages) just to throw them away in `execute()`, the system would crawl!
- **Copy-on-Write (COW)** makes `repl()` almost instant: just copy the tiny page table pointers and mark them *"Read-Only—tell the kernel if anyone writes!"*. If `execute()` runs right away, zero physical data pages were ever copied!
- Then **`execute()`** rips up the old page table cleanly, creates a blank new virtual map pointing to `fwc`'s executable sections on disk, and lets the page-fault handler pull in pages lazily as `fwc` runs!

---

### 6. Design Justification: Why Virtual Page Size Must Equal Physical Frame Size [Frequency: 1]

> **Question**: The *Mem Team* is thinking of dividing the virtual memory into pages of different sizes and then mapping them to physical memory, where, again, it is divided into blocks of different sizes to support different processes that may have different memory requirements. What are your observations and comments on this approach?

#### pre-requisite
- **Fixed-Size Paging**: The core reason paging eliminates external fragmentation is that **every virtual page and every physical frame has the exact same fixed power-of-2 size**, making all frames interchangeable.

#### Answer (Exam Writing Format)
Using arbitrary **variable-sized pages and variable-sized physical blocks** undermines the core benefits of paging and is **not recommended** as a primary mechanism:
1. **Re-introduces External Fragmentation**: Variable-sized physical blocks leave non-uniform free holes after deallocation, forcing complex placement algorithms (First/Best-fit) and costly **memory compaction**.
2. **Complex Address Translation Hardware**: Variable page sizes prevent simple bit-splitting (`VPN | Offset`) in the MMU; translation would require per-page size/limit checks (similar to Segmentation).
3. **Increased OS Allocation Overhead**: Tracking free variable-sized blocks and matching page sizes to holes adds heavy kernel bookkeeping overhead.
*(Note: Modern CPUs only support a tiny set of fixed power-of-2 "huge page" sizes like 4 KB / 2 MB / 1 GB alongside standard 4 KB frames—not arbitrary variable sizes.)*

#### IN DETAIL
Why did OS designers invent paging in the first place? To make every chunk of memory look like identical **LEGO bricks** so **any** page fits into **any** empty frame with zero leftover gaps (**zero external fragmentation**) and so splitting a 32-bit address is just chopping off the bottom 12 bits!

If you make pages and physical blocks *different arbitrary sizes*:
- You lose fast bit-shift address translation!
- You recreate variable-partition **external fragmentation**—you're basically reinventing **Segmentation** while calling it Paging!

---

### 7. Purpose of the Translation Lookaside Buffer (TLB) [Frequency: 1]

> **Question**: What is the purpose of "translation lookaside-buffer"?

#### pre-requisite
- **Paging Overhead**: Without a TLB, accessing one byte of user memory requires **at least two physical RAM accesses** (one to read the Page Table Entry in RAM, one to access the actual data byte—and even more with multi-level page tables!).

#### Answer (Exam Writing Format)
- **Definition**: The **Translation Lookaside Buffer (TLB)** is a small, ultra-fast associative hardware cache inside the MMU that stores recent `<Virtual Page Number → Physical Frame Number>` translations.
- **Purpose**:
  - **Eliminates Extra Page-Table Memory Lookups**: On a TLB hit (typically >95–99% of accesses), the physical frame address is obtained in ~1 CPU cycle without reading the page table in main memory.
  - **Dramatically Reduces Effective Memory Access Time (EMAT)**: Brings average memory access latency close to a single physical memory access instead of $2\times$ (or $4\times$ with multi-level paging).

#### IN DETAIL
Imagine calling a customer support number where every time you want to ask a question, you first have to drive across town to look up their phone number in a paper phone book (**page table in RAM**), and *then* make the call (**actual data access**)—doubling or quadrupling every single memory read!
The **TLB** is a tiny speed-dial card taped right on your phone (**MMU associative hardware cache**). If the page number is on your speed-dial (TLB Hit), you get the frame number instantly in parallel and make just **one** trip to RAM!

---

### 8. Purpose of the Dirty (Modified) Bit in a Page Table Entry [Frequency: 1]

> **Question**: What is the purpose of dirty bit entry in the page table?

#### pre-requisite
- **Page Replacement Cost**: When RAM is full and a victim page frame must be evicted to make room for a new page from disk, if the victim was modified, its new contents must be saved back to disk before reusing the frame.

#### Answer (Exam Writing Format)
- **Definition**: The **dirty (modified) bit** is a hardware flag in each Page Table Entry (PTE) automatically set to `1` by the MMU whenever the CPU writes to any byte in that page (cleared to `0` when loaded from disk).
- **Purpose**:
  - **Avoids Unnecessary Disk Writes on Eviction**: When selecting a victim page during a page fault:
    - If **`dirty == 0`** (clean): Memory still matches the copy on disk, so the frame is **simply discarded/overwritten immediately** with **zero disk write**.
    - If **`dirty == 1`** (modified): The page must be written back to swap/disk first before loading the new page.
  - This cuts page-fault disk write traffic roughly in half, vastly improving virtual memory performance (and helps replacement policies like Enhanced Second Chance prefer clean victims).

#### IN DETAIL
Think of borrowing a library book (disk page) onto your study desk (RAM frame):
- **Clean (`dirty = 0`)**: You only *read* the book without writing any notes in the margins. When your desk runs out of space, you don't need to photocopy your copy before putting it away—the library already has an identical copy, so you just toss yours onto the return cart instantly!
- **Dirty (`dirty = 1`)**: You wrote new solutions in the book! Now if you clear your desk, you *must* save your updated pages back to the library first so your edits aren't lost. The dirty bit tells the OS instantly whether a page needs that expensive disk write!

---

### 9. Inverted Page Table: Positive and Negative Aspects [Frequency: 1]

> **Question**: Explain its positive and negative aspects of inverted page table approach.

#### pre-requisite
- **Inverted Page Table (IPT)**: Instead of one page table per process indexed by virtual page number (VPN), the system keeps **one single global table** with **one entry per physical frame**, storing `<process_id, virtual_page_number>` (often paired with a hash table).

#### Answer (Exam Writing Format)
- **Positive Aspect (Low Memory Overhead)**:
  - Requires only **one global table** proportional to **physical RAM size** (1 entry per physical frame), completely independent of the number of processes or huge 64-bit virtual address spaces—dramatically saving RAM.
- **Negative Aspects (Lookup Overhead & Sharing Complexity)**:
  1. **Slower Translation Lookup**: Since the table is indexed by physical frame rather than virtual page, translating a virtual address requires searching for `<PID, VPN>` (using a hash table + collision chain), increasing memory lookup latency on a TLB miss.
  2. **Difficulty Implementing Shared Memory**: Because each physical frame has only one IPT slot storing one `<PID, VPN>`, mapping a single physical frame into multiple processes' virtual address spaces simultaneously requires extra chained structures.

#### IN DETAIL
Imagine a hotel with 100 rooms (**physical frames**), and 10,000 guests who *might* reserve rooms (**virtual address spaces**):
- **Standard Per-Process Page Table**: Every single guest carries a giant 10,000-page binder listing all possible rooms! Huge memory waste!
- **Inverted Page Table**: The front desk keeps **one 100-line clipboard**—one line per physical room—recording *"Room #7 is occupied by Guest PID=42, Virtual Page #99."* Tiny memory footprint!
- **Catch**: When Guest #42 asks *"Where is my Page #99?"*, you can't just jump to line 99! You have to **search** the 100-line clipboard (via a hash table + TLB cache) to find which room holds `<PID=42, VPN=99>`!

---

### 10. 32-Bit 2 KB Pages (100 Processes = 800 MB Metadata): Fixing Overhead & Double Access with Multi-Level Paging + TLB [Frequency: 1]

> **Question**: The team is considering a 32-bit address space with 2 KB pages. The team estimates each entry in the page table to consume 4 bytes. On average, the *Mem team* estimates any machine to support around 100 processes. This results in a memory overhead. Moreover, for any address translation, the page table needs to be first accessed, and then the page table entry needs to be fetched. This results in two accesses to the memory for each address translation. Is there some better mechanism that you can suggest to the team to handle these issues? Please elaborate on the mechanism.

#### pre-requisite
- **Quantifying the Problem**:
  - Address space = $2^{32}\text{ B}$; Page size = $2\text{ KB} = 2^{11}\text{ B} \implies 2^{21}\text{ pages/process}$.
  - Linear page table size per process = $2^{21} \times 4\text{ B} = 8\text{ MB}$.
  - Across **100 processes**, flat linear page tables consume **$800\text{ MB}$** of RAM just for metadata!

#### Answer (Exam Writing Format)
To solve **both** problems (memory overhead and double memory access latency), suggest combining:

1. **To Reduce Page Table Memory Overhead ($800\text{ MB}$)**:
   - **Multi-Level (Hierarchical) Paging** *(or Inverted Page Table)*:
     - Split the 21-bit VPN into an Outer Page Directory + Inner Page Tables (or increase page size to 4 KB). Only page-table subtrees covering **actually allocated** code/heap/stack regions are created in RAM—shrinking metadata from 8 MB/process down to a few KB/process.
     - *(Alternative)* **Inverted Page Table**: Keep one global table sized by physical RAM frames rather than virtual space.
2. **To Eliminate Extra Memory Accesses per Translation**:
   - **Translation Lookaside Buffer (TLB)**:
     - Add a fast, associative hardware MMU cache storing recent `<VPN → PFN>` translations.
     - On every memory reference, hardware checks the TLB in parallel (~1 CPU cycle). On a **TLB Hit** (~98–99%), the physical address is produced directly without touching RAM page tables—reducing access back to **1 physical memory access** for data!
     - On a **TLB Miss**, hardware/OS walks the multi-level page table and caches the translation into the TLB for future locality hits.

#### IN DETAIL
The Mem Team actually bumped into the **two classic crises of naive paging**:
1. **Crisis 1 (Space Blowup — 800 MB wasted!)**: Every single tiny `hello world` process was getting an 8 MB flat table covering all 4 GB of empty virtual space! **Multi-level paging** acts like a collapsible folder tree—if a process only uses 10 MB of code and stack, you only allocate page-table folders for those 10 MB, saving >99% of that 800 MB!
2. **Crisis 2 (Time Blowup — 2× or 3× slower RAM!)**: Multi-level paging would actually make memory accesses *even slower* (3 RAM lookups instead of 2) if you walked the tree every time! The **TLB** saves the day: it's an ultra-fast hardware speed-dial right inside the CPU that remembers recent translations so 99% of accesses skip page tables completely!

---

### 11. Numerical: Number of Page Table Levels Required for a 64-Bit Address Space (4 KB Page, 4 B PTE) [Frequency: 1]

> **Question**: Assume a page size of 4 Kbytes and that a page table entry takes 4 bytes, how many levels of page tables would be required to map a 64-bit address space, if the top level page fits into a single page?

#### pre-requisite
- **Multi-Level Paging Formula**:
  - Page Offset bits $d = \log_2(\text{Page Size})$.
  - Entries per single-page table = $\frac{\text{Page Size}}{\text{PTE Size}}$, so each page-table level resolves $\log_2(\text{Entries per Page})$ virtual address bits.
  - To ensure the top-level page directory fits inside 1 page, keep dividing the Virtual Page Number (VPN) bits by the per-level bits (taking $\lceil \text{VPN} / \text{bits per level} \rceil$).

#### Answer (Exam Writing Format)
- **Given**:
  - Virtual Address = $64\text{ bits}$
  - Page Size = $4\text{ KB} = 2^{12}\text{ bytes} \implies \text{Offset bits } (d) = 12\text{ bits}$
  - Page Table Entry (PTE) size = $4\text{ bytes} = 2^2\text{ bytes}$
- **Calculations**:
  1. **Virtual Page Number (VPN) bits**:
     $$\text{VPN bits} = 64 - 12 = 52\text{ bits}$$
  2. **PTEs per page (and bits indexed per level)**:
     $$\text{PTEs per page} = \frac{4\text{ KB}}{4\text{ bytes}} = \frac{2^{12}}{2^2} = 2^{10} = 1024\text{ entries}$$
     $$\text{Bits resolved per page table level} = \log_2(1024) = 10\text{ bits}$$
  3. **Number of Page Table Levels**:
     $$\text{Levels} = \left\lceil \frac{52}{10} \right\rceil = \mathbf{6\text{ levels}}$$
  4. **Bit Breakdown**:
     - Outer (Level 1) to Inner (Level 6) split: **$2 \mid 10 \mid 10 \mid 10 \mid 10 \mid 10 \mid 12$** (or $10|10|10|10|10|2|12$; top level needs only $2^2 = 4$ entries = $16\text{ bytes} \le 4\text{ KB}$, fitting easily into a single page!).

#### IN DETAIL
Why do we split a 64-bit page table into levels? A flat page table for $2^{52}$ pages $\times 4\text{ bytes}$ would take **$2^{54}\text{ bytes}$ (16 Million Terabytes!)** per process—obviously impossible!

Instead, we build a tree where **every node of the tree fits inside one 4 KB page**:
- Since $1\text{ page} = 4096\text{ bytes}$ and each entry is $4\text{ bytes}$, one page holds $4096 / 4 = 1024 = 2^{10}$ entries.
- That means each level consumes **$10\text{ bits}$** of the 52-bit Virtual Page Number:
  - 1 level covers $10$ bits
  - 2 levels cover $20$ bits
  - 3 levels cover $30$ bits
  - 4 levels cover $40$ bits
  - 5 levels cover $50$ bits (still $2$ bits short of $52$! If we stopped at 5 levels, the top level would need $2^{12} = 4096$ entries = $16\text{ KB}$, which is **4 pages**, violating the requirement that the top-level table fits in 1 page!).
  - **6 levels** cover up to $60$ bits ($\ge 52$), with the top level using just $2$ bits ($4$ entries = $16\text{ bytes} \le 4\text{ KB}$). Hence **6 levels** are required!

---

### 12. Numerical: 44-Bit Virtual Address, 64 KB Page, Two-Level Paging & 4 GB Program Frame Count [Frequency: 1]

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
