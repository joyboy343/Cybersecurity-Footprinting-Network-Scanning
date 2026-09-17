# Cybersecurity Footprinting & Network Scanning

This repository documents hands-on cybersecurity lab work covering domain footprinting with theHarvester and local network discovery using Zenmap/Nmap. It contains the practical commands, raw outputs, screenshots, network topology evidence, and final technical report produced during the exercises.

## Overview

| Module | Focus | Tools |
|---|---|---|
| W2-PM4 | Footprinting / OSINT | theHarvester |
| W2-PM5 | Network Scanning | Zenmap / Nmap |

The work covers two foundational cybersecurity assessment activities: passive domain footprinting and authorized local network discovery. No exploitation, credential attacks, or unauthorized access were performed in either module.

## Objectives

- Perform domain footprinting using theHarvester
- Collect publicly available host/email information from the assigned exercise
- Identify the local IP address and subnet
- Discover live hosts on the local network
- Identify available MAC address information
- Generate a network topology
- Document technical findings and evidence

## Technologies & Tools

| Tool | Purpose |
|---|---|
| Kali Linux | Operating system used to run theHarvester and native networking commands |
| theHarvester 4.10.1 | OSINT footprinting tool used to gather hosts, emails, IPs and ASNs for a target domain from public sources |
| Zenmap (Nmap GUI) 7.99 | GUI front-end for Nmap; used to run ping scans, view results and generate a network topology diagram |
| Nmap (`nmap -sn`) | Command-line host discovery via ICMP/ARP ping scan of the local subnet |
| `ip neigh` | Linux command used to inspect the ARP/neighbour cache and confirm the gateway's MAC address |

## W2-PM4 — theHarvester Footprinting

theHarvester was used to perform passive OSINT footprinting against the `microsoft.com` domain, as an educational exercise against public sources only. No active or intrusive scanning of Microsoft's infrastructure was performed — every result below came from theHarvester's public/free data sources.

### Task 1 — Baidu Search

```
theHarvester -d microsoft.com -l 1000 -b baidu
```

**Target:** microsoft.com
**Source:** Baidu
**IP results:** No IPs found
**Email results:** 2 — `is-microsoft-noreply@microsoft-com`, `microsoft-noreply@microsoft.com`
**Host results:** 10 hosts, including `adoption.microsoft.com`, `learn.microsoft.com`, `support.microsoft.com`, `news.microsoft.com`, `watson.microsoft.com`
**People results:** No people found

![PM4 Task 1 - theHarvester Baidu](W2-PM4-theHarvester/screenshots/task1-theharvester-baidu.png)
*Baidu-only theHarvester run against microsoft.com — 2 emails and 10 hosts returned.*

[View Task 1 Output](W2-PM4-theHarvester/outputs/task1-theharvester-baidu.txt)

### Task 2 — Multiple Sources

```
theHarvester -d microsoft.com -l 50 -b all
```

Most paid-API sources (Shodan, VirusTotal, Hunter, Censys, SecurityTrails, etc.) returned "missing API key" errors, which is expected without paid credentials configured. Results below come only from the free/public sources that did return data (Baidu, CRTsh, DuckDuckGo, Hackertarget, Rapiddns, Subdomaincenter, Urlscan, Yahoo, Waybackarchive, and a DNS fallback pattern search).

| Category | Result |
|---|---|
| ASNs found | 8 — AS13335, AS133618, AS16625, AS20940, AS24940, AS40034, AS8070, AS8075 |
| Interesting URLs found | 8 (public support/learn pages) |
| IP addresses found | 153 |
| Emails found | 3 — `dotnet-docker-bot@microsoft.com`, `opencode@microsoft.com`, `secure@microsoft.com` |
| Hosts/subdomains found | 9,969 — almost entirely public subdomains surfaced by certificate-transparency and search-engine sources (e.g. `*.microsoft.com`, `*.azure.microsoft.com`); this is a passive OSINT result set, not a list of systems that were probed |
| LinkedIn users/links found | 0 |
| People found | None |

![PM4 Task 2 - Start](W2-PM4-theHarvester/screenshots/task2-theharvester-all-start.png)
*All-sources run initiated — paid-API sources reporting missing keys as expected.*

![PM4 Task 2 - End](W2-PM4-theHarvester/screenshots/task2-theharvester-all-end.png)
*All-sources run completed — final subdomain results returned and captured.*

[View Task 2 Output](W2-PM4-theHarvester/outputs/task2-theharvester-all.txt)

## W2-PM5 — Zenmap Network Scanning

This activity was performed against my own local VirtualBox/Kali network, not against any third-party system.

**Scan scope:** `10.0.0.0/24`

### Task 2 — Local Network Configuration

| Item | Value |
|---|---|
| Local IP | 10.0.0.2 |
| Subnet | 10.0.0.0/24 |
| Gateway | 10.0.0.1 |
| Interface | eth0 |

> Only the gateway's MAC address (`52:54:00:12:35:00`) is captured in the evidence for this module, via `ip neigh` and the Nmap ARP ping scan below — the Kali VM's own MAC address is not visible in the captured output, since neither `ip neigh` nor a self-scan with Nmap reports the scanning host's own interface MAC. That value is intentionally left undocumented here rather than assumed.

### Tasks 3–5 — Live Host Discovery

```
nmap -sn 10.0.0.0/24
```

**Result:** 2 live hosts identified on the subnet — `10.0.0.1` and `10.0.0.2`.

- **Task 3** — list of live hosts: `10.0.0.1`, `10.0.0.2`
- **Task 4** — number of live hosts: 2
- **Task 5** — IP addresses of live hosts: `10.0.0.1`, `10.0.0.2`

![Zenmap Ping Scan](W2-PM5-Zenmap/screenshots/task3-5-ping-scan.png)
*`nmap -sn 10.0.0.0/24` run from Zenmap — 2 hosts up.*

### Task 6 — MAC Address Identification

```
sudo nmap -sn -PR 10.0.0.0/24
```

| Host | MAC address | Source |
|---|---|---|
| 10.0.0.1 (gateway) | `52:54:00:12:35:00` (QEMU virtual NIC) | Observed via Nmap ARP ping scan (`nmap -sn -PR`) and confirmed independently with `ip neigh` |
| 10.0.0.2 (Kali VM, scanning host) | Not captured | Nmap does not report a MAC address for the host's own local interface — this is expected Nmap behaviour, not a finding, and no separate interface-configuration command output for this value was captured |

The distinction above is intentional: the gateway MAC came directly from Nmap/ARP output, and no MAC address for the scanning host itself is claimed, since none was observed in the evidence.

![Zenmap MAC Address Scan](W2-PM5-Zenmap/screenshots/task6-mac-addresses.png)
*`ip neigh` and `sudo nmap -sn -PR 10.0.0.0/24` (output tee'd to `task6-mac-addresses.txt`) — gateway MAC confirmed.*

[View MAC Address Scan Output](W2-PM5-Zenmap/outputs/task6-mac-addresses.txt)

### Task 7 — Network Topology

Zenmap's Topology view was used to visualize the discovered local network. The legend was enabled and the topology was exported and saved as a PDF.

![Zenmap Network Topology](W2-PM5-Zenmap/screenshots/task7-topology.png)
*Zenmap Topology view (legend enabled) — 10.0.0.1 and 10.0.0.2 shown relative to the scanning host.*

[View Zenmap Topology PDF](W2-PM5-Zenmap/PM5-Zenmap-Topology.pdf)

## Key Findings

| Module | Observation | Evidence |
|---|---|---|
| W2-PM4 (Baidu) | 2 email addresses and 10 hosts found for microsoft.com | `task1-theharvester-baidu.txt`, Task 1 screenshot |
| W2-PM4 (all sources) | 8 ASNs, 153 IPs, 3 emails, 9,969 hosts/subdomains, 0 LinkedIn results | `task2-theharvester-all.txt`, Task 2 screenshots |
| W2-PM5 | Local subnet identified as 10.0.0.0/24, gateway 10.0.0.1, local IP 10.0.0.2 | Zenmap scan configuration |
| W2-PM5 | 2 live hosts discovered: 10.0.0.1, 10.0.0.2 | `task3-5-ping-scan.png` |
| W2-PM5 | Gateway MAC address observed: 52:54:00:12:35:00 | `task6-mac-addresses.png`, `task6-mac-addresses.txt` |
| W2-PM5 | Network topology generated and exported successfully | `task7-topology.png`, `PM5-Zenmap-Topology.pdf` |

All items above are reconnaissance-level observations, not confirmed vulnerabilities.

## Security Considerations

- Footprinting information (public emails, subdomains, ASN data) has reconnaissance value for an attacker building a target profile, but does not by itself indicate a vulnerability.
- Exposed host/email information can assist an authorized security assessment in scoping further testing.
- Internal host discovery (ping scans) helps identify which devices are actually active on a network.
- Any unexpected or unrecognized device found during network discovery should be investigated.
- None of the findings in this repository were validated as exploitable weaknesses — that would require further, separately authorized testing.

## Lessons Learned

Working through these two modules gave me a clearer picture of how OSINT and domain footprinting actually work in practice — how much information about an organization is discoverable through entirely passive sources, and how inconsistent free OSINT sources can be (most paid-API sources in theHarvester simply fail without credentials, which is normal). Running theHarvester against a large real domain also showed me how noisy and large a subdomain result set can get, and why a report needs to summarize that kind of output rather than dump it wholesale.

On the Nmap/Zenmap side, I learned the practical difference between a plain ping scan and an ARP-based scan (`-PR`), why Nmap doesn't report a MAC address for the scanning host's own interface, and how to cross-check a result (like the gateway's MAC) using more than one method (`ip neigh` alongside Nmap). Working with Zenmap's topology view also helped me understand how a visual network map complements raw scan output.

Beyond the tools themselves, this module reinforced the importance of evidence-based documentation — being precise about what a screenshot or log file actually shows, not what I expected or assumed it would show, and being careful to separate an observation from a security conclusion.

## Ethical & Legal Scope

- W2-PM4 was performed as part of the assigned educational exercise, using only theHarvester's public/passive OSINT sources against the `microsoft.com` domain — no active or intrusive scanning of that domain was performed.
- W2-PM5 was performed only against my own local VirtualBox/Kali lab network.
- Security scanning and reconnaissance should only ever be performed against systems and networks where appropriate authorization exists.
- No exploitation, credential attacks, or unauthorized access was performed as part of either module.

## Project Evidence

- PM4 screenshots: [`W2-PM4-theHarvester/screenshots/`](W2-PM4-theHarvester/screenshots/)
- PM4 outputs: [`W2-PM4-theHarvester/outputs/`](W2-PM4-theHarvester/outputs/)
- PM5 screenshots: [`W2-PM5-Zenmap/screenshots/`](W2-PM5-Zenmap/screenshots/)
- PM5 output: [`W2-PM5-Zenmap/outputs/task6-mac-addresses.txt`](W2-PM5-Zenmap/outputs/task6-mac-addresses.txt)
- Topology PDF: [`W2-PM5-Zenmap/PM5-Zenmap-Topology.pdf`](W2-PM5-Zenmap/PM5-Zenmap-Topology.pdf)
- Final report: [`report/W2-PM-FINAL.docx`](report/W2-PM-FINAL.docx)

## Final Report

[W2-PM-FINAL Report (DOCX)](report/W2-PM-FINAL.docx)
[W2-PM-FINAL Report (PDF)](report/W2-PM-FINAL.pdf)

## Repository Structure

```
Cybersecurity-Footprinting-Network-Scanning/
│
├── README.md
│
├── W2-PM4-theHarvester/
│   ├── screenshots/
│   │   ├── task1-theharvester-baidu.png
│   │   ├── task2-theharvester-all-start.png
│   │   └── task2-theharvester-all-end.png
│   │
│   └── outputs/
│       ├── task1-theharvester-baidu.txt
│       └── task2-theharvester-all.txt
│
├── W2-PM5-Zenmap/
│   ├── screenshots/
│   │   ├── task3-5-ping-scan.png
│   │   ├── task6-mac-addresses.png
│   │   └── task7-topology.png
│   │
│   ├── outputs/
│   │   └── task6-mac-addresses.txt
│   │
│   └── PM5-Zenmap-Topology.pdf
│
└── report/
    ├── W2-PM-FINAL.docx
    └── W2-PM-FINAL.pdf
```

## Conclusion

This Week 2 submission covers domain footprinting with theHarvester, local network discovery with Zenmap/Nmap, evidence collection for both modules, and technical documentation of the results in a final report and this repository. No exploitation, penetration testing success, or vulnerability discovery is claimed — the work reflects reconnaissance and authorized network discovery only.

## Acknowledgements

This project was completed as part of the cybersecurity training provided by **NetworkWalks**. I appreciate the opportunity to work through practical exercises involving reconnaissance, OSINT, network discovery, and security documentation.

Thank you to the NetworkWalks team for providing the learning resources and practical labs that supported this work.

## Author

**Karthik Raman Keerangudi Kalyanaraman**

Cybersecurity Intern
