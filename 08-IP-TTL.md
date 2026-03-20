# 🛡️ IP — Time To Live — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Very Easy

---

## 🔍 What the challenge was
A packet capture containing an ICMP exchange between a source and a target host. The objective was to identify the TTL value used by the source to successfully reach the targeted host.

---

## 🧠 What I learned
I learned the concept of TTL (Time To Live) as a packet lifespan mechanism — a counter that decrements at each router hop, and when it reaches zero, the router discards the packet and sends back an ICMP "Time Exceeded" error. I also learned what ICMP is: a network-layer protocol used not for data transfer, but for diagnostic and error communication between devices — the protocol behind ping and traceroute.

---

## 🛠️ My Approach
I opened the capture file in Wireshark and observed that all packets were ICMP, with approximately 70 "Time-to-live exceeded" error messages present. I initially identified TTL=51 as my answer, but realized I was reading the **reply** packet from the target rather than the **outgoing request** from the source. After re-reading the challenge statement carefully, I located the successful outgoing request and identified the correct TTL value.

---

## 💡 Key Skill Learned/Reinforced
Understanding the difference between **request TTL** and **reply TTL** in a packet exchange. The reply carries the target's own TTL — a separate value set by the responding machine — which is not the same as the TTL used by the source to reach it. Reading the challenge statement precisely proved as important as the technical analysis itself.

---

## 🔑 Core Takeaway
A high volume of ICMP "Time Exceeded" messages in a packet capture is a recognizable fingerprint of traceroute-based reconnaissance — an attacker systematically mapping network hops to a target by deliberately sending packets with incrementing TTL values. From a SOC detection perspective, this pattern warrants investigation as active network path discovery.

---

## 🤖 Claude's Role
Guided me through building the concept of TTL and ICMP from my own reasoning before the challenge. Corrected my initial answer by helping me distinguish between request and reply TTL. Did not provide the flag or direct solution.
