# 🛡️ POP-APOP — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Very Easy

---

## 🔍 What the challenge was
A packet capture containing a POP3 session using APOP authentication. The objective was to recover the user's plaintext password from the network frame.

---

## 🧠 What I learned
I learned how the APOP challenge-response authentication mechanism works — the server sends a unique timestamp at the start of the session, the client combines it with the password and sends back an MD5 hash, never transmitting the password in cleartext. I also learned that despite being more secure than plain POP3 at the time of its design, APOP relies on MD5 which is now considered cryptographically weak and vulnerable to wordlist attacks. I learned to independently verify tool output rather than trusting results blindly, and discovered that terminal output can be visually misleading when password entries contain characters that resemble progress indicators.

---

## 🛠️ My Approach
I filtered the capture for POP traffic and followed the TCP stream, identifying an APOP authentication exchange containing a server-issued challenge string and an MD5 digest. I extracted the hash using apop2john and ran John the Ripper with the rockyou wordlist. John returned a result that appeared to be a progress indicator followed by a password — I misread the output and submitted the wrong value. After repeated failures, I independently verified the hash using a Python script, which confirmed that the full string including the apparent progress indicator was in fact the complete password entry in the wordlist. I submitted the correct value and validated the challenge.

---

## 💡 Key Skill Learned/Reinforced
Independent verification of tool output. When John the Ripper produced an unexpected result, rather than accepting it or abandoning the approach, I cross-validated using a manual Python MD5 calculation and a direct wordlist scan. This methodology caught a misreading that would otherwise have remained unresolved. Terminal output must be treated as data — visual assumptions about formatting can be incorrect.

---

## 🔑 Core Takeaway
APOP was designed as a security improvement over plaintext POP3 authentication, but its reliance on MD5 makes it vulnerable to offline wordlist attacks when network traffic is captured. From a SOC detection perspective, APOP traffic in a modern environment indicates legacy infrastructure — a potential attack surface that warrants attention. Protocols designed decades ago may have been reasonable at the time but no longer meet current security standards.

---

## 🤖 Claude's Role
Guided the APOP challenge-response concept from first principles. Assisted in diagnosing the contradiction between John's output and direct hash verification. Did not provide the password or flag at any point.
