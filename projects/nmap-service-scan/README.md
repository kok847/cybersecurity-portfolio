# Project 2: Nmap service scan of Metasploitable 2

> Tested only on virtual machines I own, inside an isolated Host-Only network (`192.168.56.0/24`).

## Objective
Discover hosts in the lab and identify exposed services on the Metasploitable 2 target, then assess the attack surface and recommend fixes.

## Environment and tools
Kali Linux (attacker, `192.168.56.106`), Metasploitable 2 (target, `192.168.56.107`), Nmap 7.95. Built in [Project 1](../virtual-lab-setup/).

## Method
1. **Host discovery:** `nmap -sn 192.168.56.0/24` (no port scan).
2. **Service versions on the target:** `nmap -sV 192.168.56.107` (1,000 default TCP ports).
3. **Subnet-wide version scan:** `nmap -sV 192.168.56.0/24`, to confirm nothing else in the lab is exposed.
4. **Save raw output** for evidence: `nmap -sV 192.168.56.107 -oN evidence/sv-scan.txt`.

## Results
**Host discovery** found **4 hosts up**:

| IP | Identity |
|---|---|
| 192.168.56.1 | Host machine's Host-Only adapter (Jetty on port 8080 in the subnet scan) |
| 192.168.56.100 | VirtualBox DHCP server (all 1,000 ports closed) |
| 192.168.56.106 | My Kali VM (the scanner itself, shown without MAC or latency) |
| 192.168.56.107 | Metasploitable 2 target |

**Service scan:** 23 open TCP ports on the target (977 closed). Grouped by security implication:

| Category | Ports and services | Risk |
|---|---|---|
| Cleartext and legacy remote access | 21 FTP (vsftpd 2.3.4), 23 Telnet, 512 rexec, 513 rlogin, 514 rsh, 2121 ProFTPD 1.3.1 | Credentials and data sent without encryption; weak trust-based r-service authentication |
| Outdated remote administration | 22 SSH (OpenSSH 4.7p1), 5900 VNC (3.3), 6000 X11 | Old versions and weak configurations; X11 returned access denied |
| Backdoor-style services | 1524 bindshell, 6667 UnrealIRCd | Port 1524 is a root shell; UnrealIRCd has a historical backdoor (version not confirmed) |
| Exposed databases | 3306 MySQL 5.0.51a, 5432 PostgreSQL 8.3.x | Unauthorised access and attacks on outdated software |
| File sharing and RPC | 111 rpcbind, 139/445 Samba 3.x, 2049 NFS | Information disclosure and weak share or export settings |
| Web and Java services | 80 Apache 2.2.8, 8009 AJP, 8180 Tomcat 1.1, 1099 Java RMI | Outdated web stack; AJP and RMI widen the attack surface |
| Mail and DNS | 25 Postfix, 53 BIND 9.4.2 | Old versions; possible information leakage |

**Notable finding:** vsftpd 2.3.4 is linked to [CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523), a backdoor that opens a shell on TCP 6200. Only copies downloaded between 30 June and 3 July 2011 were affected, so the version string alone does not prove the backdoor is present. It needs authorised verification in the lab.

Evidence: `evidence/a6-nmap-sn.png`, `a7-nmap-sv-target.png`, `a8-nmap-sv-subnet.png`, `sv-scan.txt`.

## What I would do differently / how to fix the issues
- Disable Telnet, rexec, rlogin and rsh. Use SSH and SFTP instead of FTP.
- Remove or patch vsftpd, UnrealIRCd and the bindshell on 1524.
- Firewall database ports (3306, 5432) to trusted hosts only; restrict NFS exports, Samba shares and VNC.
- Upgrade OpenSSH, Apache, BIND and Samba, and disable AJP if unused.
- Next time: run `-sC` and `-p-` for full-port coverage, then validate findings with OpenVAS.
