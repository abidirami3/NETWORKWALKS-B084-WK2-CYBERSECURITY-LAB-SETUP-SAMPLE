# PENETRATION TESTING REPORT

### Footprinting & Network Scanning Phases
**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| Field | Detail |
|---|---|
| Pentester Name (Cybersecurity Professional) | `[FULL NAME]` |
| Program / Batch | `[BATCH, e.g. B082]` – Networkwalks Cybersecurity Internship |
| Report Date | 11 November 2026 |
| Modules completed | W2-PM1 (Multiple Kali Tools), W2-PM2 (GHDB), W2-PM4 (theHarvester), W2-PM5 (Zenmap) |
| Primary target (in scope) | `networkwalks.com` (own-course target) and the tester's own lab network |
| Authorization | Letter of Authorization `[REFERENCE NO.]` |
| Classification | Educational / training report |

---

## 1. Liability Disclaimer

This report was produced for education and research purposes only, as part of the Networkwalks Cybersecurity Internship. Its aim is to show how reconnaissance works against real-world systems so that those systems can be defended better. All activities were limited to passive information gathering and host discovery. No exploitation, brute-force, denial-of-service, credential guessing or data modification was performed. Hacking is only legal when you test a system you own or a lab environment, when you have written permission from the owner, or when you work under a signed agreement with an agreed scope.

## 2. Executive Summary

Four project modules were completed covering the first two phases of a penetration test: **footprinting/reconnaissance** and **network scanning**.

- **PM1** profiled `networkwalks.com` with six Kali tools (whois, whatweb, nslookup, curl, wafw00f, dnsrecon). The site runs WordPress on Apache, is hosted at `192.232.216.135`, and sits behind a ModSecurity WAF.
- **PM2** used Google dorks (GHDB) to locate exposed camera pages and open directories listing mathematics PDFs, showing how much a search engine alone can reveal.
- **PM4** used theHarvester against `microsoft.com` to test passive collection of e-mails, hosts and sub-domains from public sources.
- **PM5** used Zenmap (Nmap GUI) to discover live hosts on the tester's lab subnet and to draw the network topology.

**Key takeaway:** every piece of information above was obtained without sending a single attack packet. The less an organization exposes publicly, the harder every later stage becomes.

## 3. Introduction, Scope and Method

Reconnaissance is the first phase of any security assessment. This report documents the commands run, the outputs obtained, what each result means for an attacker, and how a defender can reduce the exposure.

| Module | Scope | Nature |
|---|---|---|
| PM1 | `networkwalks.com` | Passive and light active information gathering |
| PM2 | Google search index | Passive (queries to Google only; targets not contacted by scanning) |
| PM4 | `microsoft.com` public data sources | Passive OSINT (no connection to the target) |
| PM5 | Tester's own lab subnet | Host discovery (ping scan) |

**Rules followed:** no exploitation, no login attempts, no credentials used or recorded, no data copied or modified, no third-party system scanned. Third-party exposed devices found in PM2 are **anonymised** in this public report (see section 5.2).

## 4. Tools Used

| Tool | Purpose |
|---|---|
| whois | Domain registration details |
| whatweb | Web technology fingerprinting |
| nslookup | DNS resolution (domain to IP) |
| curl -I | HTTP response header analysis |
| wafw00f | Web Application Firewall detection |
| dnsrecon | DNS enumeration |
| Google Hacking Database (Exploit-DB) | Ready-made Google dorks |
| theHarvester 4.10.1 | E-mail, host and sub-domain OSINT |
| Zenmap / Nmap | Host discovery and topology |
| Kali Linux | Operating system for all tools |

---

## 5. Activities Performed

### 5.1 PM1 – Footprinting with Multiple Kali Tools (target: networkwalks.com)

#### Task 1 – whois
```
$ whois networkwalks.com
```
![whois](screenshots/pm1-1-whois.png)

| Item | Result |
|---|---|
| Registrar | GoDaddy.com, LLC (IANA ID 146) |
| Creation date | 2019-11-06 |
| Last updated | 2025-11-12 |
| Expiry date | 2027-11-06 |
| Name servers | NS6135.HOSTGATOR.COM, NS6136.HOSTGATOR.COM |
| DNSSEC | Unsigned |
| Status | clientDelete/Renew/Transfer/UpdateProhibited (registrar locks enabled) |
| Registrant | Hidden behind a privacy service (Domains By Proxy, LLC) |

**Analysis:** the name servers reveal the hosting provider (HostGator). Registrar locks and WHOIS privacy are good protections. DNSSEC being unsigned leaves DNS answers without cryptographic validation.

#### Task 2 – whatweb
```
$ whatweb networkwalks.com
```
![whatweb](screenshots/pm1-2-whatweb.png)

| Item | Result |
|---|---|
| HTTP → HTTPS | `301 Moved Permanently` then `200 OK` on HTTPS |
| Web server | Apache |
| CMS | WordPress 7.1.4 (reported in the MetaGenerator tag) |
| Plugin | WordPress Download Manager 3.3.76 |
| Libraries | jQuery 3.7.1, Bootstrap, HTML5, Google Tag Manager |
| Server IP | 192.232.216.135 (United States) |
| E-mail disclosed | info@networkwalks.com |

**Analysis:** the CMS and plugin versions are publicly visible. An attacker can match them against vulnerability databases, so version disclosure should be minimised and plugins kept up to date.

#### Task 3 – nslookup
```
$ nslookup networkwalks.com
```
![nslookup](screenshots/pm1-3-nslookup.png)

**Result:** `networkwalks.com` resolves to **192.232.216.135** (non-authoritative answer). The resolver used was the local router (192.168.1.1). Additional queries then timed out (secondary resolver timeouts / IPv6 DNS server unreachable), which did not affect the final answer.

**Analysis:** knowing the IP allows direct scanning of the server and discovery of other sites hosted on the same IP.

#### Task 4 – curl -I
```
$ curl -I https://networkwalks.com
```
![curl](screenshots/pm1-4-curl.png)

| Header | Value / Meaning |
|---|---|
| Status | `HTTP/2 200` |
| server | Apache |
| x-nginx-cache | WordPress (a caching layer sits in front) |
| x-endurance-cache-level | 0 (hosting cache plugin) |
| link | Exposes the WordPress REST API (`/wp-json/`) |
| set-cookie | `__wpdm_client` with `secure; HttpOnly` flags (good practice) |
| referrer-policy | `no-referrer-when-downgrade` |
| date | Mon, 09 Nov 2026 15:44:50 GMT |

**Analysis:** headers confirm the technology stack and reveal the REST API endpoint without loading the page. Cookies correctly use the Secure and HttpOnly flags.

#### Task 5 – wafw00f
```
$ wafw00f networkwalks.com
```
![wafw00f](screenshots/pm1-5-wafw00f.png)

**Result:** the site is behind **ModSecurity (SpiderLabs)** WAF (wafw00f v2.4.2, 2 requests).

**Analysis:** a WAF is a positive security control. For an attacker it means naive attacks will be blocked or logged.

#### Task 6 – dnsrecon
```
$ dnsrecon -d networkwalks.com
```
![dnsrecon](screenshots/pm1-6-dnsrecon.png)

**Result:** the enumeration **failed with a timeout** error (the DNS servers could not be reached from the lab network within the default 3-second lifetime), so no DNS records were returned in this run.

**Mitigation / next step:** re-run with a public resolver and a longer timeout, for example `dnsrecon -d networkwalks.com -n 8.8.8.8 --lifetime 10`, or check the local DNS configuration. This is documented as a limitation rather than a finding.

---

### 5.2 PM2 – Footprinting with GHDB (Google Hacking Database)

**Method:** dorks from Exploit-DB GHDB and other sources were entered into Google; each result was opened only to verify what the page exposes. **No credentials were entered, used or recorded.** IP addresses of private third-party devices are partly masked (`xxx`) to avoid publishing a list of exposed cameras. Findings are classified honestly: only the webcamXP servers are genuinely misconfigured devices; the others are public webcams or open directory listings.

#### Task 1 – Exposed camera pages and image directories

| No. | Resource | Dork | Category | Credentials |
|---|---|---|---|---|
| 1 | `http://122.116.xxx.xxx:8080/` | `intitle:"webcamXP" inurl:8080` | Exposed webcamXP server (no login required) | None |
| 2 | `http://139.64.xxx.xxx:8080/` | `intitle:"webcamXP" inurl:8080` | Exposed webcamXP server | None |
| 3 | skylinewebcams.com – Rome skyline | `inurl:webcam site:skylinewebcams.com inurl:roma` | Intentionally public tourist webcam | None |
| 4 | skylinewebcams.com – Playa Los Cristianos, Tenerife | `inurl:webcam site:skylinewebcams.com` | Intentionally public tourist webcam | None |
| 5 | klsimer.edu – `/img/infrastructure/lb/` | `intitle:"Index of" "DCIM/camera"` | Open directory listing of camera images | None |
| 6 | atlas.aei.uni-hannover.de – `…/DCIM/Camera/` (test-data folder) | `intitle:"Index of" "DCIM/camera"` | Open directory listing (test images) | None |
| 7 | meteo.physic.ut.ee – `/webcam/arhiiv/` | `intitle:"Index of /webcam/"` | Open directory of webcam archive | None |
| 8 | ranchturini.com – `/webcam/…/` | `intitle:"Index of /webcam/"` | Open directory of webcam archive | None |
| 9 | fotowebcam.org – Prasserbad | `intitle:"Webcam" inurl:WebCam.htm` | Intentionally public webcam | None |
| 10 | lmc.edu – `/webcam.htm` | `intitle:"Webcam" inurl:WebCam.htm` | Public institutional webcam page | None |

#### Task 2 – Open directories with downloadable mathematics PDFs

Dork used for all results: `intitle:index.of "parent directory" mathematics pdf`

| No. | Resource |
|---|---|
| 1 | education.giakonda.org.uk/Maths/ |
| 2 | erewhon.superkuh.com/library/Math/ |
| 3 | justmathbg.info/files/ |
| 4 | unm.edu/~megrad/Math/Mathematics-2017.pdf |
| 5 | ochicken.net/library/Mathematics/Mathematical_Analysis/ (Zorich, *Mathematical Analysis*) |
| 6 | netlib.org/math/docpdf/ |
| 7 | math.cmu.edu/users/math/mlavrov/arml/16-17/algebra-12-11-16.pdf |
| 8 | lyceedadultes.fr/…/00_principes_de_la_redaction_mathematique.pdf |
| 9 | inis.jinr.ru/sl/vol2/Mathematics/Math.Encyclopedia/Pdf/Math_V2.pdf |
| 10 | seedscleveland.sharepoint.com/…/Fraction Decimal Percent Chart.pdf |

**Analysis:** the dorks show that Google indexes everything a server exposes. Misconfigured directory listings and unprotected camera software are discovered in seconds, without touching the target. Several listed resources may contain copyrighted books; they were only listed, not downloaded or redistributed.

**Defensive advice:** disable directory indexing (`Options -Indexes`), put cameras behind authentication or a VPN, change default ports and credentials, and use `robots.txt`/`noindex` plus regular self-dorking of your own domains.

---

### 5.3 PM4 – Footprinting with theHarvester (target: microsoft.com)

#### Task 1 – Baidu source, limit 1000
```
$ theHarvester -d microsoft.com -l 1000 -b baidu
```
![theHarvester baidu](screenshots/pm4-1-baidu.png)

**Result (theHarvester 4.10.1):** *No IPs, e-mails, people or hosts found.* Baidu returned nothing for this query in this run. The lab guide notes that results vary over time because the underlying sources change their algorithms.

#### Task 2 – All sources, limit 50
```
$ theHarvester -d microsoft.com -l 50 -b all
```
![theHarvester all](screenshots/pm4-2-all.png)

**Result:** many modules aborted with *Missing API key* errors (e.g. bevigil, bitbucket, bufferoverun, BuiltWith, Brave Search, Censys, criminalip, dehashed). Sources that require paid or registered API keys cannot be queried without configuring `/etc/theHarvester/api-keys.yaml`.

**Analysis and next steps:**
- Free sources that need no key (e.g. `crtsh`, `rapiddns`, `otx`, `urlscan`, `duckduckgo`) can be selected individually with `-b`.
- Results can be saved with `-f <filename>` for evidence.
- For defenders: harvested e-mails feed phishing campaigns and sub-domains expand the attack surface, so monitor what your organisation leaks.

---

### 5.4 PM5 – Network Scanning with Zenmap (own lab subnet)

| Task | Result |
|---|---|
| 1. Install Zenmap | Installed from the official site (https://nmap.org/download.html) `[CONFIRM: Windows / Kali]` |
| 2. Local IP and subnet | `[FILL IN: ipconfig / ip a output]`, subnet `10.0.2.0/24` |
| 3. Live hosts | Ping scan (`nmap -sn 10.0.2.0/24`) |
| 4. Number of live hosts | **3** (including the tester's own machine) |
| 5. IP addresses | 10.0.2.2, 10.0.2.3, 10.0.2.15 |
| 6. MAC addresses | `[FILL IN from the Zenmap Nmap Output tab]` |
| 7. Topology | Saved below (PDF copy available in the submission) |

![Zenmap topology](screenshots/pm5-topology.png)

**Analysis:** the topology shows the local host at the centre connected to three live nodes. The addressing (10.0.2.2 / 10.0.2.3 / 10.0.2.15) matches the default layout of a VirtualBox NAT network (virtual gateway, virtual DHCP/DNS service and the VM itself) `[CONFIRM]`. A ping scan is a light discovery technique; it does not probe ports or services, so it only tells us which hosts answer, not what they run.

---

## 6. Consolidated Findings

| # | Finding | Module | Risk (informational rating) | Recommendation |
|---|---|---|---|---|
| 1 | CMS and plugin versions disclosed (WordPress, WP Download Manager) | PM1 | Low | Hide generator tags, keep core/plugins updated |
| 2 | Server IP and hosting provider identifiable | PM1 | Info | Accept, or front the site with a CDN/reverse proxy |
| 3 | WordPress REST API endpoint advertised in headers | PM1 | Info | Restrict unneeded REST routes |
| 4 | DNSSEC not enabled | PM1 | Low | Enable DNSSEC at the registrar |
| 5 | Public contact e-mail disclosed | PM1 | Info | Use a role mailbox, apply anti-spam filtering |
| 6 | ModSecurity WAF, cookie flags and registrar locks in place | PM1 | Positive control | Keep rules updated |
| 7 | Exposed camera servers and open directory listings indexed by Google | PM2 | High for the owners of those devices | Authentication, VPN, disable indexing |
| 8 | Passive OSINT sources depend on API keys; public data volume varies | PM4 | Info | Configure API keys, query multiple sources |
| 9 | 3 live hosts discovered on the lab subnet | PM5 | Info | Maintain an asset inventory |

## 7. Limitations and Lessons Learned

- `dnsrecon` timed out and `nslookup` showed secondary resolver timeouts: DNS enumeration depends on a reachable resolver.
- theHarvester returned little or nothing for the Baidu source, and many sources need API keys.
- Results from OSINT tools change over time; screenshots are timestamped evidence of one run.
- Footprinting is only the first phase: no vulnerabilities were confirmed or exploited in this work.

## 8. Conclusion

Reconnaissance and scanning provide a surprisingly complete picture of a target using only public information. The same tools used by attackers should be run regularly by defenders against their own assets to reduce what they leak: fewer exposed versions, no open directories, authenticated cameras and an up-to-date inventory of live hosts.

## 9. Evidence Index

| File | Content |
|---|---|
| `screenshots/pm1-1-whois.png` | whois output |
| `screenshots/pm1-2-whatweb.png` | whatweb output |
| `screenshots/pm1-3-nslookup.png` | nslookup output |
| `screenshots/pm1-4-curl.png` | curl -I output |
| `screenshots/pm1-5-wafw00f.png` | wafw00f output |
| `screenshots/pm1-6-dnsrecon.png` | dnsrecon output (timeout) |
| `screenshots/pm4-1-baidu.png` | theHarvester, Baidu source |
| `screenshots/pm4-2-all.png` | theHarvester, all sources |
| `screenshots/pm5-topology.png` | Zenmap topology |

---
*Prepared for educational purposes under the Networkwalks Cybersecurity Internship.*
