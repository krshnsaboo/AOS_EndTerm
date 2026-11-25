# Topic 10: Networking Foundations, Data Link & Medium Access Layer (Tanenbaum & Slides)

> **Overview**: Layering & distributed-vs-network prerequisites, bridges/switches, Data Link framing/error/flow control, piggybacking, Go-Back-N & Selective Repeat sliding windows, ALOHA, and CSMA / CSMA-CD MAC protocols.  
> **Reference Material**: [OverviewOfNetworking.pptx](../Faculty_Notes/OverviewOfNetworking.pptx)  
> **Questions**: 11 total questions  
> **Index**: [← Back to Topic Roadmap](./00_INDEX_AND_TOPIC_ROADMAP.md)

---

## Questions Overview

1. [Difference Between a Computer Network and a Distributed System](#1-difference-between-a-computer-network-and-a-distributed-system-frequency-1) — Frequency: 1
2. [IPC Producer–Consumer Paradigm vs. Network Host-to-Host Communication Challenges](#2-ipc-producerconsumer-paradigm-vs-network-host-to-host-communication-challenges-frequency-1) — Frequency: 1
3. [T/F: "A Change in the Service Provided at Layer k Will Impact Services at Both k+1 and k-1"](#3-tf-a-change-in-the-service-provided-at-layer-k-will-impact-services-at-both-k1-and-k-1-frequency-1) — Frequency: 1
4. [T/F: "It Makes No Difference If We Replace a Bridge with a Switch"](#4-tf-it-makes-no-difference-if-we-replace-a-bridge-with-a-switch-frequency-1) — Frequency: 1
5. [Data Link Layer Design Issues & How Sliding Window / ARQ Protocols Address Them](#5-data-link-layer-design-issues-how-sliding-window-arq-protocols-address-them-frequency-1) — Frequency: 1
6. [Piggybacking in Data Link Layer: How It Works & Delayed-ACK Issues](#6-piggybacking-in-data-link-layer-how-it-works-delayed-ack-issues-frequency-1) — Frequency: 1
7. [Tradeoffs Between Go-Back-N and Selective Repeat Sliding Window Protocols](#7-tradeoffs-between-go-back-n-and-selective-repeat-sliding-window-protocols-frequency-2) — **Frequency: 2**
8. [Pure ALOHA vs. Slotted ALOHA: Implementation Difference & Vulnerable Window Proof](#8-pure-aloha-vs-slotted-aloha-implementation-difference-vulnerable-window-proof-frequency-2) — **Frequency: 2**
9. [Why Non-Persistent CSMA Outperforms 1-Persistent CSMA Under Load](#9-why-non-persistent-csma-outperforms-1-persistent-csma-under-load-frequency-1) — Frequency: 1
10. [p-Persistent CSMA Protocol & Why It Beats 1-Persistent CSMA](#10-p-persistent-csma-protocol-why-it-beats-1-persistent-csma-frequency-1) — Frequency: 1
11. [CSMA/CD (IEEE 802.3 Ethernet): Collision Detection, Binary Exponential Backoff & Minimum Frame Size Numerical](#11-csmacd-ieee-8023-ethernet-collision-detection-binary-exponential-backoff-minimum-frame-size-numerical-syllabus-2iii-practice) — **Syllabus 2.(iii) / Practice**

---

### 1. Difference Between a Computer Network and a Distributed System [Frequency: 1]

> **Question**: What is the difference between distributed system and computer network?

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slide 7).

#### Answer (Exam Writing Format)
| Aspect | Computer Network | Distributed System |
| :--- | :--- | :--- |
| **User Visibility** | **Users are explicitly exposed to individual machines**; they must explicitly log into or address remote machines (e.g., `ssh hostB`, copying files manually). | **Appears to users as a single coherent system**; machine boundaries, data locations, and remote execution are hidden transparently by middleware/OS layers. |
| **Architecture Layer** | Focuses on **interconnecting autonomous computers** so they can exchange packets/messages over communication links. | Built **on top of a computer network** as a unified software abstraction layer (e.g., WWW, HDFS, cloud clusters). |

#### IN DETAIL
Although both connect autonomous computing nodes over physical media, the distinction hinges entirely on **system transparency (`OverviewOfNetworking.pptx`)**:
- **Computer Network**: Exposes independent hardware hosts explicitly to users and applications. Clients must know target identities, invoke explicit remote-transfer protocols (`ssh host`, `scp`, explicit socket endpoints), and manage failures/paths manually across autonomous OS instances.
- **Distributed System**: Layers unified middleware abstractions *above* a computer network so multiple independent nodes present the **coherent illusion of a single logical system** (single namespace, transparent replication, fault masking, and distributed scheduling, as in HDFS or Google Search), decoupling user tasks from specific physical server locations.

---

### 2. IPC Producer–Consumer Paradigm vs. Network Host-to-Host Communication Challenges [Frequency: 1]

> **Question**: In operating systems, we have developed a producer and consumer paradigm to communicate. Compare and contrast this OS producer–consumer paradigm with communication between two computers in a computer network.

#### Answer (Exam Writing Format)
- **Similarities**:
  1. Both transfer a stream of data/items from a sender (**Producer**) to a receiver (**Consumer**) through an intermediate channel/buffer.
  2. Both require flow-control synchronization so a fast sender does not overflow a slower receiver's finite buffer capacity (e.g., bounded buffer semaphores in OS vs. sliding window `rwnd` in TCP).
- **Differences**:

| Dimension | OS Producer–Consumer Paradigm | Computer Network Communication |
| :--- | :--- | :--- |
| **Communication Medium** | Shared memory buffer inside one machine (reliable, near-zero latency, shared clock/address space). | Unreliable physical links/routers across distant hosts (non-zero propagation delay, bit errors, packet loss, independent clocks). |
| **Direction & Channels** | Typically unidirectional via a single shared bounded buffer queue protected by locks/semaphores. | Usually **bidirectional / full-duplex** over independent transmit/receive channels (no shared-memory mutexes possible). |
| **Error & Order Handling** | Memory writes don't corrupt or drop data; ordering is preserved automatically in memory. | Requires framing, CRC/checksums, sequence numbers, ACKs, and retransmissions to recover lost/corrupted/reordered packets. |

#### IN DETAIL
While OS **Inter-Process Communication (IPC)** and **Computer Networks** both solve **Producer–Consumer flow coordination across bounded buffers**, replacing reliable shared kernel memory with physical communication links fundamentally alters protocol requirements:
- **Trusted OS Environment**: Shared-memory/pipe producers share one hardware clock, linearizable DRAM coherence, hardware-verified delivery (zero silent frame corruption or arbitrary reordering), and single-kernel scheduling locks.
- **Hostile Asynchronous Network**: Packets traverse shared noisy cables and heterogeneous intermediate queues without shared physical memory or global clocks. Frames can suffer bit-flips (requiring **CRC checksums**), drop inside congested switches (requiring **sequence numbers, ACKs, and retransmission timers**), arrive out-of-order across multipath routes, or overwhelm slow receivers lacking OS spinlock backpressure (requiring explicit **sliding-window flow control**).

---

### 3. T/F: "A Change in the Service Provided at Layer k Will Impact Services at Both k+1 and k-1" [Frequency: 1]

> **Question**: *"A change in the service provided at layer 'k', will impact services at both layers 'k-1' and 'k+1'."*

#### pre-requisite
- **Layered Protocol Stack**: Layer $k$ **uses** services provided by lower layer $k-1$ and **provides** services upward to layer $k+1$ across a clean interface.

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: A change in the service provided at layer $k$ will impact only the upper layer **$k+1$** (not layer $k-1$).
- **Justification**: In a layered architecture, lower layer $k-1$ provides services upward to layer $k$, and layer $k$ provides its service interface solely to layer $k+1$ above it. Therefore, changing the service interface exported by layer $k$ requires updating layer $k+1$, while layer $k-1$ below remains completely unaffected.

#### IN DETAIL
OSI/TCP-IP protocol layering enforces an asymmetric **vertical interface hierarchy (`Service` vs. `Protocol`)** between adjacent layers:
- **Upward Service Contract**: Layer $k$ consumes the unchanged **SAP (Service Access Point)** interface exported upward by Layer $k-1$ and constructs an enhanced **service** exported upward exclusively across the $k / (k+1)$ interface to Layer $k+1$.
- **Why $k-1$ Is Insulated**: Modifying what guarantees or primitives Layer $k$ provides upward (for example, changing Layer 4 Transport from connectionless unreliable UDP to connection-oriented reliable TCP, or updating Layer 3 routing) forces Layer $k+1$ (Application) to adapt its invocation logic—yet Layer $k-1$ (Network/Link) remains completely invariant as long as Layer $k$ continues invoking Layer $k-1$'s existing primitives properly.

---

### 4. T/F: "It Makes No Difference If We Replace a Bridge with a Switch" [Frequency: 1]

> **Question**: *"In the given network, it does not make any difference if we replace the 'bridge' with the 'switch'."*

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slide 60: "Repeater, Hub, Bridge and Switch").

#### Answer (Exam Writing Format)
- **FALSE**.
- **Corrected Statement**: Replacing a **bridge** with a **switch** improves network performance, port capacity, and collision isolation (though both perform Layer-2 MAC forwarding).
- **Justification**: A traditional **bridge** connects a few multi-host LAN segments using software-based frame inspection, often leaving multiple hosts sharing a collision domain per port. A **switch** connects many individual hosts directly on dedicated **full-duplex hardware (ASIC) ports**, providing wire-speed forwarding, lower latency, higher aggregate bandwidth, and dedicated per-host collision-free links.

#### IN DETAIL
Although both **Layer-2 Bridges** and **Ethernet Switches** execute identical Data-Link **backward MAC address learning** and spanning-tree loop avoidance (`OverviewOfNetworking.pptx` Slides 60–63), their internal hardware and collision-domain topology differ radically:
- **Collision Domain Isolation**: A traditional 2-to-4-port bridge connects shared coax/hub LAN segments where stations behind each port still share half-duplex broadcast segments and contend via CSMA/CD. A modern **switch** dedicates one isolated physical wire directly to each host port (**micro-segmentation**)—reducing every collision domain to a single full-duplex point-to-point link with **zero collisions** (`CSMA/CD` disabled).
- **Hardware Switching Fabric**: Bridges historically forwarded frames sequentially in CPU software over shared buses, whereas switches use dedicated hardware **ASIC crossbars** enabling simultaneous wire-speed parallel forwarding across independent port pairs.

---

### 5. Data Link Layer Design Issues & How Sliding Window / ARQ Protocols Address Them [Frequency: 1]

> **Question**: List the issues in designing data link layer? Briefly explain one protocol and discuss how these issues are addressed.

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slides 34–52).

#### Answer (Exam Writing Format)
- **Core Design Issues in Data Link Layer**:
  1. **Framing**: Breaking raw physical bit streams into discrete frames (identifying frame start/end boundaries).
  2. **Error Control**: Detecting/correcting bit errors and recovering lost/corrupted frames over noisy physical links.
  3. **Flow Control**: Preventing a fast sender from overwhelming a slow receiver's buffer.
  4. **Services to Network Layer & Medium Access (on broadcast links)**: Reliable/unreliable interface and channel sharing.
- **Example Protocol — Sliding Window / Go-Back-N ARQ (or Stop-and-Wait ARQ)**:
  - **Framing**: Uses flag bytes with byte/bit stuffing (or length fields) to delimit frames unambiguously.
  - **Error Control**: Appends a **CRC (Checksum)** to detect bit errors, tags frames with **sequence numbers**, and uses **ACKs + retransmission timers** so corrupted or lost frames are automatically resent.
  - **Flow Control**: Limits the sender's maximum unacknowledged in-flight frames to a bounded **sending window size ($W$)** matching the receiver's buffer capacity.

#### IN DETAIL
The Data Link Layer converts the Physical Layer's raw, unreliable stream of voltage pulses / asynchronous bits into a **framed, error-checked, pace-regulated node-to-node transport abstraction** through three core mechanisms:
- **Framing**: Physical bit streams have no intrinsic record markers; explicit delimiters (`Flag bytes 0x7E` with **byte stuffing** or bit pattern `01111110` with **zero-bit stuffing** after five `1`s) preserve transparent arbitrary payloads while letting receivers synchronize after noise corruptions.
- **Error Control (ARQ + CRC)**: Appending polynomial Frame Check Sequence (**CRC**) bits detects burst bit errors reliably, while positive/negative **ACKs + sequence numbers + Retransmission Timers** recover dropped or corrupted frames automatically.
- **Flow Control (Sliding Window)**: Bounding unacknowledged in-flight frames to the receiver's advertised buffer capacity prevents fast transmitters from overflowing slower link-endpoint NIC buffers.

---

### 6. Piggybacking in Data Link Layer: How It Works & Delayed-ACK Issues [Frequency: 1]

> **Question**: Explain how the notion of piggybacking is employed by datalink layer? What is the issue with piggybacking?

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slide 47).
- **Full-Duplex Bidirectional Communication**: Both Node A and Node B send data frames *and* need acknowledgments (ACKs) back from the other side.

#### Answer (Exam Writing Format)
- **How Piggybacking is Employed**:
  - Instead of sending a separate, dedicated control frame solely for an ACK whenever a data frame arrives, the receiver **temporarily holds (delays) the acknowledgment** and attaches ("piggybacks") the `ack` number into a small header field of its **next outgoing data frame** heading back to the sender.
  - **Benefit**: Cuts down the number of frames transmitted, saving header overhead and channel bandwidth.
- **Issues with Piggybacking**:
  1. **Increased ACK Delay / Spurious Timeouts**: If the receiver has no outgoing data frame ready to send for a long time, delaying the ACK causes the original sender's retransmission timer to expire, triggering **unnecessary duplicate retransmissions**.
  2. **Complexity of Timer Tuning**: The link layer must maintain an extra "ACK-hold timer" to determine how long to wait for outgoing reverse data before giving up and sending a standalone ACK frame (hybrid approach).

#### IN DETAIL
**Piggybacking** reduces per-frame overhead on **bidirectional full-duplex links** by collapsing reverse control acknowledgments directly into outgoing data headers:
- **Protocol Framing Efficiency**: On an active link where Node $A$ and Node $B$ exchange data in both directions (`Frame = [Preamble | MAC Header | Seq# | Ack# | Payload | FCS]`), every standalone control `ACK` frame still pays full physical/link overhead—preamble synchronization, SFD, L2 header fields, CRC trailer, inter-frame gap, and shared-channel acquisition contention—just to deliver a 2-byte sequence acknowledgment. Piggybacking reuses the `Ack#` header field already present in normal reverse data frames for free, effectively doubling link efficiency under symmetric traffic.
- **The Timer Dilemma ($T_{\text{ack\_hold}} \ll T_{\text{RTO}}$)**: Piggybacking stalls whenever reverse traffic turns **asymmetric or idle** at the receiver (`B`). Waiting indefinitely for outgoing reverse payload causes Sender `A`'s **Retransmission Timeout ($T_{\text{RTO}}$)** to expire, forcing duplicate retransmissions and wasting bandwidth. Therefore real protocols (HDLC, IEEE 802.11, TCP Delayed ACK) implement a **bounded hold timer**: start `T_ack_hold` upon frame receipt; if reverse data becomes ready beforehand, attach `Ack#` immediately and cancel the timer—otherwise, when `T_ack_hold` expires safely before $T_{\text{RTO}}$, transmit an explicit standalone `ACK` frame immediately.

---

### 7. Tradeoffs Between Go-Back-N and Selective Repeat Sliding Window Protocols [Frequency: 2]

> **Question**: Explain the tradeoffs between "Go-Back-N" Sliding Window Protocol and "Selective Repeat" Sliding Window Protocol.

#### pre-requisite
- **Sliding Window ARQ**: Allows multiple unacknowledged frames/segments in flight simultaneously ($W > 1$) for pipelined link utilization.

#### Answer (Exam Writing Format)
| Tradeoff Dimension | Go-Back-N (GBN) | Selective Repeat (SR) |
| :--- | :--- | :--- |
| **Receiver Window / Buffering** | Receiver window = **1**; discards *all* out-of-order frames after a lost frame (simple receiver, zero out-of-order buffer). | Receiver window = **$N$**; buffers out-of-order frames until the missing frame arrives (requires substantial receiver memory). |
| **Bandwidth Efficiency on Loss** | **Poor on high-error/high-BDP links**: A single lost frame causes retransmission of that frame *plus all $N-1$ subsequent frames* already sent. | **High**: Retransmits *only* the specific lost/corrupted frame(s), saving huge bandwidth. |
| **Complexity & Sequence Numbers** | **Low complexity**: Uses cumulative ACKs and a single sender timer; requires sequence space $\ge W + 1$. | **High complexity**: Requires individual ACKs (or SACKs), per-frame timers, and sequence space $\ge 2W$ (half the sequence space) to avoid wrap-around ambiguity. |

#### IN DETAIL
**Go-Back-N (GBN)** and **Selective Repeat (SR)** present a classic **receiver complexity vs. link-utilization efficiency** trade-off across high Bandwidth-Delay Product (**BDP**) or noisy channels:
- **Receiver State vs. Re-transmission Waste**: GBN keeps receiver logic trivial ($W_R = 1$, zero out-of-order memory, single cumulative ACK + single base timer), unconditionally discarding *all* frames arriving past a gap (`N+1..N+k`). When links have high bit-error rates or large $B \times \text{RTT}$ windows, one corrupted frame forces re-transmitting an entire pipe full of $W$ already-delivered frames. **Selective Repeat** buffers valid out-of-order frames up to $W_R$, issues selective ACKs/NAKs, and retransmits *only* the lost frame.
- **Sequence-Space Aliasing Constraint**: Because SR accepts buffered out-of-order frames across an active receive window $[R_{\text{base}}, R_{\text{base}} + W_R - 1]$, ambiguity occurs if the sender retransmits old unacknowledged frames while the receiver advances its window and wraps sequence numbers (`mod 2^k`). Preventing old/new sequence overlap strictly requires **$W_S + W_R \le 2^k \implies W_{\max} = 2^{k-1}$** (only half the sequence space usable, vs. $2^k - 1$ for GBN).

---

### 8. Pure ALOHA vs. Slotted ALOHA: Implementation Difference & Vulnerable Window Proof [Frequency: 2]

> **Question**: Explain the difference between "Pure ALOHA" and "Slotted ALOHA" in terms of implementation? What is the reason for improved performance of "Slotted ALOHA" over "Pure ALOHA"?

#### pre-requisite
- **Random Access MAC Protocols**: Stations share one broadcast channel and transmit frames whenever ready without prior coordination, retransmitting after a random delay if a collision occurs.

#### Answer (Exam Writing Format)
1. **Implementation Difference**:
   - **Pure ALOHA**: Completely asynchronous—a station transmits a frame **immediately at any arbitrary time** as soon as data is ready, with no time synchronization.
   - **Slotted ALOHA**: Time is divided into discrete, uniform **slots** equal to one frame transmission time ($T$); stations must synchronize to slot boundaries and transmit **only at the beginning of a time slot**.
2. **Reason for Improved Performance**:
   - **Vulnerable Period Halved**: In Pure ALOHA, a frame of duration $T$ collides with any transmission starting anywhere in a **$2T$ vulnerable window** ($[t_0 - T, t_0 + T]$), yielding maximum throughput $S_{\max} = \frac{1}{2e} \approx \mathbf{18.4\%}$.
   - In Slotted ALOHA, frames can only collide if another station transmits in the **exact same slot**—cutting the vulnerable period in half from **$2T$ to $T$** and doubling maximum throughput to $S_{\max} = \frac{1}{e} \approx \mathbf{36.8\%}$.

#### IN DETAIL
**Pure ALOHA** and **Slotted ALOHA** isolate how **discrete time synchronization** shrinks contention overlap by half on uncoordinated random-access broadcast channels:
- **Vulnerable Window Proof ($2T$ vs. $T$)**: Consider fixed-duration frames of transmission time $T_{\text{fr}}$. In **Pure ALOHA**, stations transmit asynchronously whenever ready; a frame starting at $t_0$ suffers destructive partial overlap if *any* peer starts in $(t_0 - T_{\text{fr}},\; t_0 + T_{\text{fr}})$—a continuous vulnerable span of **$2 T_{\text{fr}}$**. Enforcing discrete global slot ticks of length $T_{\text{fr}}$ in **Slotted ALOHA** forces transmissions to align strictly with slot boundaries, eliminating partial tail/head collisions so interference occurs *only* if another station generates a frame during the exact same preceding slot—halving the vulnerable interval to **$1 T_{\text{fr}}$**.
- **Poisson Throughput Derivation**: Under Poisson arrival load $G$ frames per frame-time, $P(\text{0 peers in } 2G) = e^{-2G} \implies S = G e^{-2G}$ peaks at $G = 0.5$ (**$\frac{1}{2e} \approx 18.4\%$**), whereas Slotted ALOHA yields $S = G e^{-G}$ peaking at $G = 1$ (**$\frac{1}{e} \approx 36.8\%$**).

---

### 9. Why Non-Persistent CSMA Outperforms 1-Persistent CSMA Under Load [Frequency: 1]

> **Question**: Why Nonpersistent CSMA Protocol gives better performance over 1-Persistent CSMA Protocol? Discuss.

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slide 56).
- **CSMA (Carrier Sense Multiple Access)**: Stations listen to the shared channel ("carrier sense") before transmitting.
  - **1-Persistent CSMA**: If channel is busy, continuously sense it and transmit *immediately* (probability $1$) the instant it becomes idle.
  - **Non-Persistent CSMA**: If channel is busy, *stop sensing* and wait a **random backoff time** before sensing again.

#### Answer (Exam Writing Format)
- **Flaw of 1-Persistent CSMA**:
  - When Station A is transmitting, suppose Stations B and C both become ready and continuously listen to the medium.
  - The exact instant A finishes and the channel turns idle, **both B and C transmit immediately with probability 1**, guaranteeing a **collision**! Under moderate/high load, this causes severe collision storms and throughput collapse.
- **Why Non-Persistent CSMA Performs Better**:
  - When B and C sense the channel busy, they **do not continuously wait for the exact end of A's frame**. Instead, each waits an independent **random interval** before sensing again.
  - Because their retry times are staggered randomly in time, the probability of simultaneous transmissions—and therefore **collisions**—drops sharply, yielding **much higher channel throughput at moderate-to-high traffic loads** (at the minor cost of slightly higher idle-time delay at very low load).

#### IN DETAIL
Post-carrier-sense contention rules dictate whether multi-station queues synchronize into deterministic collision storms:
- **Synchronized Queue Collision in 1-Persistent CSMA**: When Station $A$ finishes transmitting on a shared medium, any stations ($B, C, \dots$) that arrived while busy have polled the carrier continuously (`100% persistence`). The instant carrier energy drops, all waiting stations transmit simultaneously with probability $1$, guaranteeing **100% deterministic collision** whenever $\ge 2$ frames accumulated during $A$'s transmission.
- **Stochastic Decorrelation in Non-Persistent CSMA**: Sensing a busy channel immediately aborts carrier monitoring and draws an independent **random backoff delay** before re-sensing. This decorrelates waiting nodes across time—preventing clustered synchronization upon channel release and sustaining vastly higher throughput under moderate-to-heavy load $G$, trading slightly higher mean access delay under very sparse traffic.

---

### 10. p-Persistent CSMA Protocol & Why It Beats 1-Persistent CSMA [Frequency: 1]

> **Question**: Discuss p-persistent CSMA protocol. How does it improve performance over 1-persistent CSMA protocol?

#### pre-requisite
- **Reference Slides**: `OverviewOfNetworking.pptx` (Slide 56).

#### Answer (Exam Writing Format)
- **$p$-Persistent CSMA Protocol** *(used on slotted broadcast channels)*:
  1. When a station has data to send, it senses the channel. If busy, it waits until the next idle slot.
  2. When the channel is idle:
     - With **probability $p$**, it transmits immediately in the current slot.
     - With **probability $1 - p$**, it defers to the **next slot** (and repeats sensing/decision there; if another station grabs the channel, it waits random backoff).
- **How It Improves Over $1$-Persistent CSMA**:
  - In **$1$-persistent CSMA**, if multiple stations wait while the channel is busy, all of them transmit *simultaneously* with probability $1$ the exact instant the channel becomes idle—causing guaranteed collisions and throughput collapse under heavy load.
  - By introducing probabilistic deferral ($p < 1$, e.g., $p = 0.1$), **$p$-persistent CSMA** spreads contending stations across different time slots, **drastically reducing collision probability and improving channel throughput** while keeping average delay lower than pure non-persistent CSMA.

#### IN DETAIL
**$p$-Persistent CSMA** marries continuous carrier awareness with **probabilistic contention damping** on slotted multi-access channels:
- **Algorithmic Dynamics**: Unlike non-persistent CSMA (which forfeits immediate awareness of exact channel-release ticks by sleeping blindly), ready stations under $p$-persistent CSMA monitor the busy slot boundary continuously. Once idle, each ready station flips an independent Bernoulli trial with transmit probability $p$ and defers one mini-slot with probability $1 - p$.
- **Load Scaling vs. 1-Persistent**: Setting $p = 1$ collapses directly to 1-persistent CSMA (all $k$ queued stations collide simultaneously on idle release). Choosing small $p$ (satisfying $N_{\text{active}} \cdot p < 1$) ensures typically zero or one station wins any single release slot while idle wasted slots remain short—raising stable peak utilization substantially while bounding access variance.

---

### 11. CSMA/CD (IEEE 802.3 Ethernet): Collision Detection, Binary Exponential Backoff & Minimum Frame Size Numerical [Syllabus 2.(iii) / Practice]

> **Question**: Explain the working steps of **CSMA/CD** (including **Jamming Signal** and **Binary Exponential Backoff**). Derive the **minimum frame size condition** required for CSMA/CD collision detection, and compute the minimum frame size (in bytes) for a $10\text{ Mbps}$ LAN spanning $d = 2500\text{ m}$ with signal propagation speed $v = 2 \times 10^8\text{ m/s}$ (or $R = 1\text{ Gbps}$ with $d = 200\text{ m}$).

#### pre-requisite
- **Reference**: `OverviewOfNetworking.pptx` (Slides 57–58) & Tanenbaum MAC Sublayer.
- **Key Principle**: *"Listen While Talk"*—transmit only if idle, monitor channel voltage while transmitting, and ensure frame transmission time $T_{\text{tx}} \ge 2 \cdot T_{\text{prop}}$.

#### Answer (Exam Writing Format)
1. **Four Steps of CSMA/CD**:
   - **Carrier Sense**: Sense channel before sending; if busy, wait (1-persistent in classic Ethernet).
   - **Collision Detection**: Monitor voltage/signal power *while transmitting*; if sensed level exceeds normal TX level, a collision occurred.
   - **Abort & Jamming Signal**: Immediately stop transmitting data and send a short **32–48 bit Jamming Signal** so *all* stations reliably recognize the collision.
   - **Binary Exponential Backoff**: After the $c$-th consecutive collision on a frame ($c \le 16$), wait $k \times \text{Slot Time}$ ($2\tau$) where $k$ is chosen uniformly at random from $[0, 2^{\min(c, 10)} - 1]$ (abort after 16 collisions).
2. **Minimum Frame Size Derivation**:
   - Worst-case collision: Station A sends at $t = 0$; wavefront reaches farthest Station B at $t = \tau - \epsilon$ (where $\tau = d / v$); B starts sending just before receiving it; collision signal propagates back to A, arriving at **$t = 2\tau$**.
   - For A to *still be transmitting* when collision feedback returns (so A knows its frame collided rather than thinking it succeeded):
     $$T_{\text{tx}} = \frac{L_{\min}}{R} \ge 2\tau = 2 \cdot \frac{d}{v} \implies \mathbf{L_{\min} \ge 2 \cdot \frac{d}{v} \cdot R}$$
3. **Numerical Calculation ($10\text{ Mbps}, d = 2500\text{ m}, v = 2 \times 10^8\text{ m/s}$)**:
   - $\tau = \frac{2500}{2 \times 10^8} = 12.5\,\mu\text{s} \implies 2\tau = 25\,\mu\text{s}$.
   - Wait with Ethernet repeater/jam budget (standard slot time $51.2\,\mu\text{s}$ at $10\text{ Mbps}$ gives $64\text{ bytes}$); for raw cable propagation alone above: $L_{\min} = 25 \times 10^{-6}\text{ s} \times 10^7\text{ bps} = 250\text{ bits} = \mathbf{31.25\text{ bytes}}$ (rounded up to 32 bytes; Ethernet standard 51.2 $\mu\text{s}$ slot time enforces **512 bits = 64 bytes** minimum frame size! Check with $1\text{ Gbps}$ & $d=200\text{ m}$: $2\tau = 2\,\mu\text{s} \implies L_{\min} = 2\,\mu\text{s} \times 10^9 = 2000\text{ bits} = \mathbf{250\text{ bytes}}$).

#### IN DETAIL
**CSMA/CD (IEEE 802.3 Classic Half-Duplex Ethernet)** turns passive physical collision recovery into fast **intra-transmission abort**, anchored mathematically by worst-case signal propagation round-trip time ($2\tau$):
- **Minimum Frame Size ($L_{\min} \ge 2\tau \cdot R$)**: Suppose Station $A$ at one cable extreme begins transmitting at $t = 0$. Its electrical wavefront takes maximum one-way propagation delay $\tau = d_{\max} / v_{\text{prop}}$ to reach far-end Station $B$. Station $B$ can still perceive an idle wire and transmit right up to $t = \tau - \epsilon$, corrupting the frame; the returning collision interference reaches $A$ at **$t \approx 2\tau$**. For $A$ to still be actively transmitting when collision voltage returns—ensuring it detects collision, emits the $32\text{--}48$-bit **Jam Signal**, and enters **Binary Exponential Backoff** ($[0, 2^{\min(k,10)}-1] \cdot \text{slot}$) rather than misclassifying transmission as successful—every valid Ethernet frame must last at least **$2\tau \cdot R$** bits ($512\text{ bits} = 64\text{ bytes}$ at $10\text{ Mbps}$ over $2500\text{ m}$).

---

