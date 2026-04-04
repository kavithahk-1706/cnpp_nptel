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

**TRAP QUESTION: Is DNS on UDP or TCP?**
Primarily UDP but not exclusively. It switches to TCP when the message is too large for a UDP Datagram size or if it needs to transfer all its data to another server. UDP becomes too much of a risk for that so it switches to TCP since it's reliable.

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

## Lecture 09 — HTTP Proxy, HTML, TELNET

---

**Q: Is the TELNET use case correct — access a remote machine (Bengal) from a different location (Hyderabad) as if physically present?**
Yes, exactly. telnetd runs on the remote machine, the telnet client connects to it over TCP, authenticates with login/password, and from that point every command executes on the remote system. the connection persists until explicitly closed.

---

**Q: Is FTP unidirectional while TELNET is bidirectional?**
Partially correct, but the framing needs to be precise. FTP's DATA CONNECTION is unidirectional per transfer — data flows only one way at a time, either client→server (upload) or server→client (download), never both simultaneously on the same connection. The data connection can also be initiated by either side: server-initiated = active mode, client-initiated = passive mode. FTP also uses two separate TCP connections — port 21 for control, port 20 for data.

TELNET, by contrast, uses a SINGLE TCP connection for both data and control, and that connection is full-duplex — both directions can flow simultaneously. So the right comparison is: FTP data connection = unidirectional per transfer; TELNET connection = full-duplex.

---

**Q: What kind of "terminal" does NVT abstract over — like Git Bash / WSL?**
Yes, exactly that kind. Different systems historically had completely different physical/virtual terminal types (VT100, VT220, xterm, IBM 3270, etc.), each with its own escape sequences and control characters. NVT provides a common intermediate format: local terminal → NVT format → TCP → NVT → server's native terminal format. Both sides maintain an NVT layer and translate to/from it.

---

**Q: Is the bare minimum NVT capability set part of NVT or the TELNET protocol? Where do negotiated options fit?**
The minimum capability set belongs to NVT — all NVT implementations must support it by definition. The negotiated options are deliberately NOT part of the TELNET protocol spec. This separation means new terminal features (new options) can be added without modifying the TELNET protocol itself. Two endpoints negotiate extra options on top of the NVT baseline at connection time.

---

**Q: What is line mode vs character mode in TELNET negotiated options?**
Line mode: client buffers the entire input line locally and sends it in one go when you hit Enter. Character mode: every single keystroke is transmitted immediately to the server. Character mode is more responsive; line mode is more bandwidth-efficient. This is negotiated upfront between endpoints. Echo mode (local vs remote) is related — whether your terminal shows keystrokes locally or waits for the server to echo them back.

---

**Q: IAC escape — how does 255 as data vs 255 as command work?**
Rule: if you see 255 and the next byte is ALSO 255 → it's an escape sequence meaning "this 255 is literal data, pass it to the application." If 255 is followed by anything other than 255 → treat it as a command. Analogous to bit stuffing / character stuffing: same principle of using a special marker to distinguish control info from data that happens to look like control info.

## Lecture 10 — Application Layer: SMTP, MIME, POP3, IMAP, SNMP

---

**Q: What is the UA and MTA in a Gmail context?**
**UA (User Agent)** = the Gmail interface itself — the webpage or app the user interacts with to compose, read, and manage mail. The UA constructs the email's headers and body based on user input.
**MTA (Mail Transfer Agent)** = Google's backend mail servers (e.g. smtp.gmail.com) — the actual delivery machinery the user never sees. The MTA acts as a **client** when initiating a connection to send mail outward, as a **server** when receiving incoming mail on port 25, and as a **relay** when forwarding mail as an intermediary between sender and receiver MTAs — receiving from one (server role) and forwarding to another (client role). Similar in behaviour to an HTTP proxy.

---

**Q: How does BCC work in terms of the email envelope vs header?**
The envelope and header are separate constructs even when their data overlaps. The "To:" field in the **header** is human-readable metadata rendered by the UA for the recipient to see. The **envelope** is what the MTA actually uses to route the mail, and these two can differ. BCC is the classic example: a BCC'd address is present in the **envelope** (so the MTA knows to deliver there) but absent from the **header** (so visible recipients have no idea). The MTA routes purely based on the envelope and never reads the header.

---

**Q: What is the SMTP session flow step by step?**
1. TCP connection established — client connects to server on port 25
2. Server sends **220** — service ready (server announces itself first)
3. Client sends **HELO** with its MTA domain — server responds **250 OK**
4. Client sends **MAIL FROM:\<addr\>** — server responds **250 OK**
5. Client sends **RCPT TO:\<addr\>** — server responds **250 OK**
6. Client sends **DATA** — server responds **354** (positive intermediate: "go ahead, send the body")
7. Client sends headers (From, To, Subject, Date), then a blank line, then the body, terminated by a lone dot on its own line
8. Server responds **250** — mail accepted
9. Client sends **QUIT** — server responds **221** — service closing

Note: **220** is the server's opening greeting. **221** is only at the very end when closing. These are different and should not be confused.

---

**Q: What do SMTP status code ranges mean?**
- **2xx** — success, command accepted and completed
- **3xx** — positive intermediate; command accepted but more input needed (e.g. 354 after DATA)
- **4xx** — temporary failure; try again later. MTA will queue and retry automatically (e.g. mailbox temporarily full)
- **5xx** — permanent failure; do not retry. MTA bounces mail back to sender (e.g. 550 = user doesn't exist)

The key distinction between 4xx and 5xx is **retry logic** — 4xx says come back later, 5xx says don't come back at all.

---

**Q: Can you manually run any protocol session via telnet?**
Yes, but only for **text-based (ASCII-based) protocols** — ones where commands and responses are human-readable plaintext. Telnet provides a raw TCP connection and the server doesn't care whether it's talking to a proper client application or a human typing manually, as long as valid protocol messages are received. Works for SMTP, HTTP/1.1, POP3, IMAP, and FTP's control channel (port 21). Does NOT work for binary or encrypted protocols — HTTPS fails because the TLS handshake is cryptographic binary data that cannot be typed manually. FTP is a mixed case: control channel (port 21) is text-based and telnettable, data channel (port 20) can carry binary file content.

---

**Q: What is MIME and how does it relate to SMTP?**
SMTP natively handles only **7-bit ASCII text**. MIME (Multipurpose Internet Mail Extensions) is an extension built on top of SMTP that enables non-ASCII content — images, audio, video, attachments — by encoding them into 7-bit ASCII for transmission. MIME is not a separate protocol and has no port of its own; it just adds header fields between the email header and the body describing what the body contains and how it is encoded. The analogy to NVT holds: just as NVT standardises between different terminal formats, MIME standardises between ASCII and non-ASCII so binary content can travel over an ASCII-only pipe.

MIME header fields include: MIME-Version, Content-Type, Content-Transfer-Encoding, Content-ID, Content-Description.

**Content-Transfer-Encoding types:**
- **7-bit** — already plain ASCII, no conversion needed
- **8-bit** — uses full 8 bits, still mostly text-ish
- **binary** — raw binary, no conversion
- **base64** — converts binary data into a 64-character ASCII alphabet; every 3 bytes → 4 ASCII characters; bloats size ~33% but safely travels over ASCII-only SMTP. Used for image/audio/video attachments
- **quoted-printable** — for mostly-ASCII text with a few non-ASCII characters; only encodes the problematic characters as =XX (hex value); more efficient than base64 for near-ASCII content

**Content-Type** multipart = email contains multiple distinct parts (e.g. a text body + image attachment + PDF), each with its own Content-Type and encoding. The multipart type bundles them together with boundary markers.

Application-level restrictions (e.g. Gmail blocking .exe attachments or large videos) are the mail server's own policy decisions. MIME just provides the Content-Type label — what the server does with that label is entirely up to the server's configuration.

---

**Q: What ports do POP3 and IMAP run on?**
- **POP3** — port **110**
- **IMAP4** — port **143**
- **SMTP** — port **25** (already in notes)
- **SNMP** — port **161** (request/response), port **162** (traps)

---

**Q: What is the difference between POP3 and IMAP, and which does Gmail use?**
**POP3 (Post Office Protocol):** Simple download model. Pulls all messages fully from server to client. Server is a temporary holding area — once downloaded, mail is typically deleted from server. Designed for single-device use. Cannot partially fetch; must download entire message to read anything. No server-side management.

**IMAP4 (Internet Message Access Protocol):** Server is always the source of truth. Client is just a view that reflects server state. Supports partial fetch — metadata (sender, subject, preview) is fetched first to populate the inbox list; full body and attachments are fetched on demand only when a specific email is opened. Supports multi-device access, server-side folder management, header-only search, and selective sync. Gmail uses IMAP.

**On-demand behaviour in Gmail (IMAP):** Opening Gmail fetches metadata for the most recent emails. Clicking an email fetches its full body. Navigating to older pages, switching folders, or searching triggers fresh server fetches for that specific content. Everything is on demand. Caching is a client-side implementation detail — Gmail caches previously fetched data locally so repeat access is faster; clearing cache forces fresh fetches from server, which is why first load after cache clear takes slightly longer.

---

**Q: What is SNMP and what are its components?**
SNMP (Simple Network Management Protocol) has nothing to do with mail. It manages network devices (routers, switches, printers, hosts) at the application layer. Runs on **UDP** for low overhead. Two core functions: **monitor** (collect info about device health/state) and **control** (act on that info to fix or configure).

**Components:**
- **Agent** — software running on a managed device. Maintains local data about the device's configuration and state. Reports to the manager. Responds to queries and sends traps.
- **Manager** — application (management station) used by network admin. Queries or modifies agent data. Looks up MIBs to know what objects exist before sending requests.
- **MIB (Management Information Base)** — a text file on the agent describing what managed objects exist on that device and their structure, written in ASN.1 syntax. Acts as a schema/dictionary — static reference for the manager to know what questions it can ask. **MIB-2 (RFC 1213)** defines the standard baseline managed objects common to all TCP/IP devices. Vendor/device-specific extensions are defined in additional MIBs on top of this standard base.
- **Managed Objects** — specific trackable/configurable properties of a device (e.g. CPU usage, interface status, packets dropped, bandwidth). Each is assigned a unique **OID (Object Identifier)** — a sequence of integers separated by dots (e.g. 1.3.6.1.2.1...) used by the manager to request specific data from the agent.

---

**Q: What are the SNMP operations and which fall under monitor vs control?**
- **GET request** (manager→agent) — requests value of one or more managed objects → **monitor**
- **GET NEXT request** (manager→agent) — requests next object in lexicographic OID order (useful for walking the MIB tree) → **monitor**
- **GET response** (agent→manager) — response to GET, GET NEXT, or SET requests → **monitor**
- **SET request** (manager→agent) — modifies value of one or more managed objects → **control**
- **Trap** (agent→manager, port 162) — unsolicited alert pushed by agent about a specific event without waiting for a query → **monitor** (it conveys information about device state, not a change)

SET request is the **only control operation**. Everything else is monitor. Traps are the only SNMP messages initiated by the agent without a prior request from the manager. Typical flow: agent fires trap about a critical event → manager evaluates → manager sends SET request to reconfigure the device.

GET/GET NEXT/SET/GET response all use port **161** (request-response). Traps use port **162**.

---

**Q: What does lexicographic OID order mean?**
Comparison happens position by position through the integer sequence. Similar to alphabetical ordering but for numbers. OID 1.3.6.1.2 comes before 1.3.6.1.10 because at the fifth position 2 < 10. GET NEXT uses this ordering to traverse the MIB tree sequentially without needing to know every OID in advance.

---

**Q: What are the SNMP versions?**
- **SNMPv1** — community string (plaintext password) sent with every request. No encryption. Simple but completely insecure.
- **SNMPv2c** — "c" stands for community; kept the insecure community string approach from v1 (security overhaul attempted but too complex, abandoned). Added 64-bit counters for high-speed interfaces (32-bit counters overflow on fast links), better error handling, and additional PDU types. Security problem unresolved.
- **SNMPv3** — full security implemented: **integrity** (packets not tampered with), **authentication** (valid source verified), **privacy/confidentiality** (encryption). Maps to the CIA triad with availability swapped out for authentication — availability is a system-wide property not directly implementable in a protocol, whereas authentication is concretely actionable at the protocol level. Many devices support all three versions simultaneously.

---

## Lecture 11 — Transport Layer 1: Services

---

**Q: When they say data passes into the OS through the transport layer, what does that mean architecturally?**
It means TCP/UDP is where the user space/kernel space boundary is crossed. When an application wants to send data, it makes a system call — that's the literal crossing point. TCP or UDP is the first thing in kernel space that receives it. "Through the transport layer" = through the user↔kernel gate. The physical implementation breakdown: NIC handles physical layer (+ parts of data link in wireless for speed), NIC drivers handle the MAC sublayer, OS kernel handles LLC (upper data link), full IP logic, and both TCP and UDP. Application layer sits in user space and interacts with the user directly.

---

**Q: What does LLC actually handle? I said it was error detection, CRC, parity checks.**
That's wrong — CRC and framing belong to the MAC sublayer or physical layer. LLC's actual job in TCP/IP is **multiplexing** — identifying which network layer protocol (IP, ARP etc.) to hand the frame up to. In practice LLC is almost vestigial in modern TCP/IP ethernet because ethernet handles framing itself. For exam purposes: LLC = upper data link, handles multiplexing and flow control between data link and network layer.

---

**Q: Is there a formal interlayer between application and transport?**
Not in TCP/IP. The one real thing that lives in that gap is **TLS/SSL** — handles encryption and session management, technically sits between application and transport. This is why the OSI model has separate session and presentation layers that TCP/IP just collapses into the application layer. Everything else (HTTP, FTP etc.) directly calls TCP/UDP via sockets.

---

**Q: How does the transport layer detect a packet dropped at the network layer if the drop happens below it?**
Packet dropped at network layer → receiver transport layer sees nothing → no ACK sent → sender timeout expires → sender retransmits. The absence of ACK IS the signal. Additionally, 3 duplicate ACKs (same ACK number repeated) = fast retransmit signal, meaning packets after that point dropped but earlier ones arrived fine. TCP retransmits immediately without waiting for full timeout — this is called **fast retransmit.**

---

**Q: Does a webpage not load at all if a packet drops, or just that specific resource?**
Depends on the HTTP version. HTTP/1.1 over TCP: one dropped packet stalls the entire connection, page hangs. HTTP/2 over TCP: mostly stalls due to head-of-line blocking — one dropped packet stalls the whole connection even for unrelated resources. HTTP/3 over QUIC (UDP-based): each resource stream is independent, one dropped packet only stalls that specific resource, rest of page loads fine. This is exactly why HTTP/3 exists.

---

**Q: Does a video wait for the entire file to reassemble before playing?**
No — that would be unusable. Videos use **adaptive bitrate streaming (ABR)**. The video is pre-chunked server-side into small segments (a few seconds each). The player requests chunk by chunk, starts playing once it has the first few buffered, and keeps fetching ahead while you watch. The loading spinner = not enough chunks buffered ahead to play smoothly, not waiting for the whole file.

---

**Q: Is sliding window purely a reliability mechanism or also a flow control mechanism?**
Both simultaneously. As a reliability mechanism: sender keeps unACKed segments in buffer, retransmits on timeout, uses sequence numbers. As a flow control mechanism: the window SIZE dynamically reflects receiver buffer availability via rwnd advertisement. Stop-and-wait is just sliding window with fixed window size 1 — reliable but inefficient because the pipe sits idle waiting for each ACK. Sliding window keeps the pipe full while maintaining reliability.

---

**Q: How does the sender know the receiver's buffer capacity? What does "advertise" mean?**
Every TCP segment header has a **window size field.** When the receiver sends back an ACK, it fills this field with current free buffer space in bytes. So every ACK does two things simultaneously: "I received everything up to byte N" + "I currently have X bytes of free buffer space." Sender reads X and cannot have more than X bytes unacknowledged in flight. When buffer is full, receiver advertises window = 0, sender stops completely. When receiver processes data and frees buffer, it advertises a non-zero window, sender resumes.

---

**Q: What are the two factors that determine actual sliding window size?**
- **rwnd (receiver window):** advertised by receiver in TCP header alongside ACK. Reflects free buffer space. Determined by OS buffer allocation for the connection and speed at which the application is consuming incoming data.
- **cwnd (congestion window):** maintained by sender based on network conditions. Not directly observed — inferred from dropped packets (no ACK = congestion). Starts at 1 MSS, doubles per RTT in slow start, switches to linear growth (+1 MSS per RTT) after crossing ssthresh, drops on congestion detection.
- **Effective window = min(rwnd, cwnd).** Both must cooperate — even if network is fine, slow receiver pulls window down via rwnd, and even if receiver has plenty of buffer, congested network pulls it down via cwnd.

---

**Q: cwnd doubles per ACK or per RTT?**
Per RTT, not per ACK. In slow start every ACK adds 1 MSS to cwnd, but since you're sending N segments and receiving N ACKs, cwnd effectively doubles per round trip. Per ACK is slightly off and could cost marks in a numerical question.

---

**Q: What exactly does cwnd drop to when congestion is detected?**
Depends on implementation. Two main variants:

| | Timeout (complete silence) | 3 Duplicate ACKs |
|---|---|---|
| **TCP Tahoe** | drop to 1 MSS | drop to 1 MSS |
| **TCP Reno** | drop to 1 MSS | drop to ssthresh (fast recovery) |

Reno is smarter: 3 dup ACKs means some packets are still getting through so network isn't fully collapsed — no need to drop all the way to 1 MSS. Timeout means complete silence, so drop hard. Reno is the more common modern implementation.

---

**Q: Collision vs congestion — same thing?**
Completely different. **Collision** = two devices transmitting simultaneously on the same physical medium, signals physically interfere and destroy each other. Layer 1/2 problem. Fixed by CSMA/CD (wired) and CSMA/CA (wireless). **Congestion** = too much traffic arriving at a router faster than it can forward, buffer fills, packets dropped. Network layer and above. Fixed by congestion control mechanisms in TCP. Collision is a physical medium problem; congestion is a capacity problem.

---
## Lecture 12 — Transport Layer 2: Connection Establishment

---

**Q: What makes delayed duplicate connection requests non-differentiable to the receiver?**
In a naive two-way handshake, there is nothing in the packet itself that encodes when it was created or whether it belongs to a current or previous connection. If a client crashes and restarts, it may reuse the same initial sequence number or the protocol has no mechanism to encode "age." The receiver gets two structurally identical packets with no postmark or timestamp — it cannot distinguish between a zombie from a previous attempt and a legitimate retransmission. The receiver is stateless about the history of what it received; it only knows the packet arrived, not how long it spent in the network.

---

**Q: What does "performance-first" mean in the correctness vs performance tradeoff?**
Correctness-first means adding a mechanism that always distinguishes delayed duplicates from new requests — airtight but slow and expensive in overhead. Performance-first means making the confusion scenario so statistically unlikely that it stops being a practical problem, without eliminating it mathematically. TCP does this via random Initial Sequence Numbers (ISNs): the ISN is a random 32-bit number, so the probability that a delayed duplicate from a previous connection shares the same ISN as a new connection is astronomically low. You are not solving the problem, you are making the bad outcome improbable enough to be acceptable.

---

**Q: How does ISN randomization actually cause the receiver to reject a delayed duplicate?**
When a connection starts, both sides agree on a starting sequence number — the ISN. Every subsequent packet is numbered relative to that ISN. The receiver maintains a window of expected sequence numbers for the current connection. A delayed duplicate from a previous connection carries sequence numbers relative to a completely different ISN, so its numbers fall outside the current connection's expected window entirely. The receiver sees a sequence number that doesn't match anything it's waiting for and drops it. The mismatch exposes the zombie — not any explicit "old" label. This is why ISNs must be random: if both connections started from ISN 0, the old packet's sequence numbers might accidentally fall inside the new connection's window and get accepted as valid data.

---

**Q: How does restricting packet lifetime actually solve the delayed duplicate problem?**
The other two solutions try to identify duplicates at the receiver and reject them. This solution prevents duplicates from existing at all. Every packet is given a maximum lifetime enforced by the network — if a packet has been roaming longer than this bound, intermediate routers kill it before it can reach the receiver. The receiver only ever sees the retransmission; the zombie never arrives. Additionally, the sender waits for one full maximum packet lifetime before reusing the same connection parameters, guaranteeing all stragglers from the old connection are dead before the new one begins.

---

**Q: Why do throwaway ports and unique global identifiers both fail — what's the common thread?**
Both fail because they solve the problem through permanent exclusion. Throwaway ports permanently retire every used port number, and since port numbers are a finite 16-bit space (65535 values), the address space is eventually exhausted. Unique global identifiers permanently blacklist every used identifier, which requires persistent state tracking across system crashes — the algorithm must survive reboots and still guarantee no new identifier overlaps with any previously live one, which is infeasible overhead. Neither approach sets an expiry on used-up resources; they just write them off forever. Restricting packet lifetime is the opposite philosophy: reuse freely, but wait for provable death first.

---

**Q: What is the difference between TTL (hop count) and T (maximum packet lifetime)?**
These are two different things operating at two different levels. TTL with hop count is the enforcement mechanism — what physically kills packets in the network. Every router decrements the hop count; if it hits zero, the packet is dropped. This happens at the network layer per packet in real time. T, the maximum packet lifetime, is a derived design parameter — the theoretical upper bound on how long any packet could possibly survive, calculated by analysing how TTL is configured and worst-case queuing delays. T is not enforced directly; it is the answer to "after what time is it safe to reuse a sequence number?" because after T seconds, TTL enforcement guarantees everything carrying that sequence number is dead.

---

**Q: Does hop count actually bound time? What about a packet stuck in a queue?**
Hop count does not bound time — this is a known limitation. The hop count only decrements when a packet is forwarded by a router; it does not move while the packet sits in a queue. A packet with TTL 30 remaining could theoretically queue indefinitely without its hop count changing. Hop count specifically solves routing loops (bouncing forever between routers), not indefinite queuing. Queuing is bounded separately: router queues have a maximum size and active queue management algorithms like RED (Random Early Detection) probabilistically drop packets before queues fill completely. Residual edge cases are handled by ISN randomization. No single mechanism is airtight — hop count, queue management, and ISN randomization work collectively.

---

**Q: Why is restricted network design not feasible at internet scale?**
Restricted network design requires a guaranteed maximum end-to-end delay across the entire network — meaning every router on every path would need to provide timing guarantees that can be summed or bounded. The internet is decentralized; no single entity owns or controls all routers, and no global enforcement of such guarantees is possible. It is a structural promise that nobody can make.

---

**Q: Why is timestamp-per-packet not feasible despite being the most theoretically clean solution?**
Timestamp requires every router on earth to have synchronized clocks. Clock drift — the phenomenon where physical clocks on different machines run at slightly different speeds, so "60 seconds" on one machine is actually 60.003 seconds on another — compounds across internet scale into real desynchronization. One router might consider a packet alive while another considers it expired, making consistent enforcement impossible. Sequence numbers avoid this entirely because both sides agree on the numbering scheme upfront; no hardware clock is involved.

---

**Q: What is the virtual clock and how do sequence numbers act as one?**
A virtual clock is the sequence number counter itself, incrementing at a fixed rate tied to real time regardless of whether packets are being sent. When a packet is sent, it is stamped with the current sequence number value — which is equivalent to stamping it with the current virtual time. The sequence number space is sized and the increment rate is chosen such that a full wraparound (reuse of a sequence number) takes longer than T seconds. By the time a sequence number would be reused, TTL has guaranteed that every packet ever stamped with it is dead. So reuse is safe. There is no hardware clock involved, which is why clock drift is not a problem.

---

**Q: Why must T account for acknowledgements and not just packets?**
The delayed duplicate problem exists on both sides. The receiver cannot distinguish duplicate requests; but the sender also cannot distinguish acknowledgements. If a connection crashes and restarts, an acknowledgement for the original request might still be in the network — the sender cannot tell if it belongs to the old attempt or the new one. T must therefore be long enough to guarantee that not just the original packets but all acknowledgements associated with them have also been purged from the network. Only once both are guaranteed dead can sequence numbers be safely reused and any arriving acknowledgement be trusted to belong to a currently live connection.