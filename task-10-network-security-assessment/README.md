# Task 10 — Full Network Security Assessment Report

## 1. Assessment Scope
**Primary target:** `scanme.nmap.org` (45.33.32.156) — Nmap’s official test host, publicly authorised for scanning practice.  
**Secondary scope:** Local network interface (`eth0`) on the Kali Linux VM, used to demonstrate packet‑level traffic analysis.  
**Time window:** Conducted on 14 September 2026 in two sessions — Nmap reconnaissance and Wireshark capture.  
**Services in scope:**  
- Nmap port/service scanning (ports, service versions, OS fingerprinting)  
- Wireshark packet capture and protocol analysis (HTTP, DNS, TCP)  
**Out of scope:** No web vulnerability scanning (e.g., Nikto) was performed. Only the authorised test target and local traffic were assessed.

---

## 2. Executive Summary
This assessment combined **Nmap reconnaissance** and **Wireshark traffic analysis** to evaluate exposed services and demonstrate risks of unencrypted communication.  

Three open services were identified: FTP, SSH, and HTTP. FTP and HTTP pose the most risk — FTP because it transmits credentials in plain text, and HTTP because the Apache version found (2.4.7) is outdated and unencrypted.  

Wireshark confirmed this risk in practice: a plain HTTP request was captured showing the full page request, browser details, and headers in readable text.  

No critical, business‑halting vulnerabilities were found, but the issues mirror common risks in real environments.  

**Overall risk posture: Low to Moderate.** Straightforward fixes exist and are outlined in Section 5.

---

## 3. Technical Report

### Phase 1 — Reconnaissance (Nmap)
A combined Nmap scan (`nmap -sV -O -F -Pn scanme.nmap.org`) identified:

| Port | Service | Version | Notes |
|------|---------|---------|-------|
| 21/tcp | FTP | tcpwrapped | Unencrypted protocol by default |
| 22/tcp | SSH | tcpwrapped | Encrypted by design |
| 80/tcp | HTTP | Apache httpd 2.4.7 (Ubuntu) | Older release; unencrypted traffic |

OS detection returned only low‑confidence guesses due to NAT/internet routing. This is a known limitation, documented here as a constraint rather than a finding.  

Raw results: `task-1-nmap/nmap_scan_results.txt`  
Screenshots: `task-1-nmap/`

---

### Phase 2 — Traffic Analysis (Wireshark)
Traffic was captured for ~2 minutes while browsing, including a deliberate visit to **neverssl.com** (plain HTTP).  

- **HTTP traffic:** A `GET / HTTP/1.1` request was captured in full, showing headers (`Host`, `User-Agent`, `Accept`) in plain text.  
- **DNS traffic:** Normal queries/responses observed between client and resolver.  
- **TCP handshake:** A full three‑way handshake was captured, followed by TLS negotiation for an HTTPS site. This contrast illustrates the difference between encrypted and unencrypted sessions.  

Evidence: `task-8-wireshark/` (screenshots + `.pcap` file)

---

### Phase 3 — Web Vulnerability Scan
Not performed (Nikto excluded from scope).

---

## 4. Findings Register

| ID   | Description | Severity | Asset | Recommended Fix |
|------|-------------|----------|-------|-----------------|
| F‑01 | FTP service open, unencrypted | Medium | scanme.nmap.org:21 | Disable FTP or migrate to FTPS/SFTP |
| F‑02 | HTTP service running Apache 2.4.7 | Medium | scanme.nmap.org:80 | Update Apache to latest stable release |
| F‑03 | Unencrypted HTTP traffic observed | Medium | Local traffic | Enforce HTTPS site‑wide; enable HSTS |
| F‑04 | SSH service exposed | Low | scanme.nmap.org:22 | Enforce key‑based auth; disable root login |
| F‑05 | OS fingerprinting inconclusive | Informational | scanme.nmap.org | No action required |

---

## 5. Remediation Roadmap

| Priority | Finding | Action | Effort |
|----------|---------|--------|--------|
| 1 | F‑03 | Enforce HTTPS + HSTS | Medium |
| 2 | F‑02 | Patch/update Apache | Easy |
| 3 | F‑01 | Disable FTP or migrate to SFTP/FTPS | Medium |
| 4 | F‑04 | Audit SSH configuration | Easy |
| 5 | F‑05 | No action needed | — |

**Execution order:** Address F‑03 and F‑02 first for maximum risk reduction. Handle FTP migration (F‑01) next if required by workflows. SSH audit (F‑04) is a quick check and can be done in parallel.

---

## Supporting Evidence
- Nmap results/screenshots: `task-1-nmap/`  
- Wireshark screenshots + `.pcap`: `task-8-wireshark/`
