# Task 3: Low-Level Network Packet Inspection & Protocol Analysis

## 1. Executive Summary

Network packet capture was performed on an isolated lab environment with Kali Linux (192.168.56.10) and Metasploitable 3 (192.168.56.103). Analysis confirmed FTP and HTTP transmit credentials in plaintext, while SSH properly encrypts all session data.

## 2. Lab Environment

| Component | Details |
|-----------|---------|
| Attacker | Kali Linux - 192.168.56.10 |
| Target | Metasploitable 3 - 192.168.56.103 |
| Tool | tcpdump |
| Capture File | /tmp/task3.pcap |

## 3. Open Ports (Nmap)

21/tcp  open     ftp
22/tcp  open     ssh
23/tcp  filtered telnet
80/tcp  open     http
443/tcp filtered https
445/tcp open     microsoft-ds

## 4. Evidence

### 4.1 TCP Three-Way Handshake

05:47:55.180180 IP 192.168.56.10.53815 > 192.168.56.103.ftp: Flags [S]
05:47:55.180558 IP 192.168.56.103.ftp > 192.168.56.10.53815: Flags [S.]
05:47:55.239069 IP 192.168.56.10.45710 > 192.168.56.103.http: Flags [S]

### 4.2 FTP Clear-Text Credentials

05:49:24.831930 IP 192.168.56.10.34824 > 192.168.56.103.ftp: FTP: USER vagrant
05:49:28.377708 IP 192.168.56.10.34824 > 192.168.56.103.ftp: FTP: PASS vagrant

### 4.3 HTTP Clear-Text Login

POST /login.php HTTP/1.1
username=admin&password=password

POST /phpmyadmin/ HTTP/1.1
pma_username=root&pma_password=password

POST /drupal/ HTTP/1.1
name=admin&pass=password

### 4.4 SSH Encrypted (Comparison)

SSH: SSH-2.0-OpenSSH_10.4p1 Debian-5
[Encrypted binary data - not human-readable]

## 5. Findings Summary

| # | Vulnerability | Protocol | Port | Severity |
|---|---------------|----------|------|----------|
| 1 | Clear-text FTP credentials | FTP | 21 | High |
| 2 | Clear-text HTTP login | HTTP | 80 | High |
| 3 | phpMyAdmin clear-text login | HTTP | 80 | High |
| 4 | Drupal clear-text login | HTTP | 80 | High |
| 5 | Port scan detectable | TCP | - | Medium |
| 6 | SSH properly encrypted | SSH | 22 | Secure |

## 6. Recommendations

1. Replace FTP with SFTP/FTPS
2. Enforce HTTPS (TLS 1.3) on all web apps
3. Disable plain HTTP, enforce HSTS
4. Remove Telnet, use SSH only
5. Deploy IDS/IPS (Snort/Suricata)
6. Network segmentation with VLANs
7. Zero-Trust Architecture
8. Regular automated packet analysis

## 7. Conclusion

Critical clear-text credential exposure identified in FTP and HTTP. SSH confirmed secure. Immediate remediation required.

## 8. Commands Used

sudo tcpdump -i eth0 host 192.168.56.103 -w /tmp/task3.pcap
sudo tcpdump -r /tmp/task3.pcap -A 'tcp port 21' | grep -E "USER|PASS"
sudo tcpdump -r /tmp/task3.pcap -A 'tcp port 80' | grep -E "POST|username|password"
sudo tcpdump -r /tmp/task3.pcap 'tcp[tcpflags] & tcp-syn != 0' | head -20
sudo tcpdump -r /tmp/task3.pcap -A 'tcp port 22' | head -30
