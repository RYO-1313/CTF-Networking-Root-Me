# ✅ Challenge 1 — FTP Authentication

**Platform:** Root-Me  
**Category:** Network  
**Difficulty:** Very Easy  
**Status:** Completed ✅

---

## 🔍 What the Challenge Was

Given a `.pcap` packet capture file containing recorded FTP network traffic. The goal was to find the credentials used during the FTP session.

---

## 🧠 What I Learned

- **FTP (File Transfer Protocol)** runs on port 21 (control) and port 20 (data)
- FTP sends **everything in plain text** — including usernames and passwords
- This makes it extremely insecure on modern networks
- The FTP login sequence follows this pattern:

```
Server → Client : 220 Welcome
Client → Server : USER <username>
Server → Client : 331 Password required
Client → Server : PASS <password>
Server → Client : 230 Login successful
```

---

## 🛠️ My Approach

1. Opened the `.pcap` file in Wireshark
2. Filtered traffic by FTP protocol using the Wireshark filter bar:ftp
3. Used **Follow → TCP Stream** to reconstruct the full conversation
4. Read the plain text credentials directly from the stream

---

## 💡 Key Wireshark Skill Learned

**Follow TCP Stream** — Right click any packet → Follow → TCP Stream

Reassembles all packets in a conversation into one readable window.  
Color coding: 🔴 Red = client traffic, 🔵 Blue = server traffic

---

## 🔑 Core Takeaway

> Any protocol that doesn't encrypt its traffic is vulnerable to credential theft through packet capture. FTP is a textbook example of this.

---
