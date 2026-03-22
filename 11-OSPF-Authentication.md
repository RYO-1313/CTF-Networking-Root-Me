# 🛡️ OSPF Authentication — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Medium

## 🔍 What the challenge was
A network packet capture containing OSPF traffic with cryptographic MD5 authentication. The goal was to extract the OSPF authentication key used between routers.

## 🧠 What I learned
- OSPF (Open Shortest Path First) is a link-state routing protocol used by routers to exchange topology information and calculate efficient paths across a network
- OSPF supports multiple authentication types: no auth, plaintext, MD5 cryptographic, and SHA
- OSPF MD5 authentication hashes are embedded in packets using the Crypto Auth TLV field — the key itself is never transmitted, only the hash
- Ettercap, typically known as a live MITM tool, can also parse static pcap files and extract protocol-specific hashes in John-compatible format
- Tool output must be verified before use — escape sequences in terminal output can obscure actual data and require cleanup before processing

## 🛠️ My Approach
Opened the capture in Wireshark and identified OSPF packets with cryptographic (Type 2) MD5 authentication. Recognized the need to extract the hash into a crackable format before attempting to crack it. Evaluated multiple tools including PCAP2Hash (WiFi-only, not applicable) and an ospf-md5-cracker repo before confirming Ettercap as the appropriate extraction tool for static pcap files. Debugged unreadable output caused by terminal escape sequences using file verification commands. Extracted clean hash lines in $netmd5$ format and cracked the authentication key using John the Ripper with the rockyou wordlist.

## 💡 Key Skill Learned/Reinforced
Transferring a known methodology to a new protocol. The extract-then-crack workflow established in POP-APOP applied directly here — identify the hash, find the right extraction tool, format correctly, crack with John. Recognizing pattern continuity across protocols accelerates problem-solving.

## 🔑 Core Takeaway
OSPF MD5 authentication is crackable offline if an attacker captures routing traffic — and weak or dictionary-guessable keys provide no real protection. In a SOC environment, unauthorized OSPF adjacency formation or unexpected routing updates are detectable indicators of route poisoning attempts, a serious threat that can redirect enterprise traffic through attacker-controlled infrastructure.

## 🤖 Claude's Role
Guided understanding of OSPF authentication types and the reasoning behind cryptographic auth in routing protocols. Prompted critical evaluation of tool selection before execution. Tool discovery, hash extraction, debugging of output issues, and flag recovery were performed independently.
