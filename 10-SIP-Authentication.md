# 🛡️ SIP Authentication — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Very Easy

## 🔍 What the challenge was
A SIP protocol exchange captured in a text-based log file. The goal was to identify credentials used during a VoIP session involving REGISTER, INVITE, and BYE methods.

## 🧠 What I learned
- SIP (Session Initiation Protocol) is used to establish, manage, and terminate VoIP/video calls
- SIP borrows its authentication model from HTTP — specifically HTTP Digest Authentication
- Authentication schemes can vary within the same capture: some exchanges use MD5 hashing, others use PLAIN (cleartext) transmission
- PLAIN auth transmits credentials with no encoding, no hashing, and no obfuscation — the password is directly readable in the protocol field

## 🛠️ My Approach
Opened the text-based SIP log and identified three exchanges: REGISTER, INVITE, and BYE. Initially focused on the MD5 hashes present in the INVITE and BYE lines, assuming the credential would require cracking. Recognized that the REGISTER exchange used PLAIN authentication — and that the credential was already visible in cleartext within the log, requiring no further analysis.

## 💡 Key Skill Learned/Reinforced
Reading protocol fields critically and resisting the assumption that credentials are always hashed or encoded. Pattern recognition from previous challenges (MD5 hashes) almost caused me to overlook a plaintext credential sitting in plain sight.

## 🔑 Core Takeaway
Not all authentication is obfuscated. PLAIN auth over SIP transmits passwords in cleartext — any attacker or analyst with access to network traffic can extract credentials instantly, with no tooling required. In a real SOC environment, SIP traffic using PLAIN auth is a critical misconfiguration: VoIP credential harvesting requires zero effort when the protocol itself does the work for the attacker.

## 🤖 Claude's Role
Guided conceptual understanding of SIP authentication schemes and HTTP Digest Auth without revealing the solution. Prompted critical observation of protocol field differences across the three captured exchanges. The credential identification and flag validation were performed independently.
