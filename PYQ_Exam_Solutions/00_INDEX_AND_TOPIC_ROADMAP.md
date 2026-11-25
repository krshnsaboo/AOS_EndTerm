# AOS End-Semester Exam — Syllabus-Aligned Question & Solution Bank

Tailored specifically for the **CS3.304 Advanced Operating Systems EndSem Syllabus** *(scientific calculator permitted)*:
- **Kept**: All 155 syllabus-relevant + essential-prerequisite PYQs (`[Frequency: N]`)
- **Removed**: 9 out-of-syllabus OSN questions (Physical Layer, Security Ch. 16–17, HTTP Web Caches, IBM/360 Chaining trivia)
- **Added**: 16 high-yield **EndSem Syllabus Core / Calculator Practice / Cross-Chapter Integration** questions covering Disk Scheduling, CSMA/CD, DVR Count-to-Infinity, WFQ Numerical, TCP Tahoe/Reno, Coping with Complexity, Ch. 18 VMs + Live Migration, HPC + Amdahl/Gustafson, HDFS, MapReduce, and Distributed System Models

> **Folders**: [PYQ Papers](../PYQ_Papers) | [Faculty Notes](../Faculty_Notes)  
> **Quick Revision Files**: [EndSEM 2025 Complete Solutions](./EndSEM_2025_Solutions.md) *(AOS & OSN 2025)* | [High-Frequency PYQs (`Frequency > 1`)](./High_Frequency_PYQs_and_Solutions.md) *(33 recurring questions)*

---

## EndSem Topic Roadmap (12 Topics — 171 Questions: 155 PYQs + 16 Syllabus Additions)

| # | Topic | PYQs | Syllabus Core/Practice | Total |
| :---: | :--- | :---: | :---: | :---: |
| 01 | [Topic 1: Introduction, OS Structures & Coping with Complexity (Ch. 1–2 & 3.1)](./Topic_01_OS_Evolution_Hardware_and_Structures.md) | 19 | 1 | **20** |
| 02 | [Topic 2: Processes, Threading & Concurrency, and IPC (Ch. 3–4)](./Topic_02_Processes_Threads_and_IPC.md) | 17 | 0 | **17** |
| 03 | [Topic 3: CPU Scheduling Algorithms & Numerical Calculations (Ch. 5)](./Topic_03_CPU_Scheduling_Algorithms.md) | 20 | 0 | **20** |
| 04 | [Topic 4: Synchronization Tools, Monitors & Classical Problems (Ch. 6–7)](./Topic_04_Synchronization_Monitors_and_Classic_Problems.md) | 13 | 0 | **13** |
| 05 | [Topic 5: Deadlocks — Prevention, Avoidance & Recovery (Ch. 8)](./Topic_05_Deadlocks_Prevention_Avoidance_and_Recovery.md) | 8 | 0 | **8** |
| 06 | [Topic 6: Virtual Machines (Ch. 18 & Xen) and Multicore Prerequisites (3.6)](./Topic_06_Virtual_Machines_and_Multicore_Architectures.md) | 4 | 2 | **6** |
| 07 | [Topic 7: Memory Management, Segmentation, Paging & Page Tables (Ch. 9)](./Topic_07_Memory_Management_Paging_and_Address_Translation.md) | 12 | 0 | **12** |
| 08 | [Topic 8: Virtual Memory, Demand Paging & Page Replacement (Ch. 10)](./Topic_08_Virtual_Memory_Demand_Paging_and_Replacement.md) | 9 | 0 | **9** |
| 09 | [Topic 9: Mass Storage (Disk Scheduling & RAID) and File Systems (`vsfs`, Ch. 11, 13–14 & 3.2)](./Topic_09_Storage_UNIX_IO_RAID_and_File_Systems.md) | 15 | 2 | **17** |
| 10 | [Topic 10: Networking Foundations, Data Link & Medium Access Layer (Tanenbaum & Slides)](./Topic_10_Networking_Foundations_Data_Link_and_MAC.md) | 10 | 1 | **11** |
| 11 | [Topic 11: Network Routing, Transport Layer, TCP & Packet Scheduling WFQ (Tanenbaum & Slides)](./Topic_11_Network_and_Transport_Layers_Routing_TCP_QoS.md) | 26 | 3 | **29** |
| 12 | [Topic 12: HPC (3.3), HDFS (3.4), MapReduce (3.5), Distributed Models (3.7) & Cross-Chapter Integration](./Topic_12_HPC_HDFS_MapReduce_and_Distributed_Models.md) | 2 | 7 | **9** |
| **Total** | **12 Syllabus-Aligned Topic Files** | **155** | **16** | **171** |

---

## Official EndSem Syllabus Coverage Checklist

| Syllabus Item | Where Covered |
| :--- | :--- |
| **1. Silberschatz Ch. 1–10** (Intro, Structures, Processes, Threads, CPU Sched, Sync, Deadlocks, Memory, Virtual Mem) | [Topic 1](./Topic_01_OS_Evolution_Hardware_and_Structures.md) through [Topic 5](./Topic_05_Deadlocks_Prevention_Avoidance_and_Recovery.md), [Topic 7](./Topic_07_Memory_Management_Paging_and_Address_Translation.md), [Topic 8](./Topic_08_Virtual_Memory_Demand_Paging_and_Replacement.md) |
| **1. Silberschatz Ch. 11** (Mass Storage: **Disk Scheduling** & **RAID**) | [Topic 9](./Topic_09_Storage_UNIX_IO_RAID_and_File_Systems.md) |
| **1. Silberschatz Ch. 13–14** & **3.2 Simple File System** (`vsfs`) | [Topic 9](./Topic_09_Storage_UNIX_IO_RAID_and_File_Systems.md) |
| **2.(i)–(ii) Data Link Layer**: Go-Back-N & Selective Repeat Sliding Window | [Topic 10](./Topic_10_Networking_Foundations_Data_Link_and_MAC.md) |
| **2.(iii) Medium Access Layer**: ALOHA, CSMA Variants & **CSMA/CD** | [Topic 10](./Topic_10_Networking_Foundations_Data_Link_and_MAC.md) |
| **2.(iv)–(v) Network Layer**: **Distance Vector Routing** + **Weighted Fair Queuing (WFQ)** | [Topic 11](./Topic_11_Network_and_Transport_Layers_Routing_TCP_QoS.md) |
| **2.(vi) Transport Layer**: **Three-Way Handshake (Establishment & Release)**, TCP Congestion Control & Windows | [Topic 11](./Topic_11_Network_and_Transport_Layers_Routing_TCP_QoS.md) |
| **3.1 Coping with System Complexity** (Saltzer & Kaashoek Ch. 1) | [Topic 1](./Topic_01_OS_Evolution_Hardware_and_Structures.md) |
| **3.3 High Performance Computing (HPC)** + Amdahl / Gustafson Scaling | [Topic 12](./Topic_12_HPC_HDFS_MapReduce_and_Distributed_Models.md) |
| **3.4 Hadoop Distributed File System (HDFS)** & **3.5 MapReduce** | [Topic 12](./Topic_12_HPC_HDFS_MapReduce_and_Distributed_Models.md) |
| **3.6 Virtual Machines** (Silberschatz Ch. 18 & Xen) | [Topic 6](./Topic_06_Virtual_Machines_and_Multicore_Architectures.md) |
| **3.7 Distributed System Models & Enabling Technologies** (Hwang Ch. 1) | [Topic 12](./Topic_12_HPC_HDFS_MapReduce_and_Distributed_Models.md) |
| **5. Cross-Chapter High-Level Synthesis** | [Topic 12](./Topic_12_HPC_HDFS_MapReduce_and_Distributed_Models.md) |

---

## High-Importance Repeated PYQs (`Frequency >= 2` — 33 Questions)

| Frequency | Topic | Question |
| :---: | :--- | :--- |
| **3×** | Topic 01 | [Dijkstra's EWD1303: Probe Instructions (Polling) vs. Hardware Interrupts & Nondeterminism](./Topic_01_OS_Evolution_Hardware_and_Structures.md#11-dijkstras-ewd1303-probe-instructions-polling-vs-hardware-interrupts-nondeterminism-frequency-3) |
| **3×** | Topic 01 | [Evolutionary OS Transitions (Manual → Batch → Multiprogramming → Paging)](./Topic_01_OS_Evolution_Hardware_and_Structures.md#1-evolutionary-os-transitions-manual-batch-multiprogramming-paging-frequency-3) |
| **3×** | Topic 02 | [Medium-Term Scheduler (Swapper): Purpose & Issues If Omitted](./Topic_02_Processes_Threads_and_IPC.md#7-medium-term-scheduler-swapper-purpose-issues-if-omitted-frequency-3) |
| **3×** | Topic 03 | [Progressive Round Robin vs. Ordinary Round Robin (Batch vs. Interactive Users)](./Topic_03_CPU_Scheduling_Algorithms.md#10-progressive-round-robin-vs-ordinary-round-robin-batch-vs-interactive-users-frequency-3) |
| **3×** | Topic 03 | [T/F: OS Gives Higher Priority to I/O-Bound Over CPU-Bound Processes](./Topic_03_CPU_Scheduling_Algorithms.md#4-tf-os-gives-higher-priority-to-io-bound-over-cpu-bound-processes-frequency-3) |
| **3×** | Topic 04 | [Single-Lane Highway / Vermont Bridge Synchronization Using Semaphores](./Topic_04_Synchronization_Monitors_and_Classic_Problems.md#9-single-lane-highway-vermont-bridge-synchronization-using-semaphores-frequency-3) |
| **3×** | Topic 04 | [T/F: "With Monitors, It Is Possible to Reduce Busy Waiting Over Semaphores"](./Topic_04_Synchronization_Monitors_and_Classic_Problems.md#8-tf-with-monitors-it-is-possible-to-reduce-busy-waiting-over-semaphores-frequency-3) |
| **3×** | Topic 05 | [Circular-Wait Deadlock Prevention vs. Deadlock Avoidance (Banker's Algorithm): Overhead & Throughput](./Topic_05_Deadlocks_Prevention_Avoidance_and_Recovery.md#2-circular-wait-deadlock-prevention-vs-deadlock-avoidance-bankers-algorithm-overhead-throughput-frequency-3) |
| **3×** | Topic 08 | [T/F: "Global Frame Allocation Algorithm Gives Better Throughput Over Local Frame Allocation"](./Topic_08_Virtual_Memory_Demand_Paging_and_Replacement.md#5-tf-global-frame-allocation-algorithm-gives-better-throughput-over-local-frame-allocation-frequency-3) |
| **3×** | Topic 08 | [Three-Tier Memory Hierarchy Effective Access Time Numerical (Cache + RAM + Disk = 480,028 ns)](./Topic_08_Virtual_Memory_Demand_Paging_and_Replacement.md#6-three-tier-memory-hierarchy-effective-access-time-numerical-cache-ram-disk-480028-ns-frequency-3) |
| **3×** | Topic 11 | [Distance Vector Routing Numerical: Computing Router C's New Routing Table](./Topic_11_Network_and_Transport_Layers_Routing_TCP_QoS.md#2-distance-vector-routing-numerical-computing-router-cs-new-routing-table-frequency-3) |
| **3×** | Topic 11 | [TCP Connection Establishment (Three-Way Handshake) & Connection Release (Two-Army Problem / TIME_WAIT)](./Topic_11_Network_and_Transport_Layers_Routing_TCP_QoS.md#19-tcp-connection-establishment-three-way-handshake-connection-release-two-army-problem-time_wait-frequency-3) |
| **2×** | Topic 01 | [Dijkstra's THE Operating System: Level 0 & Level 1 Hierarchy vs. Combining Them](./Topic_01_OS_Evolution_Hardware_and_Structures.md#16-dijkstras-the-operating-system-level-0-level-1-hierarchy-vs-combining-them-frequency-2) |
| **2×** | Topic 01 | [Hypothetical Layered OS: Comparing Operations Across Level 11 (Devices) & Level 10 (File System)](./Topic_01_OS_Evolution_Hardware_and_Structures.md#18-hypothetical-layered-os-comparing-operations-across-level-11-devices-level-10-file-system-frequency-2) |
| **2×** | Topic 01 | [Layered OS Approach: Advantages, Disadvantages & Role in System Design](./Topic_01_OS_Evolution_Hardware_and_Structures.md#13-layered-os-approach-advantages-disadvantages-role-in-system-design-frequency-2) |
| **2×** | Topic 01 | [Separation of Mechanism and Policy (Examples & Consequences of Mixing Them)](./Topic_01_OS_Evolution_Hardware_and_Structures.md#12-separation-of-mechanism-and-policy-examples-consequences-of-mixing-them-frequency-2) |
| **2×** | Topic 02 | [Concept and Purpose of Thread Pools](./Topic_02_Processes_Threads_and_IPC.md#14-concept-and-purpose-of-thread-pools-frequency-2) |
| **2×** | Topic 02 | [Many-to-Many Threading Model on Multiprocessors: Performance for K < P, K = P, and P < K < U](./Topic_02_Processes_Threads_and_IPC.md#13-many-to-many-threading-model-on-multiprocessors-performance-for-k-p-k-p-and-p-k-u-frequency-2) |
| **2×** | Topic 03 | [CPU-Bound vs. I/O-Bound Processes & Scheduling Based on Recent CPU Usage](./Topic_03_CPU_Scheduling_Algorithms.md#3-cpu-bound-vs-io-bound-processes-scheduling-based-on-recent-cpu-usage-frequency-2) |
| **2×** | Topic 03 | [Consequence of Misclassifying CPU-Bound and I/O-Bound Processes on Throughput & Response Time](./Topic_03_CPU_Scheduling_Algorithms.md#5-consequence-of-misclassifying-cpu-bound-and-io-bound-processes-on-throughput-response-time-frequency-2) |
| **2×** | Topic 03 | [How FCFS CPU Scheduler Penalizes I/O-Bound Processes (Convoy Effect)](./Topic_03_CPU_Scheduling_Algorithms.md#6-how-fcfs-cpu-scheduler-penalizes-io-bound-processes-convoy-effect-frequency-2) |
| **2×** | Topic 03 | [Problem in Non-Preemptive CPU Scheduling (Convoy Effect & Unresponsive Starvation)](./Topic_03_CPU_Scheduling_Algorithms.md#7-problem-in-non-preemptive-cpu-scheduling-convoy-effect-unresponsive-starvation-frequency-2) |
| **2×** | Topic 03 | [T/F: "The Notion of 'Load Balancing' Counteracts with 'Processor Affinity'"](./Topic_03_CPU_Scheduling_Algorithms.md#15-tf-the-notion-of-load-balancing-counteracts-with-processor-affinity-frequency-2) |
| **2×** | Topic 04 | [Why Monitors Need Condition Variables (c.wait / c.signal) & Differences vs. Semaphores](./Topic_04_Synchronization_Monitors_and_Classic_Problems.md#7-why-monitors-need-condition-variables-cwait-csignal-differences-vs-semaphores-frequency-2) |
| **2×** | Topic 06 | [Xen Paravirtualization: Why Four Hardware Privilege Levels (Rings 0–3) & Example Operations](./Topic_06_Virtual_Machines_and_Multicore_Architectures.md#4-xen-paravirtualization-why-four-hardware-privilege-levels-rings-03-example-operations-frequency-2) |
| **2×** | Topic 08 | [FIFO vs. LRU Page Replacement Traces & Belady's Anomaly (3 vs. 4 Frames)](./Topic_08_Virtual_Memory_Demand_Paging_and_Replacement.md#7-fifo-vs-lru-page-replacement-traces-beladys-anomaly-3-vs-4-frames-frequency-2) |
| **2×** | Topic 08 | [Problems When Swapping Out a Process with Pending Asynchronous I/Os](./Topic_08_Virtual_Memory_Demand_Paging_and_Replacement.md#1-problems-when-swapping-out-a-process-with-pending-asynchronous-ios-frequency-2) |
| **2×** | Topic 09 | [RAID Level 3 & RAID Level 4: Reliability, Throughput, Merits & Demerits](./Topic_09_Storage_UNIX_IO_RAID_and_File_Systems.md#15-raid-level-3-raid-level-4-reliability-throughput-merits-demerits-frequency-2) |
| **2×** | Topic 09 | [Why RAID 4 Beats RAID 3, the RAID 4 Parity-Disk Bottleneck & How RAID 5 Solves It](./Topic_09_Storage_UNIX_IO_RAID_and_File_Systems.md#16-why-raid-4-beats-raid-3-the-raid-4-parity-disk-bottleneck-how-raid-5-solves-it-frequency-2) |
| **2×** | Topic 09 | [Why Reading a File Causes Block-Level Disk Writes (POSIX `atime` Metadata Updates)](./Topic_09_Storage_UNIX_IO_RAID_and_File_Systems.md#5-why-reading-a-file-causes-block-level-disk-writes-posix-atime-metadata-updates-frequency-2) |
| **2×** | Topic 10 | [Pure ALOHA vs. Slotted ALOHA: Implementation Difference & Vulnerable Window Proof](./Topic_10_Networking_Foundations_Data_Link_and_MAC.md#8-pure-aloha-vs-slotted-aloha-implementation-difference-vulnerable-window-proof-frequency-2) |
| **2×** | Topic 10 | [Tradeoffs Between Go-Back-N and Selective Repeat Sliding Window Protocols](./Topic_10_Networking_Foundations_Data_Link_and_MAC.md#7-tradeoffs-between-go-back-n-and-selective-repeat-sliding-window-protocols-frequency-2) |
| **2×** | Topic 11 | [Why Transport Protocols Use Much Larger Sliding Windows Than Data Link Layer (BDP)](./Topic_11_Network_and_Transport_Layers_Routing_TCP_QoS.md#17-why-transport-protocols-use-much-larger-sliding-windows-than-data-link-layer-bdp-frequency-2) |
