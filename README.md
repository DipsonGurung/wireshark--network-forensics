# 🦈 Wireshark Forensics Labs

A collection of hands-on network forensics labs using **Wireshark** to analyse real packet capture (PCAP) files. Each subdirectory contains a complete lab report with step-by-step Wireshark instructions, annotated findings, and screenshots.

---

## Repository Structure

```
wireshark/
├── README.md                          ← you are here
│
├── lab-01-network-dump-analysis/
|   ├── README.md                      ← lab overview & quick-reference answers
|   ├── NetworkDumps.pcapng            ← packet capture file
|   └── Network_Forensics_Lab.pdf
│
└── lab-02-AppleTV-Forensic/
   ├── README.md                      ← lab overview & quick-reference answers
   ├── evidence03.pcap                ← packet capture file
   └── Evidence03_AppleTV_Forensic_Report.d
```

---

## Labs at a Glance

### 🌐 [Network Forensics Lab — Packet Analysis Walkthrough](./network_forensics_lab/README.md)

| | |
|---|---|
| **PCAP file** | `NetworkDumps.pcapng` |
| **Scenario** | Academic lab — reconstructing a user's full browsing session |
| **Device** | Kali Linux VM (Links browser), IP `192.168.76.144` |
| **Protocols** | DNS, ICMP, ARP, TCP, HTTP, TLS |
| **Key skill** | Tracing a complete session from ping → DNS → TCP handshake → HTTP → web filter → HTTPS |

**What you will find:**
- A ping to `www.google.com` reconstructed from DNS + ICMP packets
- MAC address extraction via ARP
- The TCP three-way handshake (SYN → SYN-ACK → ACK) at Packets 113–115
- OS and browser fingerprinting from the HTTP User-Agent header
- A blocked website (`aaaaaaaaaaa.com`) intercepted by a Websense/Forcepoint content filter
- Subsequent visits to BBC, Guardian, and Coventry University's staff portal
- HTTP → HTTPS 302 redirect and the start of a TLS session

---


### 📺 [Evidence03 — AppleTV Activity Analysis](./evidence03_appletv/README.md)

| | |
|---|---|
| **PCAP file** | `evidence03.pcap` (1,778 packets) |
| **Scenario** | Covert surveillance — reconstructing a suspect's iTunes Store activity |
| **Device** | Apple TV 2nd gen (firmware 2.4), IP `192.168.1.10` |
| **Protocols** | DNS, TCP, HTTP, mDNS, ARP |
| **Key skill** | Reading plaintext HTTP to extract search terms, movie selections, trailer URLs, and pricing data keystroke-by-keystroke |

**What you will find:**
- The suspect's MAC address from ARP traffic
- Device identification from the HTTP User-Agent string
- Four incremental search terms typed character-by-character (`h → ha → hac → hack`)
- Two movies browsed: *Hackers* (1995) and *Sneakers* (1992)
- A full movie trailer URL extracted from iTunes Store XML
- A final search phrase — `iknowyourewatchingme` — suggesting counter-surveillance awareness


---

## Prerequisites

| Requirement | Detail |
|---|---|
| **Wireshark** | Version 4.x recommended — [download here](https://www.wireshark.org/download.html) |
| **PCAP files** | Included in each subdirectory |
| **OS** | Windows, macOS, or Linux — Wireshark runs on all three |
| **Experience** | Beginner-friendly — each lab includes step-by-step filter instructions |

---

## How to Use These Labs

1. **Clone or download** this repository
2. **Open Wireshark** and load the PCAP file from the relevant subdirectory
3. **Read the lab README** for background context and the list of questions
4. **Follow the Wireshark filter instructions** given for each question
5. **Open the `.docx` report** to see full analysis with screenshot placeholders — insert your own screenshots as you work through the lab

---

## Wireshark Quick-Reference Filters

The following filters are used across both labs. Keep this as a handy reference:

| Filter | What it shows |
|---|---|
| `arp` | All ARP packets — use to find MAC addresses |
| `dns` | All DNS queries and responses |
| `icmp` | All ping (Echo Request / Reply) packets |
| `http` | All plaintext HTTP requests and responses |
| `tcp` | All TCP segments |
| `tls` | All TLS/HTTPS traffic |
| `frame.number == N` | Jump to a specific packet number |
| `frame.number >= N and frame.number <= M` | View a range of packets |
| `http.request.uri contains "text"` | Filter HTTP requests by URL content |
| `ip.addr == 192.168.1.10` | Show all traffic to/from a specific IP |

---

## Key Concepts Covered

| Concept | Covered In |
|---|---|
| MAC address extraction from ARP | Both labs |
| Device fingerprinting via User-Agent | Both labs |
| DNS resolution before every connection | Both labs |
| Incremental / keystroke search leakage | Evidence03 |
| Plaintext HTTP data extraction | Both labs |
| XML metadata recovery from HTTP responses | Evidence03 |
| TCP three-way handshake | Network Forensics Lab |
| ICMP ping reconstruction | Network Forensics Lab |
| Web content filter detection | Network Forensics Lab |
| HTTP 302 redirect → TLS upgrade | Network Forensics Lab |
| Counter-surveillance indicators | Evidence03 |

---

## Report Documents

Each lab includes a fully formatted `.docx` report built with consistent styling:

- **Cover block** with case metadata
- **Network participants table** with IP/MAC/role breakdown
- **Per-question sections** with answer boxes and Wireshark filter instructions
- **Screenshot placeholders** — paste your own Wireshark captures directly into the document
- **Protocol summary table** covering all protocols seen in the capture
- **Key findings** bullet list with forensic commentary

---

> **Disclaimer:** All labs were produced for educational and portfolio purposes. Analysis was performed on provided PCAP files using Wireshark. No real criminal investigation is implied.
