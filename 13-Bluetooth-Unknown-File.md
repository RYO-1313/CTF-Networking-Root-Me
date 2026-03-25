# 🛡️ Bluetooth — Unknown File — Root-Me Writeup

**Platform:** Root-Me  
**Category:** Network  
**Difficulty:** Easy

---

## 🔍 What the challenge was

A binary file recovered from a hacker's machine, originating from a Bluetooth communication between a computer and a smartphone. The objective was to identify the phone's MAC address and device name, concatenate them, and submit the SHA-1 hash of the result.

---

## 🧠 What I learned

The `.bin` format was initially unfamiliar, but identifying it as a BTSnoop capture file using the `file` command immediately clarified the approach. Once opened in Wireshark, the Bluetooth packet structure was readable using the same methodology applied to previous network captures — navigating packet fields to extract meaningful information.

---

## 🛠️ My Approach

Identified the file type using the `file` command, which revealed a BTSnoop version 1 HCI UART capture. Opened the file in Wireshark and navigated the 27 HCI-EVT packets, expanding packet details to locate the BD_ADDR field containing the device MAC address and the Remote Name field containing the phone model. Concatenated both values in the format specified by the challenge and generated the SHA-1 hash using the terminal.

---

## 💡 Key Skill Learned/Reinforced

Wireshark is protocol-agnostic — the same analytical methodology used for TCP, DNS, and LDAP captures applies directly to Bluetooth captures. File type identification is always the correct first step before attempting to open or analyze an unknown binary.

---

## 🔑 Core Takeaway

Bluetooth communications expose device metadata — including MAC addresses and device names — in cleartext during the pairing and discovery process. From a SOC perspective, this is relevant in environments where Bluetooth-based reconnaissance or unauthorized device pairing represents an attack surface.

---

## 🤖 Claude's Role

Guided the file identification process and Wireshark navigation methodology. Did not provide field names or packet contents directly — those were located independently through packet analysis.
