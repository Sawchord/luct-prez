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
.columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
</style>
<link href="https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet"/>


# A decade in certificate transparency and what may come next

---
# Whoami
<!-- header: Whoami -->

TODO

---
# Recap

---

# Prior talks
<!-- header: Recap -->

- There are multiple good talks on CT. Go watch them!
    - [Everything you always wanted to know about Certificate Transparency](https://media.ccc.de/v/33c3-8167-everything_you_always_wanted_to_know_about_certificate_transparency)
    - [Who watches the watchers in Web PKI?](www.example.com)
- These talks focus on the protocol itself and the ongoing rollout
- This talk will focus on what happened since

---
## Motivation

# Why Certificate Transparency?

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

## Certificate transparency

- Log operators include logs in a merkle tree
- Entries can be precerts or full certificates (cross-posting)
- Logs regularly publish a signed tree head (STH)

---

## Validating logs

- Request and validate signed tree head against public key
- Request inclusion proof, check that cert is in the log
- Request extension proof, check that new STH is extension of log

---

## Who looks at logs?
- Monitors tail logs
- Some offer search engines such as [crt.sh](crt.sh)
- Some offer to send E-Mails if certs are issued
- Governance of CT ecosystem via mailing list

---

## Situation in 2026

- Browsers maintain a list of logs (url + pubkey) they recognize
- Logs accept certificates of a certain expiration date (temporal sharding)
- Certs require 2 (if TTL < 180 Days) or 3 SCTs to be valid

---

## Situation in 2026

# Success?

CT is widely deployed.
We are done here arent we?

![bg right 80%](pics/htph_thumbs_up.jpg)

---

## What is left to do?

Spoiler alert: A lot!

- Some ideas from original design have not caught on
- Some issues emerged over time

---
<!-- header: The ecosystem -->

![bg 70%](diagrams/CT_ecosystem_ideal.svg)

---

## Current log operators
<!-- header: Log operators -->

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
- The schema is not part of the standard but ad-hoc by Google
- The list is vendored into Chromium and Firefox
- Safari: [https://valid.apple.com/ct/log_list/current_log_list.json](https://valid.apple.com/ct/log_list/current_log_list.json)

---

## How to get on the log list?

- Write an application to Chromium (via bug tracker)
- Submit contact info, pubkeys, urls, maximum merge delay etc,
- Keep the log running, Google will test is regularly
- After a while you will get included
- Maintain 99% uptime

---

## Issues with log lists
<!-- header: Log lists -->

- Log lists are not part of the CT standard
- Ad-hoc invention by chromium

This has lead to some problems

---

## Issues with log lists

- Supposedly alternative to certificate pinning for Apps
- There is an android library for CT enforcement
- Google changes schema from v2 to v3
- Developers forget to update library
- App breaks, 100+ Million of user affected

---

## Issues with log lists

- Google added two fake logs to v2 called mimics
- Published the private keys
- Allows CAs to circumvent CT for clients that rely on the log list
---

## Issues with log lists
Log lists have a kind of chicken and egg problem

- Clients must vendor the list 
- Distribute it using established update channels
- **How to secure the update channel?**

**It can not replace certificate pinning!**

---

## Non web browser clients

- Google claims that CT should only be used in browsers

But on the other hand:

- **CT could likely have prevented Notepad++ attack**

---

## Logs v2
<!-- header: Standards -->

New RFC 9162

- Obsoletes RFC 6962
- Mostly concerned with cryptoagility
- 5 years of delay
- "Too little, too late" - The authors
- **Was never adopted :(**

---

## Static-ct API

Problem: RFC 6962 API requires server to compile proofs
- Requires computational resources on server
- Bad for caching

Solution: Store merkle tree in a static file format
- Read path of log is a static file server
- Can be georeplicated
- CDNs can cache the files

**This is being rolled right now. Currently RFC 6962 and static-ct logs coexist**

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

Not much momentum here right now

---

## How about auditing?
<!-- header: Auditing -->

- There is no cryptographic link between an SCT and the log entry
- Browsers do not check the log entries corresponding to an SCT
- Chromium seems to have a random checking mechanism
- External auditing services exist but not used widely

--- 

## Why browsers don't check logs

Scalability:
- It's an "interactive" proof against an STH
- What is the failure mode if logs are offline?
- ~100KB traffic per proof.
    Acceptable for user, but insane amount of traffic on a log

Privacy: 
- Asking a log for a proof gives away which website is visited

---

![bg 70%](diagrams/CT_ecosystem_real.svg)

---

### Conclusion

A threat actor could mount an attack by coercing 1 root CA + 2 CT logs

- Fake SCTs are very likely not be detected
- Selectively serving a forked log will very likely not be detected

It is relatively easy to become a log operator (and it should be!).

---

## What can we do?
<!-- header: luCT -->

- Browsers (and other clients) should be able to inspect the log
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

TODO: Gif showing the tool

---

## luCT privacy

Oblivious proxy (WIP):

- Uses two layers of TLS (TLS stack compiled into wasm)
- Outer TLS connection to a proxy
- Sends WebSocket data, which is forwarded as raw TCP
- Uses inner TLS connection to connect to the log
- Proxy knows IP address, log knows requested SCT

**If you can, use proper VPN or TOR :P**

---
## luCT security policy

- Let N = 2 if TTL $\leq$ 180d, else N = 3
- Cert contains N SCTs from known logs with matching signatures
- K SCTs validate against log
- SCT validates against log iff:
    - There is STH newer than 24h or log is readonly
    - SCT validated against oldest possible STH
    - Extension proof of old STH to new STH was checked
    - K = 1 if validated against STH $\geq$ 24h, else K = N
    - STH age by own timestamps NOT included timestamps

---

## luCT gossip

Nope, not yet!

**Idea:**
- Fetch STH timeline from network of checkpointing servers
- Validate extension proofs on timeline

---

## Threat model

Scenario | Attack requirement | Attack
-----|------|:------
CA only (2016) | 1 CA | Rogue cert
CA + CT (today) | 1 CA + 2 CT | Rogue cert + fake SCTs
CA + CT + luCT (soon) | 1 CA + 2 CT  | Rogue cert + forked logs
CA + CT + luCT + Gossip | 1 CA + 2 CT + checkpointer | Full eclipse?

---

![bg 70%](diagrams/CT_ecosystem_luct.svg)

---

## There is one issue left

What if an attacker just submits a rogue certificate to honest logs, but the domain owners don't care to ever check logs for rogue certificates?

**Idea:**

- Check CAA (RFC 8659) and TLSA (RFC 6698) records via DNS over HTTPs
- Display special icon if entries match

---

## Definitely try this at home!

- If you are a website admin / domain owner: 
    - Check [crt.sh](crt.sh) and subscribe to E-Mail alert
- If you have lots of compute:
    - Consider becoming a log operator
- If you can put up with half backed software:
    - **Try out luCT!**


---

## Outlook
<!-- header: Outlook -->

The story does not end here!

Some ideas:

- CT logs with private information retrieval
- Sealing STHs
- Better Log list management
- Content addressable tiles
---

## Outlook

And transparency logs have also many applications outside of certificates, e.g.
- Binary transparency logs
- Key transparency logs

These systems have been deployed to some degree, e.g. Go package manager, WhatApp

---

<!-- header: Questions? -->

<div class="columns">
<div>

# TYSM!!

Questions?

</div>
<div>

## Lets get in touch!

<i class="fa fa-at" aria-hidden="true"></i> [luct.dev](luct.dev)

<i class="fa fa-github" aria-hidden="true"></i> [github.com/Sawchord/luct](https://github.com/Sawchord/luct)

<i class="fa fa-comment" aria-hidden="true"></i> [matrix.luct.dev](matrix.luct.dev)

</div>
</div>

---

# Bonus slides!

---

## Cosmic rays vs. certificate transparency