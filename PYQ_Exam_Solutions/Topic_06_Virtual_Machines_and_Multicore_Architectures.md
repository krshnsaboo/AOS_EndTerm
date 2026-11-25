# Topic 6: Virtual Machines (Ch. 18 & Xen) and Multicore Prerequisites (3.6)

> **Overview**: Multicore OS prerequisites (`Corey`, `Barrelfish`), Silberschatz Ch. 18 Hypervisors (`Type 0/1/2`, Popek–Goldberg, `VT-x`), Xen 4 privilege rings, Memory Virtualization (`EPT`/Ballooning), and Cloud VM Live Migration.  
> **Reference Material**: [xen.pdf](../Faculty_Notes/xen.pdf) | [Corey](../Faculty_Notes/CoreyManyCoreOS123-10-Multicore.pdf) | [Barrelfish](../Faculty_Notes/barrelfish_sosp09-Multikernel.pdf)  
> **Questions**: 6 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [Multicore Prerequisite — Corey Many-Core OS: "Applications Should Control Sharing of OS Data Structures"](#1-multicore-prerequisite-corey-many-core-os-applications-should-control-sharing-of-os-data-structures-frequency-1) — Frequency: 1
2. [Multicore Prerequisite — Barrelfish Multikernel: Cross-Core Cost & Decoupled Structure](#2-multicore-prerequisite-barrelfish-multikernel-cross-core-cost-decoupled-structure-frequency-1) — Frequency: 1
3. [Virtual Machines (Silberschatz Ch. 18): Hypervisor Types, Popek–Goldberg Requirements & x86 Virtualization Techniques](#3-virtual-machines-silberschatz-ch-18-hypervisor-types-popekgoldberg-requirements-x86-virtualization-techniques-syllabus-36) — **Syllabus 3.6**
4. [Xen Paravirtualization: Why Four Hardware Privilege Levels (Rings 0–3) & Example Operations](#4-xen-paravirtualization-why-four-hardware-privilege-levels-rings-03-example-operations-frequency-2) — **Frequency: 2**
5. [Virtual Machine Operations (Multiplexing, Suspension, Provisioning, Live Migration) in Cloud Platforms](#5-virtual-machine-operations-multiplexing-suspension-provisioning-live-migration-in-cloud-platforms-frequency-1) — Frequency: 1
6. [VM Live Migration (Pre-Copy Algorithm) & Memory Virtualization (Shadow Page Tables, EPT & Ballooning)](#6-vm-live-migration-pre-copy-algorithm-memory-virtualization-shadow-page-tables-ept-ballooning-syllabus-36) — **Syllabus 3.6**

---

### 1. Multicore Prerequisite — Corey Many-Core OS: "Applications Should Control Sharing of OS Data Structures" [Frequency: 1]

> **Question**: The Corey multicore operating system is guided by the following observation: **"Applications should control sharing of operating system data structures."** Justify the observation.

#### pre-requisite
- **Reference Paper**: Boyd-Wickizer et al., *"Corey: An Operating System for Many Cores"* (`CoreyManyCoreOS123-10-Multicore.pdf`, Sections 1–3).

#### Answer (Exam Writing Format)
- **Core Bottleneck in Traditional Multicore OSes**:
  - Conventional OSes (like Linux) unconditionally use globally shared kernel data structures (e.g., shared process file-descriptor tables, shared address-space/page-table locks, shared scheduling queues) protected by spinlocks, even when an application's threads do not actually need to share them.
  - On many-core hardware, every modification to shared OS data forces expensive **cache-coherence invalidations, lock contention, remote NUMA/cache misses, and cross-core TLB shootdowns**—causing throughput to **drop** as core count rises (e.g., Linux `dup`/`close` benchmark degrades sharply from 1 to 16 cores).
- **Justification of Corey's Principle ("Applications Control Sharing")**:
  1. **Only Applications Know True Sharing Needs**: The application/runtime knows *which* resources are purely core-private vs. genuinely shared across cores (e.g., in MapReduce, Map tasks write to private buffers with zero sharing needed, whereas Reduce tasks read shared Map outputs).
  2. **Eliminate Unwanted Default Sharing**: By structuring OS data to be **core-private by default** and letting applications explicitly opt into sharing via Corey's abstractions (**Address Ranges, Kernel Cores, and Shares**), the system incurs cache-coherence and TLB-shootdown costs **only when strictly beneficial**—allowing applications like MapReduce to scale cleanly (e.g., 25% faster on 16 cores).

#### IN DETAIL
The **Corey** many-core OS demonstrated empirically that traditional kernel abstractions bottleneck severely on high core counts ($16+$ cores) due to implicit global hardware sharing:
- **Coherence Cost of Implicit Sharing**: Conventional kernels maintain single global tables (open file descriptor arrays, shared virtual address-space page-fault locks, global buffer caches). Even non-overlapping operations across separate cores ping-pong shared cache lines (`RFO` invalidations) and contend for global spinlocks—causing aggregate throughput in Linux `mmap`/`munmap`/socket microbenchmarks to peak early and collapse as cores scale.
- **Application-Managed Sharing Primitives**: Corey exposes three kernel mechanisms—**Address Ranges** (explicitly scoped virtual address regions), **Kernel Cores** (pinning kernel data/execution cleanly to dedicated cores), and **Shares** (opt-in dynamic lookup-table containers)—allowing applications to share OS metadata *only* across cores that explicitly request sharing, letting unshared operations scale nearly linearly.

---

### 2. Multicore Prerequisite — Barrelfish Multikernel: Cross-Core Cost & Decoupled Structure [Frequency: 1]

> **Question**: Regarding the development of Multikernel OS, explain the following motivation: **"Messages cost less than shared memory."**

#### pre-requisite
- **Reference Paper**: Baumann et al., *"The Multikernel: A new OS architecture for scalable multicore systems"* (`barrelfish_sosp09-Multikernel.pdf`, Section 2.4).

#### Answer (Exam Writing Format)
- **Hidden Cost of Shared Memory**: When multiple CPU cores update shared kernel data structures in shared memory, hardware **cache-coherence protocols** must constantly broadcast invalidations and transfer modified cache lines across interconnects (e.g., Barrelfish measured up to ~12,000 extra stall cycles per update on 16 cores), stalling cores on remote cache misses.
- **Why Message Passing Costs Less**:
  1. **Local Cache Containment**: Each core owns private replicated state in its local L1/L2 cache; updates happen via lightweight RPC messages to a single server core whose cache holds the data, avoiding cross-core cache-line bouncing.
  2. **Batching & Pipelining**: Explicit messages can be pipelined and batched asynchronously over the interconnect, achieving higher throughput and lower interconnect congestion than synchronous shared-memory lock contention.

#### IN DETAIL
**Barrelfish**'s principle (*"Messages cost less than shared memory"*) follows directly from physical interconnect behavior in modern heterogeneous NUMA / many-core processors:
- **Hidden Hardware Messaging Inside Shared Memory**: A normal shared-memory lock update does *not* execute directly in RAM—under the hood, CPU cache-coherence hardware already sends broadcast/directory **probe, invalidation, and ownership-transfer messages** across inter-socket links (UPI/QPI/Infinity Fabric) plus stalls cores on serialization barriers (`MFENCE`). As core count grows, implicit coherence traffic scales superlinearly across shared cache lines.
- **Explicit Message-Passing Multikernel Architecture**: Barrelfish structures the OS as a distributed system of independent per-core kernels (*CPU drivers + Monitors*) sharing zero global mutable kernel state; cross-core updates (like page-table shootdowns or capability transfers) travel via batched, cache-line-aligned asynchronous hardware/ring-buffer messages that overlap communication latency, exploit spatial NUMA locality, and tolerate heterogeneous ISAs cleanly.

---

### 3. Virtual Machines (Silberschatz Ch. 18): Hypervisor Types, Popek–Goldberg Requirements & x86 Virtualization Techniques [Syllabus 3.6]

> **Question**: Distinguish between Type 0, Type 1, and Type 2 Virtual Machine Monitors (VMMs) vs. OS-level Containers. State the **Popek–Goldberg virtualization theorem**, explain why classic x86 violated it, and compare the three virtualization techniques (**Binary Translation, Paravirtualization, Hardware-Assisted VT-x/AMD-V**).

#### pre-requisite
- **Reference**: Silberschatz Ch. 18 (*Virtual Machines*); `xen.pdf`.
- **Privileged vs. Sensitive Instructions**: *Privileged* instructions trap in user mode; *Sensitive* instructions read/modify hardware configuration state (`CR3`, interrupt flags, page tables).

#### Answer (Exam Writing Format)
1. **Hypervisor & Virtualization Taxonomy**:
   - **Type 0 Hypervisor**: Hardware/firmware-implemented partitioning (e.g., IBM LPARs, Oracle LDOMs).
   - **Type 1 Hypervisor (Bare-Metal)**: Runs directly on raw hardware at highest privilege hosting Guest OSes (e.g., Xen, VMware ESXi, KVM, Hyper-V); lowest overhead ($1\%–3\%$).
   - **Type 2 Hypervisor (Hosted)**: Runs as a user-space process atop a conventional Host OS (e.g., VirtualBox, VMware Workstation); higher I/O overhead.
   - **OS-Level Virtualization (Containers)**: Shares one single host kernel via isolated namespaces/cgroups (e.g., Docker, Solaris Zones); near-zero overhead, same OS kernel required.
2. **Popek–Goldberg Theorem & the x86 Hole**:
   - *Theorem*: A CPU is cleanly virtualizable via **Trap-and-Emulate** if and only if the set of **sensitive instructions is a subset of privileged instructions**.
   - *x86 Hole*: Classic x86 had ~17 sensitive-but-unprivileged instructions (e.g., `POPF`, `SMSW`, `SGDT`, `SIDT`) that **silently failed or leaked host state** in `Ring 1` instead of trapping!
3. **Three Solutions to the x86 Hole**:
   - **Dynamic Binary Translation (VMware)**: Rewrites guest binary instructions on the fly, substituting safe VMM emulation calls for sensitive non-trapping opcodes (zero guest OS edits; translation CPU overhead).
   - **Paravirtualization (Xen)**: Modifies Guest OS source code (`XenoLinux`) to replace sensitive instructions with explicit **hypercalls** to Xen (near-native speed; requires open/modified guest OS).
   - **Hardware-Assisted Virtualization (Intel VT-x / AMD-V)**: Adds two distinct CPU operating modes (**VMX Root** for VMM, **VMX Non-Root** for Guest OS) so any sensitive guest operation triggers an automatic hardware **`VM-Exit`** trap (run unmodified guests natively).

#### IN DETAIL
Virtual Machine Monitors (**VMMs**) achieve hardware isolation across three deployment architectures and two CPU virtualization models:
- **Popek–Goldberg Theorem**: Classical **Trap-and-Emulate** works without binary modification iff every **sensitive instruction** (reading/writing CPU mode, control registers, or interrupt flags) is strictly a subset of **privileged instructions** (instructions guaranteed to trap when invoked below `Ring 0`).
- **The Classic x86 Virtualization Hole & Modern Fixes**: Legacy 32-bit x86 violated Popek–Goldberg—instructions like `POPF` (failing silently to modify `IF` when in User Mode) or `SGDT`/`SIDT` (leaking physical host descriptor addresses silently without trapping) forced **Binary Translation** (scanning guest kernel code at runtime and rewriting unsafe opcodes, e.g., early VMware) or **Paravirtualization** (modifying guest source code to invoke explicit hypercalls, e.g., Xen) until Intel **VT-x** and AMD-V added dedicated **VMX Root vs. Non-Root** hardware execution modes that cleanly intercept sensitive guest actions via **VM Exits**.

---

### 4. Xen Paravirtualization: Why Four Hardware Privilege Levels (Rings 0–3) & Example Operations [Frequency: 2]

> **Question**: Normally an OS uses two privilege levels (user mode and system mode). Explain why Xen uses **four privilege levels** (`Ring 0`–`Ring 3`), and give 1–2 example operations performed in each privilege level.

#### pre-requisite
- **Reference Paper**: Barham et al., *"Xen and the Art of Virtualization"* (`xen.pdf`).
- **Four-Ring Protection on x86**: Traditional OSes only use Rings 0 and 3. Xen utilizes the x86 architecture's 4 hardware rings (`Ring 0` through `Ring 3`) to protect both the hypervisor from Guest OSes and Guest OSes from Guest applications.

#### Answer (Exam Writing Format)
- **Why Two Privilege Levels Are Insufficient**: A hypervisor must enforce **two independent protection boundaries**: (1) protecting the **Xen Hypervisor** from untrusted **Guest OS kernels**, and (2) protecting each **Guest OS kernel** from its own **Guest user applications**.

Xen divides system execution across **4 hardware privilege levels (Rings 0–3)**:
1. **Ring 0 — Xen Hypervisor (Highest Privilege)**:
   - Runs the hypervisor itself with direct control over physical hardware.
   - *Examples*: Scheduling physical CPUs across guest domains (`Domain 0` / `DomU`); updating physical MMU/page-table hardware; handling physical hardware interrupts and DMA.
2. **Ring 1 — Guest OS Kernel (Paravirtualized OS Mode)**:
   - Runs guest operating system kernels (more privileged than user apps, less privileged than Xen—must invoke **hypercalls** for hardware changes).
   - *Examples*: Scheduling processes/threads inside the guest OS; handling guest system calls (`read`/`fork`) or constructing batched page-table update requests sent to Xen.
3. **Ring 2 — Unused (or Optional Isolated Subsystems)**:
   - Left unused in standard Xen paravirtualization (available if an OS isolates extra drivers/services between Ring 1 and Ring 3).
   - *Examples*: Typically unused (no operations in standard Linux/Xen guests); legacy/optional intermediate driver isolation.
4. **Ring 3 — Guest User Applications (Lowest Privilege)**:
   - Runs unprivileged user programs inside each VM domain.
   - *Examples*: Executing standard application computations (`gcc`, web server logic); invoking user-space system calls (`write()`, `malloc()`) that trap into the Guest OS in Ring 1.

#### IN DETAIL
Traditional OSes require only **two hardware privilege domains** (Kernel Mode `Ring 0` vs. User Mode `Ring 3`), whereas **Xen Paravirtualization** exploits x86's four protection rings to preserve **two distinct security boundaries** simultaneously without hardware virtualization extensions:
- **Dual Isolation Requirement**: The Xen Hypervisor (`VMM`) must maintain sole absolute authority over physical MMU page tables, interrupts, and DMA hardware (**Ring 0**), preventing *any* guest OS from compromising host isolation or neighboring VMs. Simultaneously, each Guest OS kernel must remain strictly isolated from its own untrusted user applications.
- **Ring De-Privileging & Ring Compression**: Xen leaves `Ring 0` exclusively for the hypervisor, relocates the paravirtualized Guest OS kernel down to **`Ring 1`** (so privileged guest attempts trap cleanly into Xen or invoke fast `int 0x82` **Hypercalls**), and leaves Guest user applications in **`Ring 3`** (keeping `Ring 2` optionally for guest middleware or unused), enforcing full hypervisor-to-guest AND kernel-to-user isolation inside native x86 segment/ring protection.

---

### 5. Virtual Machine Operations (Multiplexing, Suspension, Provisioning, Live Migration) in Cloud Platforms [Frequency: 1]

> **Question**: Briefly explain the following operations of Virtual machine approach: (i) Multiplexing, (ii) Suspension, (iii) Provision, and (iv) Live migration. On cloud platforms, discuss how these operations will significantly enhance the utilization of server resources.

#### pre-requisite
- **Hypervisor / Virtual Machine Monitor (VMM)**: Decouples the OS + applications (packaged inside an isolated Virtual Machine image) from underlying physical server hardware.

#### Answer (Exam Writing Format)
1. **Four VM Operations**:
   - **(i) Multiplexing**: Running multiple independent VMs concurrently on a single physical host, dynamically sharing physical CPU, RAM, disk, and NIC hardware via the hypervisor.
   - **(ii) Suspension**: Pausing an executing VM and saving its entire execution state (CPU registers, memory pages, virtual device state) to disk so it can later be resumed from the exact same instruction.
   - **(iii) Provisioning**: Rapidly instantiating/cloning new VMs from pre-configured base VM templates/snapshots on demand without manual OS installation.
   - **(iv) Live Migration**: Moving a running VM—including its RAM and execution state—from one physical host to another over a network with near-zero downtime and without dropping active client connections.
2. **How Cloud Platforms Enhance Server Resource Utilization**:
   - **Server Consolidation (via Multiplexing)**: Replaces many underutilized physical servers (often running at 10–15% load) with high-density VMs packed onto fewer physical servers running at 70–85%+ utilization.
   - **Elastic Scaling & Zero-Idle Waste (via Provisioning & Suspension)**: Cloud providers rapidly spin up VMs during traffic peaks (**provisioning**) and **suspend** idle VMs to disk during off-peak hours, immediately reclaiming RAM and CPU for active paying tenants.
   - **Dynamic Load Balancing & Power Consolidation (via Live Migration)**: Underutilized servers can have their live VMs migrated onto fewer physical machines during low traffic (letting empty servers power down), and overloaded hotspots can be rebalanced seamlessly without service outages.

#### IN DETAIL
Cloud infrastructure management builds directly upon four fundamental properties unlocked by **encapsulating an entire machine's CPU, memory, and virtual disk state inside software abstractions**:
- **Multiplexing**: Time-slicing and spatial partitioning allow a hypervisor to consolidate multiple underutilized OS workloads onto one physical server with strict fault and performance isolation.
- **Suspension & Snapshotting**: Because every piece of virtual machine execution state (vCPU registers, device buffers, virtual RAM, disk blocks) lives in hypervisor-accessible software structures, a VM can be serialized cleanly to disk and resumed identically hours later or cloned (`Provisioning`).
- **Live Migration**: Encapsulation lets a VMM copy guest memory and virtual device state across a data-center network while the VM remains executing on the source host, resuming seamlessly on the target host with millisecond downtime.

---

### 6. VM Live Migration (Pre-Copy Algorithm) & Memory Virtualization (Shadow Page Tables, EPT & Ballooning) [Syllabus 3.6]

> **Question**: Explain the step-by-step **iterative pre-copy Live Migration** protocol used to move a running VM across physical hosts with sub-50 ms downtime. Also explain how VMMs virtualize memory via **Shadow Page Tables vs. Extended Page Tables (EPT/NPT)** and reclaim RAM using **Memory Ballooning**.

#### pre-requisite
- **Address Spaces in Virtualization**: Three address levels exist: **Guest Virtual Address (GVA) $\to$ Guest Physical Address (GPA) $\to$ Host Physical Address (HPA)**.

#### Answer (Exam Writing Format)
1. **Live Migration (Iterative Pre-Copy) Phases**:
   - **Phase 1 — Pre-Copy Memory Iterations**: VMM keeps VM running on Source Host while copying all RAM pages to Destination Host over the network; subsequent rounds resend only pages dirtied during the prior round until the dirty-page rate drops below a small threshold.
   - **Phase 2 — Stop-and-Copy (Brief Pause)**: Briefly pauses the VM ($<50\text{ ms}$), transfers remaining final dirty pages and CPU/device register state.
   - **Phase 3 — Activation & Network Switchover**: Resumes VM on Destination Host and broadcasts an **unsolicited (gratuitous) ARP** updating switches to route packets to the new port without dropping TCP sessions.
2. **Memory Virtualization Mechanisms**:
   - **Shadow Page Tables (Software)**: VMM traps every guest page-table edit (`CR3` update / write-protect traps) and maintains hidden `GVA → HPA` hardware page tables (`VM-Exit` overhead on page faults/context switches).
   - **Nested / Extended Page Tables (Intel EPT / AMD RVI)**: Hardware MMU walks two levels automatically (`GVA → GPA` via guest table, `GPA → HPA` via VMM EPT) without trapping on guest context switches or TLB misses (faster, though a 2D TLB miss can require up to $4 \times 4 + 4 + 4 = 24$ memory lookups vs. 4!).
3. **Memory Ballooning**: A lightweight **balloon driver** inside the Guest OS allocates pinned dummy pages inside the guest (forcing the *Guest OS* to swap out its own least-recently-used pages), then hands those freed physical frames back to the VMM for other VMs.

#### IN DETAIL
**Iterative Pre-Copy Live Migration** achieves sub-50 ms VM relocation across physical hosts without interrupting active TCP connections or guest OS execution:
- **Dirty-Page Rate vs. Network Bandwidth ($w < B_{\text{net}}$)**: During Phase 1, the VMM marks all guest pages clean in the hypervisor's **shadow/EPT page tables** via write-protection traps (`dirty-page logging`) and copies initial RAM asynchronously over the LAN. Concurrent guest writes re-dirty a working subset ($W$). Subsequent iterations copy only newly dirtied pages from the preceding round, converging swiftly as long as page dirtying rate stays well below link bandwidth.
- **Stop-and-Copy Convergence Bound**: Once remaining dirty pages drop below a small threshold (or max rounds elapse), Phase 2 momentarily pauses guest vCPUs, transfers final vCPU registers, device states, and remaining dirty pages ($T_{\text{downtime}} = V_{\text{final}} / B_{\text{net}}$), and emits a **Gratuitous ARP** broadcast updating Top-of-Rack Layer-2 switch MAC tables toward the destination host before resuming execution. *(Hypervisor memory subsystems coordinate via **EPT/NPT** nested two-dimensional paging to avoid VM exits on guest page-table updates and **Memory Ballooning** drivers to reclaim idle guest pages dynamically.)*

---

