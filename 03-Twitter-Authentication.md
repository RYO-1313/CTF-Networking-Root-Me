# ✅ Challenge 3 — Twitter Authentication

**Platform:** Root-Me  
**Category:** Network  
**Difficulty:** Very Easy  
**Status:** Completed ✅

---

## 🔍 What the Challenge Was

Given a `.pcap` file containing a single HTTP packet of a Twitter login. Goal was to extract the credentials from the packet.

---

## 🧠 What I Learned

- **HTTP Basic Authentication** sends credentials in the request header
- The credentials are **Base64 encoded** — NOT encrypted:

```
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

- Base64 is encoding, not encryption — anyone can decode it instantly
- Wireshark automatically detects and decodes Basic Auth, displaying it as:

```
Credentials: username:password
```

- This is why **HTTPS** exists — to encrypt this data in transit

---

## 🛠️ My Approach

1. Opened the `.pcap` file in Wireshark
2. Only one packet present — expanded the packet layers in the details panel:

```
▶ Frame
▶ Ethernet
▶ Internet Protocol
▶ Transmission Control Protocol
▶ Hypertext Transfer Protocol   ← expanded this
```

3. Found the `Credentials` field automatically decoded by Wireshark
4. Read the username and password directly

---

## 💡 Key Wireshark Skill Learned

Expanding **packet layers** in the details panel reveals protocol-specific information that isn't visible at the packet list level.

---

## 🔑 Core Takeaway

> HTTP Basic Auth offers no real security — Base64 is trivially reversible. Always use HTTPS to protect credentials in transit.

