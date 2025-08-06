# Table of Contents

- [A. OSI Model](#a-osi-model)
- [B. HTTP/HTTPS](#b-httphttps)
- [C. DNS](#c-dns)
- [D. TCP/UDP](#d-tcp--udp)
- [E. IP](#e-ip)

# A. OSI Model
Network Layers:
- `Physical Layer` Actual hardware transmission (e.g., `cables`, `switches`)
- `Datalink Layer` Node-to-node communication, MAC addressing (e.g., `Ethernet`, `PPP`)
- `Network Layer` Routing, addressing (e.g., `IP`, `ICMP`)
- `Transport Layer` Reliable delivery, error correction, flow control (e.g., `TCP`, `UDP`)
- `Session Layer` Manages sessions (e.g., opening, maintaining and terminating connections)
- `Presentation Layer` Data format translation, encryption, compression (e.g., `SSL`, `JPEG`, `ASCII`)
- `Application Layer` User-facing interface (e.g., HTTP, SMTP, FTP)
---
# B. HTTP/HTTPS
## 1. HTTP/HTTPS
- `HTTP` is basic protocol for transferring web pages between a browser and a server
	- Send data in `plain text`
	- **Faster** to setup but **insecure**
	- **Port**: 80
- `HTTPS` does everything `HTTPS` does but adds encryption through TLS/SSL
	- Data is **encrypted**
	- **Heavier** due to encryption overhead but **protects privacy**, ensure **data itegrity**
	- Ranking factor for SEO
	- **Port**: 443
---
## 2. HTTP/1.0 - HTTP/1.0 - HTTP/2.0
- `HTTP/1.0`
	- New TCP connection per request
	- **Data format**: Text-based
	- **Performance**: Slow, high overhead
	- Basic HTTP
- `HTTP/1.1`
	- Persistent connections (keep-alive)
	- **Data format**: Text-based
	- **Performance**: Faster than 1.0 but still head-of-line blocking
	- Chunked transfer, caching, pipelining
- `HTTP/2.0`
	- Single connection with multiplexing
	- **Data format**: Binary framing
	- **Performance**: Much faster due to parallelism
	- Headers are compressed
	- Allows server push
---
# C. DNS
- DNS is like the phonebook of the internet.
- Without DNS, you’d have to type IP addresses directly.
## How it works
When typing a URL in browser:
### Brower Cache Check
- Our browser first checks if it already knows the IP for the domain
### OS Cache Check
- If not in the browser, our operating system checks its `local DNS cache`
### Query the DNS Resolver
- Our device sends a query to `recursive DNS Resolver` (usually run but `ISP` or `public resolvers` like Google DNS `8.8.8.8`)
	- **Check the Resolver Cache**: If the resolver already knows the answer, it returns immediately
	- **If not, perform a Recursive Lookup**
		- `Root DNS Server` -> Knows where to find TLDs (`.com`, `.org`, `.net`)
		- `TLD Server` -> Knows where to find the domain's authoritative name server
		- `Authoritative DNS Server` -> Hold actual DNS Records (IP address)
### Return the IP Address
- The resolver sends it back to your browser, which connects to that IP
---
# D. TCP & UDP
## TCP
- `Connection-oriented`, handshake required
- `Reliable`, ensure delivery, retransmits loss packets
- `Error checking` with ack and retramission
- Guarantee packets order
- Packets size: Larger, includes seq/ack numbers
- Slower, More overhead due to control mechanism
## UDP
- `Connectionless`
- `Non-reliable`, no guarantee  of delivery
- `Error checking` with checksum only and no retransmission
- No ordering, package may arrive out of order
- Packets size: Smaller, minimize headers
- Faster, less overhead
---
## TCP Congestion
- Set of algorithms to detect when the network is getting overloaded and adjust the sending rate to avoid collaspe
###  **Main algorithms:**
#### a. Slow Start
-  Begins with small `cwnd` (often 1-15 MSS - Maximum Segment Size)
- For each ACK received, `cwnd` doubles every RTT - Round Trip Time (`exponential growth`)
- Continues until hitting a **slow start threshold** `ssthresh` or packet loss

#### b. Congestion Avoidance
- Once `cwnd` >= `ssthresh`, growth becomes **linear** (addtive increase +1 MSS  per RTT)
- This avoids overshooting network capacity too quickly

#### c. Fast Retransmit
- If the sender gets **3 duplicated ACKs** (same ACK number), it assumes a packet was lost
- Retransmits the missing segment immediately without waiting for timeout

#### d. Fast Recovery
- Instead of going all the way back to the slow start, `cwnd` is reduced (usually halved) and grows linearly again
- Balances recovery speed with avoiding more congestion

### How TCP Detect Packet Loss
- TCP delivers data in order - If packets #5 is missing but #6, #7, #8 arrive, the receiver can't hand #6-#8 to the application yet
- Instead, it sends another ACK for **the last in-order packet it sucessfully got** (#4)
- Every time it gets another out‑of‑order packet, it sends the same ACK again — these are duplicate ACKs.

✅ Duplicate ACKs **don’t** directly mean **loss happened**, but in TCP’s design, **multiple duplicates** are treated as a **loss signal** so the sender can **retransmit quickly** without waiting for a timeout.

# E. IP
## 1. IPv4
- 32-bit addresses (~4,3 billion)
- Dotted-decimal format (`192.168.1.1`)
- Limited space
- Relies on NAT
- Simpler headers
- Mature and widely used
- Support `Unicast`, `Multicast`, `Broadcast` and `Anycast`

## 2. IPv6
- 128-bit addresses (virtual unlimitted)
- Hexadecimal colon-seperated format (`2001:0db8::1`)
- No NAT needed, improve routing efficiency
- Built-in IPsec support
- Simplified header processing
- Designed for future scalablity
- Support `Unicast`, `Multicast`, `Anycast`. Not support `Broadcast`

## 3. IP Communication Types
### 3.1. Unicast
- `One-to-one communication` - a single sender transmit to a single specific receiver
- **Example**: A laptop requests a web page from a server
### 3.2. Broadcast
- `One-to-all communication` - a sender transmits to every device in a network segment
- **Example**

### 3.3. Multicast
- `One-to-many communication` - a sender transmits to a specific group of interested receivers
- **Example**: Streaming  a live video feed to multiple subcribers

### 3.4. Anycast
- `One-to-nearest communication` - multiple devices share the same IP, and the network routes you to the closet one
- **Example**: Global DNS Resolver like `8.8.8.8` (Google DNS)

## Why IPv6 doesn’t use broadcast but uses multicast instead?
- Because it’s **wasteful** — broadcast sends data to every device in a subnet, even those not interested, which wastes bandwidth and processing power.
- Instead, IPv6 uses multicast to target only a specific group of devices that actually need the information.