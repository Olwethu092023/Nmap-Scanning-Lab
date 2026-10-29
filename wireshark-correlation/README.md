## Correlating Nmap Scans with Wireshark Packet Captures

The goal is to show that Nmap reports in the terminal can be seen, packet by packet, on the wire in Wireshark.

## Lab Setup
Attacker: Kali Linux
Target: Metasploitable 2 
Network: isolated VirtualBox host-only network
Tools: Nmap, Wireshark


## 1. Host discovery

I ran a host discovery scan across the subnet. Nmap reported 4 hosts up: 192.168.56.1, 192.168.56.20 (my target), 192.168.56.100 and 192.168.56.10 (Kali). I then checked Wireshark to see whether the target’s discovery showed up in the traffic. 

![Host Discovery](https://i.imgur.com/mnrEjqw.png)

Nmap host discovery output. The target 192.168.56.20 is up with MAC address 08:00:27:40:C9:AF. 


![Host Discovery](https://imgur.com/n7fjNUN.png)

Wireshark, display filter arp. Kali broadcasts "Who has 192.168.56.x? Tell 192.168.56.10" for each address in the subnet, and only 192.168.56.20 replies (highlighted).

Observation: Because the target is on the same local subnet, Nmap uses ARP rather than ICMP or TCP to check whether a host is alive. ARP maps IP addresses to MAC addresses. Kali asks "Who has 192.168.56.20?" and Metasploitable replies “192.168.56.20 is at 08:00:27:40:c9:af”. That reply is what Nmap reports as "Host is up", and the MAC address in the reply matches the one in Nmap's output. 

## 2. Service and version detection

This scans Nmap's default top 1,000 TCP ports. Nmap found 23 open ports and reported 977 closed ports (reset). To check that the version information is real, I picked one service to follow in Wireshark: port 22, which Nmap reports as OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0). 


![Host Discovery](https://imgur.com/zMdoyOj.png)
Nmap -sV output listing the open ports, services and versions.


![Host Discovery](https://i.imgur.com/JMvFjFN.png)
Wireshark, SSH conversation on port 22. After the TCP handshake (SYN, SYN/ACK, ACK), the server sends its banner: SSH-2.0-OpenSSH_4.7p1 Debian-8ubuntu1 (highlighted).
The connection is then closed with FIN/ACK.

Observation: The banner in the packet is the same string Nmap shows in the VERSION column. This shows that version detection works by connecting to the service and reading what it announces about itself (and, for services that stay silent, sending probes and matching the replies). 

Open and closed ports: the TCP handshake also shows port state. An open port answers the SYN with SYN/ACK; a closed port answers with RST/ACK. Nmap’s default SYN scan is “half-open”. When a port is open, Kali does not complete the handshake; it sends a RST after the SYN/ACK. 

## 3. Default Scripts


![Host Discovery](https://i.imgur.com/UcLNfEQ.png)

The default scripts generate their own traffic on top of the port scan. I looked for packets that only the scripts would send. 

With the filter nbns, Wireshark shows Kali sending NBSTAT name queries to the target and the target answering with its NetBIOS name table. This is the NetBIOS-checking script querying the Samba service. 

![Host Discovery](https://imgur.com/3v4d5Rb.png)
Wireshark, filter nbns. NBSTAT name queries from 192.168.56.10 and the responses from 192.168.56.20.

With the filter http, I found requests from Kali to the web server on port 8180. The example below shows a full exchange: TCP handshake, GET /robots.txt, the server’s 404 Not Found response and FIN/ACK. This is consistent with the http-robots.txt script checking for a robots.txt file. Other paths I saw in the same capture, such as /.git/HEAD, /favicon.ico and /HNAP1, are also typical of script probes rather than normal browsing.




