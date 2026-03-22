# 🛡️ NTLM Authentication — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Medium

## 🔍 What the challenge was
A network packet capture containing NTLM authentication over SMB traffic. The objective was to recover the password of a suspicious user account and submit it in userPrincipalName format as the flag.

## 🧠 What I learned
- NTLM (New Technology LAN Manager) is a challenge-response authentication protocol used in Active Directory environments
- NTLM authentication follows a three-step handshake: NTLMSSP_NEGOTIATE, NTLMSSP_CHALLENGE, NTLMSSP_AUTH — each packet serving a distinct role in the exchange
- NTLMv2 uses HMAC-MD5 for stronger security than NTLMv1, but remains vulnerable to offline hash cracking if traffic is captured
- userPrincipalName (UPN) is the standard Active Directory login format combining username and domain (username@domain) — a critical identifier in SOC alert triage and Windows Event Log analysis
- Manual hash construction from Wireshark fields is error-prone — a truncated NTLMv2 response will silently fail to crack without indicating why
- BruteShark automates protocol credential extraction from pcap files, eliminating manual assembly errors

## 🛠️ My Approach
Opened the capture in Wireshark and filtered for NTLMSSP traffic. Identified two NTLM handshake cycles and extracted credentials from the second NTLMSSP_AUTH packet: username john.doe, domain catcorp.local. Retrieved the server challenge from the NTLMSSP_CHALLENGE packet and manually assembled the NTLMv2 hash in John-compatible format. Attempted cracking with John the Ripper and rockyou, then hashcat with best66 rules — both exhausted without result. Recognized that the manual hash may have been incorrectly constructed, switched to BruteShark for automated extraction, obtained the complete hash, and successfully cracked it with John the Ripper and the rockyou wordlist.

## 💡 Key Skill Learned/Reinforced
Self-doubt as a debugging strategy. When cracking attempts failed across multiple tools and wordlists, the correct next step was to question the hash itself rather than continue exhausting wordlists. Identifying the root cause — truncated manual extraction — and switching to an automated tool resolved the issue immediately.

## 🔑 Core Takeaway
NTLM authentication over SMB is a high-value target in Active Directory environments. Captured NTLMv2 hashes are crackable offline with no interaction required from the target. From a SOC perspective, unexpected SMB connections, NTLM authentication attempts from unusual hosts, or lateral movement patterns involving NTLM are detectable indicators of credential harvesting or pass-the-hash activity.

## 🤖 Claude's Role
Guided understanding of the NTLM handshake structure, NTLMv2 hash format, and userPrincipalName concept. Prompted critical evaluation of failed cracking attempts . Tool discovery, hash extraction, cracking, and flag recovery were performed independently.
