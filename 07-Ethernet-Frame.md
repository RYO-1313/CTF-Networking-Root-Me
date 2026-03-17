# 🛡️ Ethernet Frame — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Easy

---

## 🔍 What the challenge was
A raw Ethernet frame provided as a hex dump. No pcap file, no Wireshark — the goal was to manually parse the frame structure, identify and extract embedded data, and recover credentials concealed within an HTTP request.

---

## 🧠 What I learned
- Raw challenge data is not always a pcap — sometimes it requires manual byte-level reading
- An Ethernet frame begins with Destination MAC (6 bytes), Source MAC (6 bytes), and EtherType (2 bytes), followed by the payload — the preamble and SFD are typically stripped in raw captures
- EtherType `86 DD` identifies IPv6 as the encapsulated Layer 3 protocol
- Converting hex to ASCII reveals human-readable content embedded in the payload
- HTTP Basic Authentication encodes credentials in Base64 format within the Authorization header — the same pattern encountered in the Twitter Authentication challenge

---

## 🛠️ My Approach
Analyzed the hex dump structurally by mapping the first 14 bytes to their respective Ethernet header fields: Destination MAC, Source MAC, and EtherType. Identified `86 DD` as IPv6. Converted the remaining payload from hex to ASCII, which revealed a raw HTTP GET request. Located the Authorization header containing a Base64-encoded string, decoded it, and recovered the credential pair — identifying the flag from the decoded output.

---

## 💡 Key Skill Learned/Reinforced
Manual Ethernet frame dissection — reading raw hex byte by byte and mapping values to protocol fields without relying on packet analysis tools. Reinforced the ability to recognize Base64-encoded HTTP Basic Auth credentials across different challenge formats.

---

## 🔑 Core Takeaway
Protocol knowledge must be applied structurally, not just conceptually. Knowing that an Ethernet frame has a Destination MAC, Source MAC, and EtherType field only becomes useful when you can locate and read those fields in raw data. This challenge also reinforced that authentication patterns repeat across protocols — recognizing Base64 in an Authorization header is a transferable skill regardless of the capture format.

---

## 🤖 Claude's Role
Guided the structural breakdown of the Ethernet frame without revealing field values. Redirected after an incorrect assumption that the decoded output was a hash. Encouraged independent research via YouTube when the byte-counting concept was unclear, then resumed from that foundation.
