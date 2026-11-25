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
Address binding stage determines whether an executable image is permanently anchored to specific physical DRAM addresses or dynamically relocatable at load/run time:
- **Static / Compile-Time Absolute Binding ($P_s$)**: Emits literal physical addresses directly into machine instructions (loads, stores, branch targets). Loading $P_s$ at any non-identical base frame shifts every embedded target address out of alignment—causing spatial collisions with concurrently loaded processes whenever its preassigned physical region is occupied.
- **Dynamic / Execution-Time Binding ($P_d$)**: Emits zero-based logical addresses translated on every memory reference via MMU relocation hardware (`Base + Limit` registers or Page Tables). Because logical-to-physical decoupling occurs purely at hardware lookup time, $P_d$ loads into arbitrary free frames without code patching and can even migrate safely during execution.

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
**Dynamic Loading** and **Dynamic Linking** optimize distinct system dimensions—virtual memory footprint vs. cross-process code deduplication and library upgradability:
- **Dynamic Loading (Application-Controlled Lazy Routine Activation)**: Separates error-handling or optional sub-routines from the main binary initialization path; routines remain in relocatable form on disk until first invoked via explicit loader calls, saving resident memory when rare execution paths never trigger (requiring zero explicit OS kernel support).
- **Dynamic Linking & Shared Libraries (OS-Mediated Runtime Resolution)**: Defers symbol resolution of common system libraries (`libc.so`) from link time to load or first-call time (*lazy PLT/GOT binding*). Because read-only `.text` library code pages map read-execute across *every* process's page tables simultaneously, a single resident physical copy services hundreds of processes while security patches to the shared `.so` propagate transparently without relinking binaries.

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
Dynamic contiguous variable-partition allocation fragments physical RAM into alternating allocated blocks and free **holes**, making placement heuristics critically sensitive to request sequences:
- **First-Fit ($O(N)$ early termination)**: Stops at the earliest sufficient hole, concentrating small unallocatable leftovers near lower memory addresses while conserving larger upper partitions for future big allocations unless early medium holes (like `500K` for `212K`) get split prematurely ahead of larger incoming segments (`417K`, `426K`).
- **Best-Fit**: Minimizes immediate wasted space (`min(hole - size)`), maximizing successful placements for constrained multi-step sequences—at the cost of leaving behind tiny, nearly unusable **sliver fragments (external fragmentation)** that inflate search list length unless coalesced frequently.
- **Worst-Fit**: Intentionally splits the largest remaining hole to create viable medium-sized remainders, which quickly exhausts headroom for upcoming large allocation demands.

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
Segmentation hardware splits a $k$-bit virtual address explicitly into a discrete **Segment Selector (`s`)** and **Intra-Segment Offset (`d`)**, performing parallel bounds protection before relocation:
- **Binary Bit-Masking vs. Decimal Hex**: Hardware inspects the top $n$ most significant bits (here upper 2 bits `01` of `0x5100 = 0101_0001_0000_0000_2` $\implies s = 1$), rather than the whole leading hex nibble, extracting the remaining 14 bits (`0x1100`) as byte offset $d$.
- **Two-Stage MMU Check**: First verifying $d < \text{Limit}[s]$ traps invalid sparse cross-segment pointers immediately (`SIGSEGV` / Segment Fault); only valid offsets add $\text{Base}[s] + d$ to produce the linear physical address, cleanly supporting independent per-segment protection flags (`R/W/X`) and variable growth directions (upward heap vs. downward stack).

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
Pairing **non-contiguous Paging** with **`fork()` (`repl()`)** eliminates `fork()`'s historic $O(\text{RAM})$ copy bottleneck through kernel **Copy-on-Write (CoW)** state sharing:
- **Naive Fork Overhead Without CoW**: Cloning a parent's physical RAM frames during `fork()` wastes CPU and DRAM bandwidth whenever the child immediately calls `exec()` (`execute()`) microseconds later and discards the inherited address space.
- **Page-Table CoW Protocol**: On `fork()`, the OS copies *only* the parent's page tables, shares the exact same underlying physical frames via reference counts, and clears the **Write-Protect (`R/W = 0`) bit** across private writable pages in both parent and child PTEs. Any subsequent store triggers a hardware protection **Page Fault**; the kernel detects the shared CoW frame (`refcount > 1`), allocates one fresh private frame, copies that single $4\text{ KB}$ page, restores `R/W = 1`, and retries the faulting instruction.

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
Strict equality between **virtual page size ($P$)** and **physical frame size ($F$)** is the mathematical invariant enabling $O(1)$ hardware address translation without external fragmentation:
- **MMU Bit-Slice Arithmetic**: Fixing $P = F = 2^k$ bytes partitions every virtual address cleanly into upper page number bits ($p$) and lower pass-through byte offset bits ($d$, widths $k$ bits). The MMU swaps $p \mapsto f$ via the page table/TLB and passes $d$ verbatim without executing multi-word additions, multiplications, or bounds comparisons per page access.
- **Fragmentation Collapse**: With uniform blocks, *every* free physical frame fits *every* virtual page interchangeably—completely eliminating **external fragmentation** (`best-fit`/`compaction` overhead), whereas variable-sized hardware pages would reintroduce variable free-list search, segment limit checks, and relocation compaction.

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
Because paging separates logical and physical spaces through in-memory page tables, naive virtual address translation **doubles main-memory access latency** ($1$ DRAM read for PTE + $1$ DRAM read for target datum; worse with $3\text{--}4\times$ extra accesses in multi-level page trees):
- **Associative Hardware Cache (TLB)**: Implemented as tiny on-chip Content-Addressable Memory (CAM) checked in parallel with L1 cache indexing ($\sim 0.5\text{--}1\text{ ns}$). Upon a hit ($\alpha \approx 98\%\text{--}99\%$), effective access time approaches raw memory latency: $\text{EAT} = \alpha(t_{\text{TLB}} + t_{\text{mem}}) + (1-\alpha)(t_{\text{TLB}} + (k+1)t_{\text{mem}})$.
- **Context-Switch Semantics**: Unless tagged with an **Address Space Identifier (ASID / PCID)**, switching processes (`CR3` reload) invalidates non-global TLB entries to preserve cross-process isolation.

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
The hardware **Dirty (Modified) Bit (`D`)** in each Page Table Entry acts as a write-back write-amplification filter during page replacement:
- **Zero-Overhead Hardware Tracking**: Maintained in Read-Only state initially or cleared upon frame load; whenever a CPU `store` writes through a mapped TLB/PTE entry, MMU hardware atomically latches `D = 1` directly inside the PTE without raising a trap (or via a single initial minor protection fault on architectures managing `D` in software).
- **50% Eviction I/O Reduction**: On eviction, clean pages (`D = 0`) match their authoritative copy already intact on backing swap or executable image—requiring **zero disk writes** before immediate frame reclamation. Only dirty pages (`D = 1`) incur asynchronous swap-out disk writes prior to frame reuse.

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
**Inverted Page Tables (IPTs)** flip per-process virtual indexing into **single global physical-frame indexing** to constrain page-table RAM consumption on wide ($64$-bit) virtual address spaces:
- **Space Bounded by Physical DRAM**: Traditional hierarchical page tables allocate entries proportional to mapped *virtual* memory *multiplied* by process count. An IPT contains exactly one entry per *physical frame* (`<PID, Virtual_Page#>`), bounding page-table size strictly to $\text{RAM} / P$ regardless of how many sparse $64$-bit processes run concurrently.
- **Lookup Latency & Shared Memory Friction**: Because hardware accesses present virtual `<PID, p>`, finding its frame index requires scanning or **Hashed IPT chaining** (adding extra DRAM probes on TLB miss plus hash-table lock contention), and mapping one shared library physical frame cleanly into multiple `<PID, v_addr>` slots violates simple 1-to-1 frame-index uniqueness without auxiliary shared chaining structures.

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
Large virtual address spaces with small page sizes expose both **page-table memory explosion** and **double memory-access latency** simultaneously:
- **Space Scaling Problem ($800\text{ MB}$ metadata)**: A single-level flat array pre-allocates PTEs across the entire $2^{32}$ virtual space ($2^{21}$ entries $\times 4\text{ B} = 8\text{ MB}$ per process $\times 100 = 800\text{ MB}$), even when processes actively touch tiny sparse stack/heap regions separated by gigabytes of empty virtual addresses. **Multi-level hierarchical paging** allocates inner leaf tables *only* for populated virtual branches, cutting metadata footprints from megabytes to tens of kilobytes per process.
- **Latency Amortization via TLB**: Adding page-table tree depth worsens worst-case walk cost from $2$ to $k+1$ DRAM reads per access; an associative hardware **TLB** exploits spatial/temporal locality to bypass the multi-level walk on $>98\%$ of memory references.

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
Multi-level page-tree depth $L$ arises directly from forcing each individual sub-page table node to fit cleanly within **one physical page frame** (preventing contiguous multi-frame page-table allocation):
- **Branching Factor per Level**: A $4\text{ KB}$ frame ($2^{12}\text{ B}$) storing $4\text{ B}$ ($2^2\text{ B}$) PTEs holds exactly $2^{12}/2^2 = 2^{10} = 1024$ entries, consuming **$10\text{ bits}$** of virtual page number index per level.
- **Why Naive 64-Bit Trees Reach 6 Levels**: Subtracting $12$ offset bits leaves $52$ VPN bits ($\lceil 52 / 10 \rceil = 6$ sequential DRAM lookups per TLB miss!). To prevent $6$-deep pointer-chasing penalties in practice, modern $64$-bit ISAs (x86-64, ARM64) restrict canonical virtual addresses to **$48\text{ bits}$** (4-level paging) or **$57\text{ bits}$** (5-level paging) using $8\text{ B}$ PTEs ($9\text{ bits}$ per level).

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
Virtual-to-physical tree geometry balances two independent dimensional constraints: **intra-page branching capacity** and **global physical frame count**:
- **Index Bit Partitioning**: With $64\text{ KB} = 2^{16}\text{ B}$ pages and $4\text{ B} = 2^2\text{ B}$ PTEs, one page table node holds $2^{14}$ entries ($14$ bits). Subtracting $16$ offset bits from a $44$-bit virtual address leaves exactly $28$ VPN bits, splitting symmetrically into two $14$-bit levels ($L_1 = 14$, $L_2 = 14$, Offset $= 16$) so both outer directory and inner leaf tables occupy exact single $64\text{ KB}$ frames.
- **Large-Page Trade-off (Internal Fragmentation vs. Metadata)**: Coarse $64\text{ KB}$ pages dramatically collapse page-table depth and enlarge TLB coverage ($\text{TLB entries} \times 64\text{ KB}$), reducing capacity misses; however, each $4\text{ GB}$ footprint occupies $2^{16} = 65,536$ frames while small allocations suffer increased **internal fragmentation** inside the trailing frame.

---

