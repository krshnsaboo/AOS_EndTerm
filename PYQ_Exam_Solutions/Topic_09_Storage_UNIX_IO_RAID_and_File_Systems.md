# Topic 9: Mass Storage (Disk Scheduling & RAID) and File Systems (`vsfs`, Ch. 11, 13–14 & 3.2)

> **Overview**: Magnetic disk access time & scheduling (`FCFS`, `SSTF`, `SCAN`, `C-SCAN`, `LOOK`, `C-LOOK`), RAID `0/1/3/4/5/6/10`, UNIX I/O & `atime`, File System Interface & Implementation (`vsfs`/`SL-FS`), and consistency semantics.  
> **Reference Material**: [SecondaryStorageStructure_AOS.ppt](../Faculty_Notes/SecondaryStorageStructure_AOS.ppt) | [FileSystemImple-Simple.pptx](../Faculty_Notes/FileSystemImple-Simple.pptx) | [file-implementation-Simple.pdf](../Faculty_Notes/file-implementation-Simple.pdf) | [Ritchie_Unix.pdf](../Faculty_Notes/Ritchie_Unix.pdf) | [thompson-bstj78-unix.pdf](../Faculty_Notes/thompson-bstj78-unix.pdf)  
> **Questions**: 17 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [UNIX Design Advantage: Treating I/O Devices as Files](#1-unix-design-advantage-treating-io-devices-as-files-frequency-1) — Frequency: 1
2. [Block I/O System (Structured) vs. Character I/O System (Unstructured) in UNIX](#2-block-io-system-structured-vs-character-io-system-unstructured-in-unix-frequency-1) — Frequency: 1
3. [Advantages and Problems of Deferred Block I/O (Write-Back Caching) in UNIX](#3-advantages-and-problems-of-deferred-block-io-write-back-caching-in-unix-frequency-1) — Frequency: 1
4. [Advantages of Minimal Difference Between Regular Files and Directories in UNIX](#4-advantages-of-minimal-difference-between-regular-files-and-directories-in-unix-frequency-1) — Frequency: 1
5. [Why Reading a File Causes Block-Level Disk Writes (POSIX `atime` Metadata Updates)](#5-why-reading-a-file-causes-block-level-disk-writes-posix-atime-metadata-updates-frequency-2) — **Frequency: 2**
6. [Consistency Semantics: UNIX Semantics vs. Session Semantics (Pros & Cons)](#6-consistency-semantics-unix-semantics-vs-session-semantics-pros-cons-frequency-1) — Frequency: 1
7. [T/F: "From the System Side, Session Semantics Is Easy Compared to UNIX Semantics"](#7-tf-from-the-system-side-session-semantics-is-easy-compared-to-unix-semantics-frequency-1) — Frequency: 1
8. [File Block Allocation Methods Comparison: Contiguous vs. Linked vs. Indexed](#8-file-block-allocation-methods-comparison-contiguous-vs-linked-vs-indexed-frequency-1) — Frequency: 1
9. [IndixOS File System Layout: Superblock, Inode Table, Bitmaps & Data Blocks](#9-indixos-file-system-layout-superblock-inode-table-bitmaps-data-blocks-frequency-1) — Frequency: 1
10. [Step-by-Step Disk Access Trace for Reading /user1/file.txt & Multi-Level Indirection](#10-step-by-step-disk-access-trace-for-reading-user1filetxt-multi-level-indirection-frequency-1) — Frequency: 1
11. [Role of File System Page/Buffer Cache in Reducing Disk Reads/Writes](#11-role-of-file-system-pagebuffer-cache-in-reducing-disk-readswrites-frequency-1) — Frequency: 1
12. [Mass Storage Anatomy, Access Time & Disk Scheduling Algorithms Worked Numerical (FCFS, SSTF, SCAN, C-SCAN, LOOK, C-LOOK)](#12-mass-storage-anatomy-access-time-disk-scheduling-algorithms-worked-numerical-fcfs-sstf-scan-c-scan-look-c-look-syllabus-ch-11-practice) — **Syllabus Ch 11 / Practice**
13. [AgentOS SL-FS Numerical Design: Max Inodes, 1 KB Log Exhaustion, Max File Size & NAS RAID 1 vs. 5](#13-agentos-sl-fs-numerical-design-max-inodes-1-kb-log-exhaustion-max-file-size-nas-raid-1-vs-5-frequency-1) — Frequency: 1
14. [Choosing Optimal RAID Level for Data-Center Random Read/Write Workloads (RAID 1 vs. 0/4/5)](#14-choosing-optimal-raid-level-for-data-center-random-readwrite-workloads-raid-1-vs-045-frequency-1) — Frequency: 1
15. [RAID Level 3 & RAID Level 4: Reliability, Throughput, Merits & Demerits](#15-raid-level-3-raid-level-4-reliability-throughput-merits-demerits-frequency-2) — **Frequency: 2**
16. [Why RAID 4 Beats RAID 3, the RAID 4 Parity-Disk Bottleneck & How RAID 5 Solves It](#16-why-raid-4-beats-raid-3-the-raid-4-parity-disk-bottleneck-how-raid-5-solves-it-frequency-2) — **Frequency: 2**
17. [RAID Architecture Matrix (RAID 0, 1, 5, 6 & 10): Fault Tolerance, Usable Capacity & Rebuild Resilience](#17-raid-architecture-matrix-raid-0-1-5-6-10-fault-tolerance-usable-capacity-rebuild-resilience-syllabus-ch-11) — **Syllabus Ch 11**

---

### 1. UNIX Design Advantage: Treating I/O Devices as Files [Frequency: 1]

> **Question**: In UNIX, I/O devices are treated as files. Discuss the advantages. *(Note: This question is from research paper)*

#### pre-requisite
- **Reference Paper**: Dennis Ritchie & Ken Thompson, *"The UNIX Time-Sharing System"* (`Ritchie_Unix.pdf`, Section 3.3 "Special Files").
- **Special Files (`/dev/*`)**: Every I/O device in UNIX is represented as a **special file** inside the file system namespace; standard `open`, `read`, `write`, `close` syscalls on it invoke the corresponding device driver.

#### Answer (Exam Writing Format)
Treating I/O devices as special files in UNIX provides the **threefold advantage** highlighted by Ritchie and Thompson:
1. **Uniform I/O Interface**: File I/O and device I/O use the **exact same system calls** (`open`, `read`, `write`, `close`), eliminating separate device-specific APIs and simplifying both user programs and OS design.
2. **Interchangeable Naming & Tool Composability**: File and device names share the **same path syntax and semantics**—any program expecting a filename argument can transparently be passed a device file (e.g., `/dev/tty`, `/dev/lp`, `/dev/null`), enabling seamless I/O redirection and pipelines.
3. **Unified Protection Mechanism**: Special device files are protected by the **exact same permission bits and access-control mechanism** (`rwx` for owner/group/others) as ordinary files, requiring no separate security subsystem for devices.

#### IN DETAIL
Treating peripheral devices as **special file entries** (`/dev/tty`, `/dev/sda`) inside the unified filesystem hierarchy eliminates disjoint kernel I/O subsystems:
- **Polymorphic VFS Dispatching**: All regular files, terminals, pipes, and raw disks respond to one uniform system-call surface (`open, read, write, lseek, close, chmod`). Kernel VFS inspects the target inode's file type (`mode` bits) and redirects operations seamlessly via a per-driver `file_operations` / `bdevsw`/`cdevsw` dispatch table.
- **Unified Protection & Composability**: Standard `uid/gid` and `rwx` permission bits govern device access identically to regular files without separate authorization tables, while shell I/O redirection (`cmd > /dev/lp0`) works out-of-the-box without device-aware application logic.

---

### 2. Block I/O System (Structured) vs. Character I/O System (Unstructured) in UNIX [Frequency: 1]

> **Question**: Discuss the difference between block I/O system and character I/O system in UNIX.

#### pre-requisite
- **Reference Paper**: Ken Thompson, *"UNIX Implementation"* (`thompson-bstj78-unix.pdf`, Section 3).

#### Answer (Exam Writing Format)
| Dimension | Block I/O System ("Structured I/O") | Character I/O System ("Unstructured I/O") |
| :--- | :--- | :--- |
| **Device Model** | Models devices as randomly addressable arrays of fixed-size **512-byte blocks** (e.g., disks, tapes holding file systems). | Covers **all other devices** and unbuffered/unstructured access (e.g., terminals/teletypes, printers, communication lines, raw disk/tape). |
| **Buffering / Caching** | Uses a kernel **buffer pool (cache)** with LRU replacement and delayed (deferred) writes to minimize physical disk I/O. | **Unbuffered by block cache**; requests go directly to device drivers (often using character lists/queues or direct user-area DMA). |
| **Access Granularity** | Fixed block-sized transfers routed through the system buffer cache. | Arbitrary byte/character streams or custom physical record sizes. |
| **Driver Control** | Standardized block interface (`bdevsw`); hides device quirks behind uniform block abstraction. | Device-specific interpretation (`cdevsw`); driver handles raw requests directly (pins user memory during raw disk DMA). |

#### IN DETAIL
Ken Thompson emphasized that UNIX's **block I/O** and **character I/O** split fundamentally reflects **buffered structured abstraction** vs. **unbuffered stream passthrough**:
- **Block I/O ("Structured I/O")**: Targets random-addressable fixed-size sector devices through the **OS kernel buffer/page cache**. Arbitrary byte offsets are mapped transparently into fixed blocks (e.g., $512\text{ B}$ / $4\text{ KB}$), benefitting from caching, read-ahead prefetching, and deferred batched writes (`elevator` disk scheduling).
- **Character I/O ("Unstructured I/O")**: Bypasses the block buffer cache (`raw` mode), dispatching bytes or controller commands directly between user buffers and hardware device registers (`cdevsw` + `ioctl()`). Essential for variable-length serial/terminal byte streams (`tty`, pipes, sockets) and database engines managing private direct-I/O caching (`O_DIRECT`).

---

### 3. Advantages and Problems of Deferred Block I/O (Write-Back Caching) in UNIX [Frequency: 1]

> **Question**: What are the advantages and the problems of deferred block I/O system in UNIX?

#### pre-requisite
- **Reference Paper**: Ken Thompson, *"UNIX Implementation"* (`thompson-bstj78-unix.pdf`, Section 3.1).
- **Deferred (Delayed) Block I/O**: Writes go into an in-memory kernel buffer cache marked "dirty" and are not written to physical disk immediately; actual physical disk writes are deferred until the buffer is evicted/reused or flushed (`sync`).

#### Answer (Exam Writing Format)
- **Advantages**:
  1. **Dramatic Reduction in Physical Disk I/O**: Repeated writes or read-modify-write accesses to the same block (especially metadata, directory, and temporary file blocks) happen at RAM speed without hitting the slow disk each time.
  2. **Higher Throughput & Better Scheduling**: Merging multiple updates into a single physical write allows the disk driver to reorder writes to reduce seek latency.
- **Problems / Drawbacks**:
  1. **Asynchronous Error Handling**: Because `write()` returns before physical disk I/O occurs, reporting hardware write errors meaningfully back to the user process is nearly impossible.
  2. **Crash Inconsistency / Data Loss**: On an unexpected crash or power failure, "logically complete" writes still sitting in dirty RAM buffers are lost (`sync` helps periodically but doesn't eliminate the window).
  3. **Out-of-Order Physical Writes**: Buffer cache associativity/LRU eviction can write blocks to disk in a *different* order than the software issued them, risking inconsistent on-disk file system structures (and disastrous reordering on sequential devices like magnetic tapes, fixed by allowing only 1 pending write per tape drive).

#### IN DETAIL
**Deferred Block I/O (Write-Back Caching)** boosts file-system performance by decoupling `write()` system-call return latency from slow mechanical disk commits:
- **Throughput & Coalescing Gains**: Dirty blocks marked in kernel RAM cache satisfy immediate follow-up `read()` hits at DRAM speed, allow repeated updates to the same inode, free-map, or data block to coalesce into a single physical disk write, and allow the `pdflush` daemon + disk scheduler to sort pending writes by LBA cylinder order.
- **Consistency & Durability Vulnerabilities**: Because modified data resides temporarily in volatile DRAM, sudden power failure or kernel panic silently drops recent writes and can corrupt structural on-disk invariants (e.g., an allocated inode pointing to uninitialized garbage data blocks) unless mitigated via **Write-Ahead Journaling** (`ext3/ext4`), **Copy-on-Write trees** (`ZFS/Btrfs`), or synchronous **`fsync()`** barriers.

---

### 4. Advantages of Minimal Difference Between Regular Files and Directories in UNIX [Frequency: 1]

> **Question**: In the UNIX system, directory is also treated as a file and there is a minimal difference between file and directory. What is the advantage of keeping minimal difference?

#### pre-requisite
- **Reference Paper**: Dennis Ritchie & Ken Thompson, *"The UNIX Time-Sharing System"* (`Ritchie_Unix.pdf`, Section 3.2).
- **UNIX Directory Structure**: A directory in UNIX is stored as an ordinary file containing `(filename, inode_number)` entries, differing mainly in that unprivileged user processes cannot write directly to it (read permission works just like an ordinary file).

#### Answer (Exam Writing Format)
Keeping minimal difference between a regular file and a directory provides key design and operational advantages:
1. **Uniform File System Operations & Code Reuse**: Standard file system primitives (`open`, `read`, `close`, `seek`) and the same inode, block allocation, and caching mechanisms work identically for both directories and ordinary files—simplifying the kernel.
2. **Unified Protection & Permissions**: Directories use the exact same owner/group/other access-control mechanism as regular files (only kernel system calls like `mkdir`/`link` modify directory bytes to preserve hierarchy integrity).
3. **Flexible User-Space Tools**: Standard utilities and user programs with read permission can inspect/read directories directly just like any other file (e.g., `ls` reading directory entries).

#### IN DETAIL
Representing directories structurally as **ordinary byte-stream files storing `(filename, inode_number)` tables** (restricted solely against arbitrary user `write()` calls to protect structural integrity) simplifies core UNIX file system engineering:
- **Code & Mechanism Reuse**: Directories automatically inherit the exact same block allocation pointers (direct/indirect extents), buffer cache infrastructure, path traversal primitives, and metadata `inode` attributes (ownership, `rwx` ACL permissions, timestamps) without maintaining duplicate disk layout code.
- **Clean Hard-Link & Namespace Decoupling**: Storing actual file attributes and block pointers inside the **inode** rather than the directory entry lets multiple directory filenames (`hard links`) point seamlessly to one shared inode, while uniform `readdir`/`open` semantics work coherently across tree levels.

---

### 5. Why Reading a File Causes Block-Level Disk Writes (POSIX `atime` Metadata Updates) [Frequency: 2]

> **Question**: Explain why a Read operation to a file results into several block level write operations on DISK.
> *(2025 AOS Endsem Q1.9 tested this as a 1-mark True/False: "Reading a file leads to writing of disk blocks." — **TRUE**.)*

#### pre-requisite
- **Inode Metadata (`atime`)**: File systems store metadata for every file in an **inode**, including file size, permissions, block pointers, and timestamps (`mtime`, `ctime`, and **`atime` = last access time**).

#### Answer (Exam Writing Format)
- **T/F Verdict**: **TRUE** — Reading a file normally generates disk block writes unless disabled via `noatime`.

1. **Access Timestamp (`atime`) Update**: Whenever a file (and each directory along its path) is read, POSIX/UNIX file systems update the **`atime` (last-accessed time)** metadata field inside the file's (and traversed directories') inode.
2. **Resulting Disk Writes**: Modifying `atime` dirties the inode block in memory, which must eventually be flushed back as **block-level write operations to disk** (even though the user only requested a `read`!).

#### IN DETAIL
Even read-only file workloads trigger hidden metadata disk writes in default POSIX-compliant file systems due to mandatory **access-timestamp (`atime`) bookkeeping**:
- **Write Amplification from `atime`**: Opening or reading a file requires the VFS to update the inode's `st_atime` field in memory (marking the inode buffer dirty), flushing periodic dirty-inode block writes back to disk—hurting flash SSD wear and saturating read-heavy web/mail servers.
- **Mitigation Mount Options**: Modern Linux distributions default to **`relatime`** (updating `atime` only if it predates `mtime`/`ctime` or exceeds 24 hours) or **`noatime`** (disabling access timestamp updates entirely to keep pure read paths strictly read-only at block layer).

---

### 6. Consistency Semantics: UNIX Semantics vs. Session Semantics (Pros & Cons) [Frequency: 1]

> **Question**: Briefly explain the following consistency semantics with positive and negative points. (i) UNIX semantics (ii) session semantics.

#### pre-requisite
- **File Consistency Semantics**: Rules defining *when* modifications made to a shared file by one user/process become visible to other concurrent users/processes (especially in local vs. distributed file systems like NFS/AFS).

#### Answer (Exam Writing Format)
1. **(i) UNIX Semantics**:
   - **Definition**: Every `write()` to an open file is immediately visible to all other processes that currently have that file open (all operations see a single shared file pointer/image).
   - **Positives**: Strong, intuitive consistency—readers always get the exact latest committed data with zero stale reads.
   - **Negatives**: Extremely expensive to enforce in distributed file systems (requires disabling client-side caching or invalidating/synchronizing caches across the network on every single write, creating high latency and server bottlenecks).
2. **(ii) Session Semantics** *(e.g., Andrew File System / AFS)*:
   - **Definition**: Writes to an open file are **not** immediately visible to other remote clients; updates only become visible to newly opened sessions *after* the writing client calls `close()` (ending its session).
   - **Positives**: High scalability and performance in distributed systems—clients can cache whole files locally and read/write at local speed without constant network synchronization.
   - **Negatives**: Weak consistency—concurrent readers/writers see stale data during open sessions, and simultaneous writers overwrite each other on close ("last closer wins").

#### IN DETAIL
Distributed and multi-process file systems trade **strict single-node linearizability (`UNIX Semantics`)** against **scalable client caching (`Session Semantics`)**:
- **UNIX Semantics (Strict Single-Image Visibility)**: Every `write()` instantaneously updates the globally shared file view so concurrent readers immediately observe the new bytes (easy on one OS via shared kernel page cache + inode locks, prohibitively expensive across WAN networks due to mandatory cache-invalidation/lock-revocation round trips on every byte write).
- **Session Semantics (`AFS`-style Relaxed Consistency)**: Clients fetch a full cached replica upon `open()`, buffer all writes purely locally at full client RAM/disk speed with zero network traffic, and push a whole atomic file update back to the server only upon **`close()`**. Concurrent open sessions see isolated snapshots, and late `close()` writes follow last-writer-wins replacement—dramatically lowering server load on mostly-read/single-writer workloads.

---

### 7. T/F: "From the System Side, Session Semantics Is Easy Compared to UNIX Semantics" [Frequency: 1]

> **Question**: *"From the system side, it is easy to provide Session Semantics feature as compared to UNIX semantics feature."*

#### pre-requisite
- **UNIX Semantics vs. Session Semantics**: UNIX semantics maintains a single shared file image where every write is immediately visible; Session semantics (used in distributed systems like AFS) maintains separate per-client cached images during open sessions and pushes updates only on `close()`.

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: From the system implementation side, providing **Session Semantics** is **more complex** than providing basic **UNIX Semantics**.
- **Justification**: Under UNIX semantics (on a single system), all processes simply share one kernel buffer-cache image protected by simple read/write locks. In contrast, implementing Session Semantics requires maintaining multiple independent cached file images across concurrent open sessions, tracking dirty local copies, and implementing complex close-time reconciliation/synchronization protocols for subsequent sessions.

#### IN DETAIL
Comparing **Session Semantics** vs. **UNIX Semantics** from the **OS/distributed system implementation side** flips the naive performance intuition:
- **Why Basic Single-Host UNIX Semantics Is Simpler**: On a shared OS kernel, all processes resolve file accesses through a single shared in-memory `inode` and **unified kernel buffer cache**; basic mutex/page locking around `read()`/`write()` delivers immediate byte visibility automatically with minimal state tracking.
- **Why Session Semantics Increases System Complexity**: Even though network bandwidth drops, the OS/DFS must implement complete **session-boundary state machines**: intercepting `open()` to pull immutable client replicas, tracking dirty client-local shadow copies, intercepting `close()` to flush whole sessions atomically, handling failed partial uploads, and managing server-side version/conflict serialization when overlapping sessions close out of order.

---

### 8. File Block Allocation Methods Comparison: Contiguous vs. Linked vs. Indexed [Frequency: 1]

> **Question**: What file allocation method (Contiguous/Linked/Indexed) would you choose to maximize efficiency in... *(truncated in source PDF)*

#### pre-requisite
- **Three File Allocation Methods**:
  - **Contiguous**: File occupies consecutive disk blocks (`start, length`).
  - **Linked**: Each block contains a pointer to the next block on disk.
  - **Indexed**: An index block (or inode pointers) holds direct/indirect addresses of all blocks of the file.

#### Answer (Exam Writing Format)
Since the exact workload scenario is truncated, here is the **complete decision mapping** across workloads:
1. **Sequential Access & Read-Only / Fixed-Size Files** (e.g., CDs, static media archives): Choose **Contiguous Allocation** — minimal seek time, zero pointer overhead, and fastest sequential/direct access (`start + i`), though prone to external fragmentation and difficult file growth.
2. **Purely Sequential Access with Frequent File Growth** (no random access needed): Choose **Linked Allocation** — zero external fragmentation and arbitrary growth, though random access is $O(n)$ (slow pointer chasing) and pointer failures risk corruption.
3. **Mixed Sequential + Random/Direct Access with Dynamic File Growth** (general-purpose OSes like UNIX): Choose **Indexed Allocation** — supports $O(1)$ direct random access to any block, zero external fragmentation, and easy file growth via multi-level indirect blocks (minor overhead for index blocks).

#### IN DETAIL
Physical disk geometry dictates how **Contiguous, Linked, and Indexed** allocation balance sequential throughput, random offset latency, and external fragmentation:
- **Contiguous Allocation**: Stores `(start_LBA, length)` inside the inode. Sequential streaming achieves peak raw platter transfer with zero head seeks and direct random offset math (`start + offset // B`), yet dynamic file growth triggers expensive relocation copies and severe **external fragmentation**.
- **Linked Allocation**: Eliminates external fragmentation and allows arbitrary growth via per-block `next` pointers; however, random seek to offset $k$ requires $O(k)$ sequential disk reads (solved partially via in-RAM **FAT** caching), while lost/corrupted intermediate pointers sever file tails.
- **Indexed Allocation (UNIX Inodes / Extents)**: Consolidates all block pointers into an **index block / multi-level tree**—supporting $O(1)$–$O(\log N)$ direct random access with zero external fragmentation, trading small per-file pointer metadata overhead (and double-indirect reads on huge sparse files unless compressed using **extents**).

---

### 9. IndixOS File System Layout: Superblock, Inode Table, Bitmaps & Data Blocks [Frequency: 1]

> **Question**: The *F&D Team* is developing a simple file system module for the IndixOS. The team is now developing a small proof of concept with a disk that has eight **tracks** *(edited from "sectors" on exam paper)*, each comprising eight blocks of size 4 KB. The team wants to support storage of different files and directories with the given disk space (as well as their metadata). The F&D team wants the disk to support storage and retrieval of 224 KB of data. They seek your opinion on how certain aspects of the file system can be implemented:
> **a.** According to you, what is the maximum number of files/directories that can be created on the disk based on the constraints (assuming that the metadata of the file or directory consumes 256 bytes), and what does this imply? (3 points)

#### pre-requisite
- **Disk Capacity Partitioning**: Total Disk Capacity = Inode/Metadata space + Superblock/Bitmaps (if any) + User Data Blocks.

#### Answer (Exam Writing Format)
- **Given Parameters**:
  - Total blocks on disk = $8\text{ tracks} \times 8\text{ blocks/track} = 64\text{ blocks}$.
  - Total disk size = $64 \times 4\text{ KB} = 256\text{ KB}$.
  - Required user data capacity = $224\text{ KB} = \frac{224}{4} = 56\text{ data blocks}$.
- **Calculation**:
  - Remaining space available for metadata (inodes):
    $$256\text{ KB} - 224\text{ KB} = 32\text{ KB}\; (= 8\text{ blocks})$$
  - With $256\text{ bytes}$ per file/directory metadata (inode):
    $$\text{Max files/directories} = \frac{32\text{ KB}}{256\text{ bytes}} = \frac{32 \times 1024}{256} = \mathbf{128\text{ files/directories}}$$
  *(Note: If superblock and allocation bitmaps also reside in those 8 non-data blocks, e.g. 3 blocks reserved for superblock/bitmaps like vsfs, metadata gets $5 \times 4\text{ KB} = 20\text{ KB} \implies 80$ inodes; assuming all 32 KB goes to metadata gives the upper bound **128**.)*
- **Implications**:
  1. **Hard Inode Limit**: Even if files are tiny (e.g., 10 bytes each) and plenty of the 224 KB data blocks remain free, **at most 128 files/directories can ever exist** (and note the root directory `/` itself consumes 1 inode, leaving 127!).
  2. **Directory/Empty-File Capacity vs. Data Waste**: If users create 128 small 1 KB files, $128 \times 4\text{ KB} > 224\text{ KB}$, meaning **56 data blocks can only hold at most 56 non-empty files** (1 block each)—so internal fragmentation or running out of data blocks happens before 128 non-empty files!

#### IN DETAIL
A classic minimal **UNIX-like `vsfs` (Very Simple File System)** partitions contiguous raw storage into five deterministic regions whose layout anchors metadata resolution cleanly from fixed byte offsets:
- **Deterministic Fixed Bootstrap Offsets**: Block `0` holds the **Superblock** containing magic signature, filesystem geometry, inode/data block counts, and free-block counts so mounting knows exact offsets without searching. Fixed **Inode Bitmap** (`Block 1`) and **Data Bitmap** (`Block 2`) provide $O(1)$ bit-scan free-slot allocation (`1 bit = 1 object`), followed by a fixed **Inode Table** (`Blocks 3..7`) indexing `File Size`, ownership/permissions, timestamps, and direct/indirect data block pointers across remaining Data Blocks (`Blocks 8..63`).

---

### 10. Step-by-Step Disk Access Trace for Reading /user1/file.txt & Multi-Level Indirection [Frequency: 1]

> **Question**: The team wants to implement a read access operation to `/user1/file.txt` (e.g., `cat /user1/file.txt`). They would like your opinion on handling the read access in this simple file system. Further, how many reads and writes on the disks will be performed when performing this read operation?

#### pre-requisite
- **Path Traversal in a Simple File System (vsfs / UNIX)**: Every file and directory has an inode containing pointers to its data blocks. Directory data blocks simply store `(name, inode_number)` pairs. Root `/` resides at a known fixed inode number (e.g., inode `2` or `0`).

#### Answer (Exam Writing Format)
Assuming a cold cache, each directory/file fits in 1 data block, and POSIX `atime` metadata updates occur:
- **Step-by-Step Read Sequence (`/user1/file.txt`)**:
  1. **Read Root Inode (`/`)**: Locate root inode at known fixed offset in the inode table $\to$ **1 Disk Read**.
  2. **Read Root Directory Data Block**: Follow root inode's pointer to read `/` data block and search for entry `"user1"` to get `user1`'s inode number $\to$ **1 Disk Read**.
  3. **Read `user1` Inode**: Fetch `user1` directory's inode and verify execute/traverse permissions $\to$ **1 Disk Read**.
  4. **Read `user1` Directory Data Block**: Read `user1`'s data block and find entry `"file.txt"` to get `file.txt`'s inode number $\to$ **1 Disk Read**.
  5. **Read `file.txt` Inode**: Fetch `file.txt`'s inode, check read permissions, and obtain block pointer(s) $\to$ **1 Disk Read**.
  6. **Read `file.txt` Data Block(s)**: Read the $k$ data block(s) of `file.txt` (for a 1-block file, $k = 1$) $\to$ **$k$ Disk Reads** (total **$5 + k$**, or **6 reads** for a 1-block file).
- **Disk Writes Performed**:
  - **1 to 3 Disk Writes** (or **0** if `noatime` is used): Updating the **`atime` (last-access time)** field in `file.txt`'s inode (and optionally `/` and `/user1` directory inodes if directory `atime` is tracked) writes modified inode block(s) back to disk!

#### IN DETAIL
Resolving a cold-cache absolute path read (`cat /user1/file.txt`) exposes the hidden **$O(\text{depth})$ metadata traversal chain** underlying simple file I/O:
- **Root-to-Leaf Pointer Chasing**: Without warm directory caches, pathname resolution always anchors at fixed **Root Inode `#2`** in the on-disk Inode Table, reads Root's directory data block (`(name, inum)` mapping) to extract `user1`'s inode number, reads `user1`'s inode, reads `user1`'s directory block to find `file.txt`'s inode number, reads `file.txt`'s inode (following direct or indirect pointer blocks if large), and finally reads `file.txt`'s data block—followed by a metadata write updating `atime` unless `noatime`/`relatime` applies.
- **Indirect Scalability**: Adding multi-level indirect blocks (`single = 1024 ptrs`, `double = 1M ptrs`, `triple = 1B ptrs` for $4\text{ KB}$ blocks / $4\text{ B}$ pointers) expands maximum file size exponentially without bloating fixed small inodes.

---

### 11. Role of File System Page/Buffer Cache in Reducing Disk Reads/Writes [Frequency: 1]

> **Question**: Given that the file system may issue an operation to read from or write to blocks of disks, what are some mechanisms that the file system, in tandem with the disk controller, can use to improve the efficiency of the reads and writes?

#### Answer (Exam Writing Format)
1. **Buffer / Page Caching in RAM**: Cache frequently accessed metadata (root/directory inodes, bitmaps) and data blocks in memory so repeated reads/writes avoid physical disk I/O.
2. **Disk Scheduling (SCAN / C-SCAN / Elevator Algorithm)**: Reorder queued disk block requests in the controller/OS by cylinder/track position to minimize mechanical head seek time.
3. **Read-Ahead (Prefetching) & Contiguous / Extent Allocation**: Prefetch sequential blocks into cache before requested and lay out file blocks contiguously on nearby tracks to minimize seeks.
4. **Delayed (Deferred) Writes / Write Buffering & DMA**: Batch dirty block writes in memory and use DMA to transfer multi-block chunks with minimal CPU interruption.

#### IN DETAIL
Because uncached cold lookups incur numerous random metadata reads per open, modern OS storage stacks combine four architectural optimizations to eliminate redundant platter seeks:
- **Directory Entry Cache (`dcache`) & Inode Cache (`icache`)**: Pin active `<parent_inode, component_name> -> child_inode` mappings in fast hash tables inside DRAM, bypassing multi-hop directory traversals down to instant $O(1)$ memory hits.
- **Unified OS Page Cache**: Caches regular file data blocks and bitmaps in RAM, servicing repeated reads directly at memory speed while **Sequential Read-Ahead (Prefetching)** asynchronously preloads subsequent blocks ahead of user reads.
- **Elevator Disk Scheduling & Coalesced Write-Back**: Reorders dirty flush batches and scattered requests by LBA/cylinder position (`SCAN/LOOK`) to amortize mechanical seek and rotational latency.

---

### 12. Mass Storage Anatomy, Access Time & Disk Scheduling Algorithms Worked Numerical (FCFS, SSTF, SCAN, C-SCAN, LOOK, C-LOOK) [Syllabus Ch 11 / Practice]

> **Question**: List the components of magnetic disk access time ($T_{\text{access}}$). Given a disk queue with cylinder requests **`98, 183, 37, 122, 14, 124, 65, 67`** on cylinders `0–199` with the disk head initially at cylinder **`53`** (and initially moving toward `0` where direction applies), compute the **total head movement (in cylinders)** and compare **FCFS, SSTF, SCAN, C-SCAN (moving toward 199), LOOK, and C-LOOK**.

#### pre-requisite
- **Disk Access Time Formula**: $T_{\text{access}} = T_{\text{seek}} + T_{\text{rotational}} + T_{\text{transfer}} + T_{\text{controller}}$, where $\text{Avg Rotational Latency} = \frac{1}{2} \times \frac{60\text{ s}}{\text{RPM}}$ (e.g., $\approx 4.17\text{ ms}$ at 7,200 RPM). Seek time ($T_{\text{seek}}$) dominates, so schedulers minimize total cylinder travel.

#### Answer (Exam Writing Format)
Given Initial Head = **`53`**, Queue = `98, 183, 37, 122, 14, 124, 65, 67` (sorted: `14, 37, 65, 67, 98, 122, 124, 183`), Boundaries = `0` to `199`:

1. **FCFS** (serve strictly in arrival order):
   - Sequence: `53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67`
   - Movements: $|98-53|+|183-98|+|37-183|+|122-37|+|14-122|+|124-14|+|65-124|+|67-65| = 45+85+146+85+108+110+59+2 = \mathbf{640\text{ cylinders}}$.
2. **SSTF (Shortest Seek Time First)** (closest pending request):
   - Sequence: `53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183`
   - Movements: $(65-53)+(67-65)+(67-37)+(37-14)+(98-14)+(122-98)+(124-122)+(183-124) = 12+2+30+23+84+24+2+59 = \mathbf{236\text{ cylinders}}$. *(Lowest seek here, but risks **starvation** of far tracks!)*
3. **SCAN (Elevator Algorithm, initially moving toward `0`)**:
   - Hits inner end **`0`** before reversing: `53 → 37 → 14 → 0 → 65 → 67 → 98 → 122 → 124 → 183`
   - Total: $(53 - 0) + (183 - 0) = 53 + 183 = \mathbf{236\text{ cylinders}}$. *(No starvation; middle cylinders get visited twice as often as outer ones.)*
4. **LOOK (moving toward `0`, reverses at last request `14`)**:
   - Sequence: `53 → 37 → 14 → 65 → 67 → 98 → 122 → 124 → 183`
   - Total: $(53 - 14) + (183 - 14) = 39 + 169 = \mathbf{208\text{ cylinders}}$.
5. **C-SCAN (Circular SCAN, sweeping upward `53 → 199 → 0 → 37`, per Silberschatz convention)**:
   - Sequence: `53 → 65 → 67 → 98 → 122 → 124 → 183 → 199 → 0 → 14 → 37`
   - Total: $(199 - 53) + (199 - 0) + (37 - 0) = 146 + 199 + 37 = \mathbf{382\text{ cylinders}}$ *(or `183` if counting return jump `199→0` as non-serviced seek; always show both!)*. Provides **uniform wait time**.
6. **C-LOOK (sweeping upward `53 → 183 → 14 → 37`, reverses at extreme requests)**:
   - Sequence: `53 → 65 → 67 → 98 → 122 → 124 → 183 → 14 → 37`
   - Total: $(183 - 53) + (183 - 14) + (37 - 14) = 130 + 169 + 23 = \mathbf{322\text{ cylinders}}$.

| Algorithm | Total Head Movement | Key Tradeoff |
| :--- | :---: | :--- |
| **FCFS** | `640` | Fair, no starvation, huge wild arm swings |
| **SSTF** | `236` | High throughput, risks **starvation** of edge tracks |
| **SCAN** | `236` | Bounded wait, wastes travel `14 → 0` |
| **LOOK** | **`208`** | Best seek here; stops at `14` instead of `0` |
| **C-SCAN** | `382` | Most uniform variance across all cylinders |
| **C-LOOK** | `322` | Uniform variance without empty edge sweeps |

#### IN DETAIL
Magnetic hard-disk service latency splits into three physical hardware components (**$T_{\text{access}} = T_{\text{seek}} + T_{\text{rot}} + T_{\text{transfer}}$**), where mechanical actuator travel (**seek**, $3\text{--}12\text{ ms}$) typically dominates electrical transfer time:
- **Scheduling Algorithm Trade-offs**: **FCFS** guarantees zero starvation yet wastes seek distance zig-zagging across platters (`640` cylinders). **SSTF** greedily chooses the closest cylinder (`236` cylinders), delivering strong throughput while risking starvation for edge cylinders (`14`, `183`) under continuous midrange arrivals. **SCAN / LOOK** sweep monotonically across endpoints (`LOOK` reverses immediately at extreme requests `14` and `183`, trimming wasted boundary seeks down to `208` cylinders). **C-SCAN / C-LOOK** sweep unidirectionally (treating return sweep as non-servicing) to enforce uniform low variance in waiting latency across outer and inner tracks.

---

### 13. AgentOS SL-FS Numerical Design: Max Inodes, 1 KB Log Exhaustion, Max File Size & NAS RAID 1 vs. 5 [Frequency: 1]

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
Evaluating `SL-FS` highlights three critical file-system sizing and redundancy tradeoffs across disk capacity, workload block granularity, and parity overhead:
- **Metadata Starvation on Small Files (Part a)**: Because every file consumes at least one discrete **Inode (`128 B`)**, reserving a fixed metadata budget ($2\%$ of $16\text{ TB} \approx 2^{31}$ potential inodes) still easily hits artificial **inode-table exhaustion** if workloads generate billions of tiny $1\text{ KB}$ logs (which fill fewer data blocks proportionally while consuming 1 inode each)—blocking new `open(O_CREAT)` calls despite abundant free data gigabytes!
- **Multi-Level Indexing Math (Part b)** & **RAID Tradeoff (Part c)**: Expanding address pointers inside $4\text{ KB}$ blocks ($1024$ $4\text{ B}$ pointers/block) grows max file size exponentially via $D + P + P^2 + P^3$, while `8 × 4 TB` NAS designs force a fundamental choice between **RAID 1** ($16\text{ TB}$ usable, optimal random small-write IOPS via 2 mirrored writes) and **RAID 5** ($28\text{ TB}$ usable, degraded small random writes due to the 4-I/O Read-Modify-Write parity penalty).

---

### 14. Choosing Optimal RAID Level for Data-Center Random Read/Write Workloads (RAID 1 vs. 0/4/5) [Frequency: 1]

> **Question**: The *F&D team* wants IndixOS to have support for RAIDs as the OS will primarily be used in the data centre machines. The team is expecting the system to have workloads with more random reads and writes as opposed to sequential reads and writes. Which of the following RAID levels do you recommend and why? (3 points)
> - a. RAID 0
> - b. RAID 1
> - c. RAID 4
> - d. RAID 5

#### pre-requisite
- **Small-Write Penalty in Parity RAID (RAID 4/5)**: Updating a single random block in RAID 4/5 requires **4 physical I/Os** (Read old data, Read old parity, Write new data, Write new parity) to recalculate XOR parity.

#### Answer (Exam Writing Format)
- **Correct Option**: **b. RAID 1 (Disk Mirroring)**.
- **Reasoning**:
  1. **Fault Tolerance (Needed for Data Centers)**: RAID 0 offers zero redundancy—any single disk failure loses all data. RAID 1 tolerates disk failure by mirroring every block onto a twin disk.
  2. **Superior Random Read & Write Performance**:
     - **Random Writes**: RAID 1 writes directly to only the 2 mirrored disks (**2 parallel writes**, zero parity read-modify-write overhead). In contrast, RAID 4 and RAID 5 suffer the severe **Small-Write Penalty** (4 disk I/Os per random write, plus RAID 4's dedicated parity-disk bottleneck).
     - **Random Reads**: RAID 1 can serve independent random reads from **either** mirror copy in parallel, doubling random-read IOPS.

#### IN DETAIL
Data-center workloads dominated by **concurrent small random reads and small random writes** expose the exact structural divergence among RAID levels:
- **Fault Tolerance Exclusion**: **RAID 0** stripes blocks with zero redundancy; failure of any 1 drive destroys the entire array ($\text{MTTDL}$ scales inversely with $N$).
- **Parity Write Penalty vs. Mirroring**: Both **RAID 4** and **RAID 5** must maintain consistent stripe parity across partial-stripe updates via the **Read-Modify-Write cycle** ($P_{\text{new}} = P_{\text{old}} \oplus D_{\text{old}} \oplus D_{\text{new}}$)—converting every single logical random write into **4 physical disk operations** (and serializing all writes on RAID 4's dedicated parity drive). **RAID 1** updates both mirror replicas directly in **2 parallel writes** while dispatching independent random reads concurrently across either copy—making RAID 1 superior for transactional random IOPS whenever 50% capacity overhead is acceptable.

---

### 15. RAID Level 3 & RAID Level 4: Reliability, Throughput, Merits & Demerits [Frequency: 2]

> **Question**: How "RAID Level 3" and "RAID Level 4" improve reliability and disk throughput? What are the advantages of each option?

#### pre-requisite
- **Striping + Parity**: Splitting data across $N-1$ data disks and storing XOR parity on an $N$-th disk allows reconstructing any single failed disk.

#### Answer (Exam Writing Format)
1. **How Reliability & Throughput Are Improved**:
   - **Reliability**: Both store XOR parity on a dedicated parity disk; if any single disk fails, its lost bits/blocks are reconstructed by XORing the surviving disks.
   - **Throughput**: Striping data across multiple disks enables parallel disk access, overcoming single-disk serial transfer limits.
2. **RAID Level 3 (Bit/Byte-Level Striping + Synchronized Spindles)**:
   - *Mechanism*: Bits/bytes of each byte/record are striped across all $N-1$ data disks with 1 parity disk; every I/O reads/writes across **all disks simultaneously**.
   - *Advantage*: Very high **sequential transfer bandwidth** for large contiguous transfers ($N-1$ times single-disk rate).
   - *Disadvantage*: Cannot serve multiple independent random requests concurrently.
3. **RAID Level 4 (Block-Level Striping + Dedicated Parity Disk)**:
   - *Mechanism*: Whole disk blocks are striped across $N-1$ data disks with 1 dedicated parity disk.
   - *Advantage*: Supports **concurrent independent small reads** on separate data disks simultaneously (far higher random-read IOPS than RAID 3), while retaining high aggregate bandwidth on multi-block transfers.
   - *Disadvantage*: Writes bottleneck on the single shared parity disk.

#### IN DETAIL
**RAID 3** and **RAID 4** both dedicate **one XOR parity disk ($P = D_1 \oplus D_2 \oplus \dots \oplus D_{N-1}$)** alongside $N-1$ data disks to survive single-disk failure, yet their **striping granularity** splits them onto opposite performance regimes:
- **RAID 3 (Bit/Byte-Level Striping, Synchronized Spindles)**: Splits every sequential file word bit-by-bit across all $N-1$ drives rotating in lockstep. Every single I/O activates *all* spindles simultaneously—delivering $(N-1)\times$ raw **sequential streaming bandwidth** while limiting random concurrency strictly to **1 request at a time**.
- **RAID 4 (Block-Level Striping, Independent Spindles)**: Places whole multi-kilobyte blocks onto separate disks. Small independent reads hitting distinct blocks execute simultaneously on independent disk arms—multiplying random-read IOPS up to $(N-1)\times$ at the cost of lower single-file streaming speedup unless files span multiple blocks.

---

### 16. Why RAID 4 Beats RAID 3, the RAID 4 Parity-Disk Bottleneck & How RAID 5 Solves It [Frequency: 2]

> **Question**: How RAID 4 is better than RAID 3? Also, compare RAID 4 and RAID 5.
> *(2025 AOS Endsem Q8 focused on RAID Level 4 concept, its parity-disk write bottleneck, and how RAID Level 5 overcomes it.)*

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
**RAID 4** outperforms **RAID 3** on multi-user workloads by shifting from fine-grained bit/byte interleaving to **independent block-level striping**, yet inherits a fundamental write-path scalability flaw resolved by **RAID 5**:
- **Why RAID 4 Bottlenecks on Writes**: Even though independent data blocks reside across $N-1$ distinct drives, *all* XOR parity blocks across every stripe sit exclusively on **one dedicated parity drive (`Disk N`)**. Consequently, two unrelated writes to `Block A (Disk 0)` and `Block B (Disk 2)` must *both* read and overwrite `Disk N`—serializing 100% of random writes onto a single overloaded actuator while leaving `Disk N` completely idle on reads.
- **RAID 5 Rotating Distributed Parity**: Eliminates the dedicated parity drive by cycling parity placement diagonally across all $N$ disks (`Stripe i` stores parity on disk $i \bmod N$). Disjoint stripe writes update distinct parity drives simultaneously in parallel, and normal reads distribute across all $N$ spindles cleanly (though each individual small write still incurs 4 I/Os via Read-Modify-Write).

---

### 17. RAID Architecture Matrix (RAID 0, 1, 5, 6 & 10): Fault Tolerance, Usable Capacity & Rebuild Resilience [Syllabus Ch 11]

> **Question**: Compare **RAID 0, RAID 1, RAID 5, RAID 6, and RAID 10 (`1+0`)** for a storage array of $N$ disks each of capacity $C$ with respect to usable storage efficiency, maximum tolerable disk failures, read/write performance, and rebuild failure risk.

#### pre-requisite
- **Reference Slides**: `SecondaryStorageStructure_AOS.ppt` & Silberschatz Ch. 11.
- **Small-Write Penalty**: Updating 1 random block in single-parity RAID 5 requires 4 I/Os (`Read old data, Read old parity, Write new data, Write new parity`); dual-parity RAID 6 requires 6 I/Os.

#### Answer (Exam Writing Format)
| RAID Level | Architecture | Usable Capacity | Tolerated Disk Failures | Best Workload / Limitation |
| :--- | :--- | :---: | :---: | :--- |
| **RAID 0** | Pure block striping (zero redundancy) | $N \cdot C$ (**100%**) | **0** (1 failure destroys all data) | Max raw bandwidth for scratch/temp data; no reliability. |
| **RAID 1** | Disk mirroring (pairs) | $\frac{N}{2} \cdot C$ (**50%**) | **1 per mirror pair** (up to $N/2$ lucky cases) | Best small random-write IOPS (2 writes, no parity RMW penalty); high storage cost. |
| **RAID 5** | Block striping + **1 distributed XOR parity** | $(N-1) \cdot C$ | **1 disk anywhere** | Good capacity + fast reads; suffers 4-I/O small-write penalty & vulnerable to a 2nd failure during multi-TB rebuilds. |
| **RAID 6** | Block striping + **2 independent distributed parities ($P + Q$)** | $(N-2) \cdot C$ | **Any 2 disks simultaneously** | Essential for large arrays where uncorrectable read error (URE) during RAID-5 rebuild could be fatal; 6-I/O write penalty. |
| **RAID 10 (1+0)** | Stripe *across* mirrored pairs | $\frac{N}{2} \cdot C$ (**50%**) | **1 disk in every mirror pair** | Combines RAID 1 reliability/small-write speed + RAID 0 parallelism; faster, safer rebuilds than RAID 0+1 or RAID 5. |

#### IN DETAIL
Enterprise RAID engineering balances **usable storage efficiency**, **small-write IOPS penalty**, and **Mean Time To Data Loss (MTTDL) during multi-terabyte rebuilds**:
- **The RAID 5 URE Rebuild Cliff**: Reconstructing one failed $16\text{ TB}$ drive in a RAID 5 stripe forces the controller to read **every remaining bit across all $N-1$ surviving drives** continuously for hours/days. Because enterprise HDDs exhibit specified Unrecoverable Read Error (URE) probabilities ($\sim 10^{-14}$–$10^{-15}$ per bit), encountering even one unreadable sector or second mechanical failure mid-rebuild renders XOR recovery mathematically unsolvable—causing total array loss.
- **Enterprise Solutions**: **RAID 6** augments `P` XOR parity with a second algebraic Reed-Solomon / Galois-Field parity (`Q`) surviving *any two* concurrent failures (`P + Q` at 6-I/O write penalty), whereas **RAID 10 (`1+0`)** mirrors pairs before striping—isolating rebuild reads cleanly inside single mirror partners while pairing zero parity-write penalty with 50% capacity overhead.

---

