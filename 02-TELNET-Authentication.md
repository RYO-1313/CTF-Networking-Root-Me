# ✅ Challenge 2 — TELNET Authentication

**Platform:** Root-Me  
**Category:** Network  
**Difficulty:** Very Easy  
**Status:** Completed ✅

---

## 🔍 What the Challenge Was

Given a `.pcap` file of a Telnet session. Goal was to extract the login credentials from the recorded traffic.

---

## 🧠 What I Learned

- **Telnet** is a remote access protocol that sends data in **plain text**
- Before the actual session, Telnet does a **negotiation phase** where client and server agree on features:

| Message | Meaning |
|---------|---------|
| WILL | "I want to enable this option" |
| WON'T | "I refuse this option" |
| DO | "Please enable this option" |
| DON'T | "Please disable this option" |

- The actual login happens **after** this negotiation phase
- The negotiation packets can look confusing — the real data comes after

---

## 🛠️ My Approach

1. Opened the `.pcap` file in Wireshark
2. Noticed negotiation packets at the top (WON'T Authentication etc.)
3. Scrolled past the negotiation traffic
4. Right clicked a Telnet packet → **Follow → TCP Stream**
5. Found the login credentials in the reconstructed stream

---

## 💡 Key Wireshark Skill Reinforced

**Follow TCP Stream** is useful for any unencrypted TCP protocol.

It will NOT work effectively on:
- HTTPS (encrypted)
- SSH (encrypted)
- VPN traffic (encrypted)

---

## 🔑 Core Takeaway

> Telnet, like FTP, offers zero encryption. Both protocols are considered legacy and insecure — replaced by SSH (for remote access) and SFTP (for file transfer) in modern systems.

---
