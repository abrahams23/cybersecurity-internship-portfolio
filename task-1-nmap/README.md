# Task 1 — Basic Network Scanning with Nmap

## What is Nmap?
Nmap (short for Network Mapper) is a free, open-source tool that helps you discover devices on a network and see what services they're running. It works by sending packets to a target and analyzing the responses. From that, it can identify which ports are open, closed, or filtered, what software is listening on those ports, and sometimes even the operating system.

## Why network scanning matters
You can't protect what you don't know about. A scan shows the "attack surface" of a system — every open port is a possible entry point. If the service behind that port is outdated, misconfigured, or unnecessary, it becomes a risk. Security teams use Nmap to audit their own systems regularly, closing down anything that shouldn't be exposed before attackers can take advantage. Without scanning, organizations might not even realize they're running vulnerable services until it's too late.

## Ethical use guidelines
It's important to stress: scanning networks you don't own or don't have explicit permission to test is illegal in most places. Even if you don't cause damage, it's still considered unauthorized access. For this task, all scans were run against **scanme.nmap.org**, a safe practice target provided by the Nmap project itself. No scans were run against real companies, personal devices, or systems without permission.

## Installation
Nmap was already installed on the Kali Linux VM used for this internship. I confirmed the installation by running:

```bash
nmap --version
```

Which returned Nmap version 7.99. See `installation_evidence.png` for proof.

## Scans performed

| Scan type              | Command                                                   | Screenshot                  |
|-------------------------|-----------------------------------------------------------|-----------------------------|
| Basic scan              | `nmap -F -Pn scanme.nmap.org`                             | `basic_scan.png`            |
| Service version scan    | `nmap -sV -F -Pn scanme.nmap.org`                         | `service_version_scan.png`  |
| OS detection scan       | `sudo nmap -O -Pn scanme.nmap.org`                        | `os_detection_scan.png`     |
| Combined scan (to file) | `nmap -sV -O -F -Pn scanme.nmap.org -oN nmap_scan_results.txt` | `results_file_creation.png` |

> Note: I used the `-F` flag (fast scan, top 100 ports) and the `-Pn` flag (skip host discovery ping). A full 1000‑port scan against a remote target kept hanging because many filtered ports didn't respond. Documenting this choice is part of the learning process.

The full raw output is saved in `nmap_scan_results.txt`.

## Open ports found and analysis

See the port analysis section below.

## Port analysis

### Port 21 — FTP (File Transfer Protocol)
**What it does:** FTP is one of the oldest protocols for moving files between a client and a server.

**Security risk:** By default, FTP sends usernames, passwords, and file contents in plain text. That means anyone intercepting the traffic can read sensitive information. In modern environments, this is considered insecure unless it's wrapped with encryption (FTPS) or replaced with a safer alternative like SFTP.

---

### Port 22 — SSH (Secure Shell)
**What it does:** SSH provides secure, encrypted remote access to servers. It's the standard way administrators manage Linux/Unix systems.

**Security risk:** SSH is generally safe when configured properly, but risks include weak passwords, outdated versions, or allowing root login. It's also a common target for brute‑force attacks, so best practice is to use key‑based authentication and rate‑limiting.

---

### Port 80 — HTTP (Apache httpd 2.4.7, Ubuntu)
**What it does:** Port 80 serves unencrypted web traffic. In this case, the scan identified Apache HTTP Server version 2.4.7 running on Ubuntu.

**Security risk:** HTTP itself is insecure because all data (like form inputs or cookies) travels unencrypted. On top of that, Apache 2.4.7 is an older release, which increases the chance of known vulnerabilities. In a real environment, this version should be checked against vulnerability databases (like CVE listings) and updated if exploits exist.
