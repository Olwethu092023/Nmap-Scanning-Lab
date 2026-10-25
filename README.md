# Nmap-Scanning-Lab

## Objective

The Nmap Scanning Lab aimed to identify open ports, detect running services and assess potential vulnerabilities within a controlled lab environment. The primary focus was to perform comprehensive network scans, analyze service versions and identify security risks. This hands-on experience was designed to strengthen understanding of network reconnaissance, vulnerability identification and practical security assessment techniques.

## Skills Learned

- Network reconnaissance using Nmap
- Open port identification and analysis
- Service and version detection
- Detection of potentially vulnerable services
- Network scanning techniques and interpretation of scan results

## Tools Used

- **Nmap** – Network scanning, port discovery and service version detection.
- **Kali Linux** – Operating system for performing network reconnaissance and security testing.
- **Terminal / Command Line** – Executing Nmap commands and analyzing scan results.
- **Metasploitable** - Target machine

## Steps

Configured Kali Linux and Metasploitable using a host-only network adapter for the lab environment.

## Step 1: Host Discovery

Performed host discovery using Nmap to identify active hosts on the network. Four hosts were discovered.

![Host Discovery](https://i.imgur.com/eyoUk7G.png)

## Step 2: SYN Scan

Conducted a TCP SYN scan against the target machine to identify open ports and understand which services were accessible.

![Host Discovery](https://imgur.com/jvbcKvv)

## Step 3: Service and Version Detection

Scanned all ports to identify running services, their versions, and information gathered through Nmap's default scripts.

![Host Discovery](https://imgur.com/OSlmbJ7)

## Step 4: UDP Scan

Scanned the target's 20 most common UDP ports to identify available UDP services and determine which ports were open.

![Host Discovery](https://imgur.com/Xm3TL3H)

## Step 5: Aggressive Scan

Performed an aggressive scan to gather information about the target's operating system, service versions, network route and available script results.

![Host Discovery](https://imgur.com/cvp9Q2X)

## Step 6: Vulnerability Scanning

Used an Nmap Scripting Engine (NSE) script to check the target for a known vulnerability.

![Host Discovery](https://imgur.com/Gq3byLR)

## Step 7: Results and Risk Analysis

Across the full enumeration (host discovery, full TCP sweep, version/script detection, UDP scan, aggressive scan and vulnerability scripting), 28 open TCP ports were identified on the target. The table below consolidates every service, its detected version and the associated security risk.

| Port | Service | Version | Risk / Notes |
|------|---------|---------|---------------|
| 21/tcp | ftp | vsftpd 2.3.4 | **Backdoored build — CVE-2011-2523.** Also allows anonymous login. Confirmed exploitable. |
| 22/tcp | ssh | OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0) | Weak 1024-bit DSA host key present; old OpenSSH version — brute-force/credential attack risk. |
| 23/tcp | telnet | Linux telnetd | Plaintext authentication and session data — credentials sniffable on the network. |
| 25/tcp | smtp | Postfix smtpd | User enumeration risk (VRFY). |
| 53/tcp | domain | ISC BIND 9.4.2 | Old BIND release; version disclosure; check for open zone transfers. |
| 80/tcp | http | Apache httpd 2.2.8 (Ubuntu) DAV/2 | Outdated Apache with WebDAV enabled; expired self-signed cert; weak SSLv2 ciphers (RC4/DES/MD5). |
| 111/tcp | rpcbind | 2 (RPC #100000) | Exposes RPC service mapping. |
| 139/tcp | netbios-ssn | Samba smbd 3.X–4.X (workgroup: WORKGROUP) | SMB service discovery. |
| 445/tcp | microsoft-ds | Samba smbd 3.0.20-Debian | Old Samba with public RCEs; message signing disabled, blank-credential auth accepted. |
| 512/tcp | exec | netkit-rsh rexecd | Legacy r-service — plaintext authentication. |
| 513/tcp | login | OpenBSD or Solaris rlogind | Legacy r-service — historically trusted-host authentication, plaintext. |
| 514/tcp | shell | Netkit rshd | Legacy r-service — plaintext remote shell execution. |
| 1099/tcp | java-rmi | GNU Classpath grmiregistry | Java RMI registry — potential remote code execution path. |
| 1524/tcp | bindshell | Metasploitable root shell | A planted root bind shell, not a real service — classic backdoor pattern. |
| 2049/tcp | nfs | 2-4 (RPC #100003) | Network file share; check exports for anonymous mount access. |
| 2121/tcp | ftp | ProFTPD 1.3.1 | Secondary FTP daemon — non-default port is not a substitute for access control. |
| 3306/tcp | mysql | MySQL 5.0.51a-3ubuntu5 | Very old release; test for blank/weak root credentials. |
| 3632/tcp | distccd | distccd v1 | Known unauthenticated remote code execution service (CVE-2004-2687). |
| 5432/tcp | postgresql | PostgreSQL DB 8.3.0–8.3.7 | Old release; test default credentials. |
| 5900/tcp | vnc | VNC (protocol 3.3) | Old, weak RFB protocol version; unencrypted; weak authentication. |
| 6000/tcp | X11 | — | Access denied — access control appears enforced. |
| 6667/tcp | irc | UnrealIRCd | Known backdoored build — CVE-2010-2075. |
| 6697/tcp | ircs-u | — | SSL-wrapped companion to the 6667 IRC service. |
| 8009/tcp | ajp13 | Apache Jserv (Protocol v1.3) | Associated with the Ghostcat vulnerability class (CVE-2020-1938). |
| 8180/tcp | http | Apache Tomcat/Coyote JSP engine 1.1 | Tomcat manager console — test for default credentials (path to remote code execution). |
| 8787/tcp | drb | Ruby DRb RMI | Distributed Ruby service with a known unauthenticated remote code execution path. |
| 36809/tcp | mountd | 1-3 (RPC #100005) | NFS mount handler. |
| 40283/tcp | status | 1 (RPC #100024) | RPC status/NSM service. |

**Key risk themes across this target:**
- **Credential attacks** — weak/default credentials possible on SSH, MySQL, PostgreSQL, VNC and the Tomcat manager console.
- **Plaintext communication** — Telnet, the legacy r-services (exec/login/shell) and FTP control connections all transmit data unencrypted.
- **Remote exploitation / backdoors** — the vsftpd 2.3.4 backdoor was confirmed exploited to a root shell; UnrealIRCd and the ingreslock bindshell follow the same pattern; distccd and the Ruby DRb service both carry known unauthenticated RCE paths.

The highest-severity finding on this target is the vsftpd 2.3.4 backdoor. Unlike a typical version-based vulnerability, this is a known supply-chain compromise (CVE-2011-2523): the official source archive was replaced with a trojaned build that opens an unauthenticated root-level shell when triggered.
`uid=0(root) gid=0(root)`.

