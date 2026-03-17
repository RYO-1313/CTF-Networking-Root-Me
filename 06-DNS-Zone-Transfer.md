# 🛡️ DNS — Zone Transfert — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Easy

---

## 🔍 What the challenge was

A DNS service was intentionally misconfigured for the domain `ch11.challenge01.root-me.org`, hosted on a non-standard port. The objective was to exploit an unrestricted DNS Zone Transfer (AXFR) to enumerate all DNS records within the zone and retrieve the flag embedded in the zone data.

---

## 🧠 What I learned

The most significant conceptual shift in this challenge was distinguishing between three terms I initially treated as interchangeable: **server**, **domain**, and **query**.

- A **DNS server** is the machine listening for and responding to DNS requests — identified by an IP address and port
- A **domain** (or zone) is the subject of the request — the namespace whose records are being queried
- A **query type** defines what kind of information is being requested about that domain — such as an IP address record, a mail server record, or in this case, a full zone transfer

Understanding this distinction was the key that unlocked the rest of the challenge. Once those three concepts were separated, building the correct tool invocation became straightforward.

I also learned that DNS zone transfers are a legitimate administrative mechanism — they exist so that secondary DNS servers can synchronize their records from a primary server. When this mechanism is left unrestricted, however, any external party can request the entire zone, exposing every subdomain, internal hostname, and service record the organization has configured. This is a reconnaissance-enabling misconfiguration, not an attack in itself — but it hands an attacker a complete map of the attack surface before they have touched a single system.

Additionally, I discovered that AXFR zone transfers require TCP, not the UDP protocol that standard DNS queries use. This is because a full zone dump can be too large for a single UDP datagram.

---

## 🛠️ My Approach

The challenge provided a target domain and a non-standard port. Rather than immediately reaching for a tool, I first worked to understand what a zone transfer actually is and why an unrestricted one constitutes a vulnerability.

Once the concept was clear, I consulted the manual page for `dig` — a standard Linux DNS query utility — to identify the correct flags for specifying a target server, a non-standard port, a query name, a query type, and a TCP transport requirement. Reading the tool documentation directly proved more effective than searching for pre-built commands, as it reinforced understanding of each component rather than blind execution.

After a few iterations resolving connection issues — including correcting the direction of the server versus domain arguments and explicitly forcing TCP — the zone transfer returned the full record set for the target domain, which contained the flag.

---

## 💡 Key Skill Learned/Reinforced

**DNS enumeration via zone transfer abuse** — understanding that DNS infrastructure, when misconfigured, can expose an organization's entire internal naming structure in a single query, and knowing how to identify and execute an AXFR request using native Linux tooling without relying on automated scanners.

---

## 🔑 Core Takeaway

A DNS zone transfer misconfiguration does not grant direct access to any system — but it eliminates the reconnaissance phase of an attack entirely. From a SOC perspective, detecting anomalous AXFR requests in DNS logs is a meaningful early-warning signal. From a defensive posture, the remediation is straightforward: restrict zone transfer responses to known, authorized secondary DNS server IP addresses only.

---

## 🤖 Claude's Role

Claude guided the conceptual understanding of DNS zone transfers without providing commands or syntax. Key distinctions — server vs. domain vs. query type, UDP vs. TCP, and the difference between querying a server and querying a domain — were explained through dialogue after I encountered each issue independently. The final command structure was assembled entirely through my own iteration.
