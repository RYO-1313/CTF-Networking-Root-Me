# 🛡️ My CTF Networking Journey — Root-Me Writeups
A structured, hands-on approach to building network security fundamentals through protocol analysis, packet inspection, and threat-relevant tooling.

---

## 👤 About Me
I am a self-taught cybersecurity student focused on building a strong foundation in network security with the goal of working as a SOC Analyst. This repository documents my progress through networking challenges on Root-Me, one of the leading platforms for practical cybersecurity training.

Every writeup reflects not just the solution, but the methodology, concepts learned, and analytical process — the same structured thinking expected in a SOC environment.

---

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| Wireshark | Packet capture analysis, Follow TCP Stream, protocol filtering, layer inspection |
| Nmap | Host and service enumeration |
| Linux Terminal | Command execution, file management, tool usage (daily CachyOS user) |
| dig | DNS query utility — zone transfer enumeration, record-type querying |
| ldapsearch | LDAP directory enumeration and null bind testing |
| snmpv3brute | SNMPv3 authentication hash brute-forcing against pcap files |
| apop2john | APOP challenge-response hash extraction for John the Ripper |
| hashcat | Offline hash cracking with wordlists and mask attacks |
| John the Ripper | Offline hash cracking, mask-based brute force, format-aware cracking |
| hashes.com | Online rainbow table lookup for pre-cracked hashes |
| Packet6 Type 7 Decryptor | Cisco Type 7 password reversal tool |
| Ettercap | Protocol-aware pcap parsing — OSPF MD5 hash extraction from static capture files |
| BruteShark | Automated network credential extraction from pcap files |

---

## 📚 Challenges Completed

### ✅ Challenge 01 — FTP Authentication
**What the challenge was:** A pcap file containing FTP traffic. The goal was to identify login credentials transmitted over the network.

**What I learned:**
- FTP transmits credentials in plaintext — no encryption at all
- The USER and PASS commands are fully readable in the TCP stream
- Wireshark's Follow TCP Stream reassembles the full session conversation

**My Approach:**
- Opened the pcap in Wireshark
- Right-clicked a packet → Follow TCP Stream
- Read the plaintext credentials directly from the stream

**Key Skill Learned/Reinforced:** Wireshark's Follow TCP Stream — reassembles fragmented TCP packets into a readable, human-friendly conversation.

**Core Takeaway:** FTP was designed without security in mind. Credentials are fully visible to any observer on the network — this is why SFTP and FTPS exist as secure alternatives.

---

### ✅ Challenge 02 — TELNET Authentication
**What the challenge was:** A pcap file containing Telnet traffic. The goal was to extract login credentials from the captured session.

**What I learned:**
- Telnet sends every keystroke as a separate packet, including login input
- There is a Telnet negotiation phase at session start where client and server agree on terminal options
- Despite the negotiation noise, credentials still appear in plaintext in the stream

**My Approach:**
- Opened the pcap in Wireshark
- Applied a Wireshark display filter to isolate Telnet traffic
- Followed TCP Stream to reassemble the full session
- Read past the negotiation phase to locate the login prompt and credentials

**Key Skill Learned/Reinforced:** Identifying and reading through protocol negotiation phases — understanding that not all traffic at session start is credentials.

**Core Takeaway:** Telnet is as insecure as FTP — all data including passwords travels in cleartext. SSH replaced Telnet precisely because of this vulnerability.

---

### ✅ Challenge 03 — Twitter Authentication
**What the challenge was:** A pcap file containing HTTP traffic with a Twitter login. The goal was to extract credentials from an HTTP Basic Authentication header.

**What I learned:**
- HTTP Basic Authentication encodes credentials as username:password in Base64 and sends them in the request header
- Base64 is encoding, not encryption — it can be decoded by anyone instantly
- Wireshark automatically decodes Base64 in HTTP Basic Auth headers
- The credentials were visible in a single packet — no stream reassembly needed

**My Approach:**
- Opened the pcap in Wireshark
- Applied a Wireshark display filter to isolate HTTP traffic
- Located the HTTP request containing the Authorization: Basic header
- Wireshark decoded the Base64 automatically — credentials immediately visible

**Key Skill Learned/Reinforced:** HTTP Basic Auth and Base64 decoding — understanding that encoding is not encryption, and that Wireshark surfaces decoded values automatically.

**Core Takeaway:** Base64 is not a security mechanism — it is a transport encoding. HTTP Basic Auth over plain HTTP exposes credentials to any network observer.

---

### ❌ Challenge 04 — SNMP Authentification (Not Validated)
**What the challenge was:** A pcap file containing SNMPv3 traffic. Unlike previous challenges, Follow TCP Stream alone reveals nothing useful — SNMPv3 uses hash-based authentication that requires a different toolchain entirely.

**What I learned:**
- SNMP is used to monitor and manage network devices such as routers, switches, and servers
- Three SNMP versions exist with increasing security: SNMPv1 (plaintext), SNMPv2c (improved performance, still cleartext), SNMPv3 (hash-based authentication + optional encryption)
- SNMPv3 operates under three security levels: noAuthNoPriv, authNoPriv, authPriv
- The challenge used SNMPv3 with authNoPriv — authenticated but not encrypted
- SNMPv3 never transmits the password directly — only a derived HMAC hash stored in msgAuthenticationParameters
- The msgUserName field travels in plaintext — revealed username: user
- Tools like snmpv3brute are required to brute-force the hash directly from the pcap

**My Approach:**
- Opened pcap in Wireshark — Follow TCP Stream showed unreadable data
- Inspected individual SNMP packets → identified SNMPv3 and USM security model
- Read msgFlags: auth SET, encrypted NOT SET → confirmed authNoPriv
- Located msgUserName: user in the USM section
- Identified msgAuthenticationParameters as a hash, not a plaintext password
- Attempted offline hash cracking using snmpv3brute with a large wordlist — password not found

**Key Skill Learned/Reinforced:** Reading SNMPv3 packet structure in Wireshark — identifying security level from msgFlags, navigating the USM layer, and recognizing that hash-based authentication requires a cracking tool rather than simple stream reading.

**Core Takeaway:** SNMPv3 never sends your password over the wire — only a hash. This challenge introduced a fundamentally different analytical approach: extract and crack, not just read.

---

### ✅ Challenge 05 — CISCO Password
**What the challenge was:** A Cisco IOS configuration file is provided. The goal is to find the "Enable" password. The challenge hint reads: "It's not always a hash."

**What I learned:**
- Cisco IOS configuration files store passwords in different formats depending on the command used and IOS version
- Type 7 is not real encryption — it is obfuscation using a known algorithm with a fixed key, fully reversible using online tools
- Type 5 is an MD5-based hash — cannot be reversed directly, only cracked via wordlist/brute force or looked up in a rainbow table database
- The key vocabulary distinction: obfuscation hides data but is always reversible; encryption requires a secret key to reverse; hashing is a one-way transformation
- service password-encryption enables Type 7 obfuscation on plaintext passwords — it does not provide real security
- Online hash lookup databases (e.g., hashes.com) store previously cracked hashes — useful when wordlist cracking fails

**My Approach:**
- Read the config file line by line and identified all password-related entries
- Decoded all Type 7 passwords using an online Type 7 decoder (Packet6)
- Attempted decoded passwords as the flag — all rejected
- Attempted to crack the Type 5 MD5 hash using hashcat and John the Ripper — no result
- Pivoted to online rainbow table lookup at hashes.com — hash found instantly

**Key Skill Learned/Reinforced:** Online rainbow table lookup as a cracking strategy — when local wordlist and brute force attacks fail due to resource constraints, online hash databases are a viable alternative that requires no compute.

**Core Takeaway:** Not all password storage is equal. Type 7 "encryption" in Cisco configs is obfuscation — trivially reversible. Type 5 MD5 is a real hash, but MD5 is weak enough that rainbow tables make it instantly recoverable.

---

### ✅ Challenge 06 — DNS — Zone Transfert
**What the challenge was:** A DNS service was intentionally misconfigured for a target domain hosted on a non-standard port. The objective was to exploit an unrestricted DNS Zone Transfer (AXFR) to enumerate all DNS records within the zone and retrieve the flag embedded in the zone data.

**What I learned:**
- A DNS server is the machine responding to requests; a domain (zone) is the subject being queried; a query type defines what kind of information is requested — these are three distinct concepts
- DNS zone transfers are a legitimate administrative mechanism allowing secondary DNS servers to synchronize records from a primary server
- When left unrestricted, any external party can request the entire zone — exposing every subdomain, internal hostname, and service record
- AXFR zone transfers require TCP, not UDP — standard DNS queries use UDP, but a full zone dump exceeds what a single UDP datagram can carry

**My Approach:**
- Analyzed the challenge information to identify the target domain, DNS server, and non-standard port
- Researched the concept of DNS zone transfers to understand the vulnerability before touching any tool
- Consulted the dig manual to identify flags for server targeting, port specification, query name, query type, and TCP transport
- Successfully executed the AXFR request, which returned the full zone record set containing the flag

**Key Skill Learned/Reinforced:** DNS enumeration via zone transfer abuse — understanding that misconfigured DNS infrastructure can expose an organization's entire internal naming structure in a single query.

**Core Takeaway:** A DNS zone transfer misconfiguration is a reconnaissance gift to an attacker. From a SOC perspective, anomalous AXFR requests in DNS logs are a meaningful early-warning signal.

---

### ✅ Challenge 07 — LDAP — Null Bind
**What the challenge was:** A misconfigured LDAP server exposed to anonymous access. The objective was to enumerate the directory tree using a null bind and retrieve a user's email address.

**What I learned:**
- LDAP is a protocol used to access and maintain distributed directory services — commonly used in Active Directory environments
- A null bind is an anonymous authentication attempt with no username or password — when a server accepts it, the directory becomes publicly readable
- Distinguished Names (DNs) in LDAP follow a specific-to-general ordering — the most specific component comes first, the broadest comes last
- Anonymous LDAP enumeration can expose usernames, email addresses, group memberships, and organizational structure without any credentials

**My Approach:**
- Connected to the LDAP server using ldapsearch with a null bind
- Enumerated the base directory structure to identify organizational units
- Targeted the relevant ou branch containing user records
- Retrieved the user object and extracted the email address

**Key Skill Learned/Reinforced:** LDAP URI format and Distinguished Name component ordering — understanding how to navigate a directory tree programmatically and why null bind misconfiguration is a critical finding in any security assessment.

**Core Takeaway:** Anonymous LDAP access is a misconfiguration with real impact. In Active Directory environments, a null bind can expose the entire user and group structure of an organization. From a SOC perspective, unauthenticated LDAP bind attempts in logs warrant immediate investigation.

---

### ✅ Challenge 08 — IP — Time To Live
**What the challenge was:** A packet capture containing an ICMP exchange between a source and a target host. The objective was to identify the TTL value used by the source to successfully reach the targeted host.

**What I learned:**
- TTL is a packet lifespan mechanism — a counter that decrements at each router hop, and when it reaches zero, the router discards the packet and sends back an ICMP "Time Exceeded" error
- ICMP is a network-layer protocol used for diagnostic and error communication between devices — the protocol behind ping and traceroute
- The critical distinction between request TTL and reply TTL: the reply carries the target's own TTL — a separate value set by the responding machine
- Traceroute deliberately exploits TTL behavior by sending packets with incrementing TTL values to map every router hop between source and destination

**My Approach:**
- Opened the capture in Wireshark and observed all packets were ICMP, with approximately 70 "Time-to-live exceeded" error messages present
- Identified TTL=51 as an initial candidate — then recognized this was the reply TTL from the target, not the outgoing request
- Re-read the challenge statement carefully and located the successful outgoing request

**Key Skill Learned/Reinforced:** Distinguishing between request TTL and reply TTL in a packet exchange — and reading challenge statements precisely before drawing conclusions from packet data.

**Core Takeaway:** A high volume of ICMP "Time Exceeded" messages in a packet capture is a recognizable fingerprint of traceroute-based reconnaissance. From a SOC detection perspective, this pattern warrants investigation as active network path discovery.

---

### ✅ Challenge 09 — POP-APOP
**What the challenge was:** A packet capture containing a POP3 session using APOP authentication. The objective was to recover the user's plaintext password from the network frame.

**What I learned:**
- POP3 is an email retrieval protocol — clients connect to a mail server to authenticate and download messages
- APOP is a challenge-response authentication extension to POP3 — the server sends a unique timestamp, the client combines it with the password and sends back an MD5 hash, never transmitting the password in cleartext
- Despite being more secure than plain POP3 at the time of its design, APOP relies on MD5 which is now considered cryptographically weak
- Terminal output must be treated as data — visual assumptions about formatting can be incorrect
- Independent verification of tool output is essential: cross-validating with a manual Python MD5 calculation exposed a misreading that would otherwise have remained unresolved

**My Approach:**
- Filtered the capture for POP traffic and followed the TCP stream — identified an APOP exchange with a server challenge string and MD5 digest
- Extracted the hash using apop2john and ran John the Ripper with the rockyou wordlist
- John returned a result that visually appeared to be a progress indicator — submitted the wrong value repeatedly
- Independently verified the hash using a Python script, which confirmed the full string was the complete password entry
- Submitted the correct value and validated the challenge

**Key Skill Learned/Reinforced:** Independent verification of cracking tool output — when results are ambiguous or submissions fail, cross-validating with a direct hash calculation catches misreadings that tool output alone cannot surface.

**Core Takeaway:** APOP was a meaningful security improvement over plaintext POP3 in 1996, but its reliance on MD5 makes it vulnerable to offline wordlist attacks when traffic is captured. From a SOC perspective, APOP traffic in a modern environment signals legacy infrastructure — a potential attack surface.

---

### ✅ Challenge 10 — SIP Authentication
**What the challenge was:** A text-based SIP protocol log containing a VoIP session exchange. The objective was to identify credentials used during a session involving REGISTER, INVITE, and BYE methods.

**What I learned:**
- SIP (Session Initiation Protocol) is used to establish, manage, and terminate VoIP and video call sessions
- SIP borrows its authentication model from HTTP — specifically HTTP Digest Authentication
- Authentication schemes can vary within the same capture: some exchanges use MD5 hashing, others use PLAIN (cleartext) transmission
- PLAIN auth transmits credentials with no encoding, no hashing, and no obfuscation — the password is directly readable in the protocol field

**My Approach:**
- Opened the text-based SIP log and identified three exchanges: REGISTER, INVITE, and BYE
- Initially focused on the MD5 hashes present in the INVITE and BYE lines, assuming the credential would require cracking
- Recognized that the REGISTER exchange used PLAIN authentication — the credential was already visible in cleartext, requiring no further analysis

**Key Skill Learned/Reinforced:** Reading protocol fields critically and resisting the assumption that credentials are always hashed or encoded. Pattern recognition from previous challenges almost caused the plaintext credential to be overlooked entirely.

**Core Takeaway:** Not all authentication is obfuscated. PLAIN auth over SIP transmits passwords in cleartext — any attacker with access to network traffic can extract credentials instantly, with no tooling required. In a SOC environment, SIP traffic using PLAIN auth is a critical misconfiguration.

---

### ✅ Challenge 11 — OSPF Authentication
**What the challenge was:** A network packet capture containing OSPF traffic with cryptographic MD5 authentication. The goal was to extract the OSPF authentication key used between routers.

**What I learned:**
- OSPF (Open Shortest Path First) is a link-state routing protocol used by routers to exchange topology information and calculate efficient paths across a network
- OSPF supports multiple authentication types: no auth, plaintext, MD5 cryptographic, and SHA
- OSPF MD5 authentication hashes are embedded in packets using the Crypto Auth TLV field — the key itself is never transmitted, only the hash
- Ettercap, typically known as a live MITM tool, can also parse static pcap files and extract protocol-specific hashes in John-compatible format
- Tool output must be verified before use — escape sequences in terminal output can obscure actual data and require cleanup before processing

**My Approach:**
- Opened the capture in Wireshark and identified OSPF packets with cryptographic (Type 2) MD5 authentication
- Evaluated multiple tools including PCAP2Hash (WiFi-only, not applicable) before confirming Ettercap as the appropriate extraction tool
- Debugged unreadable output caused by terminal escape sequences using file verification commands
- Extracted clean hash lines in $netmd5$ format and cracked the authentication key using John the Ripper with the rockyou wordlist

**Key Skill Learned/Reinforced:** Transferring a known methodology to a new protocol. The extract-then-crack workflow established in POP-APOP applied directly here — identify the hash, find the right extraction tool, format correctly, crack with John.

**Core Takeaway:** OSPF MD5 authentication is crackable offline if an attacker captures routing traffic. In a SOC environment, unauthorized OSPF adjacency formation or unexpected routing updates are detectable indicators of route poisoning attempts — a serious threat that can redirect enterprise traffic through attacker-controlled infrastructure.

---

### ✅ Challenge 12 — NTLM Authentication
**What the challenge was:** A network packet capture containing NTLM authentication over SMB traffic. The objective was to recover the password of a suspicious user account and submit it in userPrincipalName format as the flag.

**What I learned:**
- NTLM (New Technology LAN Manager) is a challenge-response authentication protocol used in Active Directory environments
- NTLM authentication follows a three-step handshake: NTLMSSP_NEGOTIATE, NTLMSSP_CHALLENGE, NTLMSSP_AUTH — each packet serving a distinct role in the exchange
- NTLMv2 uses HMAC-MD5 for stronger security than NTLMv1, but remains vulnerable to offline hash cracking if traffic is captured
- userPrincipalName (UPN) is the standard Active Directory login format combining username and domain — a critical identifier in SOC alert triage and Windows Event Log analysis
- Manual hash construction from Wireshark fields is error-prone — a truncated NTLMv2 response will silently fail to crack without indicating why
- BruteShark automates protocol credential extraction from pcap files, eliminating manual assembly errors

**My Approach:**
- Opened the capture in Wireshark and filtered for NTLMSSP traffic
- Identified two NTLM handshake cycles and extracted credentials from the second NTLMSSP_AUTH packet: username john.doe, domain catcorp.local
- Retrieved the server challenge from the NTLMSSP_CHALLENGE packet and manually assembled the NTLMv2 hash in John-compatible format
- Attempted cracking with John the Ripper and rockyou, then hashcat — both exhausted without result
- Recognized that the manual hash may have been incorrectly constructed, switched to BruteShark for automated extraction, obtained the complete hash, and successfully cracked it with John the Ripper

**Key Skill Learned/Reinforced:** Self-doubt as a debugging strategy. When cracking attempts failed across multiple tools and wordlists, the correct next step was to question the hash itself rather than continue exhausting wordlists.

**Core Takeaway:** NTLM authentication over SMB is a high-value target in Active Directory environments. Captured NTLMv2 hashes are crackable offline with no interaction required from the target. From a SOC perspective, unexpected SMB connections, NTLM authentication attempts from unusual hosts, or lateral movement patterns involving NTLM are detectable indicators of credential harvesting or pass-the-hash activity.

---

## 🗺️ Pattern Recognized

| Challenge | Protocol | Encryption | Credentials Visible? |
|---|---|---|---|
| FTP Authentication | FTP | ❌ None | ✅ Plaintext in TCP stream |
| TELNET Authentication | Telnet | ❌ None | ✅ Plaintext in TCP stream |
| Twitter Authentication | HTTP | ❌ None (Base64 encoding only) | ✅ Decoded automatically by Wireshark |
| SNMP Authentification | SNMPv3 | ❌ authNoPriv | ⚠️ Hash only — requires cracking tool |
| CISCO Password | Cisco IOS Config | ⚠️ Type 7 obfuscation / Type 5 MD5 | ⚠️ Type 7 reversible instantly / Type 5 via rainbow table |
| DNS — Zone Transfert | DNS / AXFR | ❌ None | ✅ Full zone dump — all records exposed |
| LDAP — Null Bind | LDAP | ❌ None | ✅ Directory enumerated anonymously |
| IP — Time To Live | ICMP | ❌ None | ✅ TTL values visible in packet headers |
| POP-APOP | POP3 / APOP | ⚠️ MD5 challenge-response | ⚠️ Hash only — cracked via wordlist attack |
| SIP Authentication | SIP | ❌ None (PLAIN auth) | ✅ Plaintext credential in protocol field |
| OSPF Authentication | OSPF | ⚠️ MD5 cryptographic auth | ⚠️ Hash only — extracted via Ettercap, cracked with John |
| NTLM Authentication | NTLM / SMB | ⚠️ NTLMv2 HMAC-MD5 | ⚠️ Hash only — extracted via BruteShark, cracked with John |

---

## 📈 Skills Gained So Far

- Wireshark — Follow TCP Stream
- Wireshark — Packet layer inspection
- Wireshark — Protocol filtering (ftp, telnet, http, snmp, pop, icmp, ntlmssp)
- FTP, Telnet, HTTP Basic Auth, POP3/APOP, SIP, OSPF, NTLM/SMB protocol flows
- Base64 is encoding, not encryption
- SNMPv3 security levels (noAuthNoPriv / authNoPriv / authPriv)
- USM (User-based Security Model) in SNMPv3
- Hash vs plaintext — recognizing the difference in packet captures
- Offline hash cracking methodology with wordlists
- Nmap -Pn flag (bypass ping for unresponsive hosts)
- CTF methodology — enumerate, analyze, extract
- Cisco IOS password types (Type 0, 7, 5, 8, 9)
- Obfuscation vs encryption vs hashing — vocabulary and practical difference
- Cisco Type 7 decoding with online tools
- Hashcat and John the Ripper — wordlist and mask attacks
- Rainbow table / online hash lookup (hashes.com)
- Reading and analyzing Cisco IOS configuration files
- DNS zone transfer abuse (AXFR) — enumeration via misconfigured DNS
- dig — DNS query utility, server/domain/query-type distinction, TCP forcing
- DNS infrastructure concepts — server vs. zone vs. query type
- Reconnaissance impact of DNS misconfiguration — attack surface mapping
- LDAP protocol and directory services — null bind, DN structure, ou navigation
- Anonymous LDAP enumeration as a reconnaissance technique
- SOC detection relevance of null bind attempts in Active Directory environments
- TTL (Time To Live) — hop-based packet lifespan mechanism
- ICMP — network diagnostic and error communication protocol
- Request TTL vs reply TTL distinction in packet captures
- Traceroute behavior — TTL manipulation for network path mapping
- ICMP "Time Exceeded" as a SOC-detectable reconnaissance fingerprint
- APOP challenge-response authentication mechanism
- apop2john — hash extraction from pcap for format-aware cracking
- Independent verification of tool output via manual hash calculation
- Protocol age and cryptographic weakness — MD5 in legacy authentication
- Terminal output ambiguity — data vs. visual formatting assumptions
- SIP protocol — VoIP session establishment, REGISTER/INVITE/BYE methods
- HTTP Digest Authentication — SIP auth scheme borrowed from HTTP
- PLAIN auth misconfiguration in SIP — cleartext credential exposure
- OSPF — link-state routing protocol, router topology exchange
- OSPF authentication types — no auth, plaintext, MD5 cryptographic, SHA
- Crypto Auth TLV field — OSPF hash structure in Wireshark
- Ettercap — static pcap parsing for protocol-specific hash extraction
- Tool output debugging — escape sequences, file type verification
- NTLM challenge-response authentication — three-step handshake structure
- NTLMv2 hash format — manual construction from Wireshark fields
- userPrincipalName (UPN) — Active Directory identity format
- BruteShark — automated credential extraction from pcap files
- Self-doubt as a debugging strategy — questioning the input, not just the tool
- Pass-the-hash and credential harvesting awareness — SOC detection relevance

---

## 📖 Resources That Helped Me

| Resource | Purpose |
|---|---|
| Root-Me | CTF networking challenges platform |
| TryHackMe — Blue Team Path | Structured SOC analyst learning pathway |
| Wireshark Documentation | Protocol dissection reference and filter syntax |
| dig (man page) | DNS query utility — zone transfer and record-type reference |
| RFC 1939 | POP3 and APOP protocol specification |
| snmpv3brute | SNMPv3 pcap brute-force tool |
| apop2john | APOP hash extraction tool for John the Ripper |
| hashes.com | Online rainbow table lookup for MD5 and other hash types |
| Packet6 Type 7 Decryptor | Cisco Type 7 password reversal tool |
| Ettercap | OSPF MD5 hash extraction from static pcap files |
| BruteShark | Automated network credential extraction from pcap files |

---

*Last updated: March 2026 — 12 challenges validated | 1 attempted (SNMP)*
