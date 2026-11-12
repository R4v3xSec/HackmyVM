# HackMyVM
 
Welcome to my HackMyVM repository!
 
This repository contains the machines I have completed while practicing cybersecurity and penetration testing. Each one includes a full writeup covering reconnaissance, exploitation, and privilege escalation, along with a short summary of the core vulnerability and technique behind the box.

## Machines
- Friendly
- Friendly2
- Friendly3
- Canto
- Gift
- Noob

## Techniques Covered
 
Across these machines, the writeups touch on a range of common real-world attack patterns:
 
- **Reconnaissance:** `arp-scan`, `nmap`, `ping` OS fingerprinting via TTL
- **Web enumeration & fuzzing:** `gobuster`, `wfuzz`, `wpscan`, custom wordlists (`CEWL`)
- **Initial access:** vulnerable plugin exploitation (CVE-2023-3452), anonymous/writable FTP, Local File Inclusion, exposed file shares
- **Credential attacks:** brute-force (Hydra), offline hash cracking (`john`, `ssh2john`), credential reuse across services, weak passwords
- **Privilege escalation:** sudo misconfigurations (GTFOBins — `vim`, `cpulimit`), PATH hijacking, TOCTOU race conditions, symlink attacks
---
 
## Disclaimer
 
All machines in this repository are part of [HackMyVM](https://hackmyvm.eu/), a platform for legal, intentionally vulnerable virtual machines built for educational penetration testing practice. Nothing here targets real, production systems.


