# 🛡️ SNMP - Authentification — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Easy

---

## 🔍 What the challenge was
A packet capture (.pcap) file containing SNMP traffic. The goal was to extract authentication credentials from the capture. The challenge involved understanding how SNMP handles authentication across its different versions and security models.

---

## 🧠 What I learned
- SNMP has three versions with very different security models:
  ```
  SNMPv1 → No authentication, community string in plaintext
  SNMPv2 → Improved performance, still uses plaintext community strings
  SNMPv3 → Introduced USM (User-based Security Model) with real authentication and optional encryption
  ```
- SNMPv3 has three security levels:
  | Level | Auth | Encryption | Meaning |
  |-------|------|------------|---------|
  | noAuthNoPriv | ❌ | ❌ | No security |
  | authNoPriv | ✅ | ❌ | Authenticated but visible |
  | authPriv | ✅ | ✅ | Fully secured |
- The challenge used **SNMPv3 with authNoPriv** — authenticated but not encrypted
- In SNMPv3, the password is **never sent in plaintext** — it is hashed using HMAC-MD5 or HMAC-SHA and stored in the `msgAuthenticationParameters` field
- The `msgUserName` field travels in plaintext and revealed the username: `user`
- Cracking SNMPv3 requires extracting the hash from the pcap and running it against a wordlist — tools like `snmpv3brute` are designed specifically for this
- **Follow TCP Stream does not work** for SNMPv3 the same way it does for FTP or Telnet — the credential is a hash, not cleartext

---

## 🛠️ My Approach
1. Opened the pcap file in Wireshark
2. Attempted Follow TCP Stream — saw unreadable data, which indicated a different protocol behavior than previous challenges
3. Inspected individual SNMP packets and expanded the SNMP layer
4. Identified the SNMP version as **SNMPv3** and the security model as **USM (User-based Security Model)**
5. Found key flags in the packet: `reportable not set`, `encrypted not set`, `auth set` — confirming **authNoPriv**
6. Located the `msgUserName` field — revealed username: `user`
7. Located the `msgAuthenticationParameters` field — confirmed it contains a hash, not a plaintext password
8. Researched SNMPv3 hash cracking and identified `snmpv3brute` as the right tool
9. Attempted to crack the hash using the rockyou wordlist:
   ```bash
   python3 snmpv3brute.py -w /usr/share/wordlists/rockyou.txt -p ch16.pcap
   ```
10. Challenge not validated — wordlist did not contain the password

---

## 💡 Key Skill Learned/Reinforced
**Reading SNMPv3 packet structure in Wireshark** — specifically identifying the security level by inspecting the `msgFlags` field and USM section, and understanding the difference between `msgAuthenticationParameters` (a hash) and a plaintext credential.

---

## 🔑 Core Takeaway
> Not all protocols expose credentials in plaintext. SNMPv3 with authNoPriv authenticates using a hash — you can see the traffic but must crack the hash offline with a wordlist tool. Methodology shifts from "read the stream" to "extract and crack."

---

## 🤖 Claude's Role
Claude acted as a concept guide throughout the challenge. Rather than giving answers, Claude asked questions that pushed me to research SNMP versions, security levels, and the USM model myself. Claude helped me understand why Follow TCP Stream wouldn't work here, what `msgAuthenticationParameters` actually contains, and directed me toward the right cracking approach using `snmpv3brute`. The flag was not obtained, but the protocol understanding was built through my own research and Wireshark exploration.
