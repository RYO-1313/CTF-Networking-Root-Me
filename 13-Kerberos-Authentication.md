# 🛡️ Kerberos Authentication — Root-Me Writeup

**Platform:** Root-Me  
**Category:** Network  
**Difficulty:** Medium  
**Status:** ❌ Not Validated

---

## 🔍 What the challenge was

A pcap capture from Cat Corporation's SOC team containing suspicious Kerberos authentication traffic. The objective was to identify a user's credentials through analysis of the Kerberos exchange and recover the password via offline hash cracking.

---

## 🧠 What I learned

Developed a thorough understanding of the Kerberos authentication protocol — from the conceptual flow to the specific hash formats involved. Learned the critical distinction between two Kerberos attack surfaces: AS-REQ pre-authentication hashes (`krb5pa`) and TGS service ticket hashes (`krb5tgs`). Initially confused the two formats during hash extraction, which led to attempting to crack the wrong hash type. Identified the correct target as the AS-REQ pre-authentication hash embedded in the `pA-ENC-TIMESTAMP` field of the AS-REQ packet, encrypted with AES256 (etype 18).

---

## 🛠️ My Approach

Filtered Kerberos traffic in Wireshark and identified both AS-REQ/AS-REP and TGS-REQ/TGS-REP packet exchanges. Confirmed pre-authentication was enabled by locating the `pA-ENC-TIMESTAMP` field in the AS-REQ packet. Identified the username `william.dupond` and domain `CATCORP.LOCAL` from the cname field. Used BruteShark to extract the Kerberos hash from the pcap. Attempted offline cracking with hashcat using multiple wordlists including rockyou, SecLists corporate passwords, and leaked database wordlists — all exhausted without recovering the password. Challenge was not validated due to inability to crack the AES256 hash within available computational resources.

---

## 💡 Key Skill Learned/Reinforced

The difference between Kerberoasting (targeting TGS service ticket hashes) and AS-REQ roasting (targeting pre-authentication hashes) is critical — both produce crackable offline hashes but from different stages of the Kerberos exchange and with different hash formats. AES256 (etype 18) hashes are significantly harder to crack than RC4 (etype 23) and require substantial computational resources without a GPU.

---

## 🔑 Core Takeaway

Kerberos pre-authentication exists as a security control — when disabled, it enables AS-REP roasting without any credentials. When enabled, the encrypted timestamp in AS-REQ can still be targeted if captured. From a SOC perspective, unusual Kerberos TGS requests for sensitive services or accounts with weak passwords are indicators of Kerberoasting activity, detectable through volume-based alerting on TGS-REQ traffic.

---

## 🤖 Claude's Role

Explained the conceptual difference between AS-REQ roasting and Kerberoasting, guided Wireshark filtering for Kerberos traffic, and assisted with hash format identification and hashcat mode selection. The hash extraction and all cracking attempts were performed independently.
