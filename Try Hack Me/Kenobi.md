# Kenobi

Walkthrough on exploiting a Linux machine.  

Enumerate Samba for shares, manipulate a vulnerable version of proftpd and escalate your privileges with path
variable manipulation.  

# Task_1

Make sure you're connected to our network and deploy the machine.  

## ip `10.10.254.145`


Scan the machine with nmap, how many ports are open?  

## Command 
```sh
nmap -sV -sC -A -v 10.10.254.145 | tee nmap.txt

```
## Results
```sh
Not shown: 993 closed tcp ports (conn-refused)
PORT STATE SERVICE VERSION
21/tcp open ftp
ProFTPD 1.3.5
22/tcp open ssh
OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
| 256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_ 256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp open http Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/admin.html
|_http-title: Site doesnt have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods:
|_ Supported Methods: GET HEAD POST OPTIONS
111/tcp open rpcbind 2-4 (RPC #100000)
| rpcinfo:
| program version port/proto service
| 100000 2,3,4 111/tcp rpcbind
| 100000 2,3,4 111/udp rpcbind
| 100000 3,4
111/tcp6 rpcbind
| 100000 3,4
111/udp6 rpcbind
| 100003 2,3,4 2049/tcp nfs
| 100003 2,3,4 2049/tcp6 nfs
| 100003 2,3,4 2049/udp nfs
| 100003 2,3,4 2049/udp6 nfs
| 100005 1,2,3 39285/udp mountd
| 100005 1,2,3 51073/udp6 mountd
| 100005 1,2,3 54743/tcp mountd
| 100005 1,2,3 55149/tcp6 mountd
| 100021 1,3,4 39665/tcp nlockmgr
| 100021 1,3,4 42515/tcp6 nlockmgr
| 100021 1,3,4 45811/udp6 nlockmgr
| 100021 1,3,4 60833/udp nlockmgr
| 100227 2,3
2049/tcp nfs_acl
| 100227 2,3
2049/tcp6 nfs_acl
| 100227 2,3
2049/udp nfs_acl
|_ 100227 2,3
2049/udp6 nfs_acl
2/11139/tcp open netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open nfs
2-4 (RPC #100003)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

From the scan we can say we have 7 open ports .  
I saved the nmap full result on a separate file since it had some cool intel on scripts vuln.  

# task 2  

Enumerating Samba for shares.  
Samba is the standard Windows interoperability suite of programs for Linux and Unix.  
It allows end users to access and use files, printers and other commonly shared resources on a companies intranet or internet.  
Its often referred to as a network file system.  
```sh
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.254.145
```
```sh
Host script results:
| smb-enum-shares:
| account_used: guest
| \\10.10.254.145\IPC$:
| Type: STYPE_IPC_HIDDEN
| Comment: IPC Service (kenobi server (Samba, Ubuntu))
| Users: 2
| Max Users: <unlimited>
| Path: C:\tmp
| Anonymous access: READ/WRITE
| Current user access: READ/WRITE
| \\10.10.254.145\anonymous:
| Type: STYPE_DISKTREE
| Comment:
| Users: 0
| Max Users: <unlimited>
| Path: C:\home\kenobi\share
| Anonymous access: READ/WRITE
| Current user access: READ/WRITE
| \\10.10.254.145\print$:
| Type: STYPE_DISKTREE
| Comment: Printer Drivers
| Users: 0
| Max Users: <unlimited>
| Path: C:\var\lib\samba\printers
| Anonymous access: <none>
|_ Current user access: <none>
```

Here we get 3 shares.
```sh
smbclient //10.10.254.145/anonymous

smbget -R smb://10.10.254.145/anonymous

nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.254.145

```


# Task_3


Gain initial access with ProFtpd  
Lets get the version of ProFtpd.  
Use netcat to connect to the machine on the FTP port.  
```sh
	nc 10.10.254.145 21
```
We can use searchsploit to find exploits for a particular software version.  
user.txt = ...

# Task_4
Privilege Escalation with Path Variable Manipulation. 
To search the a system for these type of files run the following
```sh
find / -perm -u=s -type f 2>/dev/null
```
Run the binary, how many options appear?  
Strings is a command on Linux that looks for human readable strings on a binary.  

This shows us the binary is running without a full path (e.g. not using /usr/bin/curl or /usr/bin/uname).  
As this file runs as the root users privileges, we can manipulate our path gain a root shell.  
We copied the /bin/sh shell, called it curl,gave it the correct permissions and then put its location in our path.  
This meant that when the /usr/bin/menu binary was run,its using our path variable to find the "curl" binary..  
Which is actually a version of /usr/sh,as well as this file being run as root it runs our shell as root!  
