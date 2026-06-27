 TryHackMe — TakeOver Room

- Platform: TryHackMe  
- Room: TakeOver  
- Difficulty: Easy  
- Category: Web / Subdomain Takeover  
- Room Link: https://tryhackme.com/room/takeover

I worked through the TakeOver room to get a hands-on feel for subdomain takeover. This was my first focused exercise on the topic — I wanted to practice finding forgotten subdomains and understanding how DNS entries can point to deprovisioned cloud resources.

 Walk through web recon for futurevera.thm
 Find any subdomain that points to an unclaimed third-party service
 Demonstrate the takeover scenario and retrieve the room flag (redacted in this writeup)

 Ran a light nmap scan to confirm services
 Visited likely subdomains and inspected SSL certificates for hidden names
 Used /etc/hosts to test subdomains locally
 Followed a redirected URL to an AWS S3 endpoint that indicated an unclaimed resource


I started simple to avoid noise.

Nmap command I used:
```bash
nmap futurevera.thm 
```
Result 
| Port | Service |
|------|---------|
| 22   | ssh     |
| 80   | http    |
| 443  | https   |

Since this room is web-focused, I ignored SSH and concentrated on ports 80/443.

The room description mentioned the company was rebuilding their support system. That suggested a support subdomain might exist, so I tried:

 support.futurevera.thm

I didn't rely on DNS for the lab; instead I pointed the hostnames to the lab IP in my local hosts file for testing:

```bash
sudo nano /etc/hosts
# (attack machine ip) futurevera.thm support.futurevera.thm
```
After adding the entry, I loaded support.futurevera.thm in my browser and inspected the certificate details.
---

In the certificate SANs I spotted another hostname I hadn't thought to try. I added that hostname to /etc/hosts as well and visited it.

Example hosts entry I used:
```text
10.10.xx.xx futurevera.thm support.futurevera.thm secret-subdomain.support.futurevera.thm 
```

This step was the key — cert inspection quickly revealed an additional target without brute-forcing or wordlists.

---

When I visited the discovered subdomain over HTTP, it redirected me to an AWS S3 website endpoint that displayed a missing-bucket style page. In short:

- The DNS record still pointed to a third‑party service (S3)
- The backend S3 resource was gone or unclaimed
- The HTTP response and the redirected URL made the takeover obvious

The redirected URL contained the room flag — I redacted the flag here to keep this writeup safe.

Why this is a takeover: the company controls the DNS but does not control (or has deleted) the cloud resource the DNS points to; an attacker can claim the cloud resource and host content under the company’s subdomain.

---

- Certificate inspection is a fast, low-effort way to find hidden subdomains (look at SANs).
- /etc/hosts is a practical shortcut for lab environments when you need to map names to lab IPs.
- Subdomain takeover is real and impactful: phishing, brand abuse, cookie/tracking misuse, and more can follow from an unclaimed service.
- Organizations should remove DNS records that point to deprovisioned services or add monitoring to detect dangling CNAMEs.

---

- Maintain an inventory of DNS records and corresponding cloud resources.
- Remove or update DNS entries when decommissioning services.
- Use automated scans to detect dangling CNAMEs or unclaimed cloud endpoints.

---
some references :
- TryHackMe — TakeOver room: https://tryhackme.com/room/takeover  
- HackTricks — Subdomain Takeover  
- HackerOne Guide — Subdomain Takeovers  
- MDN — Subdomain security notes
