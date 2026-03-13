# 🛡️ CISCO Password — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Easy

---

## 🔍 What the challenge was

A Cisco IOS configuration file is provided. The goal is to find the "Enable" password. The challenge hint reads: *"It's not always a hash."*

---

## 🧠 What I learned

- Cisco IOS configuration files store passwords in different formats depending on the command used and the IOS version
- **Type 7** is not real encryption — it is obfuscation using a known algorithm with a fixed key. It is fully reversible using online tools
- **Type 5** is an MD5-based hash — it cannot be reversed directly, only cracked via wordlist/brute force or looked up in a rainbow table database
- The key vocabulary distinction:
  - **Obfuscation** — hides data but is always reversible with a known algorithm (no key needed)
  - **Encryption** — requires a secret key to reverse
  - **Hashing** — one-way transformation; cannot be reversed, only cracked or looked up
- `service password-encryption` enables Type 7 obfuscation on plaintext passwords in the config — it does not provide real security
- `security passwords min-length 8` applies to the **plaintext password at the time it is set**, not to the stored hash or obfuscated value
- `enable secret` (Type 5 MD5) is stronger than `enable password` (Type 7 or plaintext)
- **privilege 15** on a Cisco device means highest access level — equivalent to root/enable access
- Online hash lookup databases (e.g., hashes.com) store previously cracked hashes — useful when wordlist cracking fails
- CTF methodology: test quick hypotheses first (small targeted wordlists, pattern-based masks) before going broad

```
# Cisco password types summary
Type 0  — Plaintext (no encryption)
Type 7  — Vigenère-based obfuscation (reversible, NOT secure)
Type 5  — MD5 hash (one-way, requires cracking or lookup)
Type 8  — SHA-256 (strong)
Type 9  — Scrypt (strongest)
```

---

## 🛠️ My Approach

1. Opened the challenge and received a Cisco IOS configuration file
2. Read the config line by line and identified all password-related entries:
   - `enable secret 5 $1$p8Y6$MCdRLBzuGlfOs9S.hXOp0.` — MD5 hash
   - `username hub password 7 025017705B3907344E`
   - `username admin privilege 15 password 7 10181A325528130F010D24`
   - `username guest password 7 124F163C42340B112F3830`
   - `line con 0 password 7 144101205C3B29242A3B3C3927`
3. Decoded all four Type 7 passwords using an online Type 7 decoder (Packet6):

```
hub     → 6sK0_hub
admin   → 6sK0_admin
guest   → 6sK0_guest
con 0   → 6sK0_console
```

4. Attempted decoded passwords as the flag — all rejected
5. Attempted to crack the Type 5 MD5 hash using hashcat:

```bash
hashcat -m 500 -a 0 hash.txt /path/to/wordlist
```

6. Encountered GPU memory errors on Kali VM — attempted workarounds:

```bash
hashcat -m 500 -a 0 --force -D 1 hash.txt wordlist
```

7. Switched to John the Ripper and attempted a mask attack based on the `6sK0_` pattern observed in Type 7 passwords:

```bash
john --mask='6sK0_?a?a?a' hash.txt
```

8. Mask attack exhausted with no result — hypothesis disproved
9. Pivoted strategy: used hashes.com online rainbow table lookup with the full MD5 hash
10. Hash was found in the database — password recovered and validated

---

## 💡 Key Skill Learned/Reinforced

**Online rainbow table lookup as a cracking strategy.**

When local wordlist and brute force attacks fail — especially in a VM with limited GPU resources — online hash databases like hashes.com are a viable alternative. These databases store previously cracked hash:password pairs submitted by the community. If the password has ever been cracked before, it will likely be in the database instantly — no compute required.

---

## 🔑 Core Takeaway

> Not all password storage is equal. Type 7 "encryption" in Cisco configs is obfuscation — trivially reversible. Type 5 MD5 is a real hash, but MD5 is weak enough that rainbow tables make it instantly recoverable. The challenge hint "It's not always a hash" was a reminder to look beyond the obvious attack path.

---

## 🤖 Claude's Role

Claude acted as a guide and concept explainer throughout the challenge. The most impactful contributions were prompting research into Cisco password types and their security levels before touching the challenge, and introducing the concept of online hash lookup databases (rainbow tables) as an alternative when local cracking tools were failing due to VM resource constraints. Claude never provided the password or flag — all decoding, tool execution, and discovery was done independently.
