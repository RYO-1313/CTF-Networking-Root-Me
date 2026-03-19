# 🛡️ LDAP — Null Bind — Root-Me Writeup

**Platform:** Root-Me
**Category:** Network
**Difficulty:** Medium

---

## 🔍 What the challenge was

A misconfigured LDAP server exposed a directory containing user information including an email address. The objective was to retrieve that email by exploiting anonymous (null bind) access — without any credentials.

---

## 🧠 What I learned

- **LDAP** (Lightweight Directory Access Protocol) is a protocol designed for directory services — it organizes and exposes structured information such as users, groups, and organizational units in a hierarchical tree format.
- A **null bind** occurs when an LDAP server accepts anonymous connection requests without requiring authentication credentials. This is a misconfiguration that can expose sensitive directory data to any unauthenticated user.
- LDAP uses its own **URI format** (`ldap://host:port`) distinct from standard HTTP URLs — a syntax distinction that affects how command-line tools are invoked.
- LDAP directory entries follow a strict **Distinguished Name (DN) hierarchy**: the most specific component (such as `ou=`) is written first, followed by the domain components (`dc=`). Reversing this order results in lookup failure.
- Directory trees are organized into **branches** using Organizational Units (`ou=`). Knowing how to navigate and target the correct branch is essential to retrieving data from the right part of the tree.

---

## 🛠️ My Approach

After reading the challenge statement, I identified the vulnerability as anonymous LDAP access (null bind) and researched the appropriate Linux command-line tool for querying LDAP servers. I installed `ldap-utils` to gain access to `ldapsearch`.

My initial approach was to query the base DN with a `base` scope, which returned an insufficient access error — the server was responding but restricting access to the root entry. I adjusted the search scope to `subtree` to enumerate the full directory tree. After receiving another access error, I recognized that the server was likely restricting broad enumeration and that I needed to target a specific branch directly.

Based on the challenge statement referencing an "anonymous" user, I constructed a DN targeting `ou=anonymous` under the base domain. After correcting the component order in the DN (specific-to-general, left-to-right), the server returned two entries: the organizational unit itself and a user entry containing a name, uid, and email address.

---

## 💡 Key Skill Learned/Reinforced

**LDAP directory navigation and null bind exploitation** — understanding how directory trees are structured, how to construct valid Distinguished Names, and how to enumerate accessible branches when broad queries are restricted.

---

## 🔑 Core Takeaway

A null bind misconfiguration can expose sensitive organizational data — including user emails, usernames, and department structures — to any unauthenticated party. From a SOC analyst perspective, anonymous LDAP bind attempts against internal directory servers are a reconnaissance indicator worth monitoring. An attacker enumerating an Active Directory-backed LDAP service this way could harvest valid email addresses and usernames for phishing or credential stuffing attacks.

---

## 🤖 Claude's Role

Claude did not provide syntax or commands directly. Guidance was limited to conceptual direction: understanding what a null bind is, how LDAP URIs differ from standard URLs, and how DN component ordering works. All commands were constructed and tested independently. The DN ordering issue and branch naming were reasoned through without being given the answer.
