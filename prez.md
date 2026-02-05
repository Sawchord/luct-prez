---
theme: default
paginate: true
footer:
---

# A decade in certificate transparency and what may come next

---
# Whoami
<!-- header: Whoami -->


---

# Quick recap
<!-- header: Recap -->

- There are multiple good talks on CT. Go watch them!
    - [Everything you always wanted to know about Certificate Transparency, 33c3](https://media.ccc.de/v/33c3-8167-everything_you_always_wanted_to_know_about_certificate_transparency)
    - [Who watches the watchers in Web PKI?, emf2018](www.example.com)
- These talks focus on the protocol itself and the ongoing rollout
- I want to tell you what happens since and what is on the agenda

---

## Situation around 2016: 
Any root certificate authority (and many intermediate CAs) can issue a certificate for any website 

- There are ~180 root CAs in the trust store
- ~1800 intermediate CAs

Any of those could potentially maliciously issue a certificate which could be used to do a MitM attack on a TLS connection.

---

## This is not a theoretic threat!

Canonical example: [DigiNotar]() (2011)
TL;DR: 
- They got infiltrated
- Attackers issued certificates against 500+ domains
- Including `*.google.com`, `*.windowsupdate.com`, `*.mozilla.org`
- Very delayed response by the company
- They got kicked out of trust store and went bankrupt
- Certificates where used for attacks in Iran

--- 

## The problem:

Trusting CAs to keep a record of all the certificates they issued and investigate breaches is insufficient.

**We need a public record of all issued certificates!**

---

## Certificate transparency to the rescue!

Idea:
- Certificates need to be logged in a publicly verifiable audit log
- Certificates contain some artifact that "proves" they have been logged

---

![bg 70%](diagrams/CA_exchange.svg)

---

![bg 70%](diagrams/CT_exchange.svg)