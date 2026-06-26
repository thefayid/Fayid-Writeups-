# TryHackMe — TakeOver (Writeup)

- Platform: TryHackMe  
- Room: TakeOver  
- Difficulty: Easy  
- Category: Web / Subdomain Takeover  
- Room Link: https://tryhackme.com/room/takeover

---

## Overview
The TakeOver room focuses on identifying vulnerable subdomains that can be abused in a subdomain takeover. The goal was to enumerate the target, discover unclaimed external resources (e.g., AWS S3 endpoints) referenced by subdomains, and retrieve the room flag.

## Objective
- Enumerate web infrastructure and subdomains for futurevera.thm
- Identify any subdomain pointing to an unclaimed third-party resource
- Demonstrate a subdomain takeover scenario and retrieve the flag

## Enumeration
Tools used:
- nmap
- browser (certificate inspector)
- /etc/hosts edits
- basic web requests (curl/browser)

Nmap scan (quick host/service check):
```bash
nmap futurevera.thm -oN nmapResults.txt
```
Observed open ports:
| Port | Service |
|------|---------|
| 22   | ssh     |
| 80   | http    |
| 443  | https   |

Notes:
- Ports 80 and 443 were the focus since this lab centers on web infrastructure.
- SSL certificates are useful reconnaissance sources (SANs can reveal hidden subdomains).

## Exploitation
1. Hypothesis from room description: a rebuild of the support system → likely subdomain:
   - support.futurevera.thm
2. Added support and the main host to /etc/hosts for testing:
```bash
sudo nano /etc/hosts
# add:
# 10.10.xx.xx futurevera.thm support.futurevera.thm
```
3. Visited support.futurevera.thm in the browser and inspected the SSL certificate. Certificates often reveal Subject Alternative Names (SANs) that list additional subdomains.
4. Found another SAN entry (a hidden subdomain). Added it to /etc/hosts:
```text
10.10.xx.xx futurevera.thm support.futurevera.thm secret-subdomain.support.futurevera.thm
```
5. Accessed the discovered subdomain over HTTP and observed a redirect to an AWS S3 website endpoint that displayed a missing resource page (typical S3 "NoSuchBucket" / unconfigured website response).
   - This indicates the DNS points to a cloud provider resource that no longer exists or is unclaimed.
   - The redirected URL contained the room flag.

Proof / indicators of takeover:
- DNS CNAME or alias referencing a third-party service (S3) while the resource is absent
- HTTP redirect to a cloud provider URL showing missing resource text
- Flag present in the redirected URL (successful completion of lab)

## Privilege Escalation
- Not applicable for this web-focused lab. No local or host-level escalation steps were required.

## Lessons Learned
- SSL certificate inspection (SANs) is a fast way to discover hidden or forgotten subdomains.
- /etc/hosts is useful for lab testing when DNS is not under your control.
- Subdomain takeover is a real risk when DNS records point to third-party services that may be removed or deprovisioned.
- Impact of takeover includes phishing, brand impersonation, cookie/session abuse, and serving malicious payloads from a trusted domain.
- Always verify third-party resource ownership when decommissioning services; remove or update DNS entries.

## Short Mitigations
- Remove DNS records that point to deprovisioned third-party services.
- Implement DNS and asset inventories to track external dependencies.
- Use automated checks to detect dangling CNAMEs and unclaimed cloud resources.

## References
- TryHackMe — TakeOver room: https://tryhackme.com/room/takeover  
- HackTricks — Subdomain Takeover (recommended read)  
- HackerOne Guide — Subdomain Takeovers (recommended read)  
- MDN — Subdomain security notes
