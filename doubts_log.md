# CNPP — Doubts Log

---

## Lecture 01 / 02 — Intro + Protocol Stacks + Network Devices

---

**Q: What's L2 and L3?**
Just shorthand for Layer 2 and Layer 3 of the TCP/IP stack. An L2 switch = a switch operating at the data link layer. An L3 router = a router operating at the network layer.

---

**Q: What's ARP — isn't it Automatic Repeat something?**
That's ARQ (Automatic Repeat reQuest) — a different thing, that's an error control mechanism in the data link layer. ARP is Address Resolution Protocol. It translates an IP address (L3) into a MAC address (L2) so the data link layer knows the actual hardware address to deliver a frame to locally. This is why ARP is a cross-layer protocol sitting between network and data link.

---

**Q: What's a payload?**
The actual data/content being carried — the "thing that matters." Every layer wraps the data from the layer above in its own header (the envelope). The stuff inside = payload. The header = envelope. The layer below doesn't care what's inside, it just carries it. Each layer's payload includes the headers of all layers above it, so it keeps getting bigger as you go down the stack.

---

**Q: What's a collision domain?**
The group of devices sharing the same "wire" or medium, where simultaneous transmission causes signal collisions. Hubs = all ports in one big collision domain. Switches fix this by giving each port its own dedicated wire — each port is its own collision domain, so simultaneous transmission across ports is physically impossible.

---

**Q: Switches can intelligently route unicast traffic by MAC address — so why do they still forward broadcasts to everyone?**
Two types of traffic: unicast (specific destination MAC) and broadcast (destination MAC = FF:FF:FF:FF:FF:FF, the universal "everyone" address). Switch sees FF:FF:FF:FF:FF:FF and forwards out every port — no filtering. For unicast it looks up the MAC table and sends only to the right port. The switch differentiates purely based on the destination MAC address in the frame header.

---

**Q: How exactly does a switch prevent collisions? What's the physical mechanism?**
Each port on a switch has its own dedicated wire. When A sends to B, the signal travels on A's wire to the switch, which forwards it on B's wire. C and D can transmit simultaneously on their own wires — the signals never share a medium so they can never collide. No shared medium = no collision possible.

---

**Q: Routers "split broadcast domains" — does that mean they actively do something, or do they just not handle broadcasts at all?**
They just don't handle broadcasts at all — it's not active management, it's a wall. Broadcasts are a Layer 2 thing (FF:FF:FF:FF:FF:FF has no IP address to route). When a broadcast hits a router interface, the router sees no routable IP destination and drops it. The consequence is that every network segment separated by a router becomes its own isolated broadcast domain — not because the router manages it, but because broadcasts physically cannot cross it. Broadcasting is an intranet thing by design; inter-network broadcasting would mean your device's local "hey who has this IP" yell reaching millions of machines on the entire internet, which would collapse everything.

---

**Q: If higher-layer devices can do everything lower-layer devices can, why doesn't the router just forward broadcasts anyway?**
It CAN — it's a policy decision, not a capability limitation. By default routers are configured to drop broadcasts because their job is to be the boundary between networks. Some routers DO forward specific broadcasts when explicitly configured to (called DHCP relay / ip helper-address — forwards DHCP broadcasts as unicast across networks). So "routers split broadcast domains" really means "routers are configured to drop broadcasts by default because inter-network broadcasting would break everything" — not that they're physically incapable.

---

## Lecture 02 — Protocol Stacks + Encapsulation + Network Devices

---

**Q: How is an IP address split into network ID and host ID?**
It's NOT always "first three octets = network, last one = host." The split depends on the **address class** or **subnet mask**:

**Classful Addressing:**
- **Class A:** First octet = network, last 3 octets = host (e.g., 10.x.x.x)
- **Class B:** First 2 octets = network, last 2 = host (e.g., 172.16.x.x)
- **Class C:** First 3 octets = network, last 1 = host (e.g., 192.168.1.x)

**Modern networks use CIDR (Classless Inter-Domain Routing):** the split is variable based on subnet mask notation like /24, /16, etc. 
- /24 = first 24 bits network, last 8 bits host
- /16 = first 16 bits network, last 16 bits host

You cannot determine the network/host split just by looking at an IP address without knowing its subnet mask or class.

---

**Q: Collision domain vs broadcast domain - what's the actual difference?**
- **Collision domain:** problem is simultaneous transmission physically destroying both packets (shared medium issue). Devices transmitting at the same time = electrical signals crash, both corrupted, both need retransmission. Wastes bandwidth.
- **Broadcast domain:** problem is unnecessary traffic reaching devices that don't need it (addressing issue). One device broadcasts "HEY EVERYONE" and all devices in the domain must process it even if irrelevant.

**How they're split:**
- Collision domains split by **switches** (each port = own collision domain)
- Broadcast domains split by **routers** (broadcasts can't cross routers)

**Progression:**
- Layer 1 (hub): both problems exist, everything shared
- Layer 2 (switch): collision problem solved (each port isolated), broadcast problem still exists (all ports share broadcast domain)
- Layer 3 (router): broadcast problem solved (each network segment is isolated broadcast domain)

---

## Lecture 03 — Circuit Switching & Packet Switching

---

**Q: Why does circuit switching require both devices to operate at the same rate?**
Circuit switching allocates **fixed bandwidth** for the dedicated path. If sender transmits at 10 Mbps but receiver can only process 5 Mbps, the receiver's buffer overflows and data gets lost. There's no intermediate buffering in pure circuit switching — it's a direct pipe with fixed capacity. Packet switching solves this: intermediate nodes store-and-forward, so a slow receiver just pulls packets from buffers at its own pace. The speed mismatch is handled by the network infrastructure, not forced to match at the endpoints.

---

## Lecture 04 — Layered Services & Protocol Stack Details

---

**Q: If IP is the datagram type of packet switching, shouldn't VCs also be a network layer protocol? Why isn't VC part of the hourglass?**
Great question. VCs DO exist as real protocols — **ATM, Frame Relay, MPLS** are all VC-based technologies. BUT they're not part of the standard TCP/IP internet stack. The internet specifically chose **datagram (IP) as its universal routing layer**, which is why IP is the narrow waist of the hourglass.

VCs exist in practice but operate:
- **Below IP** (like MPLS — used inside ISP networks for traffic engineering, operates at Layer 2.5)
- **In parallel systems** (like old ATM networks, telephone networks using circuit-switched infrastructure)

The hourglass represents the **internet protocol stack specifically**, and the internet's fundamental design choice was datagram routing at the network layer. This choice enables the scalability and flexibility that makes the internet work at global scale (no per-connection state in routers). VCs are real and useful in controlled environments, just not part of TCP/IP's core architecture.

---

**Q: Service interface vs peer-to-peer interface — what's the difference?**
- **Service interface:** VERTICAL communication within the same system between adjacent layers. How a layer requests services from the layer below and provides services to the layer above. Example: application layer requests transport services through the service interface.
- **Peer-to-peer interface:** HORIZONTAL communication between corresponding layers on different systems. Transport layer on sender talks to transport layer on receiver using the peer protocol (TCP header format, sequence numbers, etc).

Vertical = same device, between layers. Horizontal = different devices, same layer talking to same layer.

---

**Q: How does non-blocking ensure the 11th connection goes through if all 10 are active?**
The 10 vs 11 example is about the switch's INTERNAL fabric, not physical ports. A blocking switch cuts corners on internal crosspoints — so even if input/output ports are free, the internal paths can be jammed. Non-blocking means the internal fabric is built with enough redundant paths that a free route always exists regardless of which other connections are active. Classic example: crossbar switch (full N×N matrix). More hardware, more expensive, but guaranteed no internal bottleneck.

---

**Q: Can non-blocking allow an 11th connection if all 10 physical ports are occupied?**
No — non-blocking only refers to internal paths between ports. If all physical ports are occupied that's a hardware limit, non-blocking can't create ports out of thin air.

---

**Q: So you can have idle input/output ports but still be blocked because internal pathways are full?**
Exactly — that's the whole problem with blocking architectures. Free ports on both ends but the internal fabric is a traffic jam. Non-blocking prevents this entirely.

---

**Q: In virtual circuits, routing decisions still happen — but per-circuit not per-packet right?**
Yes. Routing decision happens once at setup time — route is figured out, VCI table entries written at every intermediate node. After that, packets just do a fast table lookup (incoming VCI → outgoing port + new VCI). No routing algorithm per packet. One decision, many packets benefit.

---

**Q: Why doesn't the internet use virtual circuits if they're faster per-packet?**
Internet scale makes VCI tables completely impractical. Billions of devices, millions of simultaneous short-lived connections — every router would need per-connection state, constantly created and torn down. Datagram avoids this — routers just store routing tables by destination IP range, no per-connection state. VCs work in controlled environments (ATM, MPLS, inside telecom provider networks) where connections are manageable. Internet scale: no.

---

**Q: Re-establishment isn't expensive in VCs since VCIs are relative — only the failure point needs fixing?**
Actually it IS expensive. A failed node isn't just a missing table entry — the node is physically down, the entire path through it is broken. New route needed = new VCI table entries at every node along the NEW path from scratch. The locality of VCIs doesn't help because each node only knows its own local piece of the old path.

---

**Q: Is a VC failure like deleting a node in a linked list without saving the address?**
Yes — perfect analogy. Each VC node only stores "received this VCI on this port → send to next port" just like a linked list node only stores a pointer to the next. Middle node goes down = chain broken, no surviving node knows enough to repair it. Slight difference: linked list loses everything from deleted node onwards; VC has source and destination intact but path must be fully rebuilt end to end.

---

## Lecture 06 — Application Layer (DNS deep dive)

---

**Q: Port numbers are 16 bits — is that the same size as part of an IP address?**
No, different things entirely. IP addresses are 32 bits (4 octets). Port numbers are 16 bits (2 octets). Port numbers live at the transport layer to identify applications/processes; IP addresses live at the network layer to identify hosts. Different layers, different sizes, don't conflate them.

---

**Q: Do you always need to type the trailing dot in an FQDN like google.com. ?**
No — the trailing dot is technically always there, it's just silently added by your browser/OS. When you type google.com, your OS internally resolves it as google.com. (with the dot) to signal "this is absolute, start from the root." You never need to type it manually. Without the dot, something like cse.iitkgp.ac.in is technically a PQDN — the local DNS resolver may append a configured suffix to complete it before resolution.

---

**Q: What error do you get if a domain genuinely doesn't exist in DNS?**
NOT a 404 — that's an HTTP error, meaning you successfully reached a server and it told you the page doesn't exist. A nonexistent domain gives you a DNS resolution failure: you never even reach a server. Browser shows something like "Server not found" or NXDOMAIN. DNS failure happens before any HTTP request is made. Two completely separate failure points.

---

**Q: Is gov.in a single TLD or some weird hybrid?**
Neither — it's just plain hierarchy. .in is the ccTLD (India). gov is a second-level domain under .in that mirrors the naming of the generic TLD .gov. India basically recreated the gTLD category structure inside their country domain. So something.gov.in = Indian government entity. No conflict, no hybrid — just .in at the top with gov sitting under it as a second-level label.

---

**Q: What's the difference between a DNS zone and a DNS domain? Everyone seems to use them interchangeably.**
They're not the same. A domain is the full territory — ac.in as a domain includes everything under it: iitkgp.ac.in, iitb.ac.in, all of it, even stuff managed by completely separate servers. A zone is what this specific DNS server is directly responsible for — ac.in's zone covers only ac.in itself, NOT iitkgp.ac.in, because that's been delegated to IIT KGP's own DNS server. Zone = domain minus everything you've delegated out. This is also why authoritative vs non-authoritative answers exist: if you ask the ac.in server about iitkgp.ac.in, the answer is non-authoritative (borrowed from IIT KGP's server). Ask it something inside its own zone, that's authoritative — it owns that data.

Think of it like a manager who's delegated every single task to their team — they're still the manager of the whole department (domain) but their personal to-do list (zone) is basically empty. 

---

**Q: Is a subdomain always its own separate zone?**
No. Whether a subdomain is its own zone depends on delegation. cse.iitkgp.ac.in is a subdomain of iitkgp.ac.in — but if IIT KGP manages all *.iitkgp.ac.in records on one server, cse.iitkgp.ac.in is in the same zone. It only becomes a separate zone if IIT KGP explicitly delegates it to its own DNS server. Subdomain = naming relationship. Zone = administrative boundary. Different things.

---

**Q: What's the TC (truncated) flag in DNS and when does it show up?**
TC is a flag in the DNS header (used by both TCP and UDP). DNS normally runs over UDP because it's faster and lower overhead for small queries. UDP caps DNS responses at 512 bytes — if the response is too big to fit, it gets truncated and the TC flag is set, telling the client "full answer didn't fit, retry over TCP." TCP doesn't have this problem since it handles large payloads natively — you'd just use TCP from the start for known-large responses. TC basically means "UDP wasn't enough, switch."

---

**Q: What exactly is a DNS resource record?**
Just one row in the DNS database. Every record has: name (what domain), type (A = IPv4 address, NS = nameserver, MX = mail server, etc.), class (almost always IN = internet), TTL (how long to cache this), and the actual data value. The entire DNS system is just a massive distributed collection of these records. Nothing more complicated than that.

---
## Lecture 07 — Client-Server & FTP

---

**Q: Is the TCP connection setup a 3-way handshake?**
Yes. TCP uses a 3-way handshake to establish a connection before any data flows: (1) client sends SYN, (2) server responds SYN-ACK, (3) client sends ACK. Only after this is the full socket association (5-tuple) live and the socket ID assigned by the OS.

---

**Q: What exactly is the Berkeley Socket Interface?**
It's the standard API that abstracts socket programming so application code doesn't need to interact with the OS kernel directly. Defines system calls: `socket()`, `bind()`, `connect()`, `listen()`, `accept()`, `send()`, `recv()`. Originally from BSD Unix, now implemented by virtually every OS (Linux natively, Windows via Winsock, macOS). It's the standard — there's no real competing interface for TCP/IP socket programming.

---

## Lecture 08 — HTTP

---

**Q: What actually goes into an HTTP application-layer header?**
HTTP headers are plain-text key-value pairs (`Header-Name: value`) sent alongside requests and responses. They carry metadata the application needs to function — not the actual data being transferred. Categories: General headers (Cache-Control, Connection, Date — apply to both), Request headers (Accept, Accept-Language, Host, User-Agent — sent by client), Response headers (Server, Age, Retry-After — sent by server), Entity headers (Content-Type, Content-Length, Content-Encoding, Last-Modified — describe the body). The body is separate from headers, below the blank separator line.

---

**Q: What's the difference between PUT and PATCH?**
PUT replaces the entire resource at the target URL — whatever was there before is gone, replaced wholesale by what the client sent. PATCH does a partial update — only modifies the specified fields, leaving the rest intact. The lecture only covers PUT. PATCH is a real HTTP method (RFC 5789) but is out of scope for this course.

---