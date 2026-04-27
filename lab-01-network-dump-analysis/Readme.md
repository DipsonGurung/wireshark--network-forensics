# 🌐 Network Forensics Lab — Packet Analysis Walkthrough

**File:** `NetworkDumps.pcapng`  
**Tool:** Wireshark 4.x  
**Context:** Academic lab — protocol identification & web activity reconstruction

---

## Overview

This lab analyses a real packet capture taken from a networked Kali Linux machine to reconstruct the sequence of actions performed by a user. Using Wireshark, each packet is inspected to identify protocols, extract MAC addresses, follow HTTP interactions, and understand web-filtering behaviour.

The capture demonstrates a full browsing session including: a ping to Google, an attempted visit to a blocked website (intercepted by a network content filter), and subsequent visits to BBC, Guardian, and Coventry University's staff portal.

---

## Capture Details

| Field | Detail |
|---|---|
| Capture File | `NetworkDumps.pcapng` |
| Analysis Tool | Wireshark 4.x (GUI) |
| Client IP | `192.168.76.144` |
| Client MAC | `00:0c:29:87:61:c7` (VMware VM) |
| Gateway IP | `192.168.76.1` |
| Gateway MAC | `00:50:56:c0:00:08` (VMware virtual adapter) |
| DNS Resolver | `192.168.76.2` |
| Capture Date | 10 February 2014 |
| OS / Browser | Kali Linux 3.12 / Links 2.7 (text browser) |

---

## Network Participants

| IP Address | MAC Address | Role |
|---|---|---|
| `192.168.76.144` | `00:0c:29:87:61:c7` | User / Client machine (VMware VM) |
| `192.168.76.1` | `00:50:56:c0:00:08` | Default Gateway |
| `192.168.76.2` | — | DNS Resolver |
| `82.98.86.162` | — | `aaaaaaaaaaa.com` web server |
| `192.168.69.65` | — | Web Content Filter (port 9014) |
| `212.58.246.93` | — | `www.bbc.co.uk` |
| `184.168.221.51` | — | `guardian.co` (via CNAME) |
| `192.168.63.25` | — | `staff.coventry.ac.uk` (internal server) |

---

## Lab Questions & Answers

### Q1 — What was the first thing the user did?
**Answer:** The user **pinged `www.google.com`**.  
The OS first issued a DNS query to resolve the hostname to `173.194.41.112` (Packets 1 & 4), then sent 5 ICMP Echo Request packets to that IP.

**Wireshark filter:** `dns` → Packet 1

---

### Q2 — Which protocols were involved and why?
**Answer:** Two protocols:

| Packets | Protocol | Purpose |
|---|---|---|
| 1 & 4 | DNS | Resolves `www.google.com → 173.194.41.112` before the ping |
| 5, 10, 15, 20, 25 | ICMP | Echo Request (type 8) — the actual ping packets |

**Wireshark filter:** `icmp`

---

### Q3 — How many packets were sent for this protocol?
**Answer:** **5 ICMP Echo Request packets** (Packets 5, 10, 15, 20, 25).  
Each received a corresponding Echo Reply (type 0) from the server.

---

### Q4 — What was the second thing the user did?
**Answer:** The user attempted to browse to **`http://aaaaaaaaaaa.com`**.  
DNS queries appear at Packets 109–112, followed by an HTTP GET request at Packet 116.

**Wireshark filter:** `dns and frame.number >= 109`

---

### Q5 — Which protocol was used?
**Answer:** **HTTP** (Hypertext Transfer Protocol) — an unencrypted GET request over TCP port 80.

**Wireshark filter:** `http and frame.number == 116`

---

### Q6 — MAC address of `192.168.76.1`?
**Answer:** `00:50:56:c0:00:08`  
Identified from ARP traffic. OUI prefix `00:50:56` confirms this is a **VMware virtual network adapter** acting as the default gateway.

**Wireshark filter:** `arp` → look for packets from/to `192.168.76.1`

---

### Q7 — What interaction occurs at Packets 113–115?
**Answer:** The **TCP Three-Way Handshake** — how TCP establishes a reliable, connection-oriented session before any data is sent.

| Packet | Direction | TCP Flag | Meaning |
|---|---|---|---|
| 113 | Client → Server | SYN | "I want to connect" |
| 114 | Server → Client | SYN, ACK | "Acknowledged, connect to me too" |
| 115 | Client → Server | ACK | "Confirmed — connection established" |

**Wireshark filter:** `frame.number >= 113 and frame.number <= 115`

---

### Q8 — What website was the user trying to visit?
**Answer:** `aaaaaaaaaaa.com`  
Visible in the HTTP Host header of the GET request at Packet 116, directed to `82.98.86.162` on TCP port 80.

---

### Q9 — What browser and OS was the user using?
**Answer:** Extracted from the HTTP User-Agent header in Packet 116:  
`User-Agent: Links (2.7; Linux 3.12-kali1-686-pae i686; GNU C 4.7.2; text)`

| Attribute | Value |
|---|---|
| Browser | Links 2.7 (lightweight text-based browser) |
| OS / Kernel | Kali Linux, kernel `3.12-kali1-686-pae` |
| Architecture | 32-bit (i686) |
| Compiler | GNU C 4.7.2 |

**Wireshark filter:** `frame.number == 116`

---

### Q10 — What does HTTP/1.1 200 mean (Packet 128)?
**Answer:** **HTTP 200 OK** — the server successfully received the request and returned a resource. Here the 200 OK belongs to the **web content filter's block page** loading successfully, even though the actual destination (`aaaaaaaaaaa.com`) was denied.

**Wireshark filter:** `frame.number == 128`

---

### Q11 — Why did the page show "Cannot Access Webpage" (Packet 138)?
**Answer:** A **network-level web content filter** intercepted the request at Packet 126 and redirected the browser to an internal filter server (`192.168.69.65:9014`). The block event logged:

```
requestedurl=http://aaaaaaaaaaa.com/
basictype=block
categorydescriptionlist=Malicious Sites, Pornography
categorylist=130,149
actiontaken=block
actionreason=by-category
```

The site was categorised as both **Malicious** (cat. 130) and **Pornography** (cat. 149). Consistent with enterprise web filtering products such as Websense / Forcepoint.

**Wireshark filter:** `frame.number == 126`

---

### Q12 — What websites were visited next?
**Answer:** Three more sites in sequence:

| Order | Website | Packets | IP | Outcome |
|---|---|---|---|---|
| 1st | `www.bbc.co.uk` | 159–209 | `212.58.246.93` | HTTP 404 Not Found (bad path) |
| 2nd | `www.guardian.co` | 219–239 | `184.168.221.51` | HTTP 200 OK (page loaded) |
| 3rd | `staff.coventry.ac.uk` | 255–270 | `192.168.63.25` | HTTP 302 → HTTPS, TLS handshake began |

The Coventry University staff portal resolving to an internal `192.168.63.x` IP strongly suggests the user is affiliated with Coventry University. The 302 redirect forced an upgrade to HTTPS, with TLS beginning at Packet 269.

**Wireshark filter:** `http and frame.number >= 159`

---

## Protocol Summary

| Protocol | Role in This Capture | Key Packets |
|---|---|---|
| DNS | Resolves hostnames to IPs before each connection | 1, 4, 7, 30–33, 109–112, 159–162, 219–222, 255–258 |
| ICMP | Ping — Echo Request / Reply for connectivity test | 5, 6, 10, 11, 15, 16, 20, 21, 25, 26 |
| ARP | Maps IPs to MACs on the LAN | 2, 3, 9, 14, 19, 24, 29 |
| TCP | Reliable transport — 3-way handshake & teardown | 113–115, 223–225, 259–261 |
| HTTP | Application-layer web requests (unencrypted) | 116, 119, 126, 146, 166, 209, 226, 238, 262, 264 |
| TLS | Encrypted HTTPS session (after 302 redirect) | 269+ |

---

## Key Findings

- **DNS precedes every connection** — the OS resolves a hostname before any TCP handshake or HTTP request can occur.
- **ICMP (ping)** is a diagnostic tool, not a browsing protocol — it uses type-8 Echo Requests and type-0 Echo Replies.
- **ARP operates at Layer 2** — it maps IP addresses to MAC addresses within the same LAN segment, and exposes device hardware identifiers.
- The **TCP three-way handshake** (SYN → SYN-ACK → ACK) must complete before any HTTP data is exchanged.
- **HTTP is plaintext** — every request, response, User-Agent string, and Host header is fully readable in the capture.
- **Web content filters** can intercept HTTP silently at the network level, redirecting blocked requests to a denial page — visible here via the `/actionpage` parameters in Packet 126.
- A **302 redirect** signals content has moved temporarily — often used to force an upgrade from HTTP to HTTPS.
- **TLS (HTTPS)** encrypts all subsequent traffic — the content of the `staff.coventry.ac.uk` session is not readable beyond the Client Hello.

---

## Files in This Directory

| File | Description |
|---|---|
| `NetworkDumps.pcapng` | Original packet capture file |
| `Network_Forensics_Lab_Report.docx` | Full lab report with annotated screenshots |

---

> **Disclaimer:** This report was produced for educational and portfolio purposes. All analysis performed on a provided PCAP file using Wireshark.
