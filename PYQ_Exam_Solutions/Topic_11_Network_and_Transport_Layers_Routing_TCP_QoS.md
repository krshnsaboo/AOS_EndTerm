# Topic 11: Network Routing, Transport Layer, TCP & Packet Scheduling WFQ (Tanenbaum & Slides)

> **Overview**: Distance Vector (incl. Count-to-Infinity / Split Horizon) & Link State routing, broadcast & hierarchical routing, IP/ARP/DHCP/NAT/IPv6, Sockets, Checksum, Large Windows, TCP 3-Way Handshake/Release & Reno Congestion Control, QoS, and WFQ numerical scheduling.  
> **Reference Material**: [OverviewOfNetworking.pptx](../Faculty_Notes/OverviewOfNetworking.pptx) | [OverviewOfTransportLayer.pptx](../Faculty_Notes/OverviewOfTransportLayer.pptx)  
> **Questions**: 29 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [Link State Routing Algorithm: Flooding, Shortest-Path Tree & Steps](#1-link-state-routing-algorithm-flooding-shortest-path-tree-steps-frequency-1) — Frequency: 1
2. [Distance Vector Routing Numerical: Computing Router C's New Routing Table](#2-distance-vector-routing-numerical-computing-router-cs-new-routing-table-frequency-3) — **Frequency: 3**
3. [Distance Vector Routing: The Count-to-Infinity Problem, Split Horizon & Poison Reverse vs. Link State Routing](#3-distance-vector-routing-the-count-to-infinity-problem-split-horizon-poison-reverse-vs-link-state-routing-syllabus-2iv) — **Syllabus 2.(iv)**
4. [Broadcast Routing: Reverse Path Forwarding (RPF) vs. Sink Tree Packet Counts & Tradeoffs](#4-broadcast-routing-reverse-path-forwarding-rpf-vs-sink-tree-packet-counts-tradeoffs-frequency-1) — Frequency: 1
5. [T/F: "Hierarchical Routing Results in Shorter Paths Than Flat Routing"](#5-tf-hierarchical-routing-results-in-shorter-paths-than-flat-routing-frequency-1) — Frequency: 1
6. [IndixOS L2/L3 Case (a): Same-Subnet Forwarding via ARP and Ethernet Switch](#6-indixos-l2l3-case-a-same-subnet-forwarding-via-arp-and-ethernet-switch-frequency-1) — Frequency: 1
7. [IndixOS L2/L3 Case (b): Cross-Subnet Forwarding via Default Gateway Router & NAT](#7-indixos-l2l3-case-b-cross-subnet-forwarding-via-default-gateway-router-nat-frequency-1) — Frequency: 1
8. [Dynamic Host IP Configuration Protocol (DHCP) DORA Walkthrough](#8-dynamic-host-ip-configuration-protocol-dhcp-dora-walkthrough-frequency-1) — Frequency: 1
9. [AgentOS Cold-Start End-to-End Flow: OS Stack + DHCP + DNS + HTTP/TCP + ARP/Ethernet](#9-agentos-cold-start-end-to-end-flow-os-stack-dhcp-dns-httptcp-arpethernet-frequency-1) — Frequency: 1
10. [IPv4 vs. IPv6: Why IPv6 Removed Router Fragmentation, Header Checksum & NAT](#10-ipv4-vs-ipv6-why-ipv6-removed-router-fragmentation-header-checksum-nat-frequency-1) — Frequency: 1
11. [TCP Connection Multiplexing & Demultiplexing Using the 4-Tuple & Port Numbers](#11-tcp-connection-multiplexing-demultiplexing-using-the-4-tuple-port-numbers-frequency-1) — Frequency: 1
12. [LangOS Socket API: Kernel Path of a send() Call Down to NIC Frame Transmission](#12-langos-socket-api-kernel-path-of-a-send-call-down-to-nic-frame-transmission-frequency-1) — Frequency: 1
13. [Debugging TCP Connection State & SYN-Flood / Half-Open Sockets](#13-debugging-tcp-connection-state-syn-flood-half-open-sockets-frequency-1) — Frequency: 1
14. [End-to-End Network Path & Connectivity Verification Tools (ping / traceroute / nslookup)](#14-end-to-end-network-path-connectivity-verification-tools-ping-traceroute-nslookup-frequency-1) — Frequency: 1
15. [Transport Layer Data Integrity Validation: 16-Bit One's-Complement Checksum Calculation](#15-transport-layer-data-integrity-validation-16-bit-ones-complement-checksum-calculation-frequency-1) — Frequency: 1
16. [How TCP Provides Reliability Guarantees & Flow Control (Sequence Numbers, ACKs, rwnd)](#16-how-tcp-provides-reliability-guarantees-flow-control-sequence-numbers-acks-rwnd-frequency-1) — Frequency: 1
17. [Why Transport Protocols Use Much Larger Sliding Windows Than Data Link Layer (BDP)](#17-why-transport-protocols-use-much-larger-sliding-windows-than-data-link-layer-bdp-frequency-2) — **Frequency: 2**
18. [TCP Three-Way Handshake Diagnostics: Random Initial Sequence Numbers & Nq = Np + 1](#18-tcp-three-way-handshake-diagnostics-random-initial-sequence-numbers-nq-np-1-frequency-1) — Frequency: 1
19. [TCP Connection Establishment (Three-Way Handshake) & Connection Release (Two-Army Problem / TIME_WAIT)](#19-tcp-connection-establishment-three-way-handshake-connection-release-two-army-problem-time_wait-frequency-3) — **Frequency: 3**
20. [AgentOS TCP Retransmission Timeout Numerical (EstimatedRTT, DevRTT, Timeout = 72.00 ms)](#20-agentos-tcp-retransmission-timeout-numerical-estimatedrtt-devrtt-timeout-7200-ms-frequency-1) — Frequency: 1
21. [Regulating Sending Rates for Congestion Control & Why AIMD Converges to Fairness](#21-regulating-sending-rates-for-congestion-control-why-aimd-converges-to-fairness-frequency-1) — Frequency: 1
22. [TCP Congestion Control Phases: Slow Start, Congestion Avoidance (AIMD), Fast Retransmit & Fast Recovery (Tahoe vs. Reno)](#22-tcp-congestion-control-phases-slow-start-congestion-avoidance-aimd-fast-retransmit-fast-recovery-tahoe-vs-reno-syllabus-sec-2) — **Syllabus Sec 2**
23. [Importance of Quality of Service (QoS) & Admission Control](#23-importance-of-quality-of-service-qos-admission-control-frequency-1) — Frequency: 1
24. [Meeting Multimedia QoS Across Process Scheduler, Disk Scheduler & Memory Manager](#24-meeting-multimedia-qos-across-process-scheduler-disk-scheduler-memory-manager-frequency-1) — Frequency: 1
25. [Why FIFO Packet Scheduling Fails Diverse Applications (Head-of-Line Blocking)](#25-why-fifo-packet-scheduling-fails-diverse-applications-head-of-line-blocking-frequency-1) — Frequency: 1
26. [Issue with Packet-Level Round-Robin Fair Queuing & Byte-by-Byte Weighted Fair Queuing (WFQ) Fix](#26-issue-with-packet-level-round-robin-fair-queuing-byte-by-byte-weighted-fair-queuing-wfq-fix-frequency-1) — Frequency: 1
27. [Weighted Fair Queuing (WFQ / WFG) Algorithm: Fluid-Flow Bandwidth Share & Virtual Finish Time Numerical Trace](#27-weighted-fair-queuing-wfq-wfg-algorithm-fluid-flow-bandwidth-share-virtual-finish-time-numerical-trace-syllabus-2v-practice) — **Syllabus 2.(v) / Practice**
28. [Purpose of Leaky Bucket Traffic Shaping & Consequences Without It](#28-purpose-of-leaky-bucket-traffic-shaping-consequences-without-it-frequency-1) — Frequency: 1
29. [T/F: "Leaky Bucket Performs Better Than Token Bucket Algorithm"](#29-tf-leaky-bucket-performs-better-than-token-bucket-algorithm-frequency-1) — Frequency: 1

---

### 1. Link State Routing Algorithm: Flooding, Shortest-Path Tree & Steps [Frequency: 1]

> **Question**: Briefly explain the link state routing algorithm.

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Link State Routing / OSPF).

#### Answer (Exam Writing Format)
In **Link State Routing** (used in OSPF), every router builds a complete map of the entire network topology and computes shortest paths locally via 5 steps:
1. **Discover Neighbors**: Send `HELLO` packets on all links to learn neighbor router IDs/addresses.
2. **Measure Link Costs**: Measure delay, bandwidth, or administrative cost to each direct neighbor.
3. **Build Link State Packet (LSP)**: Pack `(Router ID, Neighbor list, Link costs, Sequence #, Age)` into an LSP.
4. **Flood LSPs**: Reliably flood the LSP to **all** routers in the network/area (using sequence numbers to stop duplicate/old loops).
5. **Compute Shortest Paths**: Every router holds the identical global graph and runs **Dijkstra's Shortest Path First (SPF)** algorithm locally to populate its routing table.

#### IN DETAIL
Compare **Distance Vector** vs. **Link State**:
- In **Distance Vector**, routers gossip only *"Here's how far I think everyone is"* to immediate neighbors (slow convergence, count-to-infinity bugs!).
- In **Link State**, every router announces ONLY *"Here are my direct neighbors and link costs"* to **everyone in the whole network** (flooding!). Once every router receives everyone else's puzzle pieces, every router draws the exact same global road map and runs **Dijkstra's algorithm** independently—converging fast with zero routing loops!

---

### 2. Distance Vector Routing Numerical: Computing Router C's New Routing Table [Frequency: 3]

> **Question**: Consider the following network. Distance vector routing is used and the following vectors have just come in to router $C$:
> - From $B$: $(5, 0, 8, 1, 6, 2)$
> - From $D$: $(16, 12, 6, 0, 9, 10)$
> - From $E$: $(7, 6, 3, 9, 0, 4)$
>
> The cost of the links from router $C$ to neighbors $B$, $D$, and $E$ are $6, 3,$ and $5$ respectively. What is router $C$'s new routing table?

#### pre-requisite
- **Bellman-Ford Distance Vector Update**: For each destination $y \in \{A, B, C, D, E, F\}$:
  $$D_C(y) = \min_{v \in \{B, D, E\}} \big\{ c(C, v) + D_v(y) \big\}, \quad D_C(C) = 0$$

#### Answer (Exam Writing Format)
Given link costs $c(C, B) = 6$, $c(C, D) = 3$, $c(C, E) = 5$:

| Destination | Via $B$ ($6 + D_B$) | Via $D$ ($3 + D_D$) | Via $E$ ($5 + D_E$) | Min Cost | Next Hop |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **$A$** | $6 + 5 = \mathbf{11}$ | $3 + 16 = 19$ | $5 + 7 = 12$ | **11** | **$B$** |
| **$B$** | $6 + 0 = \mathbf{6}$ | $3 + 12 = 15$ | $5 + 6 = 11$ | **6** | **$B$** |
| **$C$** | — | — | — | **0** | **— (Self)** |
| **$D$** | $6 + 1 = 7$ | $3 + 0 = \mathbf{3}$ | $5 + 9 = 14$ | **3** | **$D$** |
| **$E$** | $6 + 6 = 12$ | $3 + 9 = 12$ | $5 + 0 = \mathbf{5}$ | **5** | **$E$** |
| **$F$** | $6 + 2 = \mathbf{8}$ | $3 + 10 = 13$ | $5 + 4 = 9$ | **8** | **$B$** |

- **Router $C$'s New Routing Table**:
  - **Distance Vector**: `(11, 6, 0, 3, 5, 8)`
  - **Outgoing Lines / Next Hops**: `(B, B, -, D, E, B)`

#### IN DETAIL
Watch how simple and mechanical Distance Vector Routing is!
Router $C$ doesn't know the whole map—it only knows the cost to step onto its three neighbor bridges ($B=6$, $D=3$, $E=5$). When $B$, $D$, and $E$ mail in their distance vectors (each a 6-tuple for `A, B, C, D, E, F`), $C$ simply adds $6$ to every number from $B$, $3$ to every number from $D$, and $5$ to every number from $E$, then picks the smallest total column-by-column (always keeping $C \to C = 0$)!

**Scan Trivia (2022 Endsem Q11 Check)**:
- In `OSN_end_2022_M_key.pdf`, the printed solution key had a minor typo `6+1=18` on D's entry via B (correct is `6+1=7`), which fortunately did not alter the final routing table `(11, 6, 0, 3, 5, 8)` via `(B, B, -, D, E, B)` because direct `C→D = 3 < 7`!

---

### 3. Distance Vector Routing: The Count-to-Infinity Problem, Split Horizon & Poison Reverse vs. Link State Routing [Syllabus 2.(iv)]

> **Question**: Explain the **Count-to-Infinity** pathology in Distance Vector Routing (*"good news travels fast, bad news travels slowly"*) with a 3-router example ($A - B - C$). How do **Split Horizon** and **Split Horizon with Poison Reverse** mitigate it, and when do they still fail? Compare **Distance Vector** vs. **Link State** routing.

#### pre-requisite
- **Reference**: `OverviewOfNetworking.pptx` (Slides 82–85) & Tanenbaum Ch. 5.

#### Answer (Exam Writing Format)
1. **Count-to-Infinity Example ($A \leftrightarrow B \leftrightarrow C$, all links cost `1`)**:
   - Initially: $D_B(A) = 1$ (via $A$), $D_C(A) = 2$ (via $B$).
   - When link $A\text{--}B$ fails ($c(B,A) = \infty$), $B$ sees $C$'s stale advertisement $D_C(A) = 2$ and mistakenly concludes it can reach $A$ via $C$ with cost $1 + 2 = 3$!
   - Next round, $C$ hears $D_B(A) = 3$ and updates $D_C(A) = 1 + 3 = 4$, then $B \to 5, C \to 6, \dots$ slowly counting up to the infinity threshold (`16` in RIP) while packets ping-pong in a **routing loop**.
2. **Mitigations**:
   - **Split Horizon**: A router never advertises a route to destination $X$ back out the exact neighbor interface from which it learned the best route to $X$ ($C$ omits $A$ when updating $B$).
   - **Split Horizon with Poison Reverse**: $C$ explicitly advertises $D_C(A) = \infty$ back to $B$, stopping 2-node loops immediately. *(Limitation: fails on 3+ node cycles such as triangles/diamonds!)*
3. **Distance Vector vs. Link State**:
   - **Distance Vector (Bellman-Ford)**: Shares full routing vector **only with immediate neighbors**; small messages, suffers Count-to-Infinity & slow convergence.
   - **Link State (Dijkstra)**: Floods **only local link costs to ALL routers**; fast convergence, loop-free global map, higher memory/LSA flooding overhead.

#### IN DETAIL
**"Good news travels fast, bad news travels slowly"**:
- If a brand-new fast link appears (**good news**), every neighbor jumps onto it at 1 hop per round!
- If $A\text{--}B$ dies (**bad news**), $B$ forgets $C$ got its path *from $B$ itself*! So $B$ falls for its own echoed lie via $C$ (`3 -> 4 -> 5 -> 6...`), creating a temporary black hole loop! **Poison Reverse** tells $C$ to shout back *"My distance to A via you is INFINITY"*, breaking 2-hop ping-pong loops cleanly (though 3+ router rings still need Link State or RIP hold-downs!).

---

### 4. Broadcast Routing: Reverse Path Forwarding (RPF) vs. Sink Tree Packet Counts & Tradeoffs [Frequency: 1]

> **Question**: Looking at the following network, how many packets are generated by a broadcast from B using (a) reverse path forwarding (b) the sink tree? Which is favored for the implementation and why?
> *(Note: The source PDF `OSN_end_2021_M.pdf` does NOT contain the referenced network topology diagram!)*

#### pre-requisite
- **Reverse Path Forwarding (RPF)**: When router $R$ receives a broadcast packet from source $S$ over incoming link $L$: if $L$ is the link $R$ would normally use to send packets *back to $S$* (shortest path to $S$), $R$ forwards copies out of **all other links**; otherwise $R$ drops it.
- **Sink Tree (Spanning Tree) Broadcast**: A loop-free spanning tree rooted at source $B$ (containing $N-1$ links for $N$ nodes); packets are forwarded strictly along tree edges.

#### Answer (Exam Writing Format)
*(Because the network graph is absent in the `OSN_end_2021_M.pdf` scan, exact packet counts cannot be computed without the topology; below is the exact analytical method and conceptual comparison required by the question)*:
- **(a) Reverse Path Forwarding (RPF) Packet Count Formula**:
  - Each router accepts the broadcast only if it arrives on its **preferred shortest-path interface back to $B$**, and copies it to all other outgoing links. Because routers still forward copies across non-tree links (which neighboring routers then drop upon arrival), RPF generates **more than $N-1$ packets** (potentially up to $2E - \text{overhead}$ depending on node degrees, though avoiding infinite loops).
- **(b) Sink Tree Broadcast Packet Count Formula**:
  - Since a spanning tree over $N$ nodes has exactly **$N-1$ edges**, broadcasting from $B$ along the sink tree generates **exactly $N - 1$ packets** (each node receives the packet once, zero duplicates).
- **Which Is Favored & Why**:
  - **Sink Tree** is ideal in terms of bandwidth (minimum possible $N-1$ packets, zero duplicate packets), **if** routers already maintain global topology / spanning tree state (e.g., via Link State routing).
  - **Reverse Path Forwarding (RPF)** is favored in distributed Distance-Vector / hop-by-hop networks where routers do **not** know the full sink tree, because RPF needs **zero extra routing tables or spanning-tree setup**—it uses only the existing unicast routing table!

#### IN DETAIL
Think of spreading news from person $B$ to a town of $N = 10$ houses:
- **Sink Tree** draws a clean family tree with $N-1 = 9$ arrows connecting all 10 houses with zero loops. You send **exactly 9 messages** total—impossible to beat!
- **Reverse Path Forwarding (RPF)** doesn't require anyone to draw a town map first. Every house simply checks: *"Did this message from $B$ arrive on the shortest road back to $B$?"* If **Yes**, forward it down every *other* driveway (even if the neighbor already heard it from someone else, who will then discard the duplicate!). If **No**, discard it!
- Notice the tradeoff: Sink Tree uses minimal packets ($N-1$) and wastes zero bandwidth, whereas RPF is super easy to implement when routers only know their own unicast next-hops! *(Since `OSN_end_2021_M.pdf` omitted the figure, state this distinction clearly in an exam if ever shown a similar missing/unprinted graph.)*

---

### 5. T/F: "Hierarchical Routing Results in Shorter Paths Than Flat Routing" [Frequency: 1]

> **Question**: *"Hierarchical routing results in shorter paths than flat routing."*

#### pre-requisite
- **Hierarchical Routing**: Routers are grouped into regions/clusters/Autonomous Systems; routers know detailed internal paths within their own region and only aggregate routes toward external regions (greatly shrinking routing tables!).

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: Hierarchical routing results in **longer (suboptimal) paths** than flat routing (in exchange for much smaller routing tables and scalability).
- **Justification**: In flat routing, every router knows the exact topology to every destination and always selects the globally shortest path. In hierarchical routing, routers only route via cluster representatives/entry points without visibility into internal links of remote clusters, which often forces suboptimal (longer) paths—a trade-off accepted to shrink routing table sizes in huge networks.

#### IN DETAIL
Imagine driving from one small suburb to a neighboring suburb just across a river, but hierarchical GPS forces all inter-city traffic to pass through the **City Central Highway Exit** first! That adds extra kilometers (**longer path**), yet it saves your GPS from having to memorize every single neighborhood side-street across the entire country (**huge memory/routing-table savings**)!

---

### 6. IndixOS L2/L3 Case (a): Same-Subnet Forwarding via ARP and Ethernet Switch [Frequency: 1]

> **Question**: The *Net Team* has completed the network stack development following the OSI model. Now, they want to perform some tests to check if everything is working fine. They have a machine with an IP address `192.168.1.25`, MAC `9B:8C:6D:5E:3E:2F`, with a subnet mask of `/24`. The public IP address of the network is `17.84.18.65`, and the IP address of the router is `172.18.12.92`; MAC address ID: `7G:98:1C:1A:2D` *(note: printed as `7G` in exam paper; MACs normally use hex `0–9, A–F`)*. The team wants to understand how the data transfer can be achieved in the following cases:
> **a.** The destination machine has an IP address of `192.168.1.32` with a subnet mask `255.255.255.0`, MAC `8A:9B:1C:2F:9F:6D`. From L2 perspective, can you help the team understand how this communication can work with reasoning? (5 points)

#### pre-requisite
- **Subnet Matching**: Host applies its subnet mask (`255.255.255.0` = `/24`) using bitwise `AND` to compare its own network prefix (`192.168.1.0/24`) with the destination IP's prefix.
- **Same-Subnet L2 Forwarding**: Hosts on the exact same subnet communicate **directly over Layer 2 (Ethernet)** without needing the Layer 3 router.

#### Answer (Exam Writing Format)
1. **Subnet Check**:
   - Source: `192.168.1.25 & 255.255.255.0 = 192.168.1.0`
   - Destination: `192.168.1.32 & 255.255.255.0 = 192.168.1.0`
   - Both machines lie on the **exact same local subnet (`192.168.1.0/24`)**, so **no router forwarding is needed**.
2. **L2 Communication Steps**:
   - **ARP Resolution**: The source checks its local ARP cache for `192.168.1.32`. If missing, it broadcasts an **ARP Request** (`FF:FF:FF:FF:FF:FF`) on the LAN asking *"Who has `192.168.1.32`?"* Destination replies with a unicast **ARP Reply** containing its MAC `8A:9B:1C:2F:9F:6D`.
   - **L2 Ethernet Encapsulation**: The OS constructs an Ethernet frame with:
     - **Source MAC**: `9B:8C:6D:5E:3E:2F`
     - **Destination MAC**: `8A:9B:1C:2F:9F:6D`
     - **Payload**: IP packet (`Src IP: 192.168.1.25`, `Dst IP: 192.168.1.32`).
   - **L2 Switching / Delivery**: The local L2 switch inspects the Destination MAC, learns/looks up the destination switch port in its MAC forwarding table, and delivers the frame directly to `192.168.1.32` (never involving the default gateway router).

#### IN DETAIL
Even though the question mentions the router's IP/MAC and public IP as context (distractors!), the very first thing your OS network stack does before sending *any* packet is check: **"Is the destination in my own neighborhood (`/24` subnet)?"**
- `192.168.1.25` and `192.168.1.32` share the same first 24 bits (`192.168.1.*`). They are right next door on the same LAN switch!
- So you **never bother the router**. Your machine simply uses **ARP** to find the neighbor's MAC address (`8A:9B:1C:2F:9F:6D`), puts its own MAC as sender and the neighbor's MAC as destination on the Ethernet frame, and the local switch delivers it directly!

---

### 7. IndixOS L2/L3 Case (b): Cross-Subnet Forwarding via Default Gateway Router & NAT [Frequency: 1]

> **Question**: The destination machine has an IP address of `10.8.17.65` with a subnet mask of `255.255.255.0` and a MAC address of `1A:2B:3C:4D:9C:5A`. The machine is in a network with an outgoing IP address of the router being `18.72.19.85` and the internal address of the router is `10.8.17.89 /24`. Can you explain to the team how the data transfer works in this scenario? *(Hint: Think about both L2 and L3)* (5 points)

#### pre-requisite
- **Cross-Subnet Forwarding + NAT**: When source and destination are on **different subnets**, the source host addresses the **L3 IP packet** to the remote IP (or public NAT IP) and addresses the **L2 Ethernet frame** to its **local default gateway router's MAC** (note that local MAC addresses are *never* visible across routers!).
- *(Note on paper typo)*: In the prompt's setup, the local router's IP is written as `172.18.12.92` (likely its WAN/gateway interface or exam typo, since a host's local default gateway normally sits on `192.168.1.x/24`) with MAC `7G:98:1C:1A:2D` (also a typo `G` in hex). Explain the exact L2/L3 protocol behavior cleanly while noting the local gateway assumption!

#### Answer (Exam Writing Format)
1. **Subnet Mismatch & Routing Decision (Source Host)**:
   - Source (`192.168.1.25/24`) compares its prefix (`192.168.1.0`) with Destination (`10.8.17.65/24` $\to$ `10.8.17.0`). Since they differ, the packet **must be sent via the default gateway router** (`MAC: 7G:98:1C:1A:2D`, internal LAN gateway on source's subnet).
2. **Hop 1 — Source Host to Source Router (L2 + L3)**:
   - **L3 Header**: `Src IP = 192.168.1.25`, `Dst IP = 18.72.19.85` (if NAT/port-forwarded across public Internet, or `10.8.17.65` if directly routed across private enterprise routers).
   - **L2 Frame**: Host ARPs its local gateway MAC and sends frame with `Src MAC = 9B:8C:6D:5E:3E:2F`, `Dst MAC = 7G:98:1C:1A:2D`. *(Source never uses destination's MAC `1A:2B:3C:4D:9C:5A`, which is unreachable outside the remote LAN!)*
3. **Router Processing & NAT (Source Router $\to$ Internet $\to$ Dest Router)**:
   - Source router strips L2 header, translates private `Src IP 192.168.1.25` to public IP `17.84.18.65` (NAT, updating port table if traversing public Internet), decrements `TTL`, looks up L3 routing table, and wraps packet in a new L2 frame for the next WAN hop.
   - Across each intermediate router hop, **L2 MAC headers are stripped and rewritten per hop**, while **L3 IP addresses stay end-to-end** (except at NAT boundaries).
4. **Final Hop — Destination Router (`10.8.17.89/24`) to Destination (`10.8.17.65`)**:
   - Destination router receives the packet at external IP `18.72.19.85`, performs reverse NAT/routing lookup for `10.8.17.65` on its internal `/24` interface (`10.8.17.89`), ARPs for `10.8.17.65` to obtain MAC `1A:2B:3C:4D:9C:5A`, and transmits the final Ethernet frame (`Src MAC = Dest Router internal MAC`, `Dst MAC = 1A:2B:3C:4D:9C:5A`) for delivery.

#### IN DETAIL
Think of **IP addresses (L3)** as the sender/receiver **street addresses on the envelope**, and **MAC addresses (L2)** as the **truck driver handing the package across each individual leg of the journey**:
- Because `192.168.1.25` and `10.8.17.65` are in completely different cities (subnets) behind private NAT routers (`17.84.18.65` and `18.72.19.85`), your laptop cannot hand a packet directly to `1A:2B:3C:4D:9C:5A`—it doesn't even exist on your local switch!
- Instead, your laptop hands the L2 envelope to **its local post office (Default Gateway Router MAC)**, while writing the **public destination IP** on the L3 packet inside.
- Note also the exam's two small quirks: `192.168.1.x` and `10.8.17.x` are *private RFC 1918 IP addresses* (so over the public Internet, routers use **NAT** between `17.84.18.65` and `18.72.19.85`), and the printed router MAC `7G:...` has a non-hex `G` typo while its IP `172.18.12.92` doesn't match `192.168.1.x/24`—mentioning how private/public NAT + hop-by-hop L2 MAC rewriting works covers every mark cleanly!

---

### 8. Dynamic Host IP Configuration Protocol (DHCP) DORA Walkthrough [Frequency: 1]

> **Question**: The *Net Team*, in order to test their network stack, has decided to connect a machine with IndixOS to a network (`172.18.20.x/24`). However, the team wanted to check with you how the host machine can get an IP address in the given network. What can be the protocol that can be leveraged, and how does it work? (3 points)
> - a. DNS
> - b. HTTP
> - c. DHCP
> - d. ARP

#### pre-requisite
- **DHCP (Dynamic Host Configuration Protocol)**: Application/UDP-based bootstrap protocol that dynamically leases an IP address, subnet mask, default gateway, and DNS server to a newly joined host.

#### Answer (Exam Writing Format)
- **Correct Option**: **c. DHCP (Dynamic Host Configuration Protocol)**.
- **How It Works (DORA 4-Step Process)**:
  1. **Discover (Client $\to$ Broadcast `255.255.255.255`)**: New host (with source IP `0.0.0.0`) broadcasts a `DHCPDISCOVER` message to locate available DHCP servers on the subnet.
  2. **Offer (Server $\to$ Client)**: DHCP server(s) respond with `DHCPOFFER`, proposing an available IP (e.g., `172.18.20.15/24`), lease duration, default gateway, and DNS server.
  3. **Request (Client $\to$ Broadcast)**: Client broadcasts `DHCPREQUEST` selecting one offered IP (informing all servers which offer was accepted).
  4. **Acknowledge (Server $\to$ Client)**: Selected server replies with `DHCPACK`, confirming the IP lease so the host configures its interface.

#### IN DETAIL
When a brand-new laptop plugs into `172.18.20.x/24`, it has **no IP address yet**—so how can it even send/receive messages?
- **DHCP** uses broadcast (`255.255.255.255`, MAC `FF:FF:FF:FF:FF:FF`) over UDP ports 67/68! Remember the mnemonic **D-O-R-A**: **D**iscover (*"Anyone handing out IPs?"*) $\to$ **O**ffer (*"Take `172.18.20.15` for 24 hrs!"*) $\to$ **R**equest (*"I'll take `172.18.20.15`!"*) $\to$ **A**cknowledge (*"Deal, it's yours!"*).
- Why not the others? **DNS** turns domain names into IPs; **HTTP** transfers web pages; **ARP** finds a local MAC address *when you already know the IP*!

---

### 9. AgentOS Cold-Start End-to-End Flow: OS Stack + DHCP + DNS + HTTP/TCP + ARP/Ethernet [Frequency: 1]

> **Question**: A node running AgentOS connects to a network without having any IP address assigned. Immediately after joining, a user process attempts to send an HTTP request to `http://anotherdomain.com` (server IP: `179.20.10.6`). The AgentOS team requires your help in understanding the end-to-end sequence of steps from the moment the node connects until the HTTP request is ready to leave the link layer and move toward the next hop.
> **a.** Describe what the Operating System must provide to enable network communication. (2 points)
> **b.** Explain the mechanisms the node uses to acquire its own IP address and to resolve the destination server's IP address. (4 points)
> **c.** Describe the activities at the Application, Transport, and Link layers that prepare the HTTP request for transmission toward the next hop (excluding network-layer routing details). (4 points)

#### pre-requisite
- **Bootstrap Sequence**: A freshly joined node needs (1) OS network stack + NIC drivers, (2) **DHCP** to obtain its local IP/gateway/DNS, (3) **DNS** to map hostname $\to$ IP, and (4) **ARP** to resolve the next-hop gateway MAC address before sending an HTTP/TCP/Ethernet frame.

#### Answer (Exam Writing Format)
##### **(a) What the OS Must Provide (2 Points)**
1. **Network Interface Card (NIC) Hardware Drivers**: To interact with physical Ethernet/Wi-Fi hardware and transmit/receive raw frames via interrupts/DMA.
2. **Kernel Protocol Stack & Socket API**: Implementation of Link (ARP/Ethernet), Network (IP/ICMP), and Transport (TCP/UDP) protocols in the kernel, exposed to user processes via BSD **socket syscalls** (`socket`, `connect`, `send`, `recv`) and kernel socket buffers.

##### **(b) Mechanisms to Acquire Own IP & Resolve Destination IP (4 Points)**
1. **Acquiring Own IP Address — DHCP (DORA)**:
   - Node broadcasts `DHCPDISCOVER` $\to$ DHCP server replies with `DHCPOFFER` $\to$ Node broadcasts `DHCPREQUEST` $\to$ Server sends `DHCPACK`.
   - Along with its own **IP address + Subnet Mask**, the node simultaneously receives the **Default Gateway Router's IP** and **Local DNS Server's IP**.
2. **Resolving Destination Server's IP (`anotherdomain.com`) — DNS**:
   - The node sends a **DNS Query** (over UDP port 53) to the configured local DNS resolver (checking local hosts/DNS cache first).
   - If uncached, the resolver queries Root $\to$ TLD (`.com`) $\to$ Authoritative DNS servers recursively/iteratively and returns `179.20.10.6` to the node.

##### **(c) Activities at Application, Transport, and Link Layers (4 Points)**
1. **Application Layer**:
   - Browser/client formats the plaintext **HTTP GET request** (`GET / HTTP/1.1\r\nHost: anotherdomain.com\r\n...`), opens a TCP socket (`socket()`, `connect(179.20.10.6, 80)`), and passes the HTTP message to the OS via `send()`.
2. **Transport Layer (TCP)**:
   - Assigns an ephemeral **Source Port** and Destination Port **`80`** (HTTP), performs the **TCP Three-Way Handshake** (`SYN`, `SYN-ACK`, `ACK`) with `179.20.10.6`, segments the HTTP payload, computes the 16-bit **TCP Checksum**, sets sequence/window fields, and passes segments to IP.
3. **Link Layer (Ethernet + ARP)**:
   - Because `179.20.10.6` is outside the local subnet, IP determines the next hop is the **Default Gateway Router**.
   - Link layer checks its **ARP cache** for the Default Gateway's IP (broadcasting an **ARP Request** if missing to learn the Gateway's MAC address).
   - Encapsulates the IP packet inside an **Ethernet Frame** with `Src MAC = Node's NIC MAC`, `Dst MAC = Default Gateway Router's MAC`, appends a **32-bit CRC/FCS trailer**, and pushes bits onto the wire via the NIC driver.

#### IN DETAIL
Walk through this like flipping a light switch when you plug a brand-new laptop into campus Wi-Fi and open `http://anotherdomain.com`:
1. **OS Foundations (Part a)**: You need NIC hardware drivers + a TCP/IP stack + sockets (`connect`/`send`), otherwise apps can't even talk to the wire.
2. **Who am I? Where are they? (Part b)**:
   - **DHCP** gives your laptop an identity card (`IP, Mask, Router IP, DNS Server IP`).
   - **DNS** acts like a phonebook translating human name `"anotherdomain.com"` into `"179.20.10.6"`.
3. **Packing the Russian Nesting Dolls (Part c)**:
   - **Application Layer**: Writes the letter (`HTTP GET /`).
   - **Transport Layer (TCP)**: Calls `179.20.10.6:80` to establish a reliable 3-way handshake, stamps ports/sequence numbers/checksum onto the TCP segment.
   - **Link Layer**: Notices `"179.20.10.6"` isn't in the same room, uses **ARP** to find the local **Router's MAC address** (note: *NOT* the server's MAC!), wraps everything in an Ethernet frame with `Dst MAC = Router MAC` + CRC, and shoots it out the cable!

---

### 10. IPv4 vs. IPv6: Why IPv6 Removed Router Fragmentation, Header Checksum & NAT [Frequency: 1]

> **Question**: The LangOS team has been thinking of using IPv6 and would like to know some details from you. What part of the IPv6 header can be used to track the path of a package in a network? How is it able to track it?

#### pre-requisite
- **IPv6 Fixed Header Fields**: `Version` (4 bits), `Traffic Class` (8), `Flow Label` (20), `Payload Length` (16), `Next Header` (8), **`Hop Limit` (8)**, `Source Address` (128), `Destination Address` (128), plus optional **Extension Headers** (e.g., Routing / Hop-by-Hop headers).

#### Answer (Exam Writing Format)
1. **Primary Mechanism — `Hop Limit` Field (via `traceroute` / ICMPv6)**:
   - The **8-bit `Hop Limit`** field in the IPv6 header (equivalent to IPv4's TTL) specifies the maximum number of router hops a packet may traverse.
   - **How It Tracks the Path**: Each router decrements `Hop Limit` by 1. If `Hop Limit` reaches `0`, the router drops the packet and sends an **ICMPv6 Time Exceeded** message (carrying that router's IP address) back to the sender. By sending probes with `Hop Limit = 1, 2, 3, ...`, the sender discovers and traces every router hop along the path!
2. **Related IPv6 Path Fields**:
   - **Routing Extension Header / Segment Routing (SRv6)** explicitly embeds or records an ordered list of intermediate router hops inside the packet header, and the **20-bit `Flow Label`** enables routers to consistently route/track packets belonging to the same path/flow.

#### IN DETAIL
Suppose you want to see every single router between your laptop and `google.com` over IPv6 (`traceroute6`):
- You send Probe #1 with **`Hop Limit = 1`**. The *first* router subtracts 1 (`1 - 1 = 0`), drops it, and replies *"ICMPv6 Time Exceeded from Router 1!"*—boom, you know Hop 1!
- Next you send Probe #2 with **`Hop Limit = 2`**. Router 1 passes it (`2 - 1 = 1`), Router 2 drops it (`1 - 1 = 0`) and replies *"From Router 2!"*—boom, you know Hop 2!
- Repeating until you reach the destination maps the entire hop-by-hop path!

---

### 11. TCP Connection Multiplexing & Demultiplexing Using the 4-Tuple & Port Numbers [Frequency: 1]

> **Question**: The *Net team* has developed a simple web application to test their TCP implementation. The web application is a simple process that will be sending a stream of data `11100110011001101101010101010101` to another process in a different machine (running IndixOS).
> **a.** The Net Team would first like to understand what kind of APIs the OS needs to provide to make this communication happen (based on Unix implementation) and what is the role of the OS when such communication happens. (3 points)

#### pre-requisite
- **UNIX Socket Abstraction**: Sockets expose network endpoints as standard file descriptors (`int fd`) so applications use uniform I/O calls across networks.

#### Answer (Exam Writing Format)
1. **OS Socket APIs (BSD/POSIX Sockets)**:
   - **Server side**: `socket()` (create TCP endpoint) $\to$ `bind()` (attach IP & port) $\to$ `listen()` (mark passive) $\to$ `accept()` (block/accept client connection) $\to$ `recv()`/`read()` $\to$ `close()`.
   - **Client side**: `socket()` $\to$ `connect()` (initiate TCP 3-way handshake) $\to$ `send()`/`write()` $\to$ `close()`.
2. **Role of the OS Kernel**:
   - Manages **socket buffers** (send/receive kernel queues) and copies user data across the user–kernel boundary.
   - Executes the **TCP/IP protocol stack**: performs 3-way handshake, segments the byte stream, attaches TCP/IP/Ethernet headers, computes checksums, handles timers/ACKs/retransmissions/flow control, performs routing lookup & ARP, and hands frames to the **NIC device driver** (DMA) for transmission.

#### IN DETAIL
Your web app doesn't want to build Ethernet headers, compute IP checksums, or worry about packet retries! The OS provides the **Socket API** (which behaves like opening a file over the network): the app simply calls `connect()` and `send(fd, data)`, and the **OS kernel** handles all the dirty work inside kernel space—buffering bytes, running TCP's 3-way handshake, slicing data into packets, wrapping TCP+IP+MAC headers, and driving the physical Wi-Fi/Ethernet card!

---

### 12. LangOS Socket API: Kernel Path of a send() Call Down to NIC Frame Transmission [Frequency: 1]

> **Question**: In LangOS, a user process issues a `send()` system call to transmit data over a TCP socket. Considering the following possibilities, which among these do you think captures what may happen in such a scenario:
> - a. A context switch always happens immediately because every system call requires the process to be de-scheduled.
> - b. A context switch may happen if the kernel's TCP send buffer is full causing the process to block until space is available.
> - c. A context switch never happens because TCP send is handled entirely in user space.
> - d. A context switch occurs only if the receiver is on the same machine as inter-process communication requires switching between processes.

#### Answer (Exam Writing Format)
- **Correct Option**: **b. A context switch may happen if the kernel's TCP send buffer is full causing the process to block until space is available.**
- **Justification**: A `send()` syscall triggers a **mode switch** (User $\to$ Kernel), *not* necessarily a process context switch. If space is available in the kernel's TCP send buffer, the kernel copies the data and returns immediately to the *same* process. However, on a blocking socket, if the send buffer is full (e.g., receiver `rwnd` = 0 or congestion window full), the OS moves the process to the **Blocked** state and performs a **context switch** to run another ready process.

#### IN DETAIL
Don't confuse a **Mode Switch** with a **Context Switch**!
- **Mode Switch**: Calling `send()` flips the CPU Mode Bit from User (`1`) to Kernel (`0`) so the **same process** runs kernel socket code, copies bytes into the kernel's TCP send buffer, and flips back to User (`1`)—**zero context switch** if buffer space exists! (Eliminates **a** and **c**.)
- **Context Switch**: Only if the TCP send buffer is **completely full** (and the socket isn't non-blocking) does the kernel put the process to sleep (`Blocked`) and switch the CPU to a completely different process (**b**)!

---

### 13. Debugging TCP Connection State & SYN-Flood / Half-Open Sockets [Frequency: 1]

> **Question**: The LangOS networking group is debugging a connection and prints out part of the TCP header. One of the junior engineers claims: *"The TCP header must always include an ACK number, otherwise the receiver won't know which byte to expect next."*
> They wanted to know the correct explanation from you:
> - a. The ACK field is always present in the TCP header, but the ACK bit in the flags determines if the value is valid.
> - b. The ACK field is included only when the ACK bit is set; otherwise, it is not part of the header.
> - c. The ACK number exists only in the three-way handshake and is dropped after connection setup.
> - d. The ACK field is optional and used only if the application enables reliability.

#### Answer (Exam Writing Format)
- **Correct Option**: **a. The ACK field is always present in the TCP header, but the ACK bit in the flags determines if the value is valid.**
- **Justification**: The standard TCP header has a fixed minimum size of **20 bytes**, and the 32-bit **Acknowledgment Number** field occupies bytes 8–11 in *every* TCP segment (including the initial `SYN`). However, its value is only meaningful when the **ACK control flag bit = 1** (e.g., it is ignored in the very first `SYN` packet where `ACK = 0`).

#### IN DETAIL
Look at the fixed 20-byte TCP header layout:
- Bytes 0–3: Source Port + Destination Port
- Bytes 4–7: Sequence Number
- **Bytes 8–11: Acknowledgment Number (always physically present!)**
- Bytes 12–13: Header Length + Control Flags (`URG, ACK, PSH, RST, SYN, FIN`)
- Why keep the 4-byte ACK slot even in the initial `SYN` packet when there's nothing to acknowledge yet? Keeping a fixed-offset header makes hardware/kernel parsing super fast—you just ignore whatever is in bytes 8–11 whenever the **`ACK` flag bit is `0`**!

---

### 14. End-to-End Network Path & Connectivity Verification Tools (ping / traceroute / nslookup) [Frequency: 1]

> **Question**: The LangOS network team wants to check the end-to-end network communication capabilities. The team is investigating a scenario of a process on Machine A with IP `192.168.1.10` and MAC `AA:BB:CC:DD:EE:01` running on Port number `5000`, sends data to a process on Machine B with an IP address of `10.0.0.20`, MAC `AA:BB:CC:DD:EE:02` on Port number `6000`. The packet travels through a router R with two interfaces:
> - LAN side: IP = `192.168.1.1`, MAC = `AA:BB:CC:DD:EE:11`
> - WAN side: IP = `10.0.0.1`, MAC = `AA:BB:CC:DD:EE:22`
>
> Given this scenario, which of the following statements is correct with respect to the OSI layer?
> - a. MAC source/destination addresses are rewritten at every hop
> - b. The IP addresses of the source and destination change at each router, but others remain the same.
> - c. The port numbers are reassigned by the router (to avoid conflicts), while the IP and MAC addresses remain the same.
> - d. Both IP and MAC addresses change at every hop, but the port numbers remain unchanged.

#### Answer (Exam Writing Format)
- **Correct Option**: **a. MAC source/destination addresses are rewritten at every hop.**
- **Justification**: Layer 2 (MAC) addresses have **link-local (single-hop) scope**:
  - Hop 1 (A $\to$ R): `Src MAC = EE:01`, `Dst MAC = EE:11` (router LAN interface).
  - Hop 2 (R $\to$ B): `Src MAC = EE:22` (router WAN interface), `Dst MAC = EE:02`.
  Meanwhile, without NAT, **Layer 3 IP addresses** (`192.168.1.10` $\to$ `10.0.0.20`) and **Layer 4 TCP/UDP port numbers** (`5000` $\to$ `6000`) remain unchanged end-to-end across standard IP routing.

#### IN DETAIL
Think of sending an international courier parcel:
- The **end-to-end IP addresses and Port numbers** are the permanent sender/recipient address label printed on the inner box (`From: Machine A Port 5000` $\to$ `To: Machine B Port 6000`)—a standard router doesn't touch them!
- The **MAC addresses** are the local delivery truck tags swapped at every sorting hub (Hop 1: `A's truck` $\to$ `Router LAN dock`; Hop 2: `Router WAN truck` $\to$ `B's dock`). Hence MAC addresses change at **every single hop** (**a**)!

---

### 15. Transport Layer Data Integrity Validation: 16-Bit One's-Complement Checksum Calculation [Frequency: 1]

> **Question**: How can the process in the destination machine validate the incoming data? (4 points)

#### pre-requisite
- **TCP/Internet Checksum**: Computed by splitting data (plus TCP header & pseudo-header) into 16-bit words, summing them with **1's complement arithmetic** (wrapping carry-out back into LSB), and taking the 1's complement (`~sum`). At the receiver, summing all 16-bit words including the checksum must yield all `1`s (`0xFFFF`).

#### Answer (Exam Writing Format)
The destination validates the incoming stream `11100110011001101101010101010101` using layered integrity & ordering checks:
1. **16-bit Internet Checksum (TCP Header)**:
   - Break the 32-bit payload into two 16-bit words:
     - Word 1: `11100110 01100110`
     - Word 2: `11010101 01010101`
   - 1's complement addition (with wrap-around carry):
     `1110011001100110 + 1101010101010101 = (1) 1011101110111011` $\xrightarrow{\text{add carry } 1}$ `1011101110111100`.
   - Sender's 1's complement checksum (payload portion) = `0100010001000011` (combined with TCP header + IP pseudo-header).
   - **Validation**: Destination adds all 16-bit words (pseudo-header + TCP header + data + checksum); if the 1's complement result is **`0xFFFF` (all 1s)** (complement `0x0000`), the segment is valid; otherwise it is silently discarded as corrupted.
2. **Sequence Number & Length Validation**: TCP checks the **Sequence Number** and **Payload Length** to detect missing, duplicate, or out-of-order segments before delivering an intact ordered byte stream to the receiving application.
3. **Link-Layer CRC (FCS)**: Underneath IP/TCP, Ethernet hardware also checks a 32-bit **CRC** frame trailer to reject physical bit errors.

#### IN DETAIL
Let's verify that binary checksum calculation step-by-step so it's crystal clear in an exam:
- Given 32 bits: `11100110 01100110` + `11010101 01010101`
  - Add bit-by-bit from right to left:
    `1110011001100110` (`0xE666`) + `1101010101010101` (`0xD555`) = `1 1011101110111011` (`0x1BBBB`)
  - Notice the overflow carry bit `1` on the far left! In **1's complement arithmetic**, you wrap that carry bit back around and add `+1` to the bottom: `1011101110111011 + 1 = 1011101110111100` (`0xBBBC`).
  - Flip every bit (`~0xBBBC`) $\to$ `0100010001000011` (`0x4443`)!
- When the receiver gets both data words AND the checksum word `0x4443`, it adds all three: `0xBBBC + 0x4443 = 0xFFFF` (all `1`s)! If even a single bit flipped on the wire, the sum won't be all `1`s, so TCP catches the corruption immediately!

---

### 16. How TCP Provides Reliability Guarantees & Flow Control (Sequence Numbers, ACKs, rwnd) [Frequency: 1]

> **Question**: Further, how can the Net team provide guarantees on data reliability and flow control? (3 points)

#### Answer (Exam Writing Format)
1. **Data Reliability Guarantees (Error & Loss Recovery)**:
   - **Sequence Numbers & Cumulative ACKs**: Every byte is numbered; receiver sends `ACK` for the next expected in-order byte.
   - **Retransmission Timer (Timeout + Fast Retransmit)**: Sender starts an adaptive timer (`EstimatedRTT + 4 * DevRTT`) and retransmits unacknowledged segments on timeout (or 3 duplicate ACKs).
   - **Checksums**: Detect and drop corrupted segments so timers/ACKs trigger clean retransmission.
2. **Flow Control Guarantees (Preventing Receiver Buffer Overflow)**:
   - **Sliding Window with Advertised Receiver Window (`rwnd`)**: Every TCP ACK includes a `Window Size` (`rwnd`) field telling the sender exactly how many free bytes remain in the receiver's buffer.
   - Sender restricts unacknowledged bytes in flight (`LastByteSent - LastByteAcked <= rwnd`), pausing if `rwnd == 0` until buffer space frees up.

#### IN DETAIL
Notice the clear division of responsibility:
- **Reliability** asks: *"What if the network drops, duplicates, or scrambles a packet?"* Answer: Number every byte (**Sequence Numbers**), demand receipts (**ACKs**), inspect for damage (**Checksums**), and resend if the receipt doesn't arrive before the **Timeout**!
- **Flow Control** asks: *"What if a super-fast 10 Gbps server blasts data at a slow smartphone whose RAM buffer is full?"* Answer: The smartphone stamps a **`rwnd` (Receive Window)** number on every ACK saying *"I only have 4 KB of empty buffer left—don't send more than 4 KB until I drain it!"*

---

### 17. Why Transport Protocols Use Much Larger Sliding Windows Than Data Link Layer (BDP) [Frequency: 2]

> **Question**: Explain the reason why transport protocols use larger size sliding windows as compared to the size of sliding windows of data link layer?

#### pre-requisite
- **Reference Slides**: `OverviewOfTransportLayer.pptx` (Slide 28).
- **Bandwidth-Delay Product ($\text{BDP}$)**: Number of bits/bytes "in flight" needed to keep a communication pipe full: $\text{BDP} = \text{Bandwidth} \times \text{RTT}$.

#### Answer (Exam Writing Format)
1. **End-to-End Path vs. Single Physical Link**:
   - A **Data Link Layer** sliding window operates over a **single physical hop** (adjacent nodes), where propagation delay / RTT is tiny (microseconds to milliseconds), so only a small window is needed to keep the wire full.
   - A **Transport Layer** sliding window (e.g., TCP) spans an **entire end-to-end network path** across many routers and continents, making the Round-Trip Time (RTT) orders of magnitude larger.
2. **Keeping the Pipe Full (Large Bandwidth-Delay Product)**:
   - To achieve high throughput without stalling while waiting for end-to-end ACKs, the sender must keep $\text{BDP} = \text{Bandwidth} \times \text{RTT}$ bytes unacknowledged in flight simultaneously. Since transport RTT is vastly larger than single-link RTT, the transport layer requires a **much larger sliding window** (and larger host buffers).

#### IN DETAIL
Think of pouring water into a garden hose vs. a 10-kilometer cross-city pipeline:
- **Data Link Layer** is a 2-meter garden hose between two directly wired neighbors. Water reaches the other end almost instantly, so you only need a small cup of water in transit at once (small sliding window).
- **Transport Layer (TCP)** is a 10,000-km Internet path passing through 15 routers from Hyderabad to New York ($\text{RTT} \approx 200\text{ ms}$). On a 1 Gbps link, during the 200 ms you wait for the first ACK to return, you can pump $10^9 \times 0.2 / 8 = \mathbf{25\text{ MB}}$ of data into the pipe! If your sliding window were tiny (like a data-link window of 8 packets), the sender would finish 8 packets in microseconds and sit idle 99.9% of the time waiting on ACKs!

---

### 18. TCP Three-Way Handshake Diagnostics: Random Initial Sequence Numbers & Nq = Np + 1 [Frequency: 1]

> **Question**: As part of diagnosing intermittent network failures in AgentOS, the networking team asks you to verify whether the system is correctly following the TCP three-way handshake. A TCP client module $P$ initiates a connection to a remote TCP service $Q$. Let $N_p$ denote the sequence number in the `SYN` sent from $P$ to $Q$, and let $N_q$ denote the acknowledgement number in the `SYN-ACK` sent from $Q$ back to $P$. Based on standard TCP behaviour, which of the following statements is/are CORRECT and why?
> - a. The sequence number $N_p$ is randomly chosen
> - b. The sequence number is set to 0
> - c. The acknowledgement number is $N_q = N_p$
> - d. The acknowledgement number is $N_q = N_p + 1$ *(right edge of scan crops `1` after `+`)*

#### Answer (Exam Writing Format)
- **Correct Options**: **a** and **d**.
  - **a. ($N_p$ is randomly/pseudo-randomly chosen)**: TCP selects a random/clock-based Initial Sequence Number (ISN) rather than `0` to prevent confusion with delayed duplicate packets from old connections and to defend against TCP sequence-prediction/spoofing attacks.
  - **d. ($N_q = N_p + 1$)**: Because the `SYN` flag itself consumes **1 byte of sequence space**, $Q$ acknowledges receipt of $P$'s `SYN` by setting the acknowledgment number to the *next expected byte*: $N_q = N_p + 1$.

#### IN DETAIL
Notice that the exam instructions explicitly state *"you can select one or more options (if necessary)"*!
1. Why **not `b` (set to 0)**? If every TCP connection started at `seq = 0`, an attacker could easily forge packets—and worse, a delayed packet #5 from an old crashed session could crash straight into a brand-new session! So TCP picks a pseudo-random 32-bit Initial Sequence Number ($N_p$). *(Note: Wireshark only shows `Seq = 0` as a "relative sequence number" helper for readability—on the real wire, it's random!)*
2. Why **`d` ($N_q = N_p + 1$)**? Even though a `SYN` packet carries 0 application payload bytes, TCP treats the `SYN` control bit as occupying **1 logical sequence number** so that `SYN` itself can be reliably acknowledged (`N_p + 1`).

---

### 19. TCP Connection Establishment (Three-Way Handshake) & Connection Release (Two-Army Problem / TIME_WAIT) [Frequency: 3]

> **Question**: In TCP protocol, explain the issues with the connection establishment and connection release? Briefly explain the solutions.
> *(2020 Quiz 9 Q3 & 2022 Endsem Q10 focused on Connection Establishment and Three-Way Handshake justification.)*

#### pre-requisite
- **Reference Slides**: `OverviewOfTransportLayer.pptx` (Slides 18–23, Tomlison's Three-Way Handshake).
- **Unreliable Network Reality**: Packets can be **delayed, duplicated, and reordered** arbitrarily inside the network; old packets from previous closed connections may suddenly reappear ("delayed duplicates").
- **Connection Release Reality**: Stopping both directions cleanly faces the distributed **Two-Army Problem** impossibility.

#### Answer (Exam Writing Format)
- **Issues in Connection Establishment**:
  1. **Delayed Duplicate Packets**: In a simple two-way handshake (`CR` $\to$ `ACK`), an old delayed duplicate Connection Request (`CR`) (and old duplicate `DATA` packet) arriving at a server long after an earlier session closed would look like a brand-new valid connection, causing the server to accept phantom connections or corrupt data!
  2. **Clock/Sequence Number Wrap-around & Synchronization**: Both hosts must agree on initial sequence numbers without reusing recently active sequence numbers within $2 \times \text{MSL}$ (Maximum Segment Lifetime), and must survive host crashes without remembering every past connection.
- **Justification for Three-Way Handshake**:
  1. **Host 1 $\to$ Host 2 (`SYN`, $seq = x$)**: Host 1 proposes its random/clock-based initial sequence number $x$.
  2. **Host 2 $\to$ Host 1 (`SYN-ACK`, $seq = y, ack = x + 1$)**: Host 2 picks its *own* fresh sequence number $y$ and asks Host 1 to confirm both $x$ and $y$.
  3. **Host 1 $\to$ Host 2 (`ACK`, $ack = y + 1$)**: Host 1 confirms $y$ (connection opens!).
  - **Why this stops delayed duplicates**:
    - If an **old duplicate `CR` ($seq = x_{old}$)** arrives out of nowhere, Host 2 sends back a `SYN-ACK` asking Host 1 to verify $y$. Since Host 1 never initiated a new connection, Host 1 **rejects (`RST`)** it instead of sending the 3rd `ACK`, so no bogus connection opens!
    - If both an **old duplicate `CR` AND old duplicate `DATA`** arrive, Host 1's rejection (and mismatch with Host 2's fresh $y$) ensures neither the bogus connection nor the stale data is accepted.

**Part B — Connection Release Issues & Solution (2025 AOS Q5)**:
- **Issues**: (1) **Two-Army Problem** (neither host can be 100% certain its final `ACK` arrived without risking infinite `ACK-of-ACK` chains); (2) **Asymmetric Data Loss** if closing aborts opposite-direction in-flight segments.
- **Solution**: **Independent Half-Close (`FIN` + `ACK` in each direction) + Retransmission Timers + `TIME_WAIT` ($2\text{MSL}$)** so delayed segments drain and a lost final `ACK` can be re-sent safely.

#### IN DETAIL
Imagine a malicious or slow postal system where letters can get stuck in a mailbag for **2 minutes** and show up late!
- **Why 2 steps (`Connect` $\to$ `Accepted`) fails catastrophically**: Suppose Alice ordered a pizza (`Connect` + `Charge $50`) an hour ago, got her pizza, and hung up. A delayed duplicate of that envelope shows up at the pizza shop now! In a 2-way handshake, the shop thinks it's a brand-new order, bakes a phantom pizza, and charges Alice $50 again!
- **How the 3-Way Handshake fixes it**: Now, when an envelope `"Connect (ID = x)"` arrives, the shop does NOT bake the pizza yet! Instead, the shop replies: *"Did you really just ask for ID=x? Prove you're alive right now by echoing back my secret fresh number y!"*
  - If it was a **ghost delayed packet**, Alice says *"Huh? I didn't order anything!"* (sends `RST` or ignores it)—so the shop safely drops the ghost request!
  - Only a **live, real Alice** replies *"Yes, confirmed y!"* (Step 3), proving both sides are alive *right now* and synchronized on both directions' sequence numbers!

**And Closing Safely (`Two-Army Problem`)**:
(Two-Army Problem)**: Imagine two Blue armies trying to agree via messengers through enemy territory when to both hang up! Army 1 sends *"Let's hang up!"* Army 2 replies *"Agreed!"*—wait, how does Army 2 know Army 1 heard *"Agreed"*?! If Army 1 sends *"I heard your agreement!"*, how does Army 1 know *that* arrived? **Mathematically unsolvable with 100% perfection!**
  So TCP compromises brilliantly: **close each direction separately (`FIN` / `ACK`)**, retry with a **timeout** if a `FIN` is lost (closing forcefully after $N$ retries), and wait **`2 * MSL` (`TIME_WAIT`)** so any stray packets die off cleanly!

---

### 20. AgentOS TCP Retransmission Timeout Numerical (EstimatedRTT, DevRTT, Timeout = 72.00 ms) [Frequency: 1]

> **Question**: The AgentOS team was testing the networking stack when they observed significant network jitter and decided to inspect the TCP timeout behavior. At time $t$, the system's internal state is `EstimatedRTT = 50 ms` and `DevRTT = 4 ms`. For the most recent packet, the RTT was `60 ms`. Assuming the standard TCP parameters of $\alpha = 0.125$ and $\beta = 0.25$, what should the TCP Timeout Interval be updated to for the *next* transmission and why?
> - a. `67.25 ms`
> - b. `72.00 ms`
> - c. `51.25 ms`
> - d. `70.75 ms`

#### pre-requisite
- **RFC 6298 / Jacobson–Karels TCP RTO Equations**:
  1. $\text{EstimatedRTT} = (1 - \alpha) \cdot \text{EstimatedRTT} + \alpha \cdot \text{SampleRTT}$
  2. $\text{DevRTT} = (1 - \beta) \cdot \text{DevRTT} + \beta \cdot |\text{SampleRTT} - \text{EstimatedRTT}|$
  3. $\text{TimeoutInterval} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT}$

#### Answer (Exam Writing Format)
- **Correct Option**: **b. `72.00 ms`**.
- **Calculation**:
  1. **Updated `EstimatedRTT`**:
     $$\text{EstimatedRTT} = (1 - 0.125)(50) + 0.125(60) = 43.75 + 7.50 = \mathbf{51.25\text{ ms}}$$
  2. **Updated `DevRTT`**:
     $$\text{DevRTT} = (1 - 0.25)(4) + 0.25 \cdot |60 - 51.25| = 3.00 + 0.25(8.75) = 3.00 + 2.1875 = \mathbf{5.1875\text{ ms}}$$
  3. **Updated `TimeoutInterval`**:
     $$\text{TimeoutInterval} = 51.25 + 4 \times 5.1875 = 51.25 + 20.75 = \mathbf{72.00\text{ ms}}$$

#### IN DETAIL
Why does TCP add **`4 * DevRTT`** onto `EstimatedRTT` instead of just using `EstimatedRTT` (`51.25 ms`, which is distractor `c`)?
Because network latency has **jitter**! Even if average RTT is `51.25 ms`, half of healthy packets naturally take a bit longer than the average. Adding a **4-standard-deviation safety margin (`4 * DevRTT`)** prevents TCP from triggering false timeouts when a normal packet merely experiences slight jitter! *(Also note: always compute `DevRTT` using the **newly updated** `EstimatedRTT = 51.25`, giving exactly `72.00 ms` = option **b**.)*

---

### 21. Regulating Sending Rates for Congestion Control & Why AIMD Converges to Fairness [Frequency: 1]

> **Question**: Explain the issue of regulating the sending rates to obtain desirable bandwidth controlling the congestion? Additive Increase Multiplicative Decrease (AIMD)?

#### pre-requisite
- **Reference Slides**: `OverviewOfTransportLayer.pptx` (Slides 33–39, Chiu–Jain Fairness/Efficiency model).
- **Desired Goals**: **Efficiency** (total load near full link capacity without congestion collapse) and **Fairness** (competing flows sharing a bottleneck converge to equal bandwidth—max-min fairness).

#### Answer (Exam Writing Format)
- **The Rate-Regulation Issue**:
  - Senders do not know the exact spare capacity of the network or how many other flows are sharing a bottleneck link, yet they must dynamically adjust sending rates using only binary congestion feedback ("network free" vs. "network congested") so the system achieves **both high efficiency and fairness** without oscillating wildly or starving flows.
- **Why Simple Symmetric Adjustments Fail**:
  - **Additive Increase + Additive Decrease (AIAD)** or **Multiplicative Increase + Multiplicative Decrease (MIMD)** keep flows moving parallel to or along lines from the origin in the rate space—they **never converge to the fairness line ($x_1 = x_2$)** if two flows start unequal!
- **Why AIMD (Additive Increase, Multiplicative Decrease) Succeeds**:
  1. **Additive Increase (No Congestion)**: Slowly probe for spare bandwidth by increasing the sending rate/congestion window linearly ($W \leftarrow W + 1$ per RTT).
  2. **Multiplicative Decrease (Congestion Detected)**: On packet loss/congestion signal, cut the sending rate aggressively by a factor (e.g., $W \leftarrow W / 2$) to rapidly clear congested router queues.
  3. **Convergence to Fairness & Efficiency**: Because Multiplicative Decrease shrinks larger flows by a *larger absolute amount* than smaller flows while Additive Increase adds the *same amount* to both, repeated cycles geometrically drive unequal flows along a sawtooth path **straight onto the fair and efficient operating point ($x_1 = x_2$, $x_1 + x_2 = C$)**—which is why TCP uses AIMD.

#### IN DETAIL
Imagine two drivers ($F_1$ and $F_2$) sharing a highway whose capacity is $C = 100\text{ Mbps}$, starting unfairly: $F_1$ is taking $80\text{ Mbps}$ and $F_2$ is taking only $10\text{ Mbps}$. Neither knows the other's speed—they only get a traffic light saying **Green (keep going faster)** or **Red (too crowded, slow down!)**:
- **If both add/subtract equally (AIAD)**: On Green both add $+10$, on Red both subtract $-10$. Their *difference* ($80 - 10 = 70$) stays $70$ forever—$F_2$ stays permanently cheated!
- **If both multiply/divide equally (MIMD)**: Both keep the exact same *ratio* ($8:1$) forever—still unfair!
- **Now watch AIMD**:
  - Green: Both add $+5$: $(85, 15) \to (90, 20)$ [Red! Total $110 > 100$].
  - Red: Both **cut in half** ($\div 2$): $(45, 10)$! Look! $F_1$ dropped by $45$, $F_2$ dropped by only $5$—their gap shrank from $70$ to **$35$**!
  - Repeat a few cycles: $(45,10) \to (67.5, 32.5) \to (33.75, 16.25) \to \dots \to (50, 50)$!

That simple rule—**probe gently up (+1), back off sharply down ($\div 2$)**—automatically forces *every* competing TCP connection on the Internet to converge to an **equal, efficient share** without a central coordinator!

---

### 22. TCP Congestion Control Phases: Slow Start, Congestion Avoidance (AIMD), Fast Retransmit & Fast Recovery (Tahoe vs. Reno) [Syllabus Sec 2]

> **Question**: Explain the phases of TCP Congestion Control (**Slow Start**, **Congestion Avoidance**, **Fast Retransmit**, and **Fast Recovery**), clearly distinguishing between **RTO Timeout** vs. **3 Duplicate ACKs** reactions in **TCP Tahoe** vs. **TCP Reno**.

#### pre-requisite
- **Reference Slides**: `OverviewOfTransportLayer.pptx` (Slides 71–77).
- **Variables**: `cwnd` (Congestion Window), `ssthresh` (Slow Start Threshold); sending rate is bounded by $\min(\text{cwnd}, \text{rwnd})$.

#### Answer (Exam Writing Format)
1. **Slow Start (`cwnd < ssthresh`)**:
   - Starts at $1\text{ MSS}$ and increases **`cwnd` by $+1\text{ MSS}$ for every ACK received**, **doubling `cwnd` exponentially every RTT** ($1, 2, 4, 8, \dots$) until hitting `ssthresh` or packet loss.
2. **Congestion Avoidance / AIMD (`cwnd >= ssthresh`)**:
   - Increases **`cwnd` linearly by $\approx 1\text{ MSS}$ per RTT** ($\text{cwnd} \leftarrow \text{cwnd} + \text{MSS}^2 / \text{cwnd}$ per ACK) to probe carefully for remaining bandwidth.
3. **Loss Detection & TCP Tahoe vs. TCP Reno**:
   - **Retransmission Timeout (RTO — severe congestion)**: *Both Tahoe & Reno* set `ssthresh = cwnd / 2`, reset **`cwnd = 1 MSS`**, and re-enter **Slow Start**.
   - **3 Duplicate ACKs (Fast Retransmit — mild loss, packets still flowing)**:
     - *TCP Tahoe*: Sets `ssthresh = cwnd / 2` and drops **`cwnd = 1 MSS`** (same as timeout!).
     - *TCP Reno (Fast Recovery)*: Immediately retransmits the missing segment without waiting for timeout, sets `ssthresh = cwnd / 2`, sets **`cwnd = ssthresh` (plus `3 MSS`)**, and resumes **Congestion Avoidance**—producing the classic AIMD sawtooth without collapsing to `1 MSS`.

#### IN DETAIL
Look at why **TCP Reno** beats **TCP Tahoe** on modern networks:
- When an **RTO timeout** explodes, zero ACKs are coming back—the network pipe is severely choked, so slamming the brakes all the way to **`cwnd = 1`** is necessary.
- When **3 Duplicate ACKs** arrive, notice what that actually means: three *later* packets just made it safely to the receiver! The network isn't dead—only 1 packet dropped! Cutting `cwnd` in **half** (**Fast Recovery**) keeps the pipe half-full instead of needlessly dropping back to `1 MSS`!

---

### 23. Importance of Quality of Service (QoS) & Admission Control [Frequency: 1]

> **Question**: Briefly explain the importance of "quality of service" and "admission control".

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slides 92–93, 98–103).

#### Answer (Exam Writing Format)
1. **Quality of Service (QoS)**:
   - **Importance**: Traditional best-effort networks only optimize aggregate throughput, but diverse applications (e.g., VoIP, video conferencing, streaming) have strict requirements on **bandwidth, delay, jitter (delay variation), and packet loss**. QoS mechanisms allow networks/systems to provide **predictable performance guarantees** to critical or real-time flows instead of degrading all traffic equally during congestion.
2. **Admission Control**:
   - **Importance**: QoS guarantees cannot be met if infinite flows are allowed onto finite resources. Admission control checks whether sufficient bandwidth/router/CPU capacity exists **before** admitting a new flow or virtual circuit; if accepting it would cause congestion and break existing guarantees, the new request is rejected (e.g., busy signal in telephony), protecting active sessions.

#### IN DETAIL
Think of **QoS** as promising *"Your ambulance will reach the hospital in under 5 minutes"* or *"Your Zoom video won't stutter."* Simply adding extra capacity everywhere ("overprovisioning") is too expensive, so routers/OSes reserve queues and schedule packets carefully to meet guarantees.

**Admission Control** is the bouncer at the club door: if the fire capacity is 100 people and 100 people are already inside enjoying the show, letting another 50 people cram in ruins the experience for *everyone*! By saying *"Sorry, full right now"* to the 101st request, the system guarantees that all 100 admitted flows actually receive their promised QoS.

---

### 24. Meeting Multimedia QoS Across Process Scheduler, Disk Scheduler & Memory Manager [Frequency: 1]

> **Question**: Define the word "quality of service". Discuss what techniques could be used to meet quality-of-service requirements for multimedia applications for the following components of a system: (a) Process scheduler (b) Disk scheduler (c) Memory manager.

#### Answer (Exam Writing Format)
- **Definition of Quality of Service (QoS)**: Quality of Service is the ability of a system or network to provide **guaranteed, predictable performance bounds** (such as minimum throughput/frame rate, bounded delay, low jitter, and bounded loss) required by time-sensitive applications like real-time audio/video.
- **Techniques to Meet Multimedia QoS**:
  - **(a) Process Scheduler**:
    - Use **real-time scheduling** algorithms such as **Earliest Deadline First (EDF)** or **Rate Monotonic Scheduling (RMS)** with strict priority/preemption so periodic media frames finish before deadlines.
    - Employ **admission control** and CPU resource reservation (proportion/bandwidth guarantees) to prevent best-effort background tasks from causing jitter.
  - **(b) Disk Scheduler**:
    - Use real-time/deadline-aware disk scheduling like **EDF** or **SCAN-EDF** (sorting requests with similar deadlines using elevator SCAN order) to deliver continuous media blocks before playback buffers underrun.
    - Place contiguous blocks and use **read-ahead / double buffering** to absorb seek delays.
  - **(c) Memory Manager**:
    - **Page pinning / locking** (`mlock`) of active multimedia buffers and real-time playback threads in physical RAM to prevent page faults and unpredictable swap delays.
    - Reserve dedicated ring buffers / frame pools to smooth out disk/network jitter.

#### IN DETAIL
Watching a movie or having a video call fails completely if frames arrive 2 seconds late or freeze for a page fault—even if the "average" speed over an hour looks fine! To prevent glitches, *every* OS subsystem must cooperate:
1. **CPU (Process Scheduler)** can't let a background compiler run for 500 ms and make audio stutter; it uses deadline-based real-time scheduling (**EDF / RMS**) and CPU reservations.
2. **Disk (Disk Scheduler)** can't use pure FCFS or pure SCAN that starves an urgent video frame at the edge of the platter; **SCAN-EDF** hits urgent deadlines first while ordering nearby tracks efficiently.
3. **RAM (Memory Manager)** can't swap out your active audio/video buffer during playback! Pinning real-time memory pages eliminates sudden multi-millisecond page-fault pauses.

---

### 25. Why FIFO Packet Scheduling Fails Diverse Applications (Head-of-Line Blocking) [Frequency: 1]

> **Question**: Explain why FIFO packet scheduling algorithm fails to provide good service for diverse applications?

#### pre-requisite
- **FIFO Queuing (Tail-Drop)**: Router puts all arriving packets into a single queue in arrival order and serves them strictly first-come, first-served; packets arriving when the buffer is full are dropped.

#### Answer (Exam Writing Format)
Diverse applications have conflicting QoS needs (e.g., interactive/VoIP needs low delay & jitter; bulk file transfer pushes maximum bursts). FIFO fails for them because:
1. **No Flow Isolation (Head-of-Line Blocking)**: A single aggressive or bursty bulk flow (e.g., large file transfer or malicious UDP flood) fills the shared queue, forcing small, delay-sensitive interactive/audio packets behind hundreds of big packets, causing **high queuing delay and severe jitter**.
2. **Unfair Bandwidth Allocation**: High-rate senders grab a disproportionate share of buffer slots and bandwidth, starving compliant low-rate flows.
3. **Indiscriminate Tail Drops**: When the buffer overflows, urgent real-time or control packets are dropped just as readily as low-priority background traffic.
- **Fix**: Per-flow queuing schemes like **Fair Queuing / Weighted Fair Queuing (WFQ)** isolate flows and allocate bandwidth/priority fairly.

#### IN DETAIL
If an airport puts **ambulance medevac passengers**, **business travelers**, and **10,000 crates of bricks** into the *exact same single-file security line* (FIFO):
- Whenever a truck dumps 1,000 brick crates onto the belt, every passenger behind it gets delayed unpredictably!
- Worse, if a rude shipper keeps shoving bricks faster than the scanner runs, the waiting hall overflows and incoming emergency passengers get turned away at the door!
Diverse applications need **separation and scheduling fairness** (like WFQ or priority queues), not a blind one-size-fits-all FIFO line.

---

### 26. Issue with Packet-Level Round-Robin Fair Queuing & Byte-by-Byte Weighted Fair Queuing (WFQ) Fix [Frequency: 1]

> **Question**: Explain the issue of round robin fair queuing packet scheduling algorithm (and how byte-by-byte / Weighted Fair Queuing improves performance).

#### pre-requisite
- **Reference Context**: Router packet scheduling (`OverviewOfNetworking.pptx` / Tanenbaum Ch. 5 — Fair Queuing & Weighted Fair Queuing, also listed in AOS Endsem Syllabus!).

#### Answer (Exam Writing Format)
1. **Issue with Packet-by-Packet Round-Robin Fair Queuing (FQ)**:
   - Standard Fair Queuing maintains a separate queue per flow and scans queues in Round-Robin, sending **one whole packet per turn**.
   - **Packet-Size Unfairness**: Because packets vary widely in size, flows sending **large packets** (e.g., 1500-byte bulk file transfers) receive far more **bits per second (bandwidth)** than flows sending **small packets** (e.g., 100-byte VoIP/interactive packets) during each round!
2. **How Byte-by-Byte Simulation / Weighted Fair Queuing (WFQ) Solves It**:
   - **Ideal Fluid Model**: Conceptually drains **1 bit (or 1 byte) at a time** round-robin across all active flow queues (and allows configurable flow weights $w_i$ in WFQ so higher-weight flows drain $w_i$ bytes per round).
   - **Practical Implementation via Virtual Finishing Time**: Since real routers cannot interleave individual bytes of different packets on one wire, the router computes each arriving packet's **virtual finishing time $F_i$** (the round number when its last byte would finish draining in the ideal bit-by-bit system) and always transmits the packet with the **smallest finishing time** next—guaranteeing exact fair bandwidth sharing regardless of packet size!

#### IN DETAIL
Imagine two flows sharing a 10 Mbps router:
- **Flow A** sends tiny **100-byte** gaming packets.
- **Flow B** sends giant **1500-byte** video-backup packets.
If a router just takes turns sending *"1 packet from A, then 1 packet from B"*, Flow B steals **1500 bytes** every turn while Flow A only gets **100 bytes**—so Flow B grabs **93.75%** of the link's bandwidth!! Totally unfair!

**Bit-by-bit Fair Queuing / WFQ** fixes this brilliantly:
- Imagine an ideal magical server taking **1 byte from A, 1 byte from B** simultaneously: Flow A's tiny 100-byte packet finishes at **Round 100**, whereas Flow B's huge 1500-byte packet doesn't finish until **Round 1500**!
- Because a real router has to transmit whole packets cleanly, it simply tags every incoming packet with its **simulated finishing round** (`100` vs `1500`) and transmits the packet with the earliest finishing time first! Result: **small packets zip right out with low delay, and every flow receives its exact fair share of bytes/second!**

---

### 27. Weighted Fair Queuing (WFQ / WFG) Algorithm: Fluid-Flow Bandwidth Share & Virtual Finish Time Numerical Trace [Syllabus 2.(v) / Practice]

> **Question**: Explain the **Weighted Fair Queuing (WFQ)** packet scheduling algorithm, stating the Generalized Processor Sharing (GPS) bandwidth formula and Virtual Finish Time ($F_i^k$) equations. **Numerical**: A link serves three flows with weights $w_1 = 1, w_2 = 1, w_3 = 2$. At $t = 0$, packets arrive simultaneously: $P_{1,1}$ ($100\text{ B}$, Flow 1), $P_{2,1}$ ($100\text{ B}$, Flow 2), and $P_{3,1}, P_{3,2}$ (each $100\text{ B}$, Flow 3). Compute $F_i^k$ for all packets and determine their transmission order.

#### pre-requisite
- **Reference**: `OverviewOfNetworking.pptx` (QoS Scheduling) & Syllabus Item `2.(v)`.
- **WFQ Equations**:
  - Guaranteed minimum rate: $R_i = \frac{w_i}{\sum_{j \in \text{Active}} w_j} \cdot C$.
  - Virtual Start & Finish Times for packet $k$ of flow $i$ (length $L_i^k$, arrival $A_i^k$):
    $$S_i^k = \max\big(F_i^{k-1}, A_i^k\big), \qquad F_i^k = S_i^k + \frac{L_i^k}{w_i}$$
  - **Scheduling Rule**: Transmit the queued packet with the **smallest Virtual Finish Time $F_i^k$**.

#### Answer (Exam Writing Format)
1. **Bandwidth Share Under Active Flows ($w_1=1, w_2=1, w_3=2$, total weight = $4$)**:
   - Flow 1 gets $1/4 = 25\%$ of $C$; Flow 2 gets $1/4 = 25\%$; Flow 3 gets $2/4 = 50\%$ (exact weight proportionality).
2. **Virtual Finish Time Calculations** ($A_i^k = 0$, $F_i^0 = 0$):
   - **$P_{1,1}$** (Flow 1, $L=100, w_1=1$): $F_{1,1} = \max(0, 0) + \frac{100}{1} = \mathbf{100}$
   - **$P_{2,1}$** (Flow 2, $L=100, w_2=1$): $F_{2,1} = \max(0, 0) + \frac{100}{1} = \mathbf{100}$
   - **$P_{3,1}$** (Flow 3, $L=100, w_3=2$): $F_{3,1} = \max(0, 0) + \frac{100}{2} = \mathbf{50}$
   - **$P_{3,2}$** (Flow 3, queued behind $P_{3,1}$): $S_{3,2} = \max(F_{3,1}, 0) = 50 \implies F_{3,2} = 50 + \frac{100}{2} = \mathbf{100}$
3. **Transmission Order** (smallest $F_i^k$ first; ties broken by arrival/flow index):
   - 1st: **$P_{3,1}$** ($F = 50$)
   - 2nd: **$P_{1,1}$** ($F = 100$)
   - 3rd: **$P_{2,1}$** ($F = 100$)
   - 4th: **$P_{3,2}$** ($F = 100$)
   - **Final Schedule**: $P_{3,1} \to P_{1,1} \to P_{2,1} \to P_{3,2}$ *(Flow 3 transmitted 200 B while Flows 1 & 2 each sent 100 B)*.

#### IN DETAIL
Why calculate $L_i^k / w_i$?
- Under an imaginary bit-by-bit fluid pump, Flow 3's pipe is **$2\times$ wider** ($w_3 = 2$), so draining a 100-byte packet takes only **50 virtual seconds** instead of **100**!
- Therefore, WFQ schedules whichever packet would *finish earliest* in that ideal fluid system (`F = 50` beats `100`!). And notice how $P_{3,2}$ starts at `50` (not `0`!) because Flow 3 has to finish `P_{3,1}` before starting `P_{3,2}`, ending at `100`—delivering exact `2 : 1 : 1` weighted fairness across variable packet sizes!

---

### 28. Purpose of Leaky Bucket Traffic Shaping & Consequences Without It [Frequency: 1]

> **Question**: Explain the purpose of leaky bucket algorithm? What will happen, if you do not implement leaky bucket algorithm?

#### pre-requisite
- **Traffic Shaping**: Regulating the rate and burstiness of packets injected into a network before they enter routers.

#### Answer (Exam Writing Format)
- **Purpose of Leaky Bucket Algorithm**:
  - Smooths out irregular, **bursty traffic** from an application/host into a steady, **constant-rate stream** of packets entering the network.
  - Conceptually works like a bucket with a small hole at the bottom: packets enter at arbitrary bursty rates (filling the bucket up to capacity $B$) and leak out into the network at a fixed constant rate $R$; excess packets when full are dropped (or delayed).
- **What Happens If You Do Not Implement It**:
  1. **Router Buffer Overflows & Packet Loss**: Sudden high-speed bursts overwhelm downstream routers' finite queues, causing severe packet drops.
  2. **Network Congestion & Collapse**: Dropped packets trigger massive TCP/transport retransmissions across competing flows, inflating queuing delays, causing jitter for real-time apps, and risking congestion collapse.
  3. **Inability to Enforce SLAs / QoS**: Without traffic policing/shaping at the edge, greedy hosts inject traffic spikes that violate contracted traffic profiles and break QoS guarantees for well-behaved flows.

#### IN DETAIL
Think of pouring water rapidly out of a huge pitcher (**bursty computer traffic**—a user clicks a webpage and dumps 50 packets in 1 millisecond) into a narrow kitchen drain (**router link**):
- **Without a Leaky Bucket**: If you dump all 50 packets straight into the network at 10 Gbps when the bottleneck router downstream can only handle 100 Mbps, the downstream router's buffer overflows immediately, drops half your packets, and forces retransmissions!
- **With a Leaky Bucket**: You pour your burst into a **bucket** (buffer of size $B$) that has a fixed-width spout at the bottom leaking out cleanly at **100 Mbps**. The downstream network sees a smooth, predictable, steady trickle—virtually eliminating burst-induced congestion! *(And if a rogue sender tries to pour faster than the bucket holds, excess overflows right at the edge before hurting the rest of the network!)*

---

### 29. T/F: "Leaky Bucket Performs Better Than Token Bucket Algorithm" [Frequency: 1]

> **Question**: *"'Leaky bucket algorithm' performs better than the 'token bucket algorithm'."*

#### pre-requisite
- **Leaky Bucket vs. Token Bucket**:
  - **Leaky Bucket**: Outputs packets at a strict, rigid **constant rate** regardless of idle periods or bursts.
  - **Token Bucket**: Accumulates tokens during idle periods up to capacity $B$, allowing controlled **bursty output** up to $B$ packets immediately when traffic arrives while capping the long-term average rate.

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: The **token bucket algorithm** generally performs better (is more flexible) than the **leaky bucket algorithm** for bursty traffic.
- **Justification**: The leaky bucket enforces a rigid constant output rate even when the host was previously idle, unnecessarily delaying bursty transmissions. In contrast, the token bucket saves unused tokens during idle periods and lets bursts pass at peak rate up to bucket capacity—adapting smoothly to realistic bursty traffic while still bounding the average rate.

#### IN DETAIL
Compare two phone data plans:
- **Leaky Bucket**: You're strictly capped at **1 MB every single second**, even if you haven't used the Internet for an entire hour! When you suddenly click a webpage needing a 5 MB burst, it throttles you painfully slow!
- **Token Bucket**: You save up unused allowance (**tokens**) while idle—so when you click a webpage, you can immediately burst at full speed using your saved tokens, while still preventing sustained overload over the long run!

*(Note: Question 2.4 is missing between pages 3 and 4 of the repository scan.)*

---
