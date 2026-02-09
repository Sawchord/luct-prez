---
theme: default
paginate: true
footer:
---

<style>
table {
    border-collapse:separate;
    border-radius:12px;
    border: 1px solid black;
}
th {
    
}
tr {
    border-bottom: 1px solid #dddddd;
}
tr:last-of-type {
    border-bottom: 2px solid #009879;
}
</style>


# A decade in certificate transparency and what may come next

---
# Whoami
<!-- header: Whoami -->

TODO

---

# Quick recap
<!-- header: Recap -->

- There are multiple good talks on CT. Go watch them!
    - [Everything you always wanted to know about Certificate Transparency](https://media.ccc.de/v/33c3-8167-everything_you_always_wanted_to_know_about_certificate_transparency)
    - [Who watches the watchers in Web PKI?](www.example.com)
- These talks focus on the protocol itself and the ongoing rollout
- This talk is about what happened since and what is on the agenda

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

## And where are the logs?

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
- Governance of CT ecosystem via mailing list


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
<!-- header: State of the union -->

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

<iframe class="cf-radar-embed" width="800" height="616" src="https://radar.cloudflare.com/embed/DataExplorerVisualizer?dataset=ct&path=ct%2Ftimeseries_groups%2Flog_operator&dateRange=52w&param_limitPerGroup=20&locale=en-US&widgetState=%7B%22xy.hiddenSeries%22%3A%5B%5D%7D&ref=%2Fexplorer%3FdataSet%3Dct%26groupBy%3Dlog_operator%26dt%3D52w" title="Cloudflare Radar - Certificates by CT log operator time series for Worldwide" loading="lazy" style="border:0;max-width:100%;">
</iframe>

---

## Who maintains the list of trusted logs?

- Chromiums `log_list.json` has effectively become the consensus
- Is served at [https://www.gstatic.com/ct/log_list/v3/log_list.json](https://www.gstatic.com/ct/log_list/v3/log_list.json)
- The schema is not part of the standard but ad-hoc by google
- The list is vendored into Chromium and Firefox (and Safari ??)

---

## Relying on Google's log list

Fetching the the log list live from Google is a recipe for disaster:

- Intercept call to the endpoint and replace log list with malicious logs
- Logs will approve the certificate of the request
- Basically back to the 2016 situation

---

## Relying on Google's log list

A way more trivial issue that actually happened:

- There is an android client library for CT enforcement
- Basic idea is that it has less issues than certificate pinning
- It used log list V2
- Google shuts it down, developers miss deprecation warning
- Millions of apps break
- 100s of millions of dollars go up in smoke

---

## Logs v2

New RFC 9162

- Obsoletes RFC 6962
- Mostly concerned about cryptoagility
- **Was never adopted :(**

---

## Static-ct API

Problem: RFC 6962 API requires server to compile proofs
- Requires computational resources on server
- Bad for caching

Solution: Store merkle tree in a static file format
- Read path of log is a static file server
- CDNs can cache the files

This is being rolled out as we speak. Currently RFC 6962 and static-ct logs coexist

---

## What about Gossiping?

- Forking is computationally cheap (this is not a blockchain)
- To prevent / detect forked logs requires clients to exchange STHs
- Existing standards are quite handwavy about this

---

## Gossip is not really a thing yet

- There is a document from IETF
    - [draft-ietf-trans-gossip-05](https://datatracker.ietf.org/doc/draft-ietf-trans-gossip/)
- It names 3 gossiping methods
- **Latest revision 2018-01-14 :(**

Not much momentum here right now.

---

## How about auditing?

- There is no cryptographic link between an SCT and the log entry
- Browsers do not check the log entries corresponding to an SCT
- Chromium seems to have a stochastic checking mechanism
- Firefox does nothing

Conclusion: 
- A state level actor could mount targeted attacks by coercing 1 CA + 2 CTs
- Very likely this will not be detected


--- 

## Why browsers don't check logs

Scability:
- It's an "interactive" proof against an STH
- What if logs are offline?
- ~100KB traffic per proof.
    Acceptable for user, but insane amount of traffic on a log

Privacy: 
- Asking a log for an inclusion proof gives away which website we are visiting

---

## What can we do?
<!-- header: luCT -->

- Browsers (other clients) should be able to inspect log artifacts privately
- Should not be the default behavior
- People at risk should be able to do this nonetheless
- There should be a gossiping mechanism

Note that the log data is public. We can build those things!
We don't need to ask permission!


---

## luCT

- Firefox extension
- Keeps record of STHs, checks extension proofs
- Fetches SCTs from TLS handshakes
- Fetches and validates inclusion proofs from logs

---

## luCT privacy

Oblivious proxy:

- Uses two layers of TLS (TLS stack compiled into wasm)
- Outer TLS connection to a proxy
- Sends WebSocket data, which is forwarded as raw TCP
- Uses inner TLS connection to connect to the log
- Proxy knows IP address, log knows requested SCT

**If you can, use proper VPN or TOR :P**

---

## luCT security policy

TODO: Explain the policy

---

## Planned features

TODO: Explain Gossiping

---

## Where do we stand?


Situation | Attack requirement | Tamper evident
-----|------|:------
CA only | control 1 CA | No
CA + CT (today) | control 1 CA + 2 CT | No 
CA + CT + luCT | control 1 CA + 2 CT  | **Yes**
CA + CT + luCT + Gossip | control 1 CA + 2 CT + peers | **Yes**

---

## Definitely try this at home!


- If you are a website admin: 
    - Check [crt.sh](crt.sh) and subscribe to E-Mail alert
- If you work for an organisation that has lots of computers (like a datacenter):
    - Consider becoming a log operator
- If you can put up with half backed software:
    - **Try out luCT!**

Lets get in touch: [matrix.lucius-labs.org](matrix.lucius-labs.org)

---

## Outlook
<!-- header: Outlook -->

The story does not end here!

Some ideas:

- Log management (sealing STHs, metalogs??)
- Use private information retrieval to fetch inclusion proofs
- Content addressable tiles

And transparency logs have also many applications outside of certificates, e.g.
- Binary transparency logs
- Key transparency logs

---

# TYSM!!

Questions?

<!-- header: Questions? -->