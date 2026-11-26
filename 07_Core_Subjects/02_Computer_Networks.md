# Computer Networks — Complete Interview Guide

> For: backend SDE intern / new-grad rounds (Amazon, Google, Zomato/Eternal, Flipkart, ...).
> Goal: after this one file you can clear a **core CN round**.
> Format of every topic: **Simple explanation → Key points → Diagram/table → Code (if any) → Interview tip / trap.**
> ⭐ = asked very often. **Project** lines link a concept to your own work (LocalArena, TTL Chat, SolanaPay). Use them — they turn a textbook answer into a "this person has actually done it" answer.

---

## Table of Contents

1. [How to use this file](#1-how-to-use-this-file)
2. [Network Basics](#2-network-basics)
3. [OSI Model and TCP/IP Model](#3-osi-model-and-tcpip-model)
4. [Encapsulation](#4-encapsulation)
5. [Physical and Data Link Layer](#5-physical-and-data-link-layer)
6. [Network Layer (IP, Subnetting, NAT, ICMP, Routing)](#6-network-layer)
7. [Transport Layer (TCP, UDP, QUIC)](#7-transport-layer)
8. [Application Layer (DNS, HTTP, TLS, other protocols)](#8-application-layer)
9. [Real-Time Communication and APIs](#9-real-time-communication-and-apis)
10. [Backend Infrastructure (LB, Proxy, CDN, Gateway, Rate Limiting, CORS, Security)](#10-backend-infrastructure)
11. [⭐ What happens when you type google.com and press Enter?](#11-what-happens-when-you-type-googlecom-and-press-enter)
12. [Code Lab (C++, Go, Python)](#12-code-lab)
13. [Useful CLI Commands](#13-useful-cli-commands)
14. [Most Asked CN Interview Questions (Top 40)](#14-most-asked-cn-interview-questions-top-40)
15. [Last-Minute Revision (one page)](#15-last-minute-revision-one-page)

---

## 1. How to use this file

- **Day before the interview:** read Sections 11, 14, 15.
- **One week before:** read everything once, run the code in Section 12, do both subnetting examples on paper.
- **In the interview:** always answer in 3 steps — *what it is (1 line) → how it works (2–3 lines) → where it matters / trade-off (1 line, ideally your project).*
- Interviewers in India love: OSI layers, TCP vs UDP, 3-way handshake, "type URL and press Enter", DNS, HTTP vs HTTPS, subnetting, ARP, switch vs router, TCP congestion control. These are all ⭐ below.

---

## 2. Network Basics

### 2.1 What is a network?

**Simple explanation:** A network is a set of devices (computers, phones, servers) connected so they can share data. The Internet is a "network of networks" — millions of small networks joined by routers.

**Key points**
- **Node** = any device on the network. **Link** = the wire or radio between two nodes.
- **Protocol** = an agreed set of rules (like grammar) for how to talk. Example: HTTP, TCP, IP.
- **Bandwidth** = max capacity of a link (like the width of a pipe, in Mbps).
- **Throughput** = what you actually get (water actually flowing).
- **Latency** = time for data to go from A to B (ms).

### 2.2 LAN vs MAN vs WAN (and PAN)

| Type | Full form | Range | Example | Owner |
|---|---|---|---|---|
| PAN | Personal Area Network | ~10 m | Bluetooth earbuds + phone | You |
| LAN | Local Area Network | Room / building / campus | College lab, home Wi-Fi | One org |
| MAN | Metropolitan Area Network | A city | City cable TV network, city-wide Wi-Fi | ISP / city |
| WAN | Wide Area Network | Country / world | The Internet, a bank's branch network | Many (ISPs) |

**Interview tip:** LAN = high speed, low latency, low error; WAN = slower, higher latency, uses leased lines / ISPs.

### 2.3 Topologies

**Simple explanation:** Topology = the "shape" of how devices are wired.

| Topology | Shape | Pros | Cons |
|---|---|---|---|
| Bus | All on one cable | Cheap, simple | One cable break kills all; collisions |
| Star | All to a central switch | Easy to add/remove; one bad cable affects one node | Central device = single point of failure |
| Ring | Each to two neighbours in a circle | Ordered access (token) | One break can break ring (unless dual ring) |
| Mesh | Everyone to everyone | Very reliable, redundant | Costly: **n(n−1)/2** links |
| Tree | Hierarchy of stars | Scales, easy to manage | Root failure hurts a branch |
| Hybrid | Mix | Flexible | Complex |

```
 Bus:   A---B---C---D        Star:      A   B         Ring:  A---B
                                         \ /                 |   |
                                    C---[SW]---D             D---C
```

**Trap:** "Full mesh of 5 nodes has how many links?" → 5×4/2 = **10**.
**Project:** In TTL Chat, WebRTC video is a **mesh** of peers — capped at **up to 8 users** because each user's upload grows as **n−1** streams. That is the mesh cost formula in real life.

### 2.4 Transmission modes and cast types

| Mode | Meaning | Example |
|---|---|---|
| Simplex | One direction only | TV broadcast, keyboard → PC |
| Half duplex | Both directions, one at a time | Walkie-talkie, old hub Ethernet |
| Full duplex | Both directions at the same time | Phone call, switched Ethernet, TCP, WebSocket |

| Cast | Who receives | Example |
|---|---|---|
| Unicast | One specific host | Normal HTTP request |
| Broadcast | All hosts in the LAN | ARP request, DHCP Discover |
| Multicast | A subscribed group | IPTV, OSPF hellos (224.0.0.5) |
| Anycast | The *nearest* of many hosts with the same IP | DNS root servers, CDNs |

### 2.5 Network devices ⭐

**Simple explanation (analogy):**
- **Hub** = a loudspeaker. Whatever comes in is shouted out of every port.
- **Switch** = a smart post-room clerk inside one building. Looks at the name on the envelope (MAC address) and hands it to only that person.
- **Router** = the post office between cities. Looks at the city/pincode (IP address) and chooses the next road.
- **Gateway** = a translator at the border. Joins networks that speak different protocols.

| Device | OSI layer | Works on | Collision domains | Broadcast domains | Notes |
|---|---|---|---|---|---|
| Repeater | L1 | Signals | 1 | 1 | Regenerates weak signals |
| Hub | L1 | Bits | **1 for all ports** | 1 | Dumb, half duplex, obsolete |
| Bridge | L2 | MAC (frames) | 1 per port (usually 2 ports) | 1 | Joins two LAN segments, filters by MAC |
| Switch | L2 | MAC (frames) | **1 per port** | 1 (per VLAN) | Multi-port bridge; learns MAC table |
| Router | L3 | IP (packets) | 1 per port | **1 per port** | Joins different networks; stops broadcasts |
| L3 switch | L2+L3 | MAC + IP | 1 per port | 1 per VLAN | Switch that can route between VLANs |
| Gateway | Any (often L7) | Protocols | — | — | Protocol converter; "default gateway" = your router |
| Modem | L1 | Analog ↔ digital | — | — | Modulator-demodulator (DSL, cable) |
| Access point | L2 | Wi-Fi ↔ Ethernet | — | — | Bridges wireless clients into LAN |

**How a switch learns (MAC learning):**
1. Frame arrives on port 3 from MAC `AA`. Switch writes `AA → port 3` in its MAC table.
2. Destination MAC `BB` known? Send only to that port. Unknown? **Flood** to all ports except the incoming one.
3. Entries age out (~300 s).

**Interview tip:** "Switch breaks collision domains, router breaks broadcast domains." Say this line exactly.
**Trap:** A switch still forwards **broadcasts** to all ports (same broadcast domain). Only a router (or VLAN) stops them.

### 2.6 Circuit switching vs packet switching ⭐

**Simple explanation:**
- **Circuit switching** = booking a private train track from Delhi to Mumbai for your whole trip. Nobody else can use it, even when you are silent. (Old telephone network.)
- **Packet switching** = sending your luggage as many parcels on shared roads. Each parcel finds its own way; roads are shared by everyone. (The Internet.)

| | Circuit switching | Packet switching |
|---|---|---|
| Setup | Dedicated path set up first | No reserved path |
| Resources | Reserved for whole call | Shared, used on demand |
| Bandwidth use | Wasted when idle | Efficient |
| Delay | Fixed, low after setup | Variable (queuing) |
| Order | Always in order | May arrive out of order |
| Failure | Path break = call drops | Reroute around failure |
| Example | PSTN telephone | Internet (IP) |

Packet switching has two styles:
- **Datagram** (IP): each packet routed independently, no setup.
- **Virtual circuit** (old ATM, MPLS-ish): logical path set up first, packets follow it.
- **Message switching** (store-and-forward whole message) — old, rarely asked.

**Interview tip:** The Internet chose packet switching because traffic is **bursty** — sharing wins.

### 2.7 Delay types and the bandwidth-delay product

Total delay per hop = **processing + queuing + transmission + propagation**.

| Delay | Formula | Meaning |
|---|---|---|
| Transmission | L / R (bits / bandwidth) | Time to push all bits onto the wire |
| Propagation | distance / speed (~2×10⁸ m/s in fibre) | Time for one bit to travel |
| Queuing | varies | Waiting in router buffer (congestion) |
| Processing | µs | Router checks header, looks up route |

**Worked example:** Send 1 MB over 100 Mbps link → transmission = 8 Mb / 100 Mbps = **80 ms**.
Delhi → US ≈ 12,000 km → propagation ≈ 12,000 km / 200,000 km/s = **60 ms one way** (~120 ms RTT minimum). No software can beat the speed of light.

**Bandwidth-delay product (BDP)** = bandwidth × RTT = how many bytes are "in flight" to fill the pipe.
1 Gbps × 100 ms = 100 Mb = **12.5 MB**. TCP's window must be this large to use the full link (that is why **window scaling** exists — the old max was 64 KB).
Max TCP throughput ≈ **window / RTT** → 64 KB / 100 ms ≈ **5.2 Mbps** even on a 1 Gbps link.

**Project:** In LocalArena's multi-region week, a write to the **remote region took 805 ms vs 35 ms local**. That gap is distance (propagation delay) paid on every round trip the write needs. Lesson: keep chatty paths in-region; you cannot optimise away physics.

---

## 3. OSI Model and TCP/IP Model

### 3.1 OSI 7 layers ⭐

**Simple explanation:** OSI is a **reference model** — a way to split the huge job of networking into 7 smaller jobs. Each layer only talks to the layer just above and below it. Like a company: the CEO (application) writes a letter, the secretary formats it, the mailroom packs it, the courier delivers it.

**Mnemonic (top → bottom, 7→1):** **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing.
**(bottom → top, 1→7):** **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way.

| # | Layer | What it does (simple) | PDU name | Address | Protocols / examples | Devices |
|---|---|---|---|---|---|---|
| 7 | Application | Gives network services to apps | Data / Message | — | HTTP, HTTPS, DNS, FTP, SMTP, SSH, DHCP, gRPC, WebSocket | Gateway, L7 LB, proxy |
| 6 | Presentation | Format, encode, encrypt, compress | Data | — | TLS/SSL (often placed here), JPEG, UTF-8, gzip | — |
| 5 | Session | Open/keep/close sessions, checkpoints | Data | — | RPC sessions, NetBIOS, SOCKS, (TLS session) | — |
| 4 | Transport | Process-to-process delivery, reliability, ports | **Segment** (TCP) / **Datagram** (UDP) | **Port** | TCP, UDP, QUIC (over UDP), SCTP | L4 LB, firewall |
| 3 | Network | Host-to-host across networks, routing | **Packet** | **IP address** | IPv4, IPv6, ICMP, OSPF, BGP*, IPsec | Router, L3 switch |
| 2 | Data Link | Node-to-node in same LAN, framing, MAC, error detect | **Frame** | **MAC address** | Ethernet, Wi-Fi (802.11), PPP, ARP*, VLAN 802.1Q | Switch, bridge, AP |
| 1 | Physical | Sends raw bits as signals | **Bits** | — | Cables, fibre, radio, RJ45, voltages | Hub, repeater, modem |

\* Placement debates: **ARP** sits between L2 and L3 (uses L2 frames to find MAC for an L3 IP). **BGP** is a routing protocol but runs over **TCP port 179** (so technically application-layer). **TLS** sits between L4 and L7. **ICMP** is L3 (carried inside IP). If asked, say "it sits between layers" — shows maturity.

### 3.2 TCP/IP model (what the Internet actually uses)

| TCP/IP layer (4) | Maps to OSI | Job | Examples |
|---|---|---|---|
| Application | 7 + 6 + 5 | App protocols | HTTP, DNS, TLS, SSH, gRPC |
| Transport | 4 | Ports, reliability | TCP, UDP |
| Internet | 3 | IP addressing, routing | IP, ICMP |
| Link (Network Access) | 2 + 1 | Local delivery on wire/air | Ethernet, Wi-Fi, ARP |

Some books show a **5-layer** model (Kurose): Application, Transport, Network, Link, Physical. Both are fine — say which one you are using.

### 3.3 OSI vs TCP/IP

| | OSI | TCP/IP |
|---|---|---|
| Layers | 7 | 4 (or 5) |
| Nature | Theoretical reference model | Practical, implemented model |
| Made | Before protocols (ISO) | Protocols first, then model (DARPA) |
| Session/Presentation | Separate layers | Merged into Application |
| Used today | For teaching and talking ("L4 LB", "L7 proxy") | Runs the Internet |

**Interview tip:** Engineers use OSI numbers in daily talk: "L4 load balancer", "L7 routing", "L2 switch". Know them.
**Project:** "Envoy in LocalArena is an **L7** proxy — it understands HTTP and gRPC, so it can do route timeouts, retries and health checks. An L4 balancer would only see IP:port."

---

## 4. Encapsulation

**Simple explanation:** Like packing a gift. You put the gift (data) in a box (TCP header), put that box in a bigger box with the address (IP header), then in a courier bag (Ethernet frame). At the receiver each layer opens its own box and passes the rest up. Packing = **encapsulation**. Unpacking = **decapsulation**.

```
Application   [            HTTP data             ]                     → Data
Transport     [TCP hdr][      HTTP data          ]                     → Segment
Network       [IP hdr][TCP hdr][   HTTP data     ]                     → Packet
Data Link     [Eth hdr][IP hdr][TCP hdr][ data   ][FCS]                → Frame
Physical      010110100101110101010...                                 → Bits
```

**Key points**
- Each layer adds a **header** (Data Link also adds a **trailer** = FCS/CRC).
- Header sizes to remember: Ethernet 14 B + 4 B FCS, IPv4 20 B (min), IPv6 40 B (fixed), TCP 20 B (min), UDP 8 B.
- **MTU** (Ethernet) = **1500 bytes** of payload. TCP **MSS** = 1500 − 20 (IP) − 20 (TCP) = **1460 bytes**.
- At each **router**, the L2 frame is stripped and rebuilt (new src/dst MAC), but the **IP src/dst stay the same** (unless NAT changes them).

**Trap ⭐:** "Which addresses change hop-by-hop?" → **MAC addresses change at every hop; IP addresses stay end-to-end** (except NAT). This is asked a lot.

---

## 5. Physical and Data Link Layer

### 5.1 Physical layer (brief)

**Simple explanation:** Turns bits into signals — electricity in copper, light in fibre, radio waves in Wi-Fi.

- Media: twisted pair (Cat5e/Cat6, RJ45), coaxial, optical fibre (single-mode = long distance, multi-mode = short), wireless.
- Concepts: bandwidth, signal encoding (Manchester), modulation (AM/FM/PM, QAM), multiplexing (FDM, TDM, WDM).
- Devices: hub, repeater, modem, cables.

### 5.2 Data link layer — the job

**Simple explanation:** Moves a **frame** from one machine to the **next** machine on the same link (one hop). It packs bits into frames, adds MAC addresses, checks for errors, and decides who may talk on a shared medium.

Two sub-layers:
- **LLC** (Logical Link Control): flow control, error control, talks to L3.
- **MAC** (Media Access Control): addressing and who gets the medium (CSMA/CD, CSMA/CA).

### 5.3 MAC address

- **48 bits** (6 bytes), written in hex: `3C:22:FB:9A:10:7E`.
- First 24 bits = **OUI** (manufacturer), last 24 bits = device ID.
- Burned into the NIC (can be changed in software; phones randomise it on Wi-Fi).
- Broadcast MAC = `FF:FF:FF:FF:FF:FF`.

| | MAC address | IP address |
|---|---|---|
| Layer | L2 | L3 |
| Size | 48 bit | 32 bit (v4) / 128 bit (v6) |
| Scope | Local link only | End-to-end across Internet |
| Given by | Manufacturer | Network admin / DHCP |
| Changes? | Fixed per NIC | Changes when you move networks |
| Analogy | Your Aadhaar number (who you are) | Your postal address (where you are) |

**Interview tip:** "Why need both?" → IP is **hierarchical** so routers can route by prefix (like pincodes). MAC is **flat** (random by vendor), so you cannot route the whole Internet on it — but it is perfect for the final hop on a LAN.

### 5.4 ARP — Address Resolution Protocol ⭐

**Simple explanation:** You know someone's IP (house address) but need their MAC (face) to hand over the letter in the LAN. You shout in the room: "Who has 192.168.1.5? Tell 192.168.1.10." The owner replies privately: "That's me, my MAC is BB:BB:...".

**Steps**
1. Host A wants to send to IP `192.168.1.5`. Is it in my subnet? Yes → need its MAC. (If **not** in my subnet → I need the MAC of my **default gateway** instead.)
2. Check **ARP cache** (`arp -a`). Hit → done.
3. Miss → send **ARP Request** as **broadcast** (`FF:FF:FF:FF:FF:FF`).
4. Owner replies with **ARP Reply** as **unicast**.
5. A stores the mapping in its cache (expires after some time).

```
A ──(broadcast) "Who has 192.168.1.5?"──▶ everyone in LAN
A ◀──(unicast)  "192.168.1.5 is at BB:BB:BB:BB:BB:BB"── B
```

**Related**
- **Gratuitous ARP:** a host announces its own IP→MAC (after boot / failover, to update others' caches; also detects IP conflicts).
- **RARP:** MAC → IP (old, replaced by DHCP).
- **IPv6** has no ARP — it uses **NDP** (Neighbor Discovery Protocol, over ICMPv6).
- **ARP spoofing:** attacker replies with *its* MAC for the gateway's IP → traffic flows through attacker (MITM). Defence: dynamic ARP inspection, static ARP, and above all **TLS** for the data.

**Trap:** "Does ARP cross routers?" → **No.** ARP is local to one broadcast domain. For remote IPs you ARP for the gateway.

### 5.5 Framing

**Simple explanation:** The receiver gets a long stream of bits. Framing marks where each frame **starts and ends**.

Methods:
- **Character count** (length field) — fragile if count is corrupted.
- **Byte stuffing** — flag byte marks start/end; if flag appears in data, put an ESC before it.
- **Bit stuffing** (HDLC) — flag `01111110`; after five 1s in data, insert a 0.
- **Physical layer coding violations.**

**Ethernet II frame**
```
| Preamble 7B | SFD 1B | Dst MAC 6B | Src MAC 6B | EtherType 2B | Payload 46–1500B | FCS (CRC-32) 4B |
```
- EtherType `0x0800` = IPv4, `0x86DD` = IPv6, `0x0806` = ARP, `0x8100` = VLAN tag.
- Min frame 64 B (needed for CSMA/CD collision detection), max 1518 B (1522 with VLAN tag).

### 5.6 Error detection and correction ⭐

**Simple explanation:** Noise can flip bits. The sender adds a few extra "check" bits computed from the data. The receiver recomputes; if they don't match, the frame is corrupted.

| Method | How | Detects | Used in |
|---|---|---|---|
| Single parity | Add 1 bit so number of 1s is even (even parity) | All **odd** number of bit errors; misses 2-bit errors | Old serial, memory |
| 2-D parity | Parity per row and column | All 1/2/3-bit errors, **corrects 1-bit** | Teaching |
| Checksum | Add 16-bit words (one's complement), send complement of sum | Most errors; weak vs reordered words | IP header, TCP, UDP |
| **CRC** | Treat bits as polynomial, divide by generator (XOR division), send remainder | All burst errors ≤ r bits, all odd errors (with right generator) | Ethernet (CRC-32), Wi-Fi |
| Hamming code | Parity bits at positions 1,2,4,8...; finds the error position | **Corrects 1-bit**, detects 2-bit | ECC RAM |

**Parity example (even):** data `1011001` has four 1s → parity bit `0` → send `10110010`.

**CRC worked example ⭐ (practise once on paper)**
Data = `100100`, Generator = `1101` (degree 3 → append **3 zeros**).
```
Dividend = 100100 000   (data + 3 zeros)
Step  current 4 bits   XOR with   result (drop leading bit, bring down next)
1     1001             1101       0100 -> 1000
2     1000             1101       0101 -> 1010
3     1010             1101       0111 -> 1110
4     1110             1101       0011 -> 0110
5     0110 (lead 0)    0000       0110 -> 1100
6     1100             1101       0001 -> remainder = last 3 bits = 001
```
Remainder = **001** → transmitted frame = `100100` + `001` = **`100100001`**.
Receiver divides `100100001` by `1101` → remainder `000` → no error.
(Rule: XOR, no carries. If leading bit is 0, divide by 0000, i.e. just shift.)

**Internet checksum (simple):** split into 16-bit words → add with wrap-around carry → take one's complement → that's the checksum. Receiver adds everything including checksum → result must be all 1s (`0xFFFF`).

**Error correction approaches:**
- **FEC** (Forward Error Correction): send enough extra bits to fix errors without resending (used in satellite, video streaming, Wi-Fi).
- **ARQ** (Automatic Repeat reQuest): detect + ask to resend (stop-and-wait, Go-Back-N, Selective Repeat below).

**Trap:** Ethernet **detects** errors (CRC) but does **not** retransmit — it just drops the frame. Reliability is left to TCP. Wi-Fi does do link-layer ACK + retransmit because radio loss is common.

### 5.7 Flow control: Stop-and-Wait, Go-Back-N, Selective Repeat ⭐

**Simple explanation:** Flow control = the sender should not drown a slow receiver. Error control = resend what got lost. Sliding-window protocols do both.

**Stop-and-Wait ARQ**
- Send 1 frame → wait for ACK → send next. Timer expires → resend.
- Sequence numbers 0/1 are enough (to detect duplicates).
- Simple but slow: link sits idle during the round trip.
- Efficiency = 1 / (1 + 2a), where **a = propagation delay / transmission delay**.

```
Sender           Receiver
  |--- F0 ------->|
  |<----- ACK1 ---|
  |--- F1 ---X    |  (lost)
  |   (timeout)   |
  |--- F1 ------->|
  |<----- ACK0 ---|
```

**Go-Back-N ARQ (GBN)**
- Sender can have up to **N** unacknowledged frames in flight (window = N).
- Receiver window = **1**: accepts only the next expected frame; discards out-of-order frames.
- **Cumulative ACKs** ("I have everything up to k").
- On loss/timeout of frame k → resend **k and everything after it**.
- Window size ≤ **2^m − 1** (m = bits in sequence number).
- Simple receiver, wastes bandwidth on high-loss links.

**Selective Repeat ARQ (SR)**
- Sender window N, receiver window N too — receiver **buffers** out-of-order frames.
- Individual ACKs (or NAK) per frame.
- Only the **lost frame** is resent.
- Window size ≤ **2^(m−1)** (half the sequence space, so old and new frames are never confused).
- Efficient, but receiver needs buffers and sorting logic.

| | Stop-and-Wait | Go-Back-N | Selective Repeat |
|---|---|---|---|
| Sender window | 1 | N (≤ 2^m − 1) | N (≤ 2^(m−1)) |
| Receiver window | 1 | 1 | N |
| On loss resend | That frame | That frame + all after | Only that frame |
| ACK type | Per frame | Cumulative | Individual |
| Receiver buffer | No | No | Yes |
| Efficiency | Low | Medium | High |

```
GBN, N=4, frame 2 lost:
Sent: 0 1 2X 3 4         Receiver drops 3 and 4 (out of order)
Timeout on 2 → resend 2 3 4

SR, N=4, frame 2 lost:
Sent: 0 1 2X 3 4         Receiver buffers 3 and 4
Timeout on 2 → resend only 2 → deliver 2 3 4 in order
```

**Interview tip:** **TCP is a hybrid**: cumulative ACKs like GBN, but receivers buffer out-of-order data and modern TCP uses **SACK** (Selective ACK) so the sender resends only gaps — like SR.

### 5.8 Media access: ALOHA, CSMA/CD vs CSMA/CA ⭐

**Simple explanation:** When many devices share one medium (one cable or one radio channel), who talks when?
- **CSMA** = "listen before you talk" (Carrier Sense Multiple Access).
- **CD** = "if you hear a collision while talking, stop and retry" (Collision Detection).
- **CA** = "since you can't hear collisions, try hard to avoid them" (Collision Avoidance).

| | CSMA/CD | CSMA/CA |
|---|---|---|
| Used in | Classic wired Ethernet (half duplex, hubs) | Wi-Fi (802.11) |
| Can detect collision? | Yes (voltage change on wire) | No (radio can't send and listen well; hidden terminal) |
| Approach | Send; if collision → send **jam signal** → **binary exponential backoff** → retry | Wait for idle + random backoff (DIFS), optional **RTS/CTS**, receiver sends **ACK** |
| When collision handled | After it happens | Before it happens (avoid) |
| Today | Mostly obsolete — full-duplex switches have no collisions | Still active in every Wi-Fi network |

- **Binary exponential backoff:** after the k-th collision, wait a random slot in `[0, 2^k − 1]`.
- **Hidden terminal problem:** A and C both see access point B but not each other → both send → collision at B. **RTS/CTS** fixes it (B announces "C, stay quiet").
- **ALOHA:** pure ALOHA max efficiency **18.4%**, slotted ALOHA **36.8%**. (Asked in GATE-style rounds.)
- Other: token passing (Token Ring), polling.

**Trap:** "Does modern switched Ethernet use CSMA/CD?" → Practically no. Each port is full duplex, its own collision domain.

### 5.9 VLAN (brief)

**Simple explanation:** One physical switch, split into many **virtual** switches. Like one office building with separate locked floors — HR's broadcasts never reach Engineering.

- Each VLAN = separate **broadcast domain** (and separate subnet).
- **802.1Q** tag (4 bytes) inside the Ethernet frame, **12-bit VLAN ID** → up to 4094 VLANs.
- **Access port** = belongs to one VLAN (to a PC). **Trunk port** = carries many VLANs (switch ↔ switch), frames tagged.
- VLAN-to-VLAN traffic needs a **router or L3 switch** (inter-VLAN routing).
- Benefits: security, less broadcast traffic, group by team not by physical location.

---

## 6. Network Layer

**Simple explanation:** The network layer moves a **packet from the source host to the destination host across many networks**. It gives every host a logical address (**IP**) and lets **routers** pick the path. Data link = one hop. Network = end to end (host to host). Transport = process to process (port to port).

### 6.1 IPv4 header (basics)

```
 0                   1                   2                   3
|Version| IHL |  DSCP/ECN  |          Total Length             |
|        Identification     |Flags|     Fragment Offset         |
|   TTL   |   Protocol      |        Header Checksum            |
|                    Source IP Address (32 bit)                 |
|                 Destination IP Address (32 bit)               |
|                    Options (rare)                             |
```

| Field | Meaning |
|---|---|
| Version | 4 |
| IHL | Header length in 32-bit words (min 5 → 20 bytes) |
| DSCP/ECN | Priority (QoS) and Explicit Congestion Notification |
| Total length | Header + data, max 65,535 bytes |
| Identification, Flags (DF, MF), Fragment offset | Used for **fragmentation** and reassembly |
| **TTL** | Hop limit. Each router does TTL−1. At 0 → drop + send ICMP "Time Exceeded". Stops loops. |
| **Protocol** | What is inside: **1 = ICMP, 6 = TCP, 17 = UDP** |
| Header checksum | Checks header only; recomputed at each hop (TTL changes) |
| Source / Destination IP | 32-bit addresses |

**Fragmentation:** If a packet is bigger than the next link's MTU, a router splits it (IPv4 only). **DF (Don't Fragment)** bit set → router drops it and sends ICMP "Fragmentation Needed" → this is how **Path MTU Discovery** works.

**Trap ⭐ — two different TTLs:**
- **IP TTL** = number of **hops** (router counter, typically starts at 64 or 128).
- **DNS TTL** = number of **seconds** a DNS answer may be cached.
Same name, totally different things. Interviewers love mixing them up.

### 6.2 IPv4 vs IPv6 ⭐

| | IPv4 | IPv6 |
|---|---|---|
| Address size | 32 bit (~4.3 billion) | 128 bit (~3.4 × 10³⁸) |
| Notation | Dotted decimal `192.168.1.1` | Hex with colons `2001:db8::1` (`::` = run of zeros) |
| Header | 20–60 B, variable | **40 B fixed**, simpler, extension headers |
| Header checksum | Yes | **No** (L2 and L4 already check) |
| Fragmentation | Routers and sender | **Only sender** (routers never fragment) |
| Address config | Manual or DHCP | **SLAAC** (auto) or DHCPv6 |
| Broadcast | Yes | **No** — uses multicast and anycast |
| ARP | ARP | **NDP** (ICMPv6) |
| NAT | Common (address shortage) | Not needed (enough addresses) |
| IPsec | Optional | Built-in support (originally mandatory) |
| Loopback | `127.0.0.1` | `::1` |

**Transition:** dual stack (run both), tunnelling (6in4), translation (NAT64/DNS64).

### 6.3 Classful addressing vs CIDR ⭐

**Classful (old, still asked):**

| Class | First octet | Leading bits | Default mask | Networks / hosts | Use |
|---|---|---|---|---|---|
| A | 1–126 | 0 | /8 (255.0.0.0) | 126 nets, 16,777,214 hosts each | Huge orgs |
| B | 128–191 | 10 | /16 (255.255.0.0) | 16,384 nets, 65,534 hosts | Medium |
| C | 192–223 | 110 | /24 (255.255.255.0) | 2,097,152 nets, 254 hosts | Small |
| D | 224–239 | 1110 | — | — | **Multicast** |
| E | 240–255 | 1111 | — | — | Reserved / experimental |

- `127.x.x.x` = **loopback** (that is why class A stops at 126).
- `0.0.0.0` = "this host / any address" (a server binding to `0.0.0.0` listens on all interfaces).
- `255.255.255.255` = limited broadcast.
- Problem: very wasteful. A company needing 300 hosts got a class B (65k addresses).

**CIDR (Classless Inter-Domain Routing):** write the network length directly: `a.b.c.d/n`, where **n = number of network bits**. Any n from 0 to 32.
- Number of addresses = **2^(32−n)**. Usable hosts = **2^(32−n) − 2** (minus network and broadcast address).
- Exceptions: **/31** = 2 usable (point-to-point links, RFC 3021), **/32** = single host.
- Enables **route aggregation (supernetting)**: `192.168.0.0/24` + `192.168.1.0/24` → `192.168.0.0/23`. Smaller routing tables.
- Routers use **longest prefix match**: if a packet matches both `10.0.0.0/8` and `10.1.2.0/24`, the **/24 wins** (more specific).

**CIDR quick table (memorise the last octet column):**

| CIDR | Mask | Block size (last octet) | Addresses | Usable hosts |
|---|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 4 | 2 |
| /16 | 255.255.0.0 | — | 65,536 | 65,534 |
| /20 | 255.255.240.0 | 16 in 3rd octet | 4,096 | 4,094 |
| /22 | 255.255.252.0 | 4 in 3rd octet | 1,024 | 1,022 |

Mask octet values to remember: 128, 192, 224, 240, 248, 252, 254, 255. **Block size = 256 − mask octet.**

### 6.4 Subnetting — 2 fully worked examples ⭐

**The 4-step method (use every time)**
1. Find the **interesting octet** (where the mask is not 255 or 0).
2. **Block size** = 256 − mask value in that octet.
3. **Network address** = the largest multiple of block size ≤ the IP's value in that octet.
4. **Broadcast** = next network − 1. **Hosts** = network + 1 to broadcast − 1.

---

**Example 1: Given `192.168.10.77/26`. Find mask, network, broadcast, host range, number of hosts.**

Step 1 — mask. /26 = 26 ones:
```
11111111.11111111.11111111.11000000 = 255.255.255.192
```
Step 2 — interesting octet = 4th (192). Block size = 256 − 192 = **64**.
Subnets in the 4th octet start at: 0, 64, 128, 192.

Step 3 — 77 is between 64 and 127 → **network = 192.168.10.64**.

Binary check (AND the IP with the mask):
```
IP    77  = 0100 1101
Mask 192  = 1100 0000
AND       = 0100 0000 = 64   → network 192.168.10.64
Broadcast = network bits + all host bits = 1 → 0111 1111 = 127
```
Step 4 —
| Item | Value |
|---|---|
| Subnet mask | 255.255.255.192 |
| Network address | **192.168.10.64** |
| Broadcast address | **192.168.10.127** |
| First usable host | **192.168.10.65** |
| Last usable host | **192.168.10.126** |
| Total addresses | 2^(32−26) = 2^6 = 64 |
| Usable hosts | 64 − 2 = **62** |

---

**Example 2: You get `200.10.20.0/24`. You need 6 departments (subnets), each with at least 25 hosts. Design the subnets.**

Step 1 — how many bits to **borrow** for subnets? Need ≥ 6 subnets → 2^s ≥ 6 → **s = 3** (gives 8 subnets).
Step 2 — check hosts: host bits left = 8 − 3 = 5 → 2^5 − 2 = **30 hosts** ≥ 25 ✔.
Step 3 — new prefix = /24 + 3 = **/27**, mask = **255.255.255.224**, block size = 256 − 224 = **32**.
Step 4 — list subnets:

| # | Network | First host | Last host | Broadcast |
|---|---|---|---|---|
| 1 | 200.10.20.0/27 | .1 | .30 | .31 |
| 2 | 200.10.20.32/27 | .33 | .62 | .63 |
| 3 | 200.10.20.64/27 | .65 | .94 | .95 |
| 4 | 200.10.20.96/27 | .97 | .126 | .127 |
| 5 | 200.10.20.128/27 | .129 | .158 | .159 |
| 6 | 200.10.20.160/27 | .161 | .190 | .191 |
| 7 | 200.10.20.192/27 | .193 | .222 | .223 (spare) |
| 8 | 200.10.20.224/27 | .225 | .254 | .255 (spare) |

Answer: use /27, 8 subnets of 30 usable hosts each, 2 spare for growth.

**Follow-up they may ask — "Which subnet is 200.10.20.140 in?"** → 140 lies in 128–159 → subnet 5, `200.10.20.128/27`, broadcast `.159`.

---

**Bonus A (3rd-octet subnetting):** `172.16.45.14/20`
- Mask = 255.255.**240**.0 → interesting octet = 3rd, block = 16.
- 45 lies in 32–47 → network **172.16.32.0**, broadcast **172.16.47.255**.
- Hosts **172.16.32.1 – 172.16.47.254**, 2^12 − 2 = **4,094** hosts.

**Bonus B (VLSM — Variable Length Subnet Mask):** split `192.168.1.0/24` for LANs of 100, 50, 20 hosts and one router link (2 hosts). **Always allocate the biggest first.**

| Need | Size chosen | Subnet | Range |
|---|---|---|---|
| 100 hosts | /25 (126) | 192.168.1.0/25 | .1 – .126, bcast .127 |
| 50 hosts | /26 (62) | 192.168.1.128/26 | .129 – .190, bcast .191 |
| 20 hosts | /27 (30) | 192.168.1.192/27 | .193 – .222, bcast .223 |
| 2 hosts | /30 (2) | 192.168.1.224/30 | .225 – .226, bcast .227 |

**Interview tip:** Speak the steps aloud: "mask, block size, nearest multiple, broadcast is next minus one." Interviewers grade the method, not only the answer. A C++ subnet calculator is in [Section 12.7](#127-subnet-calculator-c).

### 6.5 Private IPs and special ranges

| Range | CIDR | Use |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | **10.0.0.0/8** | Private (big orgs, cloud VPCs) |
| 172.16.0.0 – 172.31.255.255 | **172.16.0.0/12** | Private (Docker default bridge uses 172.17.0.0/16) |
| 192.168.0.0 – 192.168.255.255 | **192.168.0.0/16** | Private (home routers) |
| 127.0.0.0/8 | Loopback | Your own machine |
| 169.254.0.0/16 | Link-local (APIPA) | Auto IP when DHCP fails; cloud metadata 169.254.169.254 |
| 100.64.0.0/10 | Carrier-grade NAT | ISP-level NAT (very common in Indian mobile networks) |
| 224.0.0.0/4 | Multicast | |

Private IPs are **not routable on the public Internet**. To reach the Internet they go through **NAT**.

### 6.6 NAT and PAT ⭐

**Simple explanation:** Your home has one public IP but 10 devices. The router is like a hotel receptionist: all outgoing letters carry the hotel's address; the receptionist keeps a register of which room sent what, so replies reach the right room.

**Types**
- **Static NAT:** 1 private ↔ 1 public, fixed (for a server).
- **Dynamic NAT:** private → any free public IP from a pool.
- **PAT / NAPT / NAT overload:** many private → **one public IP**, told apart by **port numbers**. This is what home routers do.

**PAT table example**
| Inside (private IP:port) | Outside (public IP:port) | Destination |
|---|---|---|
| 192.168.1.10:51000 | 49.36.10.5:**40001** | 142.250.x.x:443 |
| 192.168.1.11:51000 | 49.36.10.5:**40002** | 142.250.x.x:443 |

Reply comes to `49.36.10.5:40002` → router looks up → forwards to `192.168.1.11:51000`.

**Pros:** saves IPv4 addresses; hides internal hosts (a side benefit, not real security).
**Cons:** breaks end-to-end connectivity — an outside host **cannot start** a connection to you (unless **port forwarding**). Breaks peer-to-peer apps, some protocols (FTP active mode, SIP), adds state on the router.

**Project:** "NAT is why WebRTC needs help. In TTL Chat, **STUN** tells each browser its public IP:port so peers can try a direct path; when NAT/firewalls block that (e.g. symmetric NAT), **TURN** relays the media." (Details in [9.3](#93-webrtc-stun-turn-ice).)

### 6.7 ICMP

**Simple explanation:** ICMP is the network's **error and status messenger**. It does not carry user data. It is carried inside IP (protocol number 1), so it has **no ports**.

| Type | Message | Used by |
|---|---|---|
| 8 / 0 | Echo Request / Echo Reply | `ping` |
| 3 | Destination Unreachable (net, host, port, "frag needed") | Errors, Path MTU discovery |
| 11 | Time Exceeded (TTL hit 0) | `traceroute` |
| 5 | Redirect | Better route available |

- **ping** sends Echo Request, measures RTT and loss.
- **traceroute** sends packets with TTL = 1, 2, 3, ... Each router where TTL hits 0 replies "Time Exceeded" → you learn every hop. (Linux traceroute uses UDP probes by default, Windows `tracert` uses ICMP; both rely on ICMP Time Exceeded replies.)

**Trap:** "Ping fails, so the server is down?" → Not necessarily. Many firewalls/clouds **block ICMP**. Test the real port: `nc -vz host 443` or `curl -v`.

### 6.8 Routing — static vs dynamic ⭐

**Simple explanation:** A **routing table** is a router's map: "for destination prefix X, send to next hop Y via interface Z". **Routing** = building that table. **Forwarding** = using it for each packet (longest prefix match).

```
Destination      Next hop        Interface
10.0.1.0/24      directly conn.  eth0
10.0.2.0/24      10.0.1.254      eth0
0.0.0.0/0        49.36.10.1      eth1      ← default route ("everything else")
```

| | Static routing | Dynamic routing |
|---|---|---|
| Who builds table | Admin by hand | Routers exchange info automatically |
| Adapts to failure | No | Yes |
| Overhead | None | CPU, bandwidth for updates |
| Best for | Small networks, default routes, stub networks | Large / changing networks |
| Examples | `ip route add ...` | RIP, OSPF, EIGRP, BGP |

### 6.9 Distance vector vs link state ⭐

**Simple explanation:**
- **Distance vector** = "routing by rumour". Each router tells its **neighbours** its whole table: "I can reach X in 3 hops." Nobody sees the full map.
- **Link state** = "everyone gets the full map". Each router floods info about **its own links** to **all** routers; each builds the complete graph and runs **Dijkstra**.

| | Distance Vector | Link State |
|---|---|---|
| Algorithm | **Bellman-Ford** | **Dijkstra** (shortest path first) |
| What is shared | Whole routing table | Only own link states (LSAs) |
| Shared with | Neighbours only | All routers (flooding) |
| When | Periodically (RIP every 30 s) | On change (+ periodic refresh) |
| View of network | Partial (next hop + distance) | Full topology |
| Convergence | Slow | Fast |
| Problems | **Count-to-infinity**, loops | More memory/CPU |
| Example | **RIP** | **OSPF**, IS-IS |

**Count-to-infinity:** A–B–C. Link B–C breaks. A still says "I can reach C in 2". B believes A ("so I can reach C in 3"), A then says 4, and so on forever. Fixes: **split horizon** (don't advertise a route back to the neighbour you learned it from), **poison reverse** (advertise it back as infinity), **max hop count** (RIP: 16 = infinity), hold-down timers.

### 6.10 RIP, OSPF, BGP (high level)

| | RIP | OSPF | BGP |
|---|---|---|---|
| Type | Distance vector | Link state | **Path vector** |
| Scope | Interior (IGP), small | Interior (IGP), large enterprise | **Exterior (EGP) — between ASes** = the Internet's glue |
| Metric | **Hop count**, max 15 | **Cost** (based on bandwidth) | **Policy** + AS-path length, many attributes |
| Algorithm | Bellman-Ford | Dijkstra | Best-path selection by policy |
| Runs over | UDP 520 | Directly on IP (protocol 89) | **TCP 179** |
| Updates | Full table every 30 s | Triggered LSAs | Incremental |
| Notes | Simple, slow, outdated | **Areas** (area 0 = backbone) for scaling | Loop prevention by AS-path; a misconfig can take a whole company offline |

- **AS (Autonomous System)** = a network under one admin, with an ASN (e.g. an ISP like Jio, or Google).
- **IGP** = routing inside an AS (RIP, OSPF). **EGP** = routing between ASes (BGP).
- **eBGP** between ASes, **iBGP** inside one AS.

**Interview tip:** "BGP is chosen by policy, not shortest path — an ISP may prefer a cheaper peer over a shorter path." Famous outage story: Facebook 2021 withdrew its own BGP routes and its DNS servers became unreachable worldwide.

---

## 7. Transport Layer

**Simple explanation:** IP delivers a packet to the right **house** (host). The transport layer delivers it to the right **person in the house** (process), using **port numbers**. It can also make delivery reliable (TCP) or keep it fast and simple (UDP).

### 7.1 Ports and sockets

- **Port** = 16-bit number (0–65535) that identifies a process/service on a host.
  - **0–1023:** well-known (need root to bind on Linux). **1024–49151:** registered. **49152–65535:** dynamic/ephemeral (Linux default ephemeral range is 32768–60999).
- **Socket** = one endpoint = IP + port. A **TCP connection** is identified by the **4-tuple** (src IP, src port, dst IP, dst port) — plus protocol, the **5-tuple**.
- That is why one server port 443 can hold millions of connections: each has a different client IP:port.

**Ports to memorise**
| Port | Service | Port | Service |
|---|---|---|---|
| 20/21 | FTP data/control | 443 | HTTPS (and HTTP/3 on UDP 443) |
| 22 | SSH / SFTP | 465/587 | SMTP over TLS / submission |
| 23 | Telnet | 993 / 995 | IMAPS / POP3S |
| 25 | SMTP | 3306 | MySQL |
| 53 | DNS (UDP + TCP) | 5432 | PostgreSQL |
| 67/68 | DHCP server/client (UDP) | 6379 | Redis |
| 80 | HTTP | 9092 | Kafka |
| 110 | POP3 | 27017 | MongoDB |
| 143 | IMAP | 3389 | RDP |
| 161 | SNMP | 179 | BGP |

### 7.2 TCP vs UDP ⭐⭐

**Simple explanation (analogy):**
- **TCP = registered post with acknowledgement.** You first confirm the receiver is there (handshake). Every parcel is numbered, the receiver signs for it, lost parcels are re-sent, and they are given out in order.
- **UDP = shouting across the room.** You just shout. Fast. No confirmation. If they missed a word, too bad — and for live talk, that's fine.

| | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery (ACK + retransmit) | Best effort, may lose |
| Order | In order (sequence numbers) | No ordering |
| Duplicates | Removed | Possible |
| Flow control | Yes (rwnd) | No |
| Congestion control | Yes (cwnd) | No (app must be polite) |
| Data model | **Byte stream** (no message boundaries) | **Messages / datagrams** (boundaries kept) |
| Header | 20–60 bytes | **8 bytes** |
| Speed / latency | Slower to start, overhead | Fast, low latency |
| Broadcast / multicast | No | Yes |
| Used by | HTTP/1.1, HTTP/2, HTTPS, SSH, SMTP, FTP, DB connections, WebSocket, gRPC | DNS, DHCP, VoIP, video calls, online games, streaming, **QUIC/HTTP/3**, SNMP, NTP |

**When to choose UDP:** latency matters more than perfection (live media, games), tiny request/response (DNS), you want to build your own reliability (QUIC), multicast.

**Project ⭐:** "In TTL Chat, **WebRTC media goes over UDP (SRTP)** — a video frame that arrives late is useless, so TCP's retransmission would only add lag. The **signaling** (offers, answers, ICE candidates) goes over the **WebSocket, which is TCP**, because those messages must arrive and in order." Video is capped at **640×360 @ ≤30 fps, ~500 kbps/peer**.

**Trap ⭐:** "TCP is a byte stream" — one `send()` of 100 bytes may arrive as two `recv()`s of 60 + 40, or merge with the next send. **You must frame messages yourself** (length prefix or delimiter). UDP keeps boundaries: one `sendto` = one `recvfrom`.

### 7.3 TCP header (key fields)

```
| Source Port (16)          | Destination Port (16)          |
|                Sequence Number (32)                        |
|             Acknowledgement Number (32)                    |
| Data Offset | Reserved | Flags (URG ACK PSH RST SYN FIN) | Window Size (16) |
| Checksum (16)             | Urgent Pointer (16)            |
| Options (MSS, Window Scale, SACK, Timestamps) ...          |
```

| Field | Meaning |
|---|---|
| Source / Dest port | Which processes |
| **Sequence number** | Byte number of the first data byte in this segment |
| **ACK number** | "Next byte I expect from you" (cumulative ACK) |
| Data offset | Header length (20–60 B) |
| **Flags** | **SYN** (start), **ACK** (ack field valid), **FIN** (I'm done sending), **RST** (abort / "no such connection"), **PSH** (push to app now), URG (urgent) |
| **Window** | Receive window — how many bytes I can still accept (**flow control**) |
| Checksum | Over header + data + pseudo-header (IPs) |
| Options | **MSS**, **window scale** (window × 2^k, up to 1 GB), **SACK**, timestamps (RTT measurement, PAWS) |

**UDP header (8 bytes):** Source port | Dest port | Length | Checksum. That's it.

### 7.4 3-way handshake ⭐⭐

**Simple explanation:** Before talking on the phone: "Hello, can you hear me?" — "Yes, I can hear you, can you hear me?" — "Yes." Both sides now know the line works **both ways** and both have agreed their starting sequence numbers.

```
Client (SYN_SENT)                                      Server (LISTEN)
   |  ---- SYN, seq = x ---------------------------->  |  (SYN_RCVD)
   |  <--- SYN + ACK, seq = y, ack = x+1 ------------  |
   |  ---- ACK, seq = x+1, ack = y+1 ---------------->  |
(ESTABLISHED)                                      (ESTABLISHED)
```

**Key points**
- **ISN (Initial Sequence Number)** is **random** — prevents spoofing/hijacking and confusion with old connections.
- SYN and FIN each consume **one sequence number** (that's why ack = x+1 with no data).
- Options like **MSS, window scale, SACK-permitted** are agreed in the SYN/SYN-ACK.
- Cost: **1 RTT** before data. (TLS 1.3 adds 1 more RTT → HTTPS = ~2 RTT before first request; QUIC merges them into 1 RTT.)
- The **server's kernel** completes the handshake; `accept()` just takes an already-established connection from the **accept queue**. `listen(fd, backlog)` sets queue size.

**Why 3 and not 2?** With 2, the server can't know the client got its SYN-ACK, and an **old delayed SYN** could create a fake half-open connection on the server. 3 steps = both sides confirm **both** directions and both ISNs.

**SYN flood (attack):** attacker sends many SYNs, never sends ACK → server's half-open (SYN) queue fills → real users refused. Defence: **SYN cookies** (server keeps no state until the final ACK; the state is encoded in its ISN), rate limiting, bigger backlog, DDoS protection at the edge.

### 7.5 4-way teardown and TIME_WAIT ⭐

**Simple explanation:** Ending a phone call politely: "I'm done talking." — "OK." ... (the other person may still finish a sentence) ... "I'm done too." — "OK, bye." Each direction closes **separately** because TCP is full duplex.

```
Client (active closer)                               Server
ESTABLISHED                                          ESTABLISHED
   | ---- FIN, seq = u ------------------------------> | CLOSE_WAIT  (app still may send)
FIN_WAIT_1
   | <--- ACK, ack = u+1 ----------------------------- |
FIN_WAIT_2
   |                       (server app calls close())
   | <--- FIN, seq = w ------------------------------- | LAST_ACK
   | ---- ACK, ack = w+1 ----------------------------> | CLOSED
TIME_WAIT (wait 2 × MSL)
   |
CLOSED
```

- Sometimes the server's ACK and FIN are combined → it looks like **3 segments**.
- **Half-close:** after sending FIN you can still **receive** (`shutdown(fd, SHUT_WR)`).
- **RST** = abrupt close (no FIN handshake), e.g. connecting to a closed port, or app crash.

**TIME_WAIT ⭐**
- The side that **closes first** stays in TIME_WAIT for **2 × MSL** (MSL = max segment lifetime; Linux uses a fixed **60 s**).
- Why? (1) If the final ACK is lost, the server re-sends FIN — we must still be around to ACK it. (2) Let **old delayed packets** from this connection die so they don't corrupt a **new** connection with the same 4-tuple.
- Problem: a client making thousands of short connections piles up TIME_WAIT sockets → **ephemeral port exhaustion** ("cannot assign requested address").
- Fixes: **connection pooling / keep-alive** (reuse connections — best fix), `SO_REUSEADDR` on servers (rebind port after restart), `tcp_tw_reuse` on Linux clients, let the server not be the active closer.

**CLOSE_WAIT trap:** many sockets stuck in **CLOSE_WAIT** = **your app forgot to call close()** after the peer closed. That's an application bug (leaking connections), not a kernel tuning problem. Check with `ss -tan state close-wait`.

### 7.6 TCP state machine — key states

| State | Meaning |
|---|---|
| CLOSED | No connection |
| LISTEN | Server waiting for SYN (`listen()`) |
| SYN_SENT | Client sent SYN, waiting for SYN-ACK |
| SYN_RCVD | Server got SYN, sent SYN-ACK, waiting for ACK |
| **ESTABLISHED** | Open, data flows |
| FIN_WAIT_1 | I sent FIN, waiting for ACK |
| FIN_WAIT_2 | My FIN ACKed, waiting for peer's FIN |
| **CLOSE_WAIT** | Peer sent FIN; waiting for **my app** to close |
| LAST_ACK | I sent my FIN after CLOSE_WAIT, waiting for final ACK |
| **TIME_WAIT** | Waiting 2×MSL after final ACK |
| CLOSING | Both sent FIN at the same time (simultaneous close) |

### 7.7 Reliable delivery and retransmission

**How TCP makes an unreliable IP network reliable:**
1. **Sequence numbers** — every byte is numbered → receiver can reorder and drop duplicates.
2. **ACKs** — cumulative ("got everything up to byte N−1, send N next").
3. **Checksum** — detect corruption (corrupted segment = dropped = treated as lost).
4. **Retransmission timer (RTO)** — no ACK in time → resend.
   - RTO is adaptive: smoothed RTT + 4 × RTT variation (Jacobson's algorithm). Minimum ~200 ms on Linux (RFC says 1 s).
   - **Exponential backoff:** each timeout doubles RTO.
   - **Karn's algorithm:** don't measure RTT from retransmitted segments (ambiguous).
5. **Fast retransmit** — **3 duplicate ACKs** = "a segment in the middle is missing" → resend immediately without waiting for the timer.
6. **SACK** — receiver says exactly which blocks it has, so only the gaps are resent.
7. **Delayed ACK** — receiver may wait up to ~40 ms (Linux) / 200 ms to ACK, to combine ACKs.

### 7.8 Flow control (rwnd) ⭐

**Simple explanation:** The receiver says "my bucket has space for 64 KB more". The sender never sends more unacknowledged data than that. This protects the **receiver**.

- **rwnd** (receive window) is in every TCP header.
- Sender rule: bytes in flight ≤ **min(rwnd, cwnd)**.
- **Zero window:** receiver's app is slow, buffer full → rwnd = 0 → sender stops and sends small **window probes** until space opens.
- **Window scaling** option lets windows exceed 64 KB (needed for high bandwidth-delay paths, see 2.7).
- **Silly window syndrome:** sending tiny segments when window opens a few bytes → fixed by Clark's solution (receiver waits for decent space) and Nagle (sender).

**Flow control vs congestion control ⭐**
| | Flow control | Congestion control |
|---|---|---|
| Protects | The **receiver** | The **network** (routers in between) |
| Signal | rwnd advertised by receiver | Inferred from **loss**, delay, or ECN |
| Variable | rwnd | cwnd (sender's own estimate) |
| Analogy | "Don't pour faster than I can drink" | "Don't drive faster than the traffic allows" |

### 7.9 Congestion control ⭐⭐

**Simple explanation:** The sender doesn't know how much the network can handle. So it **starts slow, speeds up, and backs off sharply when it sees loss** — like merging into traffic: speed up while the road is free, brake hard at the first sign of a jam.

**Variables:** `cwnd` (congestion window, in MSS), `ssthresh` (slow-start threshold).

**Phases (TCP Reno)**
1. **Slow start:** cwnd starts at ~1 MSS (Linux today: **10 MSS**, "initial window"). **+1 MSS per ACK** → cwnd **doubles every RTT** (exponential). Continue until cwnd ≥ ssthresh.
2. **Congestion avoidance (AIMD):** **+1 MSS per RTT** (linear, Additive Increase).
3. **On 3 duplicate ACKs** (mild congestion — data still flowing):
   - **Fast retransmit:** resend the missing segment now.
   - **Fast recovery:** ssthresh = cwnd / 2, cwnd = ssthresh (+3), continue in congestion avoidance (**Multiplicative Decrease**). No slow start.
4. **On timeout** (severe congestion — nothing flowing): ssthresh = cwnd / 2, **cwnd = 1 MSS**, back to slow start.

**Worked trace (TCP Reno, initial cwnd = 1, ssthresh = 8; values in MSS, one row per RTT):**

| RTT | cwnd | Phase / event |
|---|---|---|
| 1 | 1 | Slow start |
| 2 | 2 | Slow start (doubling) |
| 3 | 4 | Slow start |
| 4 | 8 | Reached ssthresh → switch to congestion avoidance |
| 5 | 9 | +1 per RTT |
| 6 | 10 | +1 per RTT |
| 7 | 11 | **3 duplicate ACKs** → fast retransmit; ssthresh = 11/2 ≈ 5, cwnd = 5 |
| 8 | 5 | Fast recovery → congestion avoidance (no slow start) |
| 9 | 6 | +1 per RTT |
| 10 | 7 | **Timeout** → ssthresh = 7/2 ≈ 3, cwnd = 1 |
| 11 | 1 | Slow start again |
| 12 | 2 | Slow start |
| 13 | 3 | Reached ssthresh → linear again |

Plotting cwnd vs time gives the classic **sawtooth**: climb, halve, climb, halve.


| Version | Behaviour |
|---|---|
| **Tahoe** | Any loss → cwnd = 1, slow start |
| **Reno** | 3 dup ACKs → halve (fast recovery); timeout → cwnd = 1 |
| **NewReno** | Better at multiple losses in one window |
| **CUBIC** | **Linux default**. cwnd grows as a **cubic function of time since last loss** — fast when far from the old max, careful near it. Good for high-bandwidth, long-RTT links. Loss-based. |
| **BBR** (Google) | **Model-based**: estimates bottleneck **bandwidth** and **min RTT**, sends at that rate. Does not treat every loss as congestion → much better on lossy links (mobile, Wi-Fi), avoids filling buffers (bufferbloat). Used by Google/YouTube. |

**Interview tip:** Say "**AIMD** — additive increase, multiplicative decrease — is what makes many TCP flows share a link fairly."

### 7.10 Nagle's algorithm and TCP_NODELAY

**Simple explanation:** Nagle = "don't send lots of tiny packets; wait and bundle them". If there is unacknowledged data in flight, buffer small writes until the ACK arrives or you have a full MSS.

- Good for: telnet-like keystroke traffic on slow links (saves headers: 1 byte of data + 40 bytes header is wasteful).
- Bad for: **real-time / request-response** apps. **Nagle + delayed ACK** can add ~40–200 ms stalls.
- Disable with `setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &one, sizeof one)`.
- Go's `net` package sets **TCP_NODELAY = true by default**.

**Project:** "For the LocalArena game (20 Hz tick = a snapshot every 50 ms, ~15 KB/s per player) and the chat WebSockets, low latency matters more than packing bytes — that's the classic TCP_NODELAY case. Go already enables it by default."

### 7.11 Head-of-line (HOL) blocking ⭐

**Simple explanation:** One slow car at the front of a single-lane road blocks every car behind it, even if they are going elsewhere.

| Level | Where | Fixed by |
|---|---|---|
| HTTP/1.1 | Requests on one connection must be answered **in order** (pipelining) | HTTP/2 multiplexing (many streams on one connection) |
| **TCP** | One **lost packet** blocks all later bytes (TCP delivers in order) — blocks **all** HTTP/2 streams on that connection | **HTTP/3 over QUIC**: each stream is independent; a loss only stalls its own stream |

**Trap:** "Does HTTP/2 remove HOL blocking?" → **Only at the HTTP layer.** TCP-level HOL blocking still exists and can make HTTP/2 worse than HTTP/1.1 on very lossy networks (because everything shares one TCP connection).

### 7.12 QUIC ⭐

**Simple explanation:** QUIC is "TCP + TLS rebuilt on top of UDP, in user space". It was made by Google and is the base for **HTTP/3**.

**Why on UDP?** Middleboxes (firewalls, NATs) on the Internet only understand TCP and UDP. A new transport protocol would be blocked. UDP passes through, and QUIC adds reliability itself.

**Features**
- **Reliable, ordered streams** — but **per stream** → no cross-stream HOL blocking.
- **TLS 1.3 built in** → transport + crypto handshake together: **1 RTT** for new connections, **0-RTT** for resumed ones.
- **Connection ID** (not the 4-tuple) identifies a connection → **connection migration**: switch from Wi-Fi to 4G without breaking the connection.
- Always encrypted (even most headers) → middleboxes can't ossify it.
- Runs in **user space** → faster to upgrade congestion control (CUBIC, BBR) without OS updates.
- Downsides: more CPU than kernel TCP, some networks block/throttle UDP (then browsers fall back to HTTP/2 over TCP).

| | TCP + TLS 1.3 | QUIC |
|---|---|---|
| Handshake to first byte | 1 RTT (TCP) + 1 RTT (TLS) = 2 RTT | **1 RTT** (0-RTT on resume) |
| HOL blocking | Yes (whole connection) | Per stream only |
| Encryption | Optional (TLS on top) | Mandatory, built in |
| Connection identity | 4-tuple (breaks on IP change) | Connection ID (survives IP change) |
| Implementation | OS kernel | User space library |

---

## 8. Application Layer

**Simple explanation:** The layer your code lives in. Protocols here define the **meaning** of messages: "GET this page", "what is the IP of google.com", "deliver this email".

### 8.1 DNS — Domain Name System ⭐⭐

**Simple explanation:** DNS is the **phonebook of the Internet**. Humans remember `google.com`; computers need `142.250.x.x`. DNS translates names to IPs. It is a **distributed, hierarchical, cached** database — no single server knows everything.

**Hierarchy**
```
                    . (root)         13 named root server sets (a–m), hundreds of anycast copies
                  /    |    \
               .com   .org   .in      TLD servers (Top-Level Domain)
               /
          google.com                 Authoritative servers (ns1.google.com ...) — the real answer
           /
      www.google.com
```

**Step-by-step resolution of `www.google.com` ⭐**
1. **Browser DNS cache** — seen recently? Use it.
2. **OS cache / stub resolver** — also checks the **`/etc/hosts`** file.
3. Ask the **recursive resolver** (configured by DHCP: your ISP, or 8.8.8.8 / 1.1.1.1). Has it cached? Return.
4. Resolver asks a **root server**: "where is www.google.com?" → root: "I don't know, ask the **.com TLD** servers at these IPs."
5. Resolver asks **.com TLD**: → "ask Google's **authoritative** servers `ns1.google.com` ..."
6. Resolver asks **Google's authoritative server** → "**A 142.250.x.x, TTL 300**".
7. Resolver **caches** it (for TTL seconds) and returns it to the OS → browser.

```
Browser → OS → Recursive resolver ──1──▶ Root        ("ask .com")
                       │         ──2──▶ .com TLD     ("ask ns1.google.com")
                       │         ──3──▶ Authoritative ("142.250.x.x, TTL 300")
                       ◀── answer, cached by TTL
```

- **Recursive query:** client → resolver: "get me the final answer." (Resolver does all the work.)
- **Iterative query:** resolver → root/TLD/auth: "give me the answer or a referral." (Each says "ask them".)
- Transport: **UDP port 53** (small, fast, 1 round trip). **TCP 53** when the response is too big (truncated flag) and for **zone transfers** (AXFR) between DNS servers.
- **DoH / DoT** = DNS over HTTPS / TLS — encrypts DNS so the ISP can't read/modify it.

**Record types ⭐**
| Record | Maps | Example |
|---|---|---|
| **A** | Name → IPv4 | `google.com → 142.250.x.x` |
| **AAAA** | Name → IPv6 | `google.com → 2404:6800:...` |
| **CNAME** | Alias → another name | `www.shop.com → shop.cdn.net` (can't be at zone apex) |
| **MX** | Mail server for domain (with priority) | `gmail.com → 10 alt1.gmail-smtp-in...` |
| **NS** | Which servers are authoritative for the zone | `google.com → ns1.google.com` |
| **TXT** | Free text — SPF, DKIM, domain verification | `"v=spf1 include:..."` |
| **SOA** | Zone admin info, serial, refresh timers, negative-cache TTL | |
| **PTR** | IP → name (**reverse DNS**) | `8.8.8.8 → dns.google` |
| **SRV** | Service host + port | `_sip._tcp.example.com` |
| **CAA** | Which CAs may issue certs | |

**Caching and TTL ⭐**
- Every answer carries a **TTL (seconds)**. Caches (browser, OS, resolver) keep it that long.
- **Low TTL** (30–60 s): fast failover/changes, more DNS traffic and latency.
- **High TTL** (hours): fewer lookups, but changes are slow to spread ("DNS propagation").
- **Negative caching:** "name does not exist" (NXDOMAIN) is also cached (TTL from SOA).
- Some clients/resolvers **ignore or stretch TTLs**, and long-lived connections never re-resolve at all — DNS failover is never instant.

**DNS for load balancing / failover:** return multiple A records (round-robin DNS), **GeoDNS** (nearest region), health-checked failover (Route 53). Limit: you are at the mercy of caches.

**Project ⭐:** "In LocalArena's multi-region failover I set **DNS TTL = 30 s** and measured that clients behind a caching resolver took **~34 s** to move to the healthy region after a regional failover — TTL plus resolver and client caching. So DNS failover time ≈ TTL + a bit, and lowering TTL is the knob, at the cost of more DNS queries."

**Trap:** "Is DNS TCP or UDP?" → **Both.** Mostly UDP; TCP for large answers and zone transfers.

### 8.2 HTTP basics

**Simple explanation:** HTTP is a **request–response**, **stateless**, text-based (until HTTP/2) protocol. Client sends a request (method + path + headers + optional body), server sends a response (status + headers + body). "Stateless" = each request stands alone; the server remembers nothing unless you add cookies/tokens.

**Raw request and response**
```
GET /search?q=tcp HTTP/1.1          ← request line: method, path, version
Host: www.google.com                ← mandatory in HTTP/1.1
User-Agent: curl/8.4.0
Accept: text/html
Accept-Encoding: gzip, br
Connection: keep-alive
                                    ← blank line ends headers

HTTP/1.1 200 OK                     ← status line
Content-Type: text/html; charset=UTF-8
Content-Length: 5123
Cache-Control: private, max-age=0
Set-Cookie: SID=abc; Secure; HttpOnly; SameSite=Lax

<html>...                           ← body
```

### 8.3 HTTP methods ⭐

| Method | Purpose | Safe? (no change) | Idempotent? (repeat = same effect) | Body? |
|---|---|---|---|---|
| GET | Read a resource | ✔ | ✔ | No (by convention) |
| HEAD | Like GET, headers only | ✔ | ✔ | No |
| OPTIONS | What is allowed? (CORS preflight) | ✔ | ✔ | No |
| POST | Create / process / action | ✘ | **✘** | Yes |
| PUT | Replace whole resource (at known URL) | ✘ | ✔ | Yes |
| PATCH | Partial update | ✘ | ✘ (not guaranteed) | Yes |
| DELETE | Delete | ✘ | ✔ | Optional |

- **Safe** = read-only. **Idempotent** = doing it 1 or 10 times leaves the server in the same state (the response may differ, e.g. DELETE second time → 404).
- **Why it matters:** retries. Proxies/clients can **safely retry idempotent** requests on timeout. Retrying a **POST** could charge a card twice.
- **Fix for POST:** client sends an **`Idempotency-Key`** header (a unique ID per logical operation); server stores the key with the result and returns the stored result on a retry.

**Project ⭐:** "LocalArena purchases use **idempotency keys** — with deterministic IDs + dedupe + unique constraints, I replayed **1.45M events with 0 duplicate rewards**." Also good: "Envoy retries are only safe for idempotent routes."

**PUT vs POST trap:** `PUT /users/42` (client picks the ID, replace) is idempotent. `POST /users` (server picks the ID) creates a new user each time.

### 8.4 HTTP status codes ⭐

| Class | Meaning | Must-know codes |
|---|---|---|
| **1xx** | Informational | **101 Switching Protocols** (WebSocket upgrade), 100 Continue |
| **2xx** | Success | **200 OK**, **201 Created**, **202 Accepted** (async), **204 No Content** |
| **3xx** | Redirect | **301 Moved Permanently**, **302 Found** (temporary), **304 Not Modified** (cache still valid), 307/308 (temp/perm, keep method) |
| **4xx** | Client error | **400 Bad Request**, **401 Unauthorized** (not authenticated), **403 Forbidden** (authenticated but not allowed), **404 Not Found**, 405 Method Not Allowed, **409 Conflict**, 412 Precondition Failed, 413 Payload Too Large, **422 Unprocessable Entity**, **429 Too Many Requests** |
| **5xx** | Server error | **500 Internal Server Error**, **502 Bad Gateway** (proxy got a bad reply from upstream), **503 Service Unavailable** (overloaded / down), **504 Gateway Timeout** (upstream too slow) |

**Traps ⭐**
- **401 vs 403:** 401 = "who are you?" (no/invalid credentials). 403 = "I know who you are, but no."
- **502 vs 504:** 502 = upstream answered with garbage / connection refused. 504 = upstream didn't answer in time. Both come from a **proxy/LB** (e.g. Envoy, Nginx).
- **301 vs 302:** 301 is cached by browsers (permanent; SEO moves). 302 is temporary.

### 8.5 Important headers

| Header | Direction | Purpose |
|---|---|---|
| Host | Req | Which site (virtual hosting — many sites on one IP) |
| User-Agent | Req | Client software |
| Accept / Content-Type | Req / both | Format wanted / format sent (`application/json`) |
| Accept-Encoding / Content-Encoding | Req / Resp | Compression (gzip, br) |
| Content-Length / Transfer-Encoding: chunked | Both | Body size / streaming body in chunks |
| Authorization | Req | `Bearer <token>`, `Basic ...` |
| Cookie / Set-Cookie | Req / Resp | Send / set cookies |
| Cache-Control, ETag, Last-Modified, Expires | Resp | Caching (8.8) |
| If-None-Match, If-Modified-Since | Req | Conditional GET → 304 |
| Location | Resp | Redirect target (3xx) or created resource (201) |
| Connection: keep-alive / close; Upgrade | Both | Connection reuse; protocol switch (WebSocket) |
| Origin, Access-Control-Allow-* | Req / Resp | CORS |
| X-Forwarded-For, X-Forwarded-Proto | Req (added by proxy) | Original client IP / scheme behind a proxy |
| Strict-Transport-Security | Resp | HSTS: "only use HTTPS for me" |
| Retry-After | Resp | With 429/503: when to try again |
| Idempotency-Key | Req | Safe POST retries (custom, popular via Stripe) |

**Project:** SolanaPay API uses **`Authorization: Bearer sk_...`** — API keys are stored **hashed with SHA-256** (never plain text) and are revocable.

### 8.6 HTTP/1.0 vs 1.1 vs 2 vs 3 ⭐⭐

| | HTTP/1.0 | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|---|
| Year | 1996 | 1997/99 | 2015 | 2022 |
| Transport | TCP | TCP | TCP (+TLS in practice) | **QUIC over UDP** |
| Connection | **New TCP per request** | **Persistent (keep-alive) by default** | One connection, **many streams** | One QUIC connection, many independent streams |
| Format | Text | Text | **Binary frames** | Binary frames |
| Parallelism | Open many connections | Pipelining (rarely used, HOL) → browsers open ~6 connections/host | **Multiplexing** | Multiplexing without TCP HOL |
| Header compression | No | No | **HPACK** | **QPACK** |
| Host header | Optional | **Mandatory** (virtual hosting) | `:authority` pseudo-header | Same |
| Other | — | Chunked transfer, caching improvements, range requests | Stream priority, server push (now deprecated by browsers) | 0-RTT/1-RTT setup, connection migration |
| HOL blocking | — | HTTP-level | TCP-level only | **None across streams** |

**Keep-alive (persistent connections):** reuse one TCP (+TLS) connection for many requests → saves the handshake RTTs and slow start each time, and avoids piles of TIME_WAIT sockets. In backend code this is a **connection pool** (Go's `http.Client` reuses connections — as long as you read and close the response body).

**Project:** "**gRPC uses HTTP/2** — multiplexed streams on one connection; LocalArena uses gRPC for service-to-service calls."

### 8.7 Cookies, sessions, JWT ⭐

**Simple explanation:** HTTP forgets you after each request. To "stay logged in" we need state.
- **Cookie** = a small token the browser stores and **automatically sends** with every request to that site. (Like a wristband at a concert.)
- **Session** = the server keeps your data (user id, cart) in memory/Redis/DB; the cookie only holds a random **session ID**. (Wristband number → guest list at the door.)
- **JWT** = the server gives you a **signed token** containing your claims (user id, roles, expiry). The server doesn't store it — it just **verifies the signature**. (A signed, stamped ID card.)

**Cookie attributes ⭐**
| Attribute | Meaning |
|---|---|
| `HttpOnly` | JavaScript can't read it → protects from **XSS** stealing it |
| `Secure` | Only sent over HTTPS |
| `SameSite=Strict/Lax/None` | Controls cross-site sending → main **CSRF** defence (Lax is browser default) |
| `Domain`, `Path` | Scope |
| `Expires` / `Max-Age` | Lifetime (none = session cookie, deleted on browser close) |

**JWT structure:** `header.payload.signature` — each part **base64url-encoded** (NOT encrypted — anyone can read the payload!).
```
header  = {"alg":"RS256","typ":"JWT"}
payload = {"sub":"user42","iss":"https://auth...","aud":"api","exp":1735689600,"roles":["player"]}
signature = sign(header + "." + payload, private key)
```
- **HS256** = shared secret (HMAC). **RS256/ES256** = private key signs, anyone with the public key verifies (good for many services).
- Always verify: **signature, `exp`, `iss`, `aud`**, and reject `alg: none`.

| | Server session + cookie | JWT |
|---|---|---|
| State | On server (Redis/DB) | In the token (stateless) |
| Scale | Needs shared session store | Any server verifies alone |
| Revoke / logout | Easy (delete session) | **Hard** — valid until `exp` → use short expiry + refresh tokens / denylist |
| Size | Tiny cookie | Bigger (sent every request) |
| Best for | Classic web apps | APIs, microservices, mobile, SSO |

**OAuth 2.0 vs OIDC:** OAuth 2.0 = **authorization** ("this app may access my photos", access token). **OIDC** = identity layer on top of OAuth = **authentication** ("who is this user", **ID token** as a JWT).

**Project ⭐:** "LocalArena uses **OIDC/JWT via Keycloak**: Keycloak issues the tokens. The benefit of JWT here is that a service can verify the signature itself, without a shared session-store lookup on every request."

**Trap:** "Where do you store JWT in the browser?" → `localStorage` is readable by JS (XSS risk); an **HttpOnly, Secure, SameSite cookie** is safer (then defend CSRF). State the trade-off.

### 8.8 HTTP caching: Cache-Control and ETag ⭐

**Simple explanation:** Don't download the same thing twice. The server tells the client (and CDNs) **how long** a response is fresh, and gives a **fingerprint** (ETag) so the client can ask "has it changed?" cheaply.

| Directive | Meaning |
|---|---|
| `Cache-Control: max-age=3600` | Fresh for 1 hour, use without asking |
| `s-maxage=600` | Freshness for **shared caches** (CDN/proxy) only |
| `public` / `private` | Any cache may store / only the user's browser (personal data) |
| `no-cache` | **May store, but must revalidate** with server before each use (trap: it does NOT mean "don't cache") |
| `no-store` | **Never store** (banking pages, secrets) |
| `immutable` | Never changes (fingerprinted files like `app.3f9a1.js`) |
| `stale-while-revalidate=60` | Serve stale while refreshing in background |

**Validation (conditional requests)**
```
1st response:  200 OK   ETag: "v42"     Last-Modified: Tue, 01 Oct 2024 10:00:00 GMT
Later request: GET /logo.png   If-None-Match: "v42"
Response:      304 Not Modified   (no body → saves bandwidth)
```
- **ETag** = version/fingerprint of the content (strong or weak `W/"..."`). Pairs with `If-None-Match`.
- **Last-Modified** pairs with `If-Modified-Since` (second precision only).
- ETag + `If-Match` is also used for **optimistic concurrency** on writes (412 Precondition Failed if someone else changed it).

**Interview tip:** Static assets: long `max-age` + `immutable` + hashed filenames. HTML/API: `no-cache` + ETag, or short max-age.

### 8.9 HTTPS and TLS 1.3 ⭐⭐

**Simple explanation:** HTTPS = HTTP inside a **TLS** tunnel. TLS gives three things:
1. **Confidentiality** — nobody in between can read it (encryption).
2. **Integrity** — nobody can change it unnoticed (MAC/AEAD).
3. **Authentication** — you are really talking to google.com (certificate).

**Symmetric vs asymmetric crypto ⭐**
| | Symmetric | Asymmetric (public key) |
|---|---|---|
| Keys | **One shared secret key** | **Key pair**: public (share freely) + private (keep secret) |
| Analogy | Same key locks and unlocks a box | Open padlocks anyone can snap shut (public), only you have the key (private) |
| Speed | **Very fast** | ~100–1000× slower |
| Problem | How to share the key safely? | Slow for bulk data |
| Examples | **AES-GCM**, ChaCha20-Poly1305 | **RSA**, **ECDHE** (key exchange), **ECDSA/Ed25519** (signatures) |
| In TLS | Encrypts all the actual data | Key exchange + proving server identity |

**TLS idea in one line:** use **asymmetric crypto to agree on a symmetric key safely**, then use the fast **symmetric key for the data**.

Also know: **Hashing** (SHA-256) = one-way fingerprint, not encryption. **Digital signature** = hash signed with private key; anyone verifies with public key. **MAC/HMAC** = keyed hash for integrity.

**TLS 1.3 handshake (1 RTT) explained simply**
```
Client                                                  Server
  | --- ClientHello ------------------------------------> |
  |     supported versions, cipher suites,                |
  |     random, **key_share** (client's ECDHE public),    |
  |     SNI = "www.google.com", ALPN = [h2, http/1.1]     |
  |                                                       |
  | <-- ServerHello (chosen cipher, server key_share) --- |
  |     ---- from here everything is encrypted ----       |
  | <-- EncryptedExtensions (ALPN = h2) ----------------- |
  | <-- Certificate (server cert chain) ----------------- |
  | <-- CertificateVerify (signature with private key) -- |
  | <-- Finished (MAC over whole handshake) ------------- |
  |                                                       |
  | --- Finished ---------------------------------------> |
  | --- HTTP request (encrypted with session keys) -----> |
```
**In plain words:**
1. **Client:** "Hi. I speak TLS 1.3, these ciphers, here is my half of a key (ECDHE public value). I want www.google.com (SNI)."
2. **Server:** "OK, this cipher. Here is my half of the key." → Now **both compute the same shared secret** (Diffie-Hellman magic: secret never travels on the wire).
3. **Server:** "Here is my certificate, and a signature made with my private key to prove I own it."
4. **Client verifies** the certificate chain up to a trusted root CA, checks the hostname matches, checks the signature. Sends Finished.
5. Both derive **symmetric session keys** → encrypted HTTP starts. **1 RTT** total.

**TLS 1.2 vs 1.3**
| | TLS 1.2 | TLS 1.3 |
|---|---|---|
| Handshake | **2 RTT** | **1 RTT**, **0-RTT** on resumption |
| Key exchange | RSA or (EC)DHE | **Only (EC)DHE** → always **forward secrecy** |
| Ciphers | Many, some weak (CBC, RC4, SHA-1) | Few strong AEAD ciphers only |
| Encrypted cert | No (sent in clear) | Yes |

- **Forward secrecy:** new ephemeral keys per session → even if the server's private key leaks later, past recorded traffic can't be decrypted.
- **0-RTT risk:** early data can be **replayed** → only allow it for idempotent requests.
- **SNI** tells the server which certificate to present (many sites on one IP). **ALPN** picks the app protocol (h2 vs http/1.1) during the handshake.
- **mTLS** = client also presents a certificate (service-to-service auth, service meshes).

### 8.10 Certificates and CAs ⭐

**Simple explanation:** A certificate is like a **passport**: it says "this public key belongs to www.google.com", and it is **signed by a trusted authority (CA)** like a government stamps a passport. Your browser/OS ships with a list of trusted **root CAs**.

**Chain of trust**
```
Root CA (self-signed, pre-installed in OS/browser trust store)
   └── signs → Intermediate CA
                  └── signs → Leaf certificate: CN/SAN = www.google.com, public key, validity dates
```
**Browser checks:**
1. Each signature in the chain is valid, up to a trusted root.
2. **Hostname** matches the **SAN** (Subject Alternative Name) list.
3. Not expired (validity dates).
4. Not revoked (**OCSP** / CRL; OCSP stapling = server attaches fresh proof).
5. Then server proves it owns the private key (CertificateVerify).

- Certificate format: **X.509**. Free automated CA: **Let's Encrypt** (ACME protocol, 90-day certs).
- Self-signed cert → browser warning (no trusted CA vouches for it) — fine for local dev only.
- **TLS termination:** the LB/reverse proxy (e.g. Envoy) holds the cert, decrypts, forwards to backends (plain or re-encrypted).

**Trap:** "Does HTTPS hide the website I visit?" → It hides the **path, headers, body**. The **domain** can still leak via DNS (unless DoH) and SNI (unless ECH), and the **IP** is visible.

### 8.11 Other application protocols (brief)

| Protocol | Port / transport | What it does | One-liner to remember |
|---|---|---|---|
| **FTP** | TCP 21 (control) + 20 (data) | File transfer | **Two connections** (control + data); **active** mode (server connects back to client — breaks with NAT) vs **passive** mode (client connects to server); plain text → use **SFTP** (over SSH) or FTPS |
| **SMTP** | TCP 25 (server-server), 587 (submission), 465 (TLS) | **Sends/pushes** email | Client → its mail server → recipient's mail server (found via **MX record**) |
| **POP3** | TCP 110 (995 TLS) | **Downloads** email, usually deletes from server | Single device, offline |
| **IMAP** | TCP 143 (993 TLS) | **Syncs** email, stays on server, folders | Multi-device (Gmail app) |
| **DHCP** | UDP 67 (server) / 68 (client) | Gives a device IP, mask, gateway, DNS automatically | **DORA**: Discover (broadcast) → Offer → Request → Ack; leases expire and renew |
| **SSH** | TCP 22 | Encrypted remote shell, file copy (scp/sftp), **port forwarding/tunnels** | Key-based auth (public key on server); replaces insecure **Telnet** (23) |
| Telnet | TCP 23 | Remote shell, **plain text** | Never on the Internet |
| NTP | UDP 123 | Clock sync | TLS certs and JWT `exp` break if clocks drift |
| SNMP | UDP 161 | Monitor network devices | |

**DHCP DORA detail:** the client has no IP yet → it sends Discover from `0.0.0.0` to `255.255.255.255` (broadcast). In big networks a **DHCP relay** forwards it across routers.

**Email flow ⭐ (asked sometimes):** Alice (Outlook) —**SMTP**→ Alice's mail server —**SMTP** (lookup Bob's **MX**)→ Bob's mail server ←**IMAP/POP3**— Bob's app.

---

## 9. Real-Time Communication and APIs

**The problem:** HTTP is "client asks, server answers". How does the **server push** a new chat message to you the moment it arrives?

### 9.1 Short polling vs long polling vs SSE vs WebSocket ⭐⭐

**Simple explanation (analogy):**
- **Short polling** = a child asking "are we there yet?" every 2 seconds.
- **Long polling** = "Tell me when we're there" — the parent stays silent until there is news, then answers; the child asks again immediately.
- **SSE** = a radio broadcast: one-way stream from server to you.
- **WebSocket** = a phone call: both talk any time, line stays open.

| | Short polling | Long polling | SSE (Server-Sent Events) | WebSocket |
|---|---|---|---|---|
| Direction | Client pull | Client pull (server holds) | **Server → client only** | **Full duplex** |
| Connection | New request each time | Request held open until data / timeout | One long HTTP response (`text/event-stream`) | One upgraded TCP connection |
| Latency | Up to poll interval | Low | Low | Lowest |
| Overhead | High (headers every poll) | Medium | Low | Very low (2–14 B frame header) |
| Auto-reconnect | n/a | Manual | **Built in** (`EventSource`, `Last-Event-ID`) | Manual |
| Binary | — | — | Text only | Text + binary |
| Through proxies | Easy | Easy | Easy (plain HTTP) | Needs Upgrade support |
| Use case | Simple dashboards | Fallback, older systems | Notifications, live scores, **LLM token streaming** | Chat, multiplayer games, collaborative editing |

**Project ⭐:** "LLM APIs usually stream tokens via **SSE** because data flows one way. My chat (TTL Chat) and game (LocalArena) use **WebSockets** because the client also sends frequently."

### 9.2 WebSocket handshake and frames ⭐

**Simple explanation:** A WebSocket starts life as a normal HTTP/1.1 request that says "let's switch to WebSocket". The server agrees with **101 Switching Protocols**, and from then on the same TCP connection carries WebSocket **frames** in both directions.

**Handshake**
```
GET /ws HTTP/1.1
Host: chat.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==        ← random 16 bytes, base64
Sec-WebSocket-Version: 13
Origin: https://chat.example.com                    ← CHECK THIS (CSWSH)
Cookie: session=...

HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  ← base64(SHA1(key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"))
```
`Sec-WebSocket-Accept` proves the server really speaks WebSocket (not a confused HTTP cache). It is **not** security.

**Frame format (simplified)**
```
| FIN(1) | RSV(3) | Opcode(4) | MASK(1) | Payload len(7, or 7+16, or 7+64) | Masking key(0 or 4 B) | Payload |
```
| Opcode | Meaning |
|---|---|
| 0x0 | Continuation (fragmented message) |
| 0x1 | **Text** (UTF-8) |
| 0x2 | **Binary** |
| 0x8 | **Close** (with status code, e.g. 1000 normal, 1001 going away) |
| 0x9 / 0xA | **Ping / Pong** (heartbeat, keep NAT/LB idle timers alive, detect dead peers) |

- **Client → server frames must be masked** (XOR with a 4-byte key) — protects proxies from cache-poisoning attacks. Server → client frames are not masked.
- `ws://` = plain (port 80), `wss://` = over TLS (port 443). **Always use wss in production.**
- WebSockets are **stateful, long-lived** connections → scaling is harder: sticky routing or a pub/sub backplane (e.g. Redis) so a message reaching server A can be delivered to a user connected to server B; LB idle timeouts need ping/pong.

**CSWSH — Cross-Site WebSocket Hijacking ⭐**
- **CORS does not protect WebSockets.** A malicious site `evil.com` can open `new WebSocket("wss://chat.example.com/ws")`, and the browser **sends your cookies**. If the server only checks the cookie, evil.com now has your logged-in chat.
- **Fix:** server **checks the `Origin` header** against an allowlist during the handshake (and/or use a token in the first message instead of cookies, plus SameSite cookies).

**Project ⭐:** "In TTL Chat I check `Origin` during the upgrade to stop CSWSH. Each connection has **2 goroutines**: a **reader** (runs handlers) and a **writer** — the **only goroutine allowed to write** to the socket, because gorilla/websocket supports one concurrent writer." (Code: [12.5](#125-websocket-echo-server-in-go-gorilla-with-origin-check).)

### 9.3 WebRTC, STUN, TURN, ICE ⭐

**Simple explanation:** WebRTC lets two browsers send **audio/video/data directly to each other** (peer-to-peer), with low latency over UDP. The hard part is that both are behind NATs — they don't know their own public address and can't accept incoming connections. STUN/TURN/ICE solve that.

| Piece | Role | Analogy |
|---|---|---|
| **Signaling** | Exchange **SDP offer/answer** (codecs, keys) and **ICE candidates**. WebRTC does **not** define it — you build it (WebSocket is common). | Two people exchanging phone numbers through a friend |
| **STUN** server | Tells you your **public IP:port** as seen from outside the NAT ("server-reflexive" address). Cheap, no media passes through. | Asking someone outside "what's my address from where you stand?" |
| **TURN** server | **Relays** all media when a direct path is impossible (symmetric NAT, strict firewall). Costs bandwidth. | A friend who carries every letter between you |
| **ICE** | Collects all **candidates** (host/local, server-reflexive via STUN, relay via TURN), runs **connectivity checks** on pairs, picks the best that works (direct first, relay last). | Try every road, keep the fastest open one |
| **DTLS-SRTP** | Keys exchanged with DTLS (TLS for UDP); media encrypted with **SRTP**. Always encrypted. | |
| **Data channel** | Arbitrary data over SCTP over DTLS (can be reliable or unreliable) | |

**Flow**
```
Peer A                     Signaling server (WebSocket)            Peer B
  |-- offer (SDP) ------------------> | --------------------------> |
  |<--------------------------------- | <------------- answer (SDP) |
  |<== ICE candidates exchanged both ways via signaling ==========>|
  |  ICE checks: try host → STUN (reflexive) → TURN (relay)         |
  |<================ media: SRTP over UDP (direct or via TURN) ====>|
```

**Project ⭐:** "TTL Chat: signaling over the WebSocket (TCP), media over **UDP/SRTP**, **STUN** to discover public addresses, **TURN** as fallback relay. It's a **mesh** — **up to 8 users**, because each user uploads n−1 streams at ~500 kbps/peer. Beyond that, the standard answer is an **SFU** (server receives one stream per user and forwards)."

### 9.4 REST vs gRPC vs GraphQL ⭐

**Simple explanation:**
- **REST** = a library with labelled shelves. Each resource has a URL; use HTTP verbs to act on it.
- **gRPC** = calling a function on another machine as if it were local, with a strict typed contract (Protobuf).
- **GraphQL** = a waiter who lets you order exactly the dishes and portions you want in one go.

| | REST | gRPC | GraphQL |
|---|---|---|---|
| Transport | HTTP/1.1 or 2 | **HTTP/2** | HTTP (usually POST to one endpoint) |
| Format | JSON (text) | **Protobuf (binary)** — small, fast | JSON |
| Contract | Optional (OpenAPI) | **Required** `.proto` → generated typed clients/servers | Schema (SDL), strongly typed |
| Endpoints | Many (`/users/42`, `/orders`) | Service methods (`GetUser`) | **One** (`/graphql`) |
| Streaming | Not native (use SSE/WS) | **Unary, server-stream, client-stream, bidirectional** | Subscriptions (usually over WS) |
| HTTP caching | **Easy** (GET + Cache-Control, CDNs) | Hard | Hard (POST) |
| Browser support | Native | Needs **gRPC-Web** + proxy | Native |
| Over/under-fetching | Common | Fixed messages | **Solved** — client picks fields |
| Best for | Public APIs, CRUD, simple integration | **Internal service-to-service**, low latency, polyglot microservices | Frontends with varied data needs (mobile + web), aggregating many services |
| Pitfalls | Chatty (N calls), versioning | Debugging binary, LB needs L7 (HTTP/2 long-lived connections) | N+1 queries on server, complex rate limiting / caching |

**gRPC must-knows:** **deadlines** propagate across calls (always set them), status codes (`DEADLINE_EXCEEDED`, `UNAVAILABLE`...), Protobuf field numbers give backward compatibility.
**Trap:** gRPC over one long-lived HTTP/2 connection — an **L4** balancer pins all requests to one backend. You need **L7** (per-request) load balancing, e.g. Envoy, or client-side LB.

**Project:** "LocalArena uses gRPC between services and WebSockets for game clients. An L7 proxy like Envoy understands HTTP/2 + gRPC, so it can balance and retry per request."

---

## 10. Backend Infrastructure

### 10.1 Load balancers: L4 vs L7 ⭐⭐

**Simple explanation:** A load balancer is a **receptionist** that spreads incoming customers across many counters (servers) so no counter is overloaded and a closed counter is skipped (health checks).

| | L4 load balancer | L7 load balancer |
|---|---|---|
| Looks at | IP + port (TCP/UDP) | HTTP method, path, headers, cookies, gRPC method |
| Decision granularity | Per **connection** | Per **request** |
| TLS | Passes through (usually) | **Terminates TLS** (must read HTTP) |
| Features | Fast, simple, cheap, any protocol | Path/host routing (`/api` → A, `/img` → B), **retries, timeouts**, header rewrite, auth, rate limits, canary by header, compression, caching |
| Cost | Very low latency | More CPU (parsing) |
| Examples | AWS NLB, LVS, HAProxy (TCP mode), Maglev | **Envoy**, Nginx, AWS ALB, HAProxy (HTTP mode), Traefik |

**Project ⭐:** "Envoy in LocalArena is **L7** — route timeouts, retries, health checks. API through Envoy: **p99 8 ms at 300 req/s**. And during deploys (rolling, canary, blue-green at 600 req/s) **0 of 90,000** requests failed (with HPA scaling 2→5→2). Zero-failure deploys like that depend on the LB concepts below: health checks and connection draining."

**Health checks:** **active** (LB pings `/health` every N s) and **passive** / outlier detection (eject a backend after consecutive 5xx).
**Connection draining:** on deploy, stop sending **new** requests to an instance but let in-flight ones finish.
**Sticky sessions (session affinity):** same client → same server (cookie or IP hash). Needed for in-memory state / WebSockets; hurts even balancing. Better: keep servers stateless, store state in Redis.

### 10.2 Load-balancing algorithms ⭐

| Algorithm | How | Good for | Weakness |
|---|---|---|---|
| **Round robin** | 1, 2, 3, 1, 2, 3... | Identical servers, similar requests | Ignores load |
| Weighted round robin | Bigger servers get more turns | Mixed server sizes, canary (5% weight) | Static |
| **Least connections** | Send to server with fewest active connections | Long/variable requests, WebSockets | Needs tracking |
| Least response time | Lowest latency + fewest connections | Latency-sensitive | Measurement noise |
| Power of two choices (P2C) | Pick 2 random, choose less loaded | Large fleets; Envoy's least-request uses this | — |
| IP hash | hash(client IP) % N | Simple stickiness | N changes → almost everyone remapped |
| **Consistent hashing** | Servers and keys on a hash **ring**; key goes to next server clockwise | Caches, sharding, stickiness that survives scaling | Uneven without virtual nodes |

**Consistent hashing ⭐ (simple):** Put servers on a clock face by hash. A key walks clockwise to the first server. Add/remove a server → only the keys **between it and its neighbour** move (≈ **K/N** keys) instead of almost all keys with `hash % N`. Use **virtual nodes** (each server at many points) for even spread. Used by DynamoDB/Cassandra partitioning, CDN and cache routing, Envoy ring-hash/Maglev.

```
hash % N with N: 3 → 4     → ~75% of keys move (cache stampede)
consistent hashing 3 → 4   → ~25% of keys move (only to the new node)
```

### 10.3 Forward proxy vs reverse proxy ⭐

**Simple explanation:**
- **Forward proxy** works **for clients**. The server sees the proxy, not you. (A friend who buys things on your behalf.) Corporate proxies, VPN-like egress, content filtering, Squid.
- **Reverse proxy** works **for servers**. The client sees the proxy, not the backends. (A company receptionist — you never see which employee handled it.) Nginx, Envoy, Cloudflare.

```
Forward:  [Clients] → [Forward Proxy] → Internet → [Server]
Reverse:  [Client]  → Internet → [Reverse Proxy] → [Server 1, Server 2, ...]
```

| | Forward proxy | Reverse proxy |
|---|---|---|
| Represents | Client | Server |
| Hides | Client identity | Backend topology |
| Configured by | Client / client's network | Server owner |
| Uses | Filtering, caching, egress control, anonymity | **TLS termination, load balancing, caching, compression, rate limiting, WAF, auth** |

**Reverse proxy vs load balancer vs API gateway:** a load balancer is a reverse proxy whose main job is spreading load. An API gateway is a reverse proxy specialised for APIs (auth, quotas, versions). One product (Envoy, Nginx) can play all three roles.

### 10.4 CDN — Content Delivery Network ⭐

**Simple explanation:** Instead of every Indian user fetching images from a server in the US, copies are kept in **edge servers (PoPs)** in Mumbai, Chennai, Delhi. Like a chain of local warehouses instead of one central factory.

- **Pull CDN:** edge fetches from origin on first miss, then caches (most common). **Push CDN:** you upload content to the CDN in advance.
- Routing users to the nearest edge: **Anycast IPs** or **GeoDNS**.
- Controlled by **Cache-Control / s-maxage / ETag**, cache keys (URL + some headers), and **invalidation/purge** (or versioned file names — better).
- Benefits: lower latency, offload origin, absorb **DDoS**, TLS termination at edge, edge compute.
- Also accelerates **dynamic** content: keeps warm, optimised connections from edge to origin.
- Metric: **cache hit ratio**.
- Examples: CloudFront, Akamai, Cloudflare, Fastly.

### 10.5 API gateway

**Simple explanation:** The **single front door** for all your microservices. Clients call the gateway; it handles cross-cutting work so each service doesn't have to.

Responsibilities: **routing** (`/orders` → order-service), **authentication** (verify JWT/API key), **rate limiting / quotas**, request/response transformation, **aggregation** (one client call → several services; "BFF" pattern), TLS termination, logging/metrics/tracing, API versioning, caching.
Examples: AWS API Gateway, Kong, Apigee, Envoy-based gateways.
Risk: single point of failure / bottleneck → run multiple instances behind an L4 LB; don't put business logic in it.

### 10.6 Rate limiting ⭐

**Simple explanation:** Limit how many requests a client can make in a time window — protects the service from abuse, bugs (retry loops), and noisy neighbours. Return **429 Too Many Requests** with **Retry-After**.

| Algorithm | Idea | Bursts? | Notes |
|---|---|---|---|
| **Token bucket** | Bucket holds up to **B** tokens, refilled at **r**/s. Each request takes 1 token; empty → reject | **Allows bursts up to B** | Most popular (AWS, Stripe, Envoy) |
| Leaky bucket | Requests enter a queue, processed at a fixed rate | Smooths to constant rate | Good for shaping output |
| Fixed window counter | Count per minute; reset at :00 | 2× burst at window edges | Simplest |
| Sliding window log | Store timestamps of each request | Exact | Memory heavy |
| Sliding window counter | Weighted mix of current + previous window counts | Approx, smooth | Good balance |

- **Key** by API key, user ID, or IP (IP is weak behind NAT — many users share one IP!).
- **Distributed:** central counter in **Redis** (`INCR` + `EXPIRE`, or a Lua script for token bucket, atomic) or local limits per instance × N.
- **Where:** API gateway / edge proxy (Envoy has local + global rate limiting), and in the app for expensive operations.
- **Client side:** respect 429, **exponential backoff + jitter**, retry budgets.

**Project ⭐:**
- "SolanaPay depends on a devnet RPC limited to **~100 requests / 10 s per IP** — that's a token-bucket-style upstream limit, and it's why a **verification queue** is my next step (smooth bursts instead of hammering the RPC)."
- "In LocalArena I saw a **retry storm: 7,482 retries** during a DB brownout — the 'timeout-ordering' story. Retries without limits multiply load exactly when the system is weakest; rate limits, backoff with jitter and retry budgets are the defence."

(Token-bucket code: [12.8](#128-token-bucket-rate-limiter-go).)

### 10.7 CORS ⭐

**Simple explanation:** By default a browser's **Same-Origin Policy** stops JavaScript on `site-a.com` from reading responses from `api.site-b.com`. **CORS** is how `api.site-b.com` says "it's OK, I allow site-a.com".

- **Origin** = scheme + host + port. `https://a.com` ≠ `http://a.com` ≠ `https://a.com:8443` ≠ `https://api.a.com`.
- **Simple request** (GET/HEAD/POST with simple headers and content types): browser sends it with `Origin`; if the response lacks a matching `Access-Control-Allow-Origin`, **JS can't read the response** (the request still reached the server!).
- **Preflight:** for PUT/DELETE, JSON content type, custom headers like `Authorization` → browser first sends **`OPTIONS`** with `Access-Control-Request-Method/Headers`; server replies with allowed origins/methods/headers (`Access-Control-Max-Age` caches it).
- Cookies cross-origin need `credentials: 'include'` **and** `Access-Control-Allow-Credentials: true` **and** a specific origin (not `*`).

```
OPTIONS /api/orders            Origin: https://shop.com
                               Access-Control-Request-Method: PUT
                               Access-Control-Request-Headers: Authorization, Content-Type
204 No Content                 Access-Control-Allow-Origin: https://shop.com
                               Access-Control-Allow-Methods: GET, PUT
                               Access-Control-Allow-Headers: Authorization, Content-Type
                               Access-Control-Max-Age: 600
```

**Traps ⭐:**
- CORS is enforced **by the browser only**. curl/Postman/servers ignore it. **CORS is not authentication** and does not protect your API from non-browser clients.
- CORS **does not apply to WebSockets** → check `Origin` yourself (**CSWSH**, TTL Chat).
- `Access-Control-Allow-Origin: *` with credentials is not allowed.

### 10.8 Network security basics ⭐

**Firewall:** filters traffic by rules.
| Type | Layer | Decides by |
|---|---|---|
| Packet filter (stateless) | L3/L4 | IP, port, protocol per packet (ACLs) |
| **Stateful** firewall | L4 | Tracks connections — allows replies to outgoing connections automatically (cloud **security groups** are stateful; AWS NACLs are stateless) |
| **WAF** (Web App Firewall) | L7 | HTTP content — blocks SQLi/XSS patterns, bad bots |
| Next-gen firewall | L3–L7 | App awareness, IDS/IPS |

**DDoS (Distributed Denial of Service):** many machines (botnet) flood you.
- **Volumetric** (L3/L4): UDP floods, **DNS/NTP amplification** (small spoofed request → huge reply to victim).
- **Protocol**: **SYN flood** (→ SYN cookies).
- **Application (L7)**: HTTP floods of expensive endpoints, Slowloris (hold connections open slowly).
- Defences: **CDN / Anycast** to absorb, scrubbing services (AWS Shield, Cloudflare), **rate limiting**, WAF, autoscaling, timeouts, caching.

**MITM (Man-in-the-Middle):** attacker sits between you and the server (fake Wi-Fi, ARP spoofing, DNS spoofing) to read/modify traffic.
- Defences: **TLS with proper certificate validation**, **HSTS** (never downgrade to HTTP; preload list), certificate pinning (mobile apps), DNSSEC, mTLS for services. Never skip cert verification in code (`InsecureSkipVerify` = MITM welcome mat).

**XSS (Cross-Site Scripting):** attacker gets **their JavaScript to run in your page** (e.g. a comment containing `<script>`), steals cookies/tokens or acts as the user.
- Types: stored, reflected, DOM-based.
- Defences: **escape/encode output** (React escapes by default), **Content-Security-Policy**, **HttpOnly** cookies, sanitise HTML input.

**CSRF (Cross-Site Request Forgery):** evil.com makes **your browser** send a request to bank.com; the browser attaches your cookies automatically → unwanted transfer.
- Defences: **SameSite cookies** (Lax/Strict), **CSRF tokens** (random token in form/header that evil.com can't read), check `Origin`/`Referer`, require custom headers (forces CORS preflight), use `Authorization` header tokens instead of cookies.

**SQL Injection:** user input becomes part of SQL: `' OR '1'='1`.
- Defence: **parameterized queries / prepared statements** (never string-concatenate SQL), least-privilege DB users, input validation, ORM.

```go
// BAD:  db.Query("SELECT * FROM users WHERE name = '" + name + "'")
// GOOD: db.Query("SELECT * FROM users WHERE name = $1", name)
```

| Attack | Victim trust abused | Key defence |
|---|---|---|
| XSS | User trusts the site's content | Escape output, CSP, HttpOnly |
| CSRF | Site trusts the user's browser (cookies) | SameSite, CSRF token |
| CSWSH | WebSocket server trusts cookies | Check Origin |
| SQLi | DB trusts the app's query string | Parameterized queries |
| MITM | Client trusts the network | TLS + cert validation + HSTS |

---

## 11. What happens when you type google.com and press Enter?

⭐⭐⭐ **The single most-asked networking question.** Interviewers use it to see how deep you can go. Give the **2-minute summary first**, then say: "I can go deeper into any step — DNS, TCP, TLS, or rendering." Let them pick.

### 11.1 The 30-second version (say this first)

> "The browser parses the URL and checks its caches and HSTS, so it will use HTTPS. It resolves google.com to an IP through DNS — browser cache, OS cache, then the recursive resolver which walks root → .com → Google's authoritative servers. It opens a TCP connection with a 3-way handshake (or QUIC for HTTP/3), does a TLS 1.3 handshake to verify Google's certificate and agree on keys, then sends an HTTP GET. The request travels through my home router's NAT, my ISP, and BGP-routed networks to Google's nearest edge, where load balancers send it to a web server that builds the response from backend services. The browser gets the HTML, then fetches CSS/JS/images over the same connection, builds the DOM and CSSOM, lays out and paints the page."

### 11.2 The detailed walkthrough

**Step 0 — Keyboard and URL parsing**
- Keypresses → OS → browser. The address bar may already **autocomplete** and even **pre-connect / prefetch** before you hit Enter.
- Is it a URL or a search term? `google.com` has no scheme → browser assumes a URL.
- Scheme: the browser checks the **HSTS preload list** — google.com is on it → **use `https://` directly** (port 443), never plain HTTP. (Without HSTS, it would try `http://` and get a **301/307 redirect** to https — one wasted round trip and a downgrade risk.)
- URL parts: scheme `https`, host `google.com`, port `443` (default), path `/`.

**Step 1 — Browser checks its own caches**
- **HTTP cache / service worker**: is there a fresh cached copy of the page? (For google.com the HTML is `private, max-age=0`, so no.)
- Is there already an open **connection** to google.com (connection pool)? If yes, skip DNS/TCP/TLS.

**Step 2 — DNS resolution (name → IP)**
1. Browser DNS cache → 2. OS cache + `/etc/hosts` → 3. **Recursive resolver** (ISP or 8.8.8.8, address learned via **DHCP**).
4. Cache miss at the resolver → iterative queries: **root** server ("ask .com") → **.com TLD** ("ask ns1.google.com") → **Google's authoritative** server → "google.com A 142.250.x.x, TTL 300".
5. The query is a **UDP** packet to port 53. Google's DNS often returns an IP for a **nearby** edge (GeoDNS / anycast), so users in India hit Indian PoPs.
6. Browser may ask for **A and AAAA** at once and prefer IPv6 ("Happy Eyeballs" races both). It may also get an **HTTPS/SVCB record** or later an **`Alt-Svc: h3`** header telling it HTTP/3 is available.

**Step 3 — Getting the packet out of your machine (ARP, gateway, NAT)**
- Is 142.250.x.x in my subnet? (IP AND mask) → **No** → send to the **default gateway** (home router).
- Need the router's MAC → **ARP** cache or ARP broadcast "who has 192.168.1.1?".
- Frame: dst MAC = router, dst IP = Google. The router does **NAT/PAT**: replaces private `192.168.1.10:51000` with its public IP and a new port, records it in the NAT table.
- (Even the DNS query in step 2 needed this ARP + NAT path.)

**Step 4 — Routing across the Internet**
- ISP routers forward hop by hop using **longest-prefix match**; between ISPs and Google, routes are learned via **BGP**. Each hop decrements **IP TTL** and rewrites **MAC** addresses; IP addresses stay the same.
- Google announces its IPs via **anycast** from many locations → you reach the **nearest Google Front End (edge)**. Often Google even has servers inside your ISP's network (peering / edge caches).

**Step 5 — TCP 3-way handshake (or QUIC)**
- Browser: `SYN` → Google: `SYN-ACK` → Browser: `ACK`. **1 RTT.** Options agreed: MSS, window scaling, SACK.
- If the browser knows Google supports **HTTP/3**, it uses **QUIC over UDP 443** instead, merging transport + TLS into 1 RTT (0-RTT on resumption).

**Step 6 — TLS 1.3 handshake**
- **ClientHello**: TLS versions, ciphers, ECDHE key share, **SNI = google.com**, **ALPN = h2, http/1.1**.
- **ServerHello** + key share → both derive the shared secret. Then encrypted: **certificate chain**, **CertificateVerify** signature, **Finished**.
- Browser **verifies the certificate**: chain up to a trusted root CA, hostname in SAN, not expired, not revoked.
- ALPN picks **HTTP/2**. **1 RTT** (TLS 1.3). Now all data is encrypted with symmetric session keys (AES-GCM / ChaCha20).

**Step 7 — HTTP request**
```
GET / HTTP/2
:authority: google.com
user-agent: Mozilla/5.0 ...
accept: text/html,...
accept-encoding: gzip, deflate, br
accept-language: en-IN,en;q=0.9
cookie: NID=...; SID=...
```
- In HTTP/2 it is a binary **HEADERS frame** on stream 1, headers compressed with **HPACK**.
- `google.com` responds with **301 → `https://www.google.com/`**, so the browser repeats for `www.google.com` (DNS again; connection may be reused via HTTP/2 connection coalescing because the certificate covers both names).

**Step 8 — Inside Google (server side)**
- **Edge / L4 load balancer** (Google uses **Maglev**, consistent hashing) → **L7 reverse proxy (GFE)** terminates TLS, applies DDoS protection, routes by host/path.
- → **Web/application servers** → many **backend services** called via **RPC (gRPC)**: search, personalisation, ads, spell check... with caches everywhere.
- The response is built, **compressed (gzip/brotli)**, sent with headers: `content-type: text/html`, `cache-control: private, max-age=0`, `set-cookie`, `strict-transport-security`, `alt-svc: h3=":443"`.

**Step 9 — Response travels back**
- Same path in reverse: Google → Internet → your ISP → your router (NAT table maps back to your private IP:port) → your machine.
- TCP: ACKs, flow control (rwnd), congestion control (cwnd grows in slow start), retransmits if anything is lost.

**Step 10 — Browser renders the page**
1. **Parse HTML → DOM** tree.
2. Finds `<link>` CSS, `<script>`, `<img>` → fetches them (multiplexed on the **same HTTP/2 connection**, many from cache or a CDN). Blocking scripts pause parsing (`async`/`defer` avoid this).
3. **Parse CSS → CSSOM**.
4. **DOM + CSSOM → render tree** → **layout** (sizes/positions) → **paint** → **composite** layers on the GPU.
5. **JavaScript** runs (event loop), may make more XHR/fetch calls (e.g. search suggestions).
6. Connection stays open (**keep-alive**) for later requests; idle timeout closes it eventually (FIN → TIME_WAIT on the closer).

### 11.3 Round-trip budget (great to mention)

| Step | Cost (no caching) |
|---|---|
| DNS | 0 RTT (cached) to several RTTs (root, TLD, authoritative) |
| TCP handshake | 1 RTT |
| TLS 1.3 | 1 RTT (TLS 1.2: 2 RTT) |
| HTTP request → first byte | 1 RTT + server time |
| **Total before first byte** | **~3 RTT + DNS** on TCP; **~2 RTT + DNS** with QUIC (or 1 RTT on 0-RTT resume) |

### 11.4 Extra points that impress

- **HSTS** avoids the insecure first request; **HSTS preload** avoids even the first one ever.
- **Connection reuse / keep-alive / HTTP/2 multiplexing** avoid repeated handshakes.
- **CDN edges** serve static assets close to the user.
- **DNS TTL trade-off** — *Project:* "In LocalArena, a DNS TTL of 30 s meant ~34 s for resolver-cached clients to move after a regional failover — so step 2's caching directly controls how fast users escape a dead region."
- **Distance matters** — *Project:* "805 ms remote-region vs 35 ms local write in LocalArena: each RTT in this flow costs real time, which is why we terminate TLS at the edge, reuse connections, and keep services in-region."
- **What can go wrong at each step**: DNS failure (NXDOMAIN, timeout), TCP refused (RST) or timeout (firewall drop), TLS certificate error, HTTP 4xx/5xx, slow page from render-blocking JS. Mapping errors to steps shows real debugging skill.

**Interview tip:** Draw the boxes: **Browser → DNS → TCP → TLS → HTTP → LB → App → DB → back → Render.** Then expand whichever box they point at.

---

## 12. Code Lab

All code is short and runnable. Servers bind to **127.0.0.1** (local only) — a safe default; use `0.0.0.0` only when you really want other machines to connect.

### 12.1 TCP echo server and client (C++, POSIX sockets) ⭐

**Call → network mapping**
| Call | Who | What happens on the wire / in the kernel | TCP state |
|---|---|---|---|
| `socket()` | both | Create endpoint (file descriptor). Nothing sent. | CLOSED |
| `bind()` | server | Attach to IP:port | CLOSED |
| `listen()` | server | Kernel will now answer SYNs | **LISTEN** |
| `connect()` | client | **SYN → SYN-ACK → ACK** (3-way handshake) | SYN_SENT → **ESTABLISHED** |
| `accept()` | server | Takes an **already-established** connection from the queue; returns a new fd | ESTABLISHED |
| `read()/write()` | both | Data segments + ACKs | ESTABLISHED |
| `close()` | both | **FIN / ACK** (4-way teardown) | FIN_WAIT/CLOSE_WAIT → TIME_WAIT/CLOSED |

**`tcp_server.cpp`**
```cpp
// tcp_server.cpp — TCP echo server (POSIX sockets)
// Build: g++ -std=c++17 -O2 tcp_server.cpp -o tcp_server     Run: ./tcp_server
#include <arpa/inet.h>   // htons, inet_ntop
#include <netinet/in.h>  // sockaddr_in
#include <sys/socket.h>  // socket, bind, listen, accept
#include <unistd.h>      // read, write, close
#include <cstdio>
#include <cstring>

int main() {
    // 1. socket(): create an endpoint. AF_INET = IPv4, SOCK_STREAM = TCP.
    //    No packets are sent yet. State: CLOSED.
    int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd < 0) { perror("socket"); return 1; }

    // Let us restart the server at once even if old sockets are in TIME_WAIT.
    int yes = 1;
    setsockopt(listen_fd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof yes);

    // 2. bind(): attach the socket to an IP:port. 127.0.0.1 = only this machine
    //    can connect (safe default). Use INADDR_ANY (0.0.0.0) to accept from the LAN.
    sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9000);                    // host → network byte order
    addr.sin_addr.s_addr = htonl(INADDR_LOOPBACK);  // 127.0.0.1
    if (bind(listen_fd, (sockaddr*)&addr, sizeof addr) < 0) { perror("bind"); return 1; }

    // 3. listen(): mark as passive. State: LISTEN.
    //    From now on the KERNEL answers SYNs with SYN-ACK by itself.
    //    backlog 16 = size of the queue of fully established connections.
    if (listen(listen_fd, 16) < 0) { perror("listen"); return 1; }
    std::printf("Echo server on 127.0.0.1:9000\n");

    while (true) {
        // 4. accept(): take ONE already-finished connection (SYN, SYN-ACK, ACK
        //    are done) from the queue. Returns a NEW socket for this client.
        //    The listening socket keeps listening for others.
        sockaddr_in client{};
        socklen_t len = sizeof client;
        int conn_fd = accept(listen_fd, (sockaddr*)&client, &len);
        if (conn_fd < 0) { perror("accept"); continue; }

        char ip[INET_ADDRSTRLEN];
        inet_ntop(AF_INET, &client.sin_addr, ip, sizeof ip);
        std::printf("Client %s:%d connected (ESTABLISHED)\n", ip, ntohs(client.sin_port));

        // 5. read()/write(): TCP is a BYTE STREAM. One read may return part of a
        //    message or two messages glued together. Echo whatever arrives.
        char buf[1024];
        ssize_t n;
        while ((n = read(conn_fd, buf, sizeof buf)) > 0) {
            write(conn_fd, buf, n);  // send the same bytes back
        }
        // read() == 0 means the client sent FIN (we are now in CLOSE_WAIT).

        // 6. close(): we send our FIN → LAST_ACK → CLOSED.
        //    Forgetting close() = sockets stuck in CLOSE_WAIT (a leak).
        close(conn_fd);
        std::printf("Client disconnected\n");
    }
    // One client at a time to keep it simple. Real servers use threads,
    // epoll/kqueue (event loop) or a thread pool.
}
```

**`tcp_client.cpp`**
```cpp
// tcp_client.cpp — TCP echo client
// Build: g++ -std=c++17 -O2 tcp_client.cpp -o tcp_client     Run: ./tcp_client "hello"
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>
#include <cstdio>
#include <cstring>

int main(int argc, char* argv[]) {
    const char* msg = argc > 1 ? argv[1] : "hello over TCP";

    // 1. socket(): TCP endpoint. The OS will pick an ephemeral source port.
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) { perror("socket"); return 1; }

    sockaddr_in server{};
    server.sin_family = AF_INET;
    server.sin_port = htons(9000);
    inet_pton(AF_INET, "127.0.0.1", &server.sin_addr);  // text IP → binary

    // 2. connect(): THIS runs the 3-way handshake.
    //    Kernel sends SYN (SYN_SENT) → gets SYN-ACK → sends ACK → ESTABLISHED.
    //    Fails with "Connection refused" if nobody listens (server replied RST).
    if (connect(fd, (sockaddr*)&server, sizeof server) < 0) { perror("connect"); return 1; }

    // 3. write(): bytes go into the kernel send buffer; TCP segments, numbers,
    //    retransmits and ACKs them for us.
    write(fd, msg, std::strlen(msg));

    // 4. read(): wait for the echo. (May need a loop for big messages —
    //    TCP has no message boundaries.)
    char buf[1024];
    ssize_t n = read(fd, buf, sizeof buf - 1);
    if (n > 0) { buf[n] = '\0'; std::printf("Echo: %s\n", buf); }

    // 5. close(): we are the active closer → FIN_WAIT_1 → FIN_WAIT_2 → TIME_WAIT.
    close(fd);
}
```

**Try it**
```bash
g++ -std=c++17 -O2 tcp_server.cpp -o tcp_server && ./tcp_server     # terminal 1
g++ -std=c++17 -O2 tcp_client.cpp -o tcp_client && ./tcp_client hi  # terminal 2 → "Echo: hi"
nc 127.0.0.1 9000                        # or type lines interactively with netcat
sudo tcpdump -i lo0 -nn port 9000        # macOS (Linux: -i lo) — watch SYN, SYN-ACK, ACK, FIN
```

**Interview tip:** "`accept()` does **not** do the handshake — the kernel already did it. `accept()` only dequeues a finished connection." Also: "this server handles one client at a time; to scale I'd use a thread per connection, a thread pool, or an **epoll/kqueue event loop** (what Nginx, Node.js and Go's runtime do)."

### 12.2 UDP sender and receiver (C++)

**`udp_receiver.cpp`**
```cpp
// udp_receiver.cpp — UDP receiver
// Build: g++ -std=c++17 -O2 udp_receiver.cpp -o udp_receiver   Run: ./udp_receiver
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>
#include <cstdio>

int main() {
    // SOCK_DGRAM = UDP. No listen(), no accept(), no handshake.
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (fd < 0) { perror("socket"); return 1; }

    sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(9001);
    addr.sin_addr.s_addr = htonl(INADDR_LOOPBACK);
    if (bind(fd, (sockaddr*)&addr, sizeof addr) < 0) { perror("bind"); return 1; }
    std::printf("UDP receiver on 127.0.0.1:9001\n");

    char buf[1500];  // one datagram; 1500 ≈ Ethernet MTU
    while (true) {
        sockaddr_in from{};
        socklen_t len = sizeof from;
        // recvfrom(): returns exactly ONE datagram (message boundaries kept)
        // and tells us who sent it. Lost or reordered datagrams are NOT fixed.
        ssize_t n = recvfrom(fd, buf, sizeof buf - 1, 0, (sockaddr*)&from, &len);
        if (n < 0) { perror("recvfrom"); continue; }
        buf[n] = '\0';
        std::printf("Got %zd bytes from port %d: %s\n", n, ntohs(from.sin_port), buf);
    }
}
```

**`udp_sender.cpp`**
```cpp
// udp_sender.cpp — UDP sender
// Build: g++ -std=c++17 -O2 udp_sender.cpp -o udp_sender     Run: ./udp_sender
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>
#include <cstdio>
#include <string>

int main() {
    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (fd < 0) { perror("socket"); return 1; }

    sockaddr_in to{};
    to.sin_family = AF_INET;
    to.sin_port = htons(9001);
    inet_pton(AF_INET, "127.0.0.1", &to.sin_addr);

    for (int i = 1; i <= 5; i++) {
        std::string msg = "packet #" + std::to_string(i);
        // sendto(): fire and forget. Succeeds even if NOBODY is listening —
        // UDP gives no delivery confirmation (at most an ICMP "port unreachable").
        sendto(fd, msg.data(), msg.size(), 0, (sockaddr*)&to, sizeof to);
    }
    std::printf("Sent 5 datagrams (no ACKs, no guarantee)\n");
    close(fd);
}
```

**Try it:** run `./udp_receiver`, then `./udp_sender`. Now stop the receiver and run the sender again — it **still "succeeds"**. That is UDP: no connection, no delivery guarantee.

| TCP calls | UDP calls |
|---|---|
| socket, bind, listen, accept, read/write, close | socket, bind (receiver), **sendto / recvfrom**, close |
| Stream, need your own message framing | One `sendto` = one datagram = one `recvfrom` |

### 12.3 Simple HTTP server in Go (net/http) + raw requests

**`main.go`**
```go
// Simple HTTP server. Run: go run main.go   then: curl -v http://127.0.0.1:8080/hello
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func hello(w http.ResponseWriter, r *http.Request) {
	// Log the request line and headers the client sent.
	log.Printf("%s %s %s from %s", r.Method, r.URL.Path, r.Proto, r.RemoteAddr)
	for name, values := range r.Header {
		log.Printf("  %s: %v", name, values)
	}

	if r.Method != http.MethodGet {
		w.Header().Set("Allow", "GET")
		http.Error(w, "method not allowed", http.StatusMethodNotAllowed) // 405
		return
	}

	// Response headers must be set BEFORE WriteHeader / the first Write.
	w.Header().Set("Content-Type", "application/json")
	w.Header().Set("Cache-Control", "no-cache")
	w.WriteHeader(http.StatusOK) // 200
	fmt.Fprintf(w, "{\"message\":\"hello\",\"method\":%q}\n", r.Method)
}

func health(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusNoContent) // 204: LB health checks hit this
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/hello", hello)
	mux.HandleFunc("/health", health)

	srv := &http.Server{
		Addr:              "127.0.0.1:8080", // local only
		Handler:           mux,
		ReadHeaderTimeout: 5 * time.Second,  // protects against Slowloris
		ReadTimeout:       10 * time.Second,
		WriteTimeout:      10 * time.Second,
		IdleTimeout:       60 * time.Second, // how long keep-alive connections may idle
	}
	log.Println("listening on http://127.0.0.1:8080")
	log.Fatal(srv.ListenAndServe())
}
```
(Go runs every request in its **own goroutine**, and keeps HTTP/1.1 connections alive by default.)

**`curl -v` shows every layer:**
```
$ curl -v http://127.0.0.1:8080/hello
*   Trying 127.0.0.1:8080...
* Connected to 127.0.0.1 (127.0.0.1) port 8080       ← TCP handshake done
> GET /hello HTTP/1.1                                  ← ">" = request we sent
> Host: 127.0.0.1:8080
> User-Agent: curl/8.4.0
> Accept: */*
>                                                      ← blank line ends headers
< HTTP/1.1 200 OK                                      ← "<" = response received
< Cache-Control: no-cache
< Content-Type: application/json
< Date: Wed, 25 Nov 2026 10:00:00 GMT
< Content-Length: 35
<
{"message":"hello","method":"GET"}
* Connection #0 to host 127.0.0.1 left intact          ← keep-alive: connection reused if possible
```

**Raw HTTP with netcat (you type the protocol yourself):**
```bash
printf 'GET /hello HTTP/1.1\r\nHost: localhost\r\nConnection: close\r\n\r\n' | nc 127.0.0.1 8080
```
Remove the `Host:` line and Go replies `400 Bad Request` — HTTP/1.1 requires Host.

**More curl:**
```bash
curl -I https://www.google.com             # HEAD: headers only
curl -X POST -H 'Content-Type: application/json' -d '{"a":1}' http://127.0.0.1:8080/hello   # → 405
curl -v --http2 https://www.google.com -o /dev/null     # see TLS + ALPN h2 lines
curl -w 'dns=%{time_namelookup} tcp=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total}\n' -o /dev/null -s https://www.google.com
```
The last one prints the time of **each step of Section 11** — DNS, TCP, TLS, first byte. Great to show in an interview.

### 12.4 Python: DNS lookup + raw HTTP GET over a socket

**`dns_http.py`**
```python
# dns_http.py — DNS lookup with getaddrinfo + raw HTTP GET over a TCP socket
# Run: python3 dns_http.py example.com
import socket
import sys

host = sys.argv[1] if len(sys.argv) > 1 else "example.com"

# 1) DNS: ask the OS resolver (it uses its cache, /etc/hosts, then the DNS server).
#    getaddrinfo returns (family, type, proto, canonname, sockaddr) tuples.
print(f"--- DNS lookup for {host} ---")
seen = set()
for family, _, _, _, sockaddr in socket.getaddrinfo(host, 80, proto=socket.IPPROTO_TCP):
    ip = sockaddr[0]
    if ip not in seen:
        seen.add(ip)
        kind = "IPv6 (AAAA)" if family == socket.AF_INET6 else "IPv4 (A)"
        print(f"{kind:12} {ip}")

# 2) Raw HTTP/1.1 GET: open TCP (3-way handshake happens inside create_connection),
#    send the request text ourselves, read the reply until the server closes.
print(f"\n--- Raw HTTP GET http://{host}/ ---")
request = (
    f"GET / HTTP/1.1\r\n"
    f"Host: {host}\r\n"          # mandatory in HTTP/1.1
    f"User-Agent: raw-socket-demo\r\n"
    f"Accept: */*\r\n"
    f"Connection: close\r\n"     # ask server to close after the response
    f"\r\n"                      # blank line = end of headers
)
with socket.create_connection((host, 80), timeout=5) as sock:
    sock.sendall(request.encode())
    chunks = []
    while True:
        data = sock.recv(4096)   # TCP is a byte stream: loop until EOF (FIN)
        if not data:
            break
        chunks.append(data)

response = b"".join(chunks).decode(errors="replace")
headers, _, body = response.partition("\r\n\r\n")
print(headers)                   # status line + headers
print(f"\n[body: {len(body)} characters]")
```

**Real output (`python3 dns_http.py example.com`, trimmed):**
```
--- DNS lookup for example.com ---
IPv6 (AAAA)  2606:4700:90c5:72db:f2a8:4af:ef6b:ff98
IPv4 (A)     172.66.147.243
IPv4 (A)     104.20.23.154

--- Raw HTTP GET http://example.com/ ---
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked          ← body sent in chunks, no Content-Length
Connection: close
Server: cloudflare                  ← a CDN edge answered, not the origin
Age: 14014                          ← seconds this copy has been in the CDN cache
cf-cache-status: HIT                ← CDN cache hit
CF-RAY: ...-DEL                     ← served from the Delhi edge (anycast/GeoDNS)
alt-svc: h3=":443"; ma=86400        ← "I also speak HTTP/3 on UDP 443"
```
One small script shows DNS (multiple A + AAAA records), the HTTP format, CDN caching and the HTTP/3 upgrade hint.

### 12.5 WebSocket echo server in Go (gorilla) with origin check

**Setup:** `go mod init wsdemo && go get github.com/gorilla/websocket`

**`main.go`**
```go
// WebSocket echo server with Origin allowlist (stops Cross-Site WebSocket Hijacking).
// Run: go run main.go
package main

import (
	"log"
	"net/http"

	"github.com/gorilla/websocket"
)

// Only pages from these origins may open a WebSocket to us.
// CORS does NOT protect WebSockets, so we must check Origin ourselves.
var allowedOrigins = map[string]bool{
	"http://localhost:3000":    true, // local frontend
	"https://chat.example.com": true, // production frontend
}

var upgrader = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
	CheckOrigin: func(r *http.Request) bool {
		return allowedOrigins[r.Header.Get("Origin")] // false → gorilla replies 403
	},
}

func wsHandler(w http.ResponseWriter, r *http.Request) {
	// Upgrade validates the handshake (Upgrade, Connection, Sec-WebSocket-Key,
	// Version 13, Origin) and replies "101 Switching Protocols".
	conn, err := upgrader.Upgrade(w, r, nil)
	if err != nil {
		log.Println("upgrade failed:", err)
		return
	}
	defer conn.Close() // close the underlying TCP connection when we return

	conn.SetReadLimit(64 * 1024) // drop clients that send huge frames

	for {
		// ReadMessage returns one full message (text or binary),
		// already un-masked and re-assembled from fragments.
		msgType, data, err := conn.ReadMessage()
		if err != nil {
			log.Println("read end:", err) // close frame, network error, or limit hit
			return
		}
		// Only this goroutine writes, so we respect gorilla's
		// "one concurrent writer per connection" rule.
		if err := conn.WriteMessage(msgType, data); err != nil {
			log.Println("write failed:", err)
			return
		}
	}
}

func main() {
	http.HandleFunc("/ws", wsHandler)
	log.Println("ws://127.0.0.1:8080/ws")
	log.Fatal(http.ListenAndServe("127.0.0.1:8080", nil))
}
```

**Test the handshake and the origin check with curl:**
```bash
# Allowed origin → "HTTP/1.1 101 Switching Protocols" + Sec-WebSocket-Accept
curl -i -N http://127.0.0.1:8080/ws \
  -H 'Connection: Upgrade' -H 'Upgrade: websocket' \
  -H 'Sec-WebSocket-Version: 13' -H 'Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==' \
  -H 'Origin: http://localhost:3000'

# Evil origin → "HTTP/1.1 403 Forbidden"
curl -i http://127.0.0.1:8080/ws \
  -H 'Connection: Upgrade' -H 'Upgrade: websocket' \
  -H 'Sec-WebSocket-Version: 13' -H 'Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==' \
  -H 'Origin: https://evil.com'
```
Browser test (from a page on `http://localhost:3000`): `const ws = new WebSocket("ws://127.0.0.1:8080/ws"); ws.onmessage = e => console.log(e.data); ws.onopen = () => ws.send("hi");`

**Scaling note (TTL Chat pattern):** when *other* goroutines also need to send (e.g. broadcast to a room), don't call `WriteMessage` from many places. Give each connection a **reader goroutine** and **one writer goroutine** that drains a channel — exactly the **2 goroutines per connection** design in TTL Chat. Add ping/pong + read deadlines to detect dead clients.

### 12.6 Sec-WebSocket-Accept in one line (bonus, Python)
```python
import base64, hashlib
key = "dGhlIHNhbXBsZSBub25jZQ=="
print(base64.b64encode(hashlib.sha1((key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11").encode()).digest()).decode())
# → s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### 12.7 Subnet calculator (C++)

**`subnet.cpp`**
```cpp
// subnet.cpp — subnet calculator: IP/CIDR → mask, network, broadcast, host range, count
// Build: g++ -std=c++17 -O2 subnet.cpp -o subnet     Run: ./subnet 192.168.10.77/26
#include <cstdint>
#include <cstdio>
#include <string>

// 32-bit number → "a.b.c.d"
std::string toIp(uint32_t x) {
    return std::to_string(x >> 24) + "." + std::to_string((x >> 16) & 255) + "." +
           std::to_string((x >> 8) & 255) + "." + std::to_string(x & 255);
}

int main(int argc, char* argv[]) {
    const char* input = argc > 1 ? argv[1] : "192.168.10.77/26";
    unsigned a, b, c, d, prefix;
    if (std::sscanf(input, "%u.%u.%u.%u/%u", &a, &b, &c, &d, &prefix) != 5 ||
        a > 255 || b > 255 || c > 255 || d > 255 || prefix > 32) {
        std::printf("Usage: ./subnet a.b.c.d/n\n");
        return 1;
    }
    uint32_t ip = (a << 24) | (b << 16) | (c << 8) | d;

    // Mask = 'prefix' ones followed by zeros. Shifting a 32-bit value by 32 is
    // undefined in C++, so handle /0 separately.
    uint32_t mask = prefix == 0 ? 0 : 0xFFFFFFFFu << (32 - prefix);

    uint32_t network = ip & mask;        // keep network bits, zero host bits
    uint32_t broadcast = network | ~mask; // set all host bits to 1
    uint64_t total = 1ULL << (32 - prefix);

    uint32_t first, last;
    uint64_t usable;
    if (prefix == 32) {        // single host
        first = last = network; usable = 1;
    } else if (prefix == 31) { // point-to-point link: both addresses usable (RFC 3021)
        first = network; last = broadcast; usable = 2;
    } else {
        first = network + 1; last = broadcast - 1; usable = total - 2;
    }

    std::printf("Input        : %s\n", input);
    std::printf("Subnet mask  : %s\n", toIp(mask).c_str());
    std::printf("Network      : %s/%u\n", toIp(network).c_str(), prefix);
    std::printf("Broadcast    : %s\n", toIp(broadcast).c_str());
    std::printf("Host range   : %s - %s\n", toIp(first).c_str(), toIp(last).c_str());
    std::printf("Total addrs  : %llu\n", (unsigned long long)total);
    std::printf("Usable hosts : %llu\n", (unsigned long long)usable);
}
```

**Real output** (matches the worked examples in 6.4):
```
$ ./subnet 192.168.10.77/26
Subnet mask  : 255.255.255.192
Network      : 192.168.10.64/26
Broadcast    : 192.168.10.127
Host range   : 192.168.10.65 - 192.168.10.126
Total addrs  : 64
Usable hosts : 62

$ ./subnet 200.10.20.140/27
Network      : 200.10.20.128/27   Broadcast: 200.10.20.159   Hosts: .129 - .158 (30)

$ ./subnet 172.16.45.14/20
Network      : 172.16.32.0/20     Broadcast: 172.16.47.255   Hosts: 4094
```
**Key idea to say aloud:** `network = ip & mask`, `broadcast = network | ~mask`. Two bit operations.

### 12.8 Token-bucket rate limiter (Go)

**`ratelimit.go`**
```go
// Token bucket: capacity B (max burst), refill r tokens/second.
// Run: go run ratelimit.go
package main

import (
	"fmt"
	"net/http"
	"sync"
	"time"
)

type TokenBucket struct {
	mu       sync.Mutex
	capacity float64   // max tokens = max burst
	tokens   float64   // tokens available now
	rate     float64   // tokens added per second
	last     time.Time // last refill time
}

func NewTokenBucket(capacity, ratePerSec float64) *TokenBucket {
	return &TokenBucket{capacity: capacity, tokens: capacity, rate: ratePerSec, last: time.Now()}
}

// Allow refills lazily (based on elapsed time) and spends 1 token if possible.
// No background goroutine needed.
func (b *TokenBucket) Allow() bool {
	b.mu.Lock()
	defer b.mu.Unlock()

	now := time.Now()
	b.tokens += now.Sub(b.last).Seconds() * b.rate
	if b.tokens > b.capacity {
		b.tokens = b.capacity
	}
	b.last = now

	if b.tokens >= 1 {
		b.tokens--
		return true
	}
	return false
}

// Limiter keeps one bucket per client key (API key, user ID, or IP).
// (Production: evict idle buckets, or keep counters in Redis for many instances.)
type Limiter struct {
	mu       sync.Mutex
	buckets  map[string]*TokenBucket
	capacity float64
	rate     float64
}

func NewLimiter(capacity, rate float64) *Limiter {
	return &Limiter{buckets: map[string]*TokenBucket{}, capacity: capacity, rate: rate}
}

func (l *Limiter) Allow(key string) bool {
	l.mu.Lock()
	b, ok := l.buckets[key]
	if !ok {
		b = NewTokenBucket(l.capacity, l.rate)
		l.buckets[key] = b
	}
	l.mu.Unlock()
	return b.Allow()
}

// Middleware: reject with 429 + Retry-After when the client is over its limit.
func (l *Limiter) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		key := r.Header.Get("X-API-Key")
		if key == "" {
			key = r.RemoteAddr // fallback; weak behind NAT (many users, one IP)
		}
		if !l.Allow(key) {
			w.Header().Set("Retry-After", "1")
			http.Error(w, "too many requests", http.StatusTooManyRequests)
			return
		}
		next.ServeHTTP(w, r)
	})
}

func main() {
	lim := NewLimiter(5, 2) // burst of 5, then 2 requests/second
	for i := 1; i <= 8; i++ {
		fmt.Printf("request %d allowed=%v\n", i, lim.Allow("alice"))
	}
	time.Sleep(1 * time.Second) // ~2 tokens refill
	fmt.Println("after 1s:", lim.Allow("alice"), lim.Allow("alice"), lim.Allow("alice"))
	fmt.Println("bob (own bucket):", lim.Allow("bob"))
}
```
**Expected output:** requests 1–5 `true` (burst), 6–8 `false`, after 1 s: `true true false`, bob: `true`.

**Interview follow-ups:**
- *Distributed?* Store `tokens` and `last` per key in **Redis** and update them in one **Lua script** (atomic), or use `INCR` + `EXPIRE` for a fixed window.
- *Token vs leaky bucket?* Token bucket allows bursts up to capacity; leaky bucket outputs a smooth constant rate.
- *Project:* SolanaPay's upstream devnet RPC allows ~100 req / 10 s per IP → a limiter (plus a verification queue) on our side keeps us under it.

---

## 13. Useful CLI Commands

| Command | What it shows | Example / what to look for |
|---|---|---|
| `ping google.com` | Reachability + **RTT** + packet loss (ICMP echo) | `time=23.4 ms`, `0% packet loss`. No reply ≠ down (ICMP may be blocked) |
| `traceroute google.com` (Windows: `tracert`) | Every **router hop** and its latency (increasing TTL) | `* * *` = hop doesn't reply to probes; a big jump = long link (e.g. undersea cable) |
| `mtr google.com` | ping + traceroute live, loss per hop | Find where loss starts |
| `nslookup google.com` | Simple DNS lookup | Which resolver answered, IPs |
| `dig google.com` | Detailed DNS answer | `ANSWER SECTION` with **TTL**, `Query time`, `SERVER` |
| `dig +short google.com AAAA` | Just the IPv6 addresses | |
| `dig +trace google.com` | Full iterative resolution | Root → .com → google.com authoritative — Section 8.1 live |
| `dig MX gmail.com`, `dig NS google.com`, `dig -x 8.8.8.8` | Mail servers, name servers, reverse DNS (PTR) | |
| `ss -tlnp` (Linux) | **Listening** TCP sockets + process | Is my server really on `127.0.0.1:8080` or `0.0.0.0:8080`? |
| `ss -tan state time-wait \| wc -l` | Count TIME_WAIT sockets | Thousands → add connection pooling |
| `ss -tan state close-wait` | Sockets the app forgot to close | App bug |
| `netstat -an` / `netstat -anv -p tcp` (macOS) | All sockets and states (older tool) | `LISTEN`, `ESTABLISHED`, `TIME_WAIT` |
| `lsof -i :8080` | Which process holds port 8080 | Fix "address already in use" |
| `netstat -rn` / `ip route` | **Routing table** | `default via 192.168.1.1` = gateway |
| `ip addr` / `ifconfig` | Interfaces, IPs, MACs, MTU | `inet 192.168.1.10/24` |
| `arp -a` / `ip neigh` | ARP cache (IP → MAC) | Gateway's MAC |
| `curl -v https://site` | DNS/TCP/TLS/HTTP steps, request + response headers | `* SSL connection using TLSv1.3`, `ALPN: server accepted h2`, `>` request, `<` response |
| `curl -I url` | Response headers only (HEAD) | `cache-control`, `etag`, `server` |
| `curl -w '...%{time_connect}...'` | Timing per phase | See 12.3 |
| `nc -vz host 443` | Is a TCP port open? | `succeeded` vs `Connection refused` (RST) vs timeout (firewall drop) |
| `nc -l 9000` | Quick TCP listener for testing | |
| `sudo tcpdump -i any -nn port 443` | Raw packets on port 443 | `Flags [S]` SYN, `[S.]` SYN-ACK, `[.]` ACK, `[P.]` data, `[F.]` FIN, `[R]` RST |
| `sudo tcpdump -i any -nn 'tcp[tcpflags] & tcp-syn != 0'` | Only SYN packets (new connections) | Watch handshakes |
| `sudo tcpdump -i any -nn udp port 53` | DNS queries/answers live | `A? google.com` |
| `sudo tcpdump -i any -w cap.pcap port 8080` | Save to file → open in **Wireshark** | |
| `openssl s_client -connect google.com:443 -servername google.com` | TLS handshake details, **certificate chain**, protocol, cipher | `Protocol: TLSv1.3`, `Cipher: TLS_AES_256_GCM_SHA384`, `Verify return code: 0 (ok)` |
| `openssl s_client -connect site:443 -servername site </dev/null 2>/dev/null \| openssl x509 -noout -dates -subject -issuer` | Cert expiry, subject, issuer | Debug "certificate expired" |
| `iperf3 -s` / `iperf3 -c host` | Measure throughput between two hosts | |

**tcpdump of a 3-way handshake (what it looks like):**
```
IP 127.0.0.1.51514 > 127.0.0.1.9000: Flags [S],  seq 1234567890, win 65535, options [mss 16344,...]
IP 127.0.0.1.9000 > 127.0.0.1.51514: Flags [S.], seq 987654321, ack 1234567891, win 65535, ...
IP 127.0.0.1.51514 > 127.0.0.1.9000: Flags [.],  ack 987654322, win 6379
```
`[S]` = SYN, `[S.]` = SYN+ACK, `[.]` = ACK. Note **ack = seq + 1**.

**Debugging ladder (say this when asked "the service is unreachable, how do you debug?")** ⭐
1. **DNS:** `dig service.com` — does it resolve to the right IP?
2. **Network path:** `ping` / `traceroute` / `mtr` — reachable? where does it stop?
3. **Port:** `nc -vz ip 443` — refused (nothing listening / RST) or timeout (firewall / security group drop)?
4. **Server side:** `ss -tlnp` — is the app listening, on the right interface (`127.0.0.1` vs `0.0.0.0`)?
5. **TLS:** `openssl s_client` — cert valid, right hostname, not expired?
6. **HTTP:** `curl -v` — status code, headers, redirects; check LB/proxy logs for 502/503/504.
7. **Packets:** `tcpdump` — are SYNs arriving? are we replying? retransmissions?

---

## 14. Most Asked CN Interview Questions (Top 40)

⭐ = top 15, asked in almost every core-CN round at Indian product companies. Answer each in 2–5 lines, then add a project line if you have one.

**1. ⭐ Explain the OSI model.**
7 layers, each with one job: Physical (bits), Data Link (frames, MAC, one hop), Network (packets, IP, routing), Transport (segments, ports, TCP/UDP), Session (sessions), Presentation (encoding, encryption), Application (HTTP, DNS). Each layer serves the one above and uses the one below. The Internet actually runs the 4-layer TCP/IP model, which merges L5–L7 into Application and L1–L2 into Link.

**2. ⭐ TCP vs UDP — when would you use each?**
TCP: connection-oriented, reliable, ordered, flow + congestion control, byte stream — for web, APIs, DBs, file transfer. UDP: connectionless, no guarantees, 8-byte header, keeps message boundaries — for DNS, live video/voice, games, QUIC. *Project:* WebRTC media in TTL Chat is UDP/SRTP (late frames are useless), signaling is over a TCP WebSocket.

**3. ⭐ Explain the TCP 3-way handshake. Why not 2?**
Client SYN (seq=x) → server SYN-ACK (seq=y, ack=x+1) → client ACK (ack=y+1). It confirms both directions work and syncs both random initial sequence numbers. With 2 steps the server can't know its SYN-ACK arrived, and a delayed old SYN could open a fake connection. Costs 1 RTT; SYN floods abuse it (defence: SYN cookies).

**4. ⭐ What happens when you type google.com and press Enter?**
URL parse + HSTS → HTTPS; caches; DNS (browser → OS → resolver → root → .com → authoritative); ARP for gateway + NAT; routing via BGP to Google's nearest edge; TCP handshake (or QUIC); TLS 1.3 with cert verification; HTTP/2 GET; LB → servers → backend RPCs; response; browser builds DOM/CSSOM, layout, paint, fetches sub-resources on the same connection. (Full answer: Section 11.)

**5. ⭐ How does DNS work? What record types do you know?**
Hierarchical, distributed, cached name → IP lookup. Stub resolver asks a recursive resolver, which asks root, then TLD, then the authoritative server iteratively and caches the answer for its TTL. Mostly UDP 53, TCP for big answers/zone transfers. Records: A, AAAA, CNAME, MX, NS, TXT, SOA, PTR, SRV. *Project:* 30 s TTL → ~34 s failover for resolver-cached clients in LocalArena.

**6. ⭐ HTTP vs HTTPS. How does TLS work?**
HTTPS = HTTP over TLS (port 443): confidentiality, integrity, authentication. TLS 1.3: ClientHello with key share + SNI → ServerHello with key share → both derive the same secret (ECDHE) → server sends certificate + signature → client verifies chain to a trusted CA → symmetric keys (AES-GCM) encrypt the data. 1 RTT; asymmetric crypto only to agree on keys and prove identity.

**7. ⭐ Hub vs switch vs router.**
Hub (L1): repeats bits to all ports, one collision domain. Switch (L2): learns MAC → port table, forwards frames only to the right port; each port is its own collision domain, but one broadcast domain. Router (L3): forwards packets between different networks by IP; each interface is its own broadcast domain. "Switch breaks collision domains, router breaks broadcast domains."

**8. ⭐ What is ARP and how does it work?**
Maps an IP to a MAC inside a LAN. Check ARP cache; if missing, broadcast "who has 192.168.1.5?"; the owner unicasts its MAC; cache it. For remote IPs you ARP for the default gateway. It doesn't cross routers; IPv6 uses NDP instead. ARP spoofing enables MITM.

**9. ⭐ IPv4 vs IPv6.**
32-bit vs 128-bit addresses. IPv6: fixed 40-byte header, no header checksum, no router fragmentation, no broadcast (multicast/anycast), SLAAC auto-config, NDP instead of ARP, NAT unnecessary. IPv4 survives on NAT and CIDR. Transition: dual stack, tunnelling, NAT64.

**10. ⭐ What is a subnet mask / CIDR? Find the network and broadcast of 192.168.10.77/26.**
The mask marks which bits are network vs host; /26 = 255.255.255.192. Block size 256−192 = 64 → 77 is in 64–127 → network 192.168.10.64, broadcast .127, hosts .65–.126, 62 usable. CIDR replaced classful addressing so prefixes can be any length (less waste, route aggregation).

**11. ⭐ HTTP/1.1 vs HTTP/2 vs HTTP/3.**
1.1: text, persistent connections, one request at a time per connection (HOL), browsers open ~6 connections. 2: binary frames, multiplexed streams on one TCP connection, HPACK — but TCP-level HOL remains. 3: over QUIC/UDP, independent streams, TLS 1.3 built in, 1-RTT/0-RTT, connection migration. gRPC runs on HTTP/2.

**12. ⭐ Flow control vs congestion control.**
Flow control protects the receiver: sender stays within the advertised receive window (rwnd). Congestion control protects the network: sender keeps its own cwnd — slow start (exponential), congestion avoidance (+1 MSS/RTT), halve on 3 dup ACKs (fast retransmit/recovery), reset to 1 on timeout. Bytes in flight ≤ min(rwnd, cwnd).

**13. ⭐ WebSocket vs long polling vs SSE.**
Long polling: client request held open until data, then re-request. SSE: one-way server → client stream over HTTP with auto-reconnect (notifications, LLM tokens). WebSocket: HTTP Upgrade → 101 → full-duplex frames on one TCP connection (chat, games). *Project:* TTL Chat and LocalArena use WebSockets; in TTL Chat I check `Origin` to prevent CSWSH.

**14. ⭐ L4 vs L7 load balancer.**
L4 routes per connection on IP/port — fast, protocol-agnostic. L7 parses HTTP/gRPC and routes per request on path/headers; does TLS termination, retries, timeouts, health checks, canaries. gRPC on long-lived HTTP/2 needs L7 balancing. *Project:* Envoy (L7) in LocalArena, p99 8 ms at 300 req/s.

**15. ⭐ What is NAT? Why does it cause problems?**
NAT/PAT lets many private hosts share one public IP; the router rewrites source IP:port and keeps a mapping table. It saves IPv4 addresses but breaks end-to-end reachability: outsiders can't initiate connections (need port forwarding). Peer-to-peer needs STUN (find public address) and TURN (relay) — used in TTL Chat's WebRTC.

**16. Explain the 4-way teardown and TIME_WAIT.**
FIN → ACK, then the other side's FIN → ACK, because each direction closes separately (half-close possible). The active closer waits in TIME_WAIT for 2×MSL to re-ACK a lost final FIN and let old packets die. Too many short connections → TIME_WAIT pile-up → port exhaustion → use keep-alive/pooling. Many CLOSE_WAIT = app not calling close().

**17. How does TCP guarantee reliable delivery?**
Sequence numbers (order, dedupe), cumulative ACKs, checksums, retransmission on timeout (adaptive RTO with exponential backoff), fast retransmit on 3 duplicate ACKs, SACK to resend only gaps, plus flow and congestion control.

**18. Explain TCP congestion control phases. What are CUBIC and BBR?**
Slow start doubles cwnd each RTT up to ssthresh; congestion avoidance adds 1 MSS per RTT (AIMD); 3 dup ACKs → ssthresh = cwnd/2, fast recovery; timeout → cwnd = 1, slow start. CUBIC (Linux default) grows cwnd as a cubic function of time since last loss. BBR (Google) models bottleneck bandwidth and min RTT instead of reacting to every loss — better on lossy mobile links.

**19. What is head-of-line blocking?**
A stalled item at the front blocks everything behind it. HTTP/1.1: responses must come in order. HTTP/2 fixes that with streams, but one lost TCP packet still stalls all streams. HTTP/3/QUIC makes streams independent at the transport layer.

**20. What is QUIC and why is it built on UDP?**
A transport protocol (base of HTTP/3) giving TCP-like reliability and congestion control plus built-in TLS 1.3, multiplexed independent streams, 1-RTT/0-RTT setup, and connection IDs for migration across networks. Built on UDP because middleboxes block unknown protocols and kernels are slow to change; QUIC lives in user space.

**21. MAC address vs IP address — why do we need both?**
MAC (48-bit, L2) identifies a NIC on the local link; flat, assigned by vendor. IP (L3) is logical and hierarchical, so routers can aggregate and route by prefix. MACs change at every hop; IPs stay end-to-end (except NAT). Like Aadhaar number (who) vs postal address (where).

**22. What are private IP ranges? What is 127.0.0.1 / 0.0.0.0?**
10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 — not routed on the Internet, need NAT. 127.0.0.1 is loopback (this machine). Binding a server to 0.0.0.0 means "all interfaces" (reachable from the network); 127.0.0.1 means local only.

**23. What is ICMP? How do ping and traceroute work?**
ICMP carries network errors and diagnostics inside IP (no ports). Ping sends Echo Request and measures RTT from Echo Reply. Traceroute sends probes with TTL 1, 2, 3...; each router that drops a probe at TTL 0 replies "Time Exceeded", revealing the path.

**24. Distance vector vs link state routing (RIP, OSPF, BGP).**
Distance vector (RIP, Bellman-Ford): share your table with neighbours periodically; slow convergence, count-to-infinity (fix: split horizon, max 15 hops). Link state (OSPF, Dijkstra): flood your link info to everyone, each router computes shortest paths; fast convergence. BGP is path-vector, runs between autonomous systems over TCP 179, and chooses routes by policy.

**25. Latency vs bandwidth vs throughput.**
Latency: time for data to travel (ms; RTT = round trip). Bandwidth: max capacity of the link (Mbps). Throughput: what you actually achieve, limited by loss, congestion and window/RTT. *Project:* 805 ms remote vs 35 ms local write in LocalArena — a latency problem that more bandwidth would not fix.

**26. HTTP methods — which are idempotent? PUT vs POST?**
GET, HEAD, OPTIONS, PUT, DELETE are idempotent; POST and PATCH are not. PUT replaces a resource at a client-known URL; POST creates/acts and may create duplicates if retried. Make POSTs retry-safe with an Idempotency-Key. *Project:* LocalArena replayed 1.45M events with 0 duplicate rewards using deterministic IDs + dedupe.

**27. Explain status codes: 301 vs 302, 401 vs 403, 502 vs 503 vs 504.**
301 permanent redirect (cached), 302 temporary. 401 not authenticated, 403 authenticated but not allowed. 502 the proxy got an invalid response from upstream; 503 service unavailable/overloaded; 504 the upstream timed out. 429 = rate limited, 304 = cached copy still valid.

**28. Cookies vs sessions vs JWT.**
Cookie: small client-side value auto-sent by the browser. Session: server-side state looked up by a session-ID cookie — easy revoke, needs shared store. JWT: signed, self-contained token (header.payload.signature) verified without a lookup — scales well, hard to revoke, so use short expiry + refresh tokens. *Project:* LocalArena uses OIDC/JWT via Keycloak.

**29. Symmetric vs asymmetric encryption.**
Symmetric: one shared key, fast (AES, ChaCha20), problem is key distribution. Asymmetric: public/private pair, slow (RSA, ECC), solves key exchange and signatures. TLS uses asymmetric (ECDHE + certificate signature) to agree on a symmetric session key, then symmetric for bulk data. Hashing (SHA-256) is one-way, not encryption.

**30. What is a digital certificate and how is it verified?**
An X.509 document binding a domain to a public key, signed by a Certificate Authority. The browser follows the chain leaf → intermediate → root in its trust store, checks signatures, hostname (SAN), validity dates, revocation, and that the server proves ownership of the private key in the handshake.

**31. REST vs gRPC vs GraphQL.**
REST: resources + HTTP verbs, JSON, cacheable, universal — public APIs. gRPC: Protobuf over HTTP/2, typed contracts, streaming, low latency — internal service-to-service (LocalArena). GraphQL: one endpoint, client selects fields — flexible frontends, but harder caching and N+1 risks.

**32. Forward proxy vs reverse proxy.**
Forward proxy acts for clients and hides them (corporate proxy, filtering). Reverse proxy acts for servers and hides backends (Nginx, Envoy): TLS termination, load balancing, caching, compression, rate limiting, WAF.

**33. What is a CDN and how does it work?**
A network of edge servers that cache content near users. Users reach the nearest edge via anycast or GeoDNS; on a miss the edge pulls from origin and caches per Cache-Control/TTL. Lower latency, origin offload, DDoS absorption. Invalidation via purge or versioned filenames.

**34. Load-balancing algorithms. What is consistent hashing?**
Round robin, weighted RR, least connections, least response time, power of two choices, IP hash, consistent hashing. Consistent hashing puts servers and keys on a ring; a key goes to the next server clockwise, so adding/removing a server moves only ~K/N keys (vs nearly all with hash % N). Virtual nodes even out the load.

**35. How would you implement rate limiting?**
Token bucket per client key (burst B, refill r/s), or sliding-window counters; reject with 429 + Retry-After. In a distributed system keep counters in Redis with atomic Lua or INCR+EXPIRE, enforce at the gateway/edge. Clients must back off with jitter. *Project:* the devnet RPC limit (~100 req/10 s per IP) in SolanaPay; the 7,482-retry storm in LocalArena.

**36. What is CORS? Does it protect my API?**
Browsers block JS from reading cross-origin responses unless the server returns Access-Control-Allow-Origin; non-simple requests get an OPTIONS preflight. It is browser-enforced only — not auth, not protection from curl/bots. WebSockets aren't covered → check Origin yourself (CSWSH).

**37. How does DHCP work?**
DORA over UDP 67/68: client broadcasts Discover → server Offers an IP → client Requests it → server Acks. The lease includes IP, subnet mask, default gateway, DNS servers and lease time; the client renews before expiry. DHCP relay forwards broadcasts across routers.

**38. Circuit switching vs packet switching.**
Circuit: a dedicated path is reserved for the whole session (old phone network) — fixed delay, wasted when idle. Packet: data split into packets sharing links on demand, routed independently (Internet) — efficient for bursty traffic, variable delay, may reorder.

**39. Go-Back-N vs Selective Repeat. How does CRC detect errors?**
GBN: sender window N, receiver accepts only in-order, cumulative ACK, resend from the lost frame onward; window ≤ 2^m − 1. SR: receiver buffers out-of-order frames, only the lost one is resent; window ≤ 2^(m−1). CRC: append r zeros, XOR-divide by a generator, send the remainder; receiver's remainder must be 0 (catches all bursts ≤ r bits).

**40. Common attacks: SYN flood, DDoS, MITM, XSS, CSRF, SQL injection — and defences.**
SYN flood fills half-open queue → SYN cookies. DDoS → CDN/anycast, scrubbing, rate limiting, WAF. MITM → TLS with cert validation + HSTS. XSS (injected JS) → output escaping, CSP, HttpOnly cookies. CSRF (forged request with your cookies) → SameSite cookies, CSRF tokens. SQLi → parameterized queries.

---

## 15. Last-Minute Revision (one page)

**Layers:** Physical (bits) · Data Link (frame, MAC, switch) · Network (packet, IP, router) · Transport (segment/datagram, port, TCP/UDP) · Session · Presentation (TLS-ish) · Application (HTTP, DNS). *"All People Seem To Need Data Processing."* MACs change per hop, IPs don't.

**Devices:** Hub L1 (one collision domain) · Switch L2 (collision domain per port, MAC table) · Router L3 (broadcast domain per port, routing table).

**Data link:** ARP = IP→MAC by broadcast, reply unicast, local only. CRC = XOR division (100100 / 1101 → remainder 001). Stop-and-wait (window 1) · GBN (resend from loss, ≤2^m−1) · SR (resend only lost, ≤2^(m−1)). CSMA/CD = wired, detect + backoff · CSMA/CA = Wi-Fi, avoid (RTS/CTS, ACK). VLAN = separate broadcast domain, 802.1Q, 12-bit ID.

**IP:** IPv4 32-bit, header 20 B, TTL = hops, protocol 1/6/17 = ICMP/TCP/UDP. IPv6 128-bit, 40 B header, no broadcast/NAT/ARP. Private: 10/8, 172.16/12, 192.168/16. Hosts = 2^(32−n) − 2. **Block = 256 − mask octet; network = largest multiple ≤ IP; broadcast = next − 1.** /26 → 64, /27 → 32, /28 → 16. Longest prefix match wins. NAT/PAT = many private → one public via ports.

**Routing:** Static vs dynamic. Distance vector (RIP, Bellman-Ford, hops ≤ 15, count-to-infinity) vs link state (OSPF, Dijkstra, areas). BGP = between ASes, policy, TCP 179.

**TCP:** SYN → SYN-ACK → ACK (1 RTT). FIN/ACK ×2 to close; active closer → TIME_WAIT (2×MSL); CLOSE_WAIT pile = app bug. Reliability = seq, ACK, checksum, RTO, 3-dup-ACK fast retransmit, SACK. Flow = rwnd (receiver). Congestion = cwnd: slow start ×2/RTT → +1/RTT (AIMD) → halve on 3 dup ACKs → 1 on timeout. CUBIC (Linux default), BBR (bandwidth + RTT model). Nagle batches small writes → TCP_NODELAY for real-time. Byte stream → frame your messages.

**UDP:** 8 B header, no connection/order/retransmit; DNS, DHCP, VoIP, games, QUIC. *TTL Chat: WebRTC media UDP/SRTP, signaling over WebSocket (TCP).*

**QUIC/HTTP/3:** UDP + TLS 1.3 built in, 1-RTT/0-RTT, independent streams (no TCP HOL), connection migration.

**DNS:** browser → OS (/etc/hosts) → recursive resolver → root → TLD → authoritative; cached by TTL (seconds ≠ IP TTL hops). UDP 53 (TCP for big). A, AAAA, CNAME, MX, NS, TXT, PTR, SOA. *30 s TTL ≈ 34 s failover (LocalArena).*

**HTTP:** stateless request/response. Idempotent: GET PUT DELETE HEAD OPTIONS; not: POST PATCH → Idempotency-Key. 200/201/204 · 301/302/304 · 400/401/403/404/409/429 · 500/502/503/504. 1.0 new conn each · 1.1 keep-alive + Host · 2 binary + multiplex + HPACK · 3 QUIC. Cache-Control: max-age, no-cache (= revalidate), no-store (= never), public/private; ETag + If-None-Match → 304.

**Auth:** Cookie (HttpOnly, Secure, SameSite) · Session (server state, easy revoke) · JWT (signed not encrypted; verify sig/exp/iss/aud; hard to revoke). OAuth = authorization, OIDC = authentication (*Keycloak in LocalArena*).

**TLS 1.3:** ClientHello (key share, SNI, ALPN) → ServerHello + cert + CertificateVerify + Finished → Finished. 1 RTT, ECDHE → forward secrecy. Asymmetric to agree a key, symmetric (AES-GCM) for data. Cert chain → trusted root CA, hostname in SAN, dates, revocation.

**Real-time:** Long polling (held request) · SSE (server→client, auto-reconnect, LLM streaming) · WebSocket (Upgrade → 101, full duplex, opcodes text/binary/close/ping/pong, client frames masked). CORS doesn't cover WS → **check Origin (CSWSH)**. WebRTC: signaling (yours) + ICE (host/STUN/TURN candidates) + DTLS-SRTP. *TTL Chat mesh ≤ 8 users (n−1 uploads).*

**APIs:** REST (resources, JSON, cacheable) · gRPC (Protobuf, HTTP/2, streaming, deadlines; needs L7 LB) · GraphQL (one endpoint, client picks fields).

**Infra:** L4 LB (IP:port, per connection) vs L7 (HTTP, per request, TLS termination, retries) — *Envoy L7, p99 8 ms @ 300 req/s; 0 of 90,000 failed during deploys.* Algorithms: RR, weighted, least-conn, P2C, IP hash, consistent hashing (~K/N keys move, virtual nodes). Forward proxy = for clients; reverse proxy = for servers. CDN = edge cache (anycast/GeoDNS, Cache-Control). API gateway = auth, routing, rate limits. Rate limit = token bucket (burst B, refill r) → 429 + Retry-After; Redis for distributed. *Devnet ~100 req/10 s; 7,482-retry storm.*

**Security:** Firewall (stateless/stateful/WAF). SYN flood → SYN cookies. DDoS → CDN, scrubbing, rate limit. MITM → TLS + HSTS. XSS → escape, CSP, HttpOnly. CSRF → SameSite, tokens. SQLi → parameterized queries.

**google.com in one breath:** URL + HSTS → caches → DNS → ARP gateway + NAT → BGP routing to nearest edge → TCP (or QUIC) → TLS 1.3 → HTTP/2 GET → LB → servers → RPCs → response → DOM + CSSOM → layout → paint. ~3 RTT + DNS before first byte (2 with QUIC).

**Numbers:** MTU 1500 · MSS 1460 · IPv4 hdr 20 B · TCP hdr 20 B · UDP hdr 8 B · IPv6 hdr 40 B · MAC 48 bit · ports 16 bit · ephemeral 49152–65535 · DNS 53 · HTTP 80 · HTTPS 443 · SSH 22 · DHCP 67/68 · BGP 179 · TIME_WAIT 60 s on Linux · speed in fibre ≈ 200,000 km/s.

**Answer pattern:** *what it is → how it works → trade-off → my project.*
