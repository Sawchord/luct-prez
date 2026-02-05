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
- Certificates embed some artifact that "proves" they have been logged
- Standardized in RFC 6962
- There are more ways to use CT but they have not seen adoption

---

![bg 70%](diagrams/CA_exchange.svg)

---
<!-- header: Situation today -->

![bg 70%](diagrams/CT_exchange.svg)

---

## And where is the log?

- Log operators maintain a merkle tree containing log entries
- Entries can be precerts or full certificates
- A signed certificate timestamp (SCT) promises to include the cert within maximum merge delay (MMD)
- Logs regularly publish current merkle root called signed tree head (STH)
- Public API to request inclusion proof (check that cert is in the log)
- Public API to request extension proof (check that new STH is extension of log)

---

## Situation in 2026

- Browsers maintain a list of log operators and logs they recognize
- Logs accept certificates of a certain expiration date (temporal sharding)
- Certs require 2 (if TTL < 180 Days) or 3 SCTs to be valid


---

## How do we know which logs to trust?

- Effectively follow Chromium's log list
- Governance via Mailing list

---

## Who is looking at the logs?

- Monitors / auditors tail logs
- Some offer search engines such as [crt.sh](crt.sh)
- Some offer to send E-Mails if certs are issued

---

## Situation in 2026

# Success!!!

We are done here arent we?

![bg right 80%](pics/htph_thumbs_up.jpg)

---

## What is left to do?
<!-- header: Current challenges -->

Spoiler alert: A lot!

---

## Current log operators

- Google (US)
- Cloudflare (US)
- Digicert (US)
- Sectigo (US)
- Let's Encrypt (US)
- TrustAsia (China)
- Geomys (Italy ??)
- IPng Networks (Switzerland)

--- 

## Logs v2

New RFC 9162

- Obsoletes RFC 6962
- Mostly concerst about cryptoagility
- **Was never adopted**

---

## Static-ct API

Problem: RFC 6962 API requires server to compile proofs
- Requires compuational resources on server
- Bad for caching

Solution: Store merkle tree in a static file format
- Read path of log is a static file server
- CDNs can cache the files

This is being rolled out as we speak. Currently RFC 6962 and static-ct logs coexist

---

## What about Gossiping?

- Forking is computationally cheap
- To prevent / detect forked logs requires clients to exchange STHs
- Existing standards are quite handwavy about this

---