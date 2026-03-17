# 🛡️ My CTF Networking Journey — Root-Me Writeups
*A structured, hands-on approach to building network security fundamentals through protocol analysis, packet inspection, and threat-relevant tooling.*

---

## 👤 About Me

I am a self-taught cybersecurity student focused on building a strong foundation in network security with the goal of working as a SOC Analyst. This repository documents my progress through networking challenges on Root-Me, one of the leading platforms for practical cybersecurity training.

Every writeup reflects not just the solution, but the methodology, concepts learned, and analytical process — the same structured thinking expected in a SOC environment.

---

## 🧰 Tools Used

| Tool | Purpose |
|------|---------|
| Wireshark | Packet capture analysis, Follow TCP Stream, protocol filtering, layer inspection |
| Nmap | Host and service enumeration |
| Linux Terminal | Command execution, file management, tool usage (daily CachyOS user) |
| dig | DNS query utility — zone transfer enumeration, record-type querying |
| snmpv3brute | SNMPv3 authentication hash brute-forcing against pcap files |
| hashcat | Offline hash cracking with wordlists and mask attacks |
| John the Ripper | Offline hash cracking, mask-based brute force |
| hashes.com | Online rainbow table lookup for pre-cracked hashes |
| Packet6 Type 7 Decryptor | Cisco Type 7 password reversal tool |

---

## 📚 Challenges Completed

### ✅ Challenge 01 — FTP Authentication

**What the challenge was:** A pcap file containing FTP traffic. The goal was to identify login credentials transmitted over the network.

**What I learned:**
- FTP transmits credentials in plaintext — no encryption at all
- The `USER` and `PASS` commands are fully readable in the TCP stream
- Wireshark's Follow TCP Stream reassembles the full session conversation

**My Approach:**
1. Opened the pcap in Wireshark
2. Right-clicked a packet → Follow TCP Stream
3. Read the plaintext credentials directly from the stream

**Key Skill Learned/Reinforced:** Wireshark's Follow TCP Stream — reassembles fragmented TCP packets into a readable, human-friendly conversation.

**Core Takeaway:**
> FTP was designed without security in mind. Credentials are fully visible to any observer on the network — this is why SFTP and FTPS exist as secure alternatives.

---

### ✅ Challenge 02 — TELNET Authentication

**What the challenge was:** A pcap file containing Telnet traffic. The goal was to extract login credentials from the captured session.

**What I learned:**
- Telnet sends every keystroke as a separate packet, including login input
- There is a Telnet negotiation phase at session start where client and server agree on terminal options
- Despite the negotiation noise, credentials still appear in plaintext in the stream

**My Approach:**
1. Opened the pcap in Wireshark
2. Applied a Wireshark display filter to isolate Telnet traffic
3. Followed TCP Stream to reassemble the full session
4. Read past the negotiation phase to locate the login prompt and credentials

**Key Skill Learned/Reinforced:** Identifying and reading through protocol negotiation phases — understanding that not all traffic at session start is credentials.

**Core Takeaway:**
> Telnet is as insecure as FTP — all data including passwords travels in cleartext. SSH replaced Telnet precisely because of this vulnerability.

---

### ✅ Challenge 03 — Twitter Authentication

**What the challenge was:** A pcap file containing HTTP traffic with a Twitter login. The goal was to extract credentials from an HTTP Basic Authentication header.

**What I learned:**
- HTTP Basic Authentication encodes credentials as `username:password` in Base64 and sends them in the request header
- Base64 is encoding, not encryption — it can be decoded by anyone instantly
- Wireshark automatically decodes Base64 in HTTP Basic Auth headers
- The credentials were visible in a single packet — no stream reassembly needed

**My Approach:**
1. Opened the pcap in Wireshark
2. Applied a Wireshark display filter to isolate HTTP traffic
3. Located the HTTP request containing the `Authorization: Basic` header
4. Wireshark decoded the Base64 automatically — credentials immediately visible

**Key Skill Learned/Reinforced:** HTTP Basic Auth and Base64 decoding — understanding that encoding is not encryption, and that Wireshark surfaces decoded values automatically.

**Core Takeaway:**
> Base64 is not a security mechanism — it is a transport encoding. HTTP Basic Auth over plain HTTP exposes credentials to any network observer.

---

### ❌ Challenge 04 — SNMP Authentification *(Not Validated)*

**What the challenge was:** A pcap file containing SNMPv3 traffic. Unlike previous challenges, Follow TCP Stream alone reveals nothing useful — SNMPv3 uses hash-based authentication that requires a different toolchain entirely.

**What I learned:**
- SNMP is used to monitor and manage network devices such as routers, switches, and servers
- Three SNMP versions exist with increasing security:
  - SNMPv1 → Community string only, plaintext, no authentication
  - SNMPv2c → Improved performance, still cleartext
  - SNMPv3 → Hash-based authentication + optional encryption
- SNMPv3 operates under three security levels:

| Level | Authentication | Encryption | Meaning |
|-------|---------------|------------|---------|
| noAuthNoPriv | ❌ | ❌ | No security |
| authNoPriv | ✅ | ❌ | Authenticated, data visible |
| authPriv | ✅ | ✅ | Fully secured |

- The challenge used SNMPv3 with `authNoPriv` — authenticated but not encrypted
- SNMPv3 never transmits the password directly — only a derived HMAC hash stored in `msgAuthenticationParameters`
- The `msgUserName` field travels in plaintext — revealed username: `user`
- Tools like `snmpv3brute` are required to brute-force the hash directly from the pcap

**My Approach:**
1. Opened pcap in Wireshark — Follow TCP Stream showed unreadable data
2. Inspected individual SNMP packets → identified SNMPv3 and USM security model
3. Read `msgFlags`: auth SET, encrypted NOT SET → confirmed `authNoPriv`
4. Located `msgUserName: user` in the USM section
5. Identified `msgAuthenticationParameters` as a hash, not a plaintext password
6. Attempted offline hash cracking using snmpv3brute with a large wordlist — password not found

**Key Skill Learned/Reinforced:** Reading SNMPv3 packet structure in Wireshark — identifying security level from `msgFlags`, navigating the USM layer, and recognizing that hash-based authentication requires a cracking tool rather than simple stream reading.

**Core Takeaway:**
> SNMPv3 never sends your password over the wire — only a hash. This challenge introduced a fundamentally different analytical approach: extract and crack, not just read.

---

### ✅ Challenge 05 — CISCO Password

**What the challenge was:** A Cisco IOS configuration file is provided. The goal is to find the "Enable" password. The challenge hint reads: *"It's not always a hash."*

**What I learned:**
- Cisco IOS configuration files store passwords in different formats depending on the command used and IOS version
- **Type 7** is not real encryption — it is obfuscation using a known algorithm with a fixed key, fully reversible using online tools
- **Type 5** is an MD5-based hash — cannot be reversed directly, only cracked via wordlist/brute force or looked up in a rainbow table database
- The key vocabulary distinction:
  - **Obfuscation** — hides data but is always reversible with a known algorithm (no key needed)
  - **Encryption** — requires a secret key to reverse
  - **Hashing** — one-way transformation; cannot be reversed, only cracked or looked up
- `service password-encryption` enables Type 7 obfuscation on plaintext passwords — it does not provide real security
- `enable secret` (Type 5 MD5) is stronger than `enable password` (Type 7 or plaintext)
- **privilege 15** on a Cisco device means highest access level — equivalent to root/enable access
- Online hash lookup databases (e.g., hashes.com) store previously cracked hashes — useful when wordlist cracking fails

```
# Cisco password types summary
Type 0  — Plaintext (no encryption)
Type 7  — Vigenère-based obfuscation (reversible, NOT secure)
Type 5  — MD5 hash (one-way, requires cracking or lookup)
Type 8  — SHA-256 (strong)
Type 9  — Scrypt (strongest)
```

**My Approach:**
1. Read the config file line by line and identified all password-related entries
2. Decoded all four Type 7 passwords using an online Type 7 decoder (Packet6) — all followed the same naming pattern
3. Attempted decoded passwords as the flag — all rejected
4. Attempted to crack the Type 5 MD5 hash using hashcat with targeted wordlists and mask attacks — encountered GPU memory limitations on the Kali VM
5. Switched to John the Ripper and tested a pattern-based mask attack — exhausted with no result
6. Pivoted to online rainbow table lookup at hashes.com — hash found instantly, password recovered

**Key Skill Learned/Reinforced:** Online rainbow table lookup as a cracking strategy — when local wordlist and brute force attacks fail due to resource constraints, online hash databases are a viable alternative that requires no compute.

**Core Takeaway:**
> Not all password storage is equal. Type 7 "encryption" in Cisco configs is obfuscation — trivially reversible. Type 5 MD5 is a real hash, but MD5 is weak enough that rainbow tables make it instantly recoverable. The hint "It's not always a hash" was a reminder to look beyond the obvious attack path.

---

### ✅ Challenge 06 — DNS — Zone Transfert

**What the challenge was:** A DNS service was intentionally misconfigured for a target domain hosted on a non-standard port. The objective was to exploit an unrestricted DNS Zone Transfer (AXFR) to enumerate all DNS records within the zone and retrieve the flag embedded in the zone data.

**What I learned:**
- A DNS **server** is the machine responding to requests; a **domain** (zone) is the subject being queried; a **query type** defines what kind of information is requested — these are three distinct concepts that must not be conflated
- DNS zone transfers are a legitimate administrative mechanism allowing secondary DNS servers to synchronize records from a primary server
- When left unrestricted, any external party can request the entire zone — exposing every subdomain, internal hostname, and service record the organization has configured
- This is a reconnaissance-enabling misconfiguration: it does not grant access to any system, but it eliminates the reconnaissance phase of an attack entirely
- AXFR zone transfers require TCP, not UDP — standard DNS queries use UDP, but a full zone dump exceeds what a single UDP datagram can carry
- Reading tool documentation directly (`man dig`) is an effective way to understand each component of a query rather than relying on pre-built commands

**My Approach:**
1. Analyzed the challenge information to identify the target domain, DNS server, and non-standard port
2. Researched the concept of DNS zone transfers to understand the vulnerability before touching any tool
3. Consulted the `dig` manual to identify flags for server targeting, port specification, query name, query type, and TCP transport
4. Iterated through connection issues — correcting the server vs. domain argument order and explicitly forcing TCP transport
5. Successfully executed the AXFR request, which returned the full zone record set containing the flag

**Key Skill Learned/Reinforced:** DNS enumeration via zone transfer abuse — understanding that misconfigured DNS infrastructure can expose an organization's entire internal naming structure in a single query, and identifying this using native Linux tooling.

**Core Takeaway:**
> A DNS zone transfer misconfiguration is a reconnaissance gift to an attacker. From a SOC perspective, anomalous AXFR requests in DNS logs are a meaningful early-warning signal. The fix is straightforward: restrict zone transfer responses to authorized secondary DNS server IP addresses only.

---

## 🗺️ Pattern Recognized

| Challenge | Protocol | Encryption | Credentials Visible? |
|-----------|----------|------------|----------------------|
| FTP Authentication | FTP | ❌ None | ✅ Plaintext in TCP stream |
| TELNET Authentication | Telnet | ❌ None | ✅ Plaintext in TCP stream |
| Twitter Authentication | HTTP | ❌ None (Base64 encoding only) | ✅ Decoded automatically by Wireshark |
| SNMP Authentification | SNMPv3 | ❌ authNoPriv | ⚠️ Hash only — requires cracking tool |
| CISCO Password | Cisco IOS Config | ⚠️ Type 7 obfuscation / Type 5 MD5 | ⚠️ Type 7 reversible instantly / Type 5 via rainbow table |
| DNS — Zone Transfert | DNS / AXFR | ❌ None | ✅ Full zone dump — all records exposed |

---

## 📈 Skills Gained So Far

- [x] Wireshark — Follow TCP Stream
- [x] Wireshark — Packet layer inspection
- [x] Wireshark — Protocol filtering (ftp, telnet, http, snmp)
- [x] FTP, Telnet, and HTTP Basic Auth flows
- [x] Base64 is encoding, not encryption
- [x] SNMPv3 security levels (noAuthNoPriv / authNoPriv / authPriv)
- [x] USM (User-based Security Model) in SNMPv3
- [x] Hash vs plaintext — recognizing the difference in packet captures
- [x] Offline hash cracking methodology with wordlists
- [x] Nmap -Pn flag (bypass ping for unresponsive hosts)
- [x] CTF methodology — enumerate, analyze, extract
- [x] Cisco IOS password types (Type 0, 7, 5, 8, 9)
- [x] Obfuscation vs encryption vs hashing — vocabulary and practical difference
- [x] Cisco Type 7 decoding with online tools
- [x] Hashcat and John the Ripper — wordlist and mask attacks
- [x] Rainbow table / online hash lookup (hashes.com)
- [x] Reading and analyzing Cisco IOS configuration files
- [x] DNS zone transfer abuse (AXFR) — enumeration via misconfigured DNS
- [x] dig — DNS query utility, server/domain/query-type distinction, TCP forcing
- [x] DNS infrastructure concepts — server vs. zone vs. query type
- [x] Reconnaissance impact of DNS misconfiguration — attack surface mapping

---

## 📖 Resources That Helped Me

| Resource | Purpose |
|----------|---------|
| Root-Me | CTF networking challenges platform |
| TryHackMe — Blue Team Path | Structured SOC analyst learning pathway |
| Wireshark Documentation | Protocol dissection reference and filter syntax |
| dig (man page) | DNS query utility — zone transfer and record-type reference |
| snmpv3brute | SNMPv3 pcap brute-force tool |
| hashes.com | Online rainbow table lookup for MD5 and other hash types |
| Packet6 Type 7 Decryptor | Cisco Type 7 password reversal tool |

---

*Last updated: March 2026 — 5 challenges validated | 1 attempted (SNMP)*
