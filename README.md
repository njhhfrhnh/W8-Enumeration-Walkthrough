# W8 Enumeration Lab Challenge 

## Target Information

| Item              | Value                                    |
| ----------------- | ---------------------------------------- |
| Target Machine    | Metasploitable 2                         |
| Target IP Address | 192.168.100.198                          |
| Attacker Machine  | Kali Linux, Windows                      |

---
# Basic Enumeration

## Challenge 2 — Fast Nmap Scan

To identify open ports and active network services on the target machine using a fast Nmap scan.

```bash
nmap -F 192.168.100.198
```
<img width="546" height="370" alt="image" src="https://github.com/user-attachments/assets/aeddfac9-e69f-44fb-81e2-b362eee931cf" />

<img width="668" height="676" alt="image" src="https://github.com/user-attachments/assets/01e1f1c0-664c-433f-a425-73d9d5cb7527" />


### Findings:
| Port | Service      |
| ---- | ------------ |
| 21   | FTP          |
| 22   | SSH          |
| 23   | Telnet       |
| 25   | SMTP         |
| 53   | DNS          |
| 80   | HTTP         |
| 111  | RPCBind      |
| 139  | NetBIOS-SSN  |
| 445  | Microsoft-DS |
| 2049 | NFS          |
| 3306 | MySQL        |
| 5432 | PostgreSQL   |
| 5900 | VNC          |
| 8009 | AJP13        |

The target machine exposed multiple network services, increasing the attack surface available for enumeration and exploitation.

---

## Challenge 4 — SNMPwalk

To determine whether the target system exposes SNMP services for information enumeration.

```bash
snmpwalk -v1 -c public 192.168.100.198
```

```bash
nmap -sU -p161 192.168.100.198
```
<img width="576" height="201" alt="image" src="https://github.com/user-attachments/assets/9e399296-5f78-4225-bd35-37a73020a6c5" />

### Findings:

```text
161/udp closed snmp
```

SNMP services were not available on the target machine. Since UDP port 161 was closed, SNMP enumeration could not be performed.

---

## Challenge 5 — TTL OS Fingerprinting

To identify the target operating system based on observed TTL values.

```bash
ping 192.168.100.198
```

<img width="440" height="176" alt="image" src="https://github.com/user-attachments/assets/8ceab714-e276-400f-92f0-8562e3282ff8" />


### Findings:

```text
ttl=64
```

TTL fingerprinting indicated that the target machine is likely running a Linux-based operating system.

---

## Challenge 9 — FTP Banner Enumeration

To identify the FTP service version running on the target machine.


```bash
nc 192.168.100.198 21
```

<img width="277" height="42" alt="image" src="https://github.com/user-attachments/assets/ecd9bfd5-39d3-4d6d-975e-52801eb7605a" />

### Findings:

```text
220 (vsFTPd 2.3.4)
```

The FTP banner disclosed the exact FTP software version. Version disclosure may assist attackers in identifying known vulnerabilities associated with the FTP service.

---

## Challenge 10 — Anonymous FTP Login

To determine whether the FTP server allows anonymous authentication.

```bash
ftp 192.168.100.198
```

<img width="490" height="202" alt="image" src="https://github.com/user-attachments/assets/f367f54b-774a-45b8-be09-a4e11b2143ea" />

<img width="390" height="135" alt="image" src="https://github.com/user-attachments/assets/da484816-f3fb-4e97-b19c-4a5908f84cfe" />

### Findings:

```text
230 Login successful.
```

Anonymous login was accepted successfully.

Allowing anonymous FTP authentication may expose sensitive files and permit unauthorized access to shared resources.

---
# Intermediate Enumeration

## Challenge 11 — SMB NSE Enumeration

To enumerate SMB information including operating system, domain, and NetBIOS details.

```bash
nmap --script smb-os-discovery -p445 192.168.100.198
```

<img width="602" height="218" alt="image" src="https://github.com/user-attachments/assets/8214c40b-05d1-43b4-8378-d8b7e03b2533" />

<img width="657" height="672" alt="image" src="https://github.com/user-attachments/assets/9dd87ad4-ee59-45e0-99ef-f07e2ee2bb76" />


### Findings:

| Information   | Result                     |
| ------------- | -------------------------- |
| OS            | Unix (Samba 3.0.20-Debian) |
| Computer Name | metasploitable             |
| Domain Name   | localdomain                |
| FQDN          | metasploitable.localdomain |


SMB enumeration successfully disclosed operating system information, domain details and host identification data which may assist attackers during reconnaissance.

---

## Challenge 12 — Enum4linux

To perform detailed SMB enumeration against the target system.

```bash
enum4linux -a 192.168.100.198
```

<img width="602" height="457" alt="image" src="https://github.com/user-attachments/assets/47dca46f-6b22-42ba-baa8-24fb4fae6436" />
<img width="602" height="409" alt="image" src="https://github.com/user-attachments/assets/b95e0b11-de2b-4789-b12c-0716e1324135" />
<img width="602" height="303" alt="image" src="https://github.com/user-attachments/assets/13c4f346-ede5-4e28-b9c1-05e4b239def0" />
<img width="602" height="360" alt="image" src="https://github.com/user-attachments/assets/8ed8f995-5cee-439b-a0e3-577629372913" />
<img width="602" height="417" alt="image" src="https://github.com/user-attachments/assets/08d511d1-d308-411d-8358-19383401cdd1" />

### Findings:

### Workgroup Information
```text
WORKGROUP
```

### SMB Shares

* tmp
* opt
* IPC$
* ADMIN$

### Enumerated Users

* root
* msfadmin
* mysql
* postgres
* ftp
* www-data

### Password Policy

```text
Password Complexity: Disabled
Minimum Password Length: 0
```
Enum4linux revealed extensive SMB information including usernames, shares and weak password policies. Weak SMB configurations may increase the risk of unauthorized access and credential attacks.

---

## Challenge 13 — NFS Exports

To enumerate exported NFS shares from the target system.

```bash
showmount -e 192.168.100.198
```

<img width="298" height="62" alt="image" src="https://github.com/user-attachments/assets/212a20cf-9b0a-4b73-aefb-808f0c1f36d6" />

### Findings:

```text
/ *
```

The root directory was exported to all hosts without restriction. Misconfigured NFS exports may expose sensitive files and directories to unauthorized users.

---

## Challenge 16 — Version Detection

To identify software versions running on exposed network services.

```bash
nmap -sV 192.168.100.198
```

<img width="602" height="378" alt="image" src="https://github.com/user-attachments/assets/32b0b96f-3731-4942-9649-265ba9825c26" />

### Findings:

| Service    | Version                         |
| ---------- | ------------------------------- |
| FTP        | vsFTPd 2.3.4                    |
| SSH        | OpenSSH 4.7p1                   |
| HTTP       | Apache httpd 2.2.8              |
| SMB        | Samba smbd 3.X–4.X              |
| MySQL      | MySQL 5.0.51a                   |
| PostgreSQL | PostgreSQL 8.3.x                |
| IRC        | UnrealIRCd                      |
| Tomcat     | Apache Tomcat/Coyote JSP Engine |


Version detection identified multiple outdated services that may contain publicly known vulnerabilities. Attackers may use version disclosure to identify exploitable targets.

---

# Advanced Enumeration

## Challenge 29 — SMTP Enumeration via Nmap

To enumerate SMTP-related information and test for open relay vulnerabilities.

```bash
nmap -p25 --script=smtp-enum-users 192.168.100.198
```

```bash
nmap -p25 --script=smtp-open-relay 192.168.100.198
```

<img width="602" height="317" alt="image" src="https://github.com/user-attachments/assets/b18ff2f5-5354-4cff-b2d3-813b303d76b5" />

### Findings:

```text
Method RCPT returned a unhandled status code.
```

```text
Server doesn't seem to be an open relay
```

SMTP enumeration confirmed that the target system exposes an SMTP service on port 25. Open relay testing indicated that the server was not configured as an open relay, reducing the risk of unauthorized email forwarding abuse.


---

