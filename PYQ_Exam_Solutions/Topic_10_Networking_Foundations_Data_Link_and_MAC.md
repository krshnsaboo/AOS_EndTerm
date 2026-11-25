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
As the lecture slide (`OverviewOfNetworking.pptx` Slide 7) states:
- In a **computer network**, you have a collection of independent computers connected by cables/Wi-Fi, and **you know which machine is which** (you explicitly type `scp file user@192.168.1.5:/tmp`).
- A **distributed system** puts a software layer *on top of* that computer network so all those machines act like **one single unified computer**—like Google Search or HDFS, where you submit one query or open one file path without ever knowing or caring which physical server out of 10,000 holds your data!

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
Notice how the **conceptual problem** is identical—a fast producer feeding a slower consumer across a finite buffer—but the **physical environment** changes everything!
Inside one OS, RAM is reliable, shared locks work, and data never gets lost in transit. Across a **computer network**, you have no shared memory locks, messages can travel along different paths or get dropped by noisy cables, and hosts run on completely separate clocks—which is why networking adds headers, checksums, ACKs, timeouts, and sequence numbers on top of the exact same producer–consumer flow-control idea!

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
Think of a chef (Layer $k$) buying raw vegetables from a farmer (Layer $k-1$) and serving plated meals to customers (Layer $k+1$): if the chef changes the restaurant menu (changes service exported at Layer $k$), only the customers ($k+1$) have to adapt—the farmer below ($k-1$) still delivers tomatoes the exact same way!

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
While both bridges and switches operate at **Layer 2** using backward-learning MAC tables (`OverviewOfNetworking.pptx` Slide 60–63), replacing a traditional software bridge with a switch makes a big practical difference: switches have dozens of dedicated full-duplex ASIC ports (one computer per port $\implies$ zero collisions and simultaneous multi-port parallel forwarding), whereas bridges historically joined shared-bus LAN segments in software!

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
Look at what the Physical Layer actually gives the Data Link Layer: just a messy stream of noisy `0`s and `1`s where bits sometimes flip or vanish!
To turn that raw wire into a clean reliable link between two neighbors, the Data Link Layer solves 3 big problems:
1. **Framing**: Putting clear `START` and `END` markers around packets (byte/bit stuffing) so the receiver knows where one message stops and the next begins.
2. **Error Control**: Stamping a **CRC** math check at the end of every frame + demanding numbered receipts (**ACKs**) before deleting old copies!
3. **Flow Control**: Restricting the sender's **Sliding Window** so it never blasts 1,000 frames into a receiver that only has space for 8!

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
Imagine you and a friend are sending letters back and forth by courier. You receive a package from your friend and want to say *"Got package #4!"*.
- **Without piggybacking**: You immediately pay a courier to deliver an empty envelope carrying just *"Got #4!"*—wasting an entire delivery trip and envelope!
- **With piggybacking**: Since you were about to mail back a book anyway, you just write *"P.S. Got #4!"* in the corner of your outgoing package's address label! Free ACK!
- **The catch**: What if you end up having nothing to mail back for 3 days? If you wait forever to piggyback, your friend panics thinking package #4 was lost and mails it again! That's why real link/TCP protocols use a **hybrid rule**: wait briefly (e.g., a few milliseconds) for outgoing data to piggyback onto; if none appears before the ACK timer expires, send a standalone ACK frame immediately!

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
Suppose you mail 10 numbered letters (#1 to #10), and Letter #3 gets lost in the rain:
- **Go-Back-N** is simple-minded at the receiver: when Letters #4–#10 arrive, the receiver shouts *"I'm still waiting for #3!"* and throws #4–#10 into the trash! So you have to re-mail #3, #4, #5, #6, #7, #8, #9, and #10 all over again! Cheap receiver, terrible waste of bandwidth if lines are noisy or very fast/long!
- **Selective Repeat** puts #4–#10 neatly into a holding shelf (**receiver buffer**) and simply asks: *"Please resend ONLY #3!"* When #3 arrives, it delivers #3–#10 cleanly in order. Saves massive bandwidth, at the cost of extra buffer memory, per-packet timers, and needing twice as many sequence numbers ($2W$) so old buffered packets aren't confused with wrapped-around new ones!

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
Think of people speaking on a shared walkie-talkie channel, each message taking **1 minute**:
- In **Pure ALOHA**, you start talking whenever you want. If Alice starts at `12:01:30`, she overlaps and ruins Bob's message if Bob started *anytime* between `12:00:30` and `12:02:30`—a **2-minute vulnerable window** ($2T$)! Almost every message gets clipped at the front or back!
- In **Slotted ALOHA**, a global clock beeps once every minute (`12:00`, `12:01`, `12:02`...). Everyone is forced to start talking *only* on the beep! Now Bob's message at `12:01` can *only* collide if someone else also starts at `12:01`—cutting partial overlaps completely and shrinking the danger zone from $2T$ to **$1T$**, which mathematically doubles peak channel efficiency from **18.4% to 36.8%**!

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
Think of two people (B and C) waiting politely outside a single phone booth while Person A is talking:
- **1-Persistent CSMA**: Both B and C stand with their hands glued to the booth door waiting for A to step out. The split second A opens the door, **both B and C dive into the phone booth at the exact same instant and collide!** Then both have to back out and try again!
- **Non-Persistent CSMA**: B checks the booth, sees it busy, and decides *"I'll walk around for a random 2 minutes."* C sees it busy and says *"I'll walk around for a random 5 minutes."* When A finishes, B returns first and gets the booth cleanly with **zero collision**! Only when traffic is super light does 1-persistent feel slightly faster; as soon as multiple stations get busy, Non-persistent easily beats 1-persistent!

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
Suppose 5 cars are stopped at a red construction zone (**channel busy**):
- **1-Persistent CSMA ($p = 1$)**: The exact millisecond the light turns green, all 5 drivers stomp the gas simultaneously—instant 5-car pileup (**100% collision**)!
- **$p$-Persistent CSMA ($p = 0.2$)**: When the light turns green, each driver flips a weighted coin (20% chance go now, 80% wait 1 second). Usually just **1 lucky driver** goes cleanly while the others hold back a slot, cutting collisions dramatically!

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
Why can't classic Ethernet allow tiny 10-byte packets on a long cable?
- Imagine Station A at one end of a 2.5 km hallway shouts a super short **1-word tweet (`Hi!`)** in 1 second, while it takes **12.5 seconds** for sound to cross the hallway!
- At $t = 12.4\text{ s}$, Station B at the far end hasn't heard anything yet and shouts too—crashing right into `Hi!`! The crash echo reaches A at $t = 25\text{ s}$, **long after A already stopped talking and walked away thinking delivery succeeded!**
- Solution: **Keep talking for at least one full round-trip echo time ($2\tau$)** (`L_min = 64 bytes` in IEEE 802.3 Ethernet, padding small packets if needed) so you are guaranteed to still be on the mic if a collision echo comes back!

---
